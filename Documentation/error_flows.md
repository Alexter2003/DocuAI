# Error Flows & Edge Cases

This document details how DocuAI handles failures, edge cases, and graceful degradation across all system domains.

---

## 1. GitHub Actions → Webhook Delivery

### 1.1 Webhook Delivery Failure
**Scenario:** GitHub Action tries to POST to `api.docuai.com/v1/webhook/doc-generated` but endpoint is unreachable.

**Flow:**
1. Action receives HTTP error (5xx, timeout, or network error)
2. Action retries with exponential backoff:
   - Attempt 1: immediate
   - Attempt 2: wait 5 seconds
   - Attempt 3: wait 30 seconds
3. If all 3 attempts fail:
   - Action logs failure to GitHub Actions console (visible to repo admin)
   - Payload is sent to dead-letter queue for manual inspection
   - User receives **dashboard notification:** "Documentation generation failed — contact support"
   - PR comment is NOT posted (partial failure, cleaner UX)

**User Action Required:** Contact support with PR URL; we replay from dead-letter queue once fixed.

---

### 1.2 Action Execution Timeout
**Scenario:** GitHub Action takes >15 minutes (exceeds job timeout).

**Flow:**
1. Action has been running, collecting diffs and generating docs
2. GitHub kills the job at 15 minute mark
3. If partial payload was already sent to webhook:
   - Backend receives payload with `status: "incomplete"`
   - Stores what was generated
   - Marks documentation as draft/incomplete in database
   - User sees in dashboard: "Documentation for this PR is incomplete — regenerate"
4. If no payload sent yet:
   - User sees nothing; PR comment is not posted
   - User can manually trigger regeneration (future feature)

**Prevention:** Document generation should complete in <10 minutes for typical repos.

---

### 1.3 Malformed Webhook Payload
**Scenario:** Action sends invalid JSON or missing required fields.

**Flow:**
1. Backend receives malformed payload
2. Returns HTTP 400 (Bad Request)
3. Logs detailed error
4. Does NOT retry (client error, not server error)
5. Backend alerts engineering team (logging/monitoring)
6. User sees: PR comment not posted; PR review UI shows no docs (unclear why)
7. **Note:** This is a code bug, not a user-facing error — should not happen in production

---

## 2. AI Generation Failures

### 2.1 Invalid or Expired API Key
**Scenario:** Admin configured a Google Gemini API key; key has been revoked or expired.

**Flow:**
1. Action calls AI provider → receives 401 (Unauthorized)
2. Action detects auth error and stops (no retry needed)
3. **PR Comment Posted:** "Documentation generation failed — API key invalid. [Admin, check settings](link-to-dashboard)"
4. **Dashboard Notification:** Admin receives "API Key Error" alert
5. Admin navigates to Settings → API Key → updates it
6. Next PR merge uses new key (no manual regeneration needed)

---

### 2.2 AI Provider Rate Limit Hit
**Scenario:** Organization hits free tier quota or rate limits mid-generation.

**Flow:**
1. Action calls AI provider → receives 429 (Too Many Requests)
2. Action catches rate limit error
3. **If Free Tier:** Action queues job for retry in 24 hours (backlog)
   - **PR Comment Posted:** "Documentation delayed — free tier quota exceeded. [Upgrade to Pro for priority](link)"
   - **Dashboard:** Shows "Processing delayed" status
   - Job silently retries next day when quota resets
4. **If Pro Tier / BYOK:** Action immediately retries (Pro users have higher limits)
   - If retry fails, posts error comment same as #2.1

---

### 2.3 Empty or Trivial Code Changes
**Scenario:** PR modifies only comments, whitespace, or config files with no functional code.

**Flow:**
1. Action scans PR diffs
2. Detects: "No meaningful code changes"
3. **Action Skips Documentation Generation** (no AI call, saves cost)
4. **PR Comment:** NOT posted (clean PR thread)
5. User sees: Nothing in PR, but PR is tagged `documentation-not-generated` (in GitHub labels)

---

### 2.4 AI Generation Times Out
**Scenario:** AI provider is slow; generation takes >2 minutes.

**Flow:**
1. Action has 15-minute timeout; AI call has 2-minute timeout
2. If AI takes >2 minutes:
   - Action cancels request
   - Logs timeout
   - **PR Comment Posted:** "Documentation generation timed out — retry manually [here](link)"
   - User can click link to trigger regeneration manually (future feature)

---

## 3. Vector Embedding & Q&A Failures

### 3.1 Embedding Generation Fails
**Scenario:** Document is received, but embedding generation fails (e.g., AI provider error).

**Flow:**
1. Backend receives documentation markdown
2. Attempts to chunk and embed it
3. Embedding API returns error (429, 500, or timeout)
4. **Document is still saved** in `Documentation` table with status: `not_searchable`
5. **Q&A Falls Back:** When user asks Q&A, backend falls back to full-text search instead of vector similarity
   - Results may be less relevant, but functional
6. **Dashboard:** User sees "Q&A may be less accurate for recent docs" notice
7. **Backend:** Logs error; retries embedding job in background (async)
8. Once embedding succeeds, `status` changes to `searchable`; notice disappears

---

### 3.2 Q&A Search Returns No Results
**Scenario:** User asks "How does module X work?" but no documentation chunks match the query.

**Flow:**
1. Backend runs vector similarity search
2. No results above confidence threshold returned
3. **Backend Response (Graceful):**
   ```
   "I couldn't find documentation about that. Try:
   - Browsing the dashboard for module structure
   - Asking a more specific question
   - Checking if this feature is documented yet"
   ```
4. **No Error Alert** — expected behavior, not a system failure

---

### 3.3 Q&A Query Quota Exceeded
**Scenario:** Free tier user has used 100 Q&A queries this month; tries to ask another.

**Flow:**
1. Backend checks monthly quota
2. User at limit → returns HTTP 429 (Quota Exceeded)
3. **Dashboard Shows:**
   ```
   "You've used 100/100 Q&A queries this month.
   Upgrade to Pro for unlimited queries."
   [Upgrade Button]
   ```
4. User can either upgrade or wait for next month (quota resets on billing cycle)

---

## 4. Configuration & Input Validation

### 4.1 Malformed `.docuai.yml`
**Scenario:** Developer adds `.docuai.yml` with invalid YAML syntax or unsupported fields.

**Flow:**
1. Action reads `.docuai.yml`
2. Parser fails (YAML error) OR recognizes unsupported field
3. **Action Behavior:** Applies defaults and continues (no failure)
4. **PR Comment:** Includes small warning: ⚠️ `.docuai.yml` has errors; using defaults. [View guide](link)"
5. **Dashboard Notification:** Optional alert to admin: "Config parsing warning in repo X"
6. Developer can fix `.docuai.yml` for next PR merge

---

### 4.2 `.docuai.yml` Missing
**Scenario:** `.docuai.yml` doesn't exist in repo; global defaults apply.

**Flow:**
1. Action looks for `.docuai.yml` → not found
2. Applies global organization defaults (language, ignore patterns, etc.)
3. **No error or warning** — expected behavior
4. **Dashboard:** Optional hint: "No `.docuai.yml` found; using organization defaults" (informational, not an error)

---

### 4.3 Unsupported Language in `.docuai.yml`
**Scenario:** Developer specifies `language: "Klingon"` (not supported).

**Flow:**
1. Action validates language code
2. Language not in supported list → falls back to English
3. **PR Comment:** ⚠️ "Language 'Klingon' not supported; generating in English instead"
4. Documentation generated in English
5. Developer can update `.docuai.yml` for next PR

---

## 5. Repository Access & Permissions

### 5.1 Repository Not Connected
**Scenario:** Action fires in a GitHub repo not connected to DocuAI.

**Flow:**
1. Action attempts POST to webhook
2. Backend checks: is this repo in our database?
3. Repo not found → returns HTTP 404
4. **Action Response:** Posts PR comment: "This repository is not connected to DocuAI. [Connect it](onboarding-link)"
5. Admin receives dashboard alert: "Unregistered webhook call from repo X"

---

### 5.2 User Loses Repository Access
**Scenario:** Developer was invited to org; organization admin revokes their access mid-session.

**Flow:**
1. User is logged in, browsing docs
2. Admin revokes access
3. User's session token is still valid (not immediately invalidated)
4. User continues browsing until session expires (refresh required)
5. **On Next Login:** User not in access list → 403 Forbidden
6. User sees: "You no longer have access to this organization"

---

### 5.3 Viewer Tries to Access Admin Features
**Scenario:** Viewer receives dashboard URL with admin settings panel.

**Flow:**
1. User navigates to `/settings`
2. Frontend checks user role (viewer)
3. **Frontend Blocks:** Redirects to `/docs` (read-only view)
4. Or backend returns 403 if they somehow bypass frontend
5. **No error page** — seamless redirect to allowed area

---

## 6. Storage & Cleanup

### 6.1 Storage Quota Exceeded
**Scenario:** Pro user has generated docs for 500 PRs; storage limit approached.

**Flow:**
1. Backend monitors storage usage
2. When 90% quota reached:
   - **Dashboard Alert:** "You're using 90% of your documentation storage"
   - Admin can manually delete old docs or upgrade plan
3. When 100% quota reached:
   - Next PR merge → Action succeeds, but doc storage attempt fails
   - **PR Comment:** "Documentation generated but could not be saved (storage full). [Manage docs](dashboard-link)"
   - Free tier: older docs auto-deleted (FIFO, max 30 days anyway)
   - Pro tier: requires manual intervention

---

### 6.2 Free Tier: Doc Auto-Cleanup
**Scenario:** Free tier documentation is older than 30 days.

**Flow:**
1. Backend runs daily cleanup job (async)
2. Deletes docs older than 30 days
3. Vector embeddings also deleted
4. **User Impact:** Minimal — docs still visible in PR comments; just not in dashboard
5. **Next PR Merge:** Generates fresh docs; overrides deleted version

---

## Summary Table

| Domain | Failure | User Sees | Resolution |
|--------|---------|-----------|-----------|
| **Webhook** | Delivery fails | Dashboard alert + no PR comment | Automatic retry; manual support if persistent |
| **Webhook** | Action timeout | Dashboard incomplete notice | Regenerate manually |
| **AI** | Auth fails | PR comment + admin alert | Admin updates API key |
| **AI** | Rate limit | PR comment + queue notification | Wait 24h or upgrade plan |
| **AI** | Empty diff | Nothing (expected) | PR labeled `documentation-not-generated` |
| **Embedding** | Fails | Doc saved, fallback to full-text | Auto-retry in background |
| **Q&A** | No results | Graceful message | User reformulates question |
| **Q&A** | Quota exceeded | Quota notice + upgrade button | Upgrade or wait for next month |
| **Config** | `.docuai.yml` malformed | Warning + defaults applied | User fixes and retries |
| **Access** | Repo not connected | PR comment with onboarding link | Admin connects repo |
| **Access** | Permission revoked | 403 on next login | User removed intentionally |
