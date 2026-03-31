# User Journeys

This document details the step-by-step flows for each user role interacting with DocuAI.

---

## 1. Admin (Team Owner) Journey

**Goal:** Set up DocuAI for the organization and manage team access.

### 1.1 Initial Onboarding
1. Visits `app.docuai.com`
2. Clicks "Sign Up with GitHub"
3. GitHub OAuth redirects back → DocuAI creates User account
4. Prompted to create or name organization
5. Selects plan: **Free** (3 repos, 1 user, Gemini Free Tier) or **Pro** (unlimited repos, unlimited team members, BYOK)
6. Dashboard shows empty state: "Connect your first repository"

### 1.2 Repository Connection
1. Clicks "+ Connect Repository"
2. Selects from list of GitHub repos they own/have admin access to
3. DocuAI generates `.github/workflows/docuai.yml` and opens a PR to add it OR provides copy-paste instructions
4. Once merged, repo is "Active" and listening for PR merges

### 1.3 Team Management
1. From Settings → Team, clicks "Invite Member"
2. Enters email or GitHub username
3. Selects role: **Developer** (can view docs, use Q&A, configure `.docuai.yml`) or **Viewer** (read-only, no GitHub required)
4. Invitee receives email with join link
5. Invitee logs in (GitHub OAuth for Developer, email magic link for Viewer)
6. Admin can revoke access anytime

### 1.4 Organization Settings
1. From Settings, configures:
   - **Default AI Provider:** Google Gemini (free tier) or custom API key (BYOK)
   - **Global Language:** Default language for all doc generation (if not overridden in `.docuai.yml`)
   - **Billing:** Manage subscription, payment method, usage analytics
2. Changes are effective immediately for future PR merges

---

## 2. Developer Journey

**Goal:** Contribute code, read documentation, and ask questions about the codebase.

### 2.1 Accepting Team Invitation
1. Receives email invite to join organization
2. Clicks link → GitHub OAuth → Logged in
3. Sees onboarded organization and list of connected repositories
4. Can immediately browse documentation and use Q&A

### 2.2 Creating Pull Request
1. Creates feature branch locally
2. Makes commits (20+ commits possible)
3. Opens Pull Request on GitHub
4. PR is reviewed and **Merged** into main/master
5. GitHub Actions workflow `docuai.yml` triggers automatically
6. ~30 seconds later, a **DocuAI PR Comment** appears with:
   - High-level summary of changes
   - Mermaid architecture diagrams
   - Link to full documentation in dashboard: `https://app.docuai.com/org/repo/docs`
7. Developer can click through to see full deep documentation and Q&A

### 2.3 Dashboard: Browsing Documentation
1. Logs into `app.docuai.com`
2. Selects repository from sidebar
3. Views structured documentation:
   - Overview / README
   - Module architecture
   - API endpoints
   - Data models
   - Integration points
4. Can search across all org documentation
5. Last 30 days of docs visible (Free tier) or unlimited (Pro tier)

### 2.4 Dashboard: Q&A Assistant
1. From Dashboard, opens "Ask a Question" panel
2. Types: "How does the auth module work?"
3. Backend runs vector similarity search against documentation chunks
4. Assistant retrieves relevant context and generates answer
5. Gets response within 3 seconds
6. Can refine question or ask follow-up
7. Monthly quota: 100 questions (Free tier) or unlimited (Pro tier)

### 2.5 Configuration (Optional)
1. Adds `.docuai.yml` to repository root
2. Specifies custom settings:
   - `language: "Spanish"` (overrides global default)
   - `ignore: ["/tests", "/node_modules"]`
   - `custom_prompt: "Focus on API contracts"` (optional)
3. On next PR merge, these rules apply automatically
4. No deployment needed — config lives in version control

---

## 3. Viewer Journey

**Goal:** Read documentation and ask questions without code access or admin duties.

### 3.1 Invitation & Access
1. Admin invites via email (not GitHub OAuth required)
2. Receives email with magic link
3. Clicks link → Email verification → Logged in
4. Sees invited organization and repositories
5. No GitHub authentication needed

### 3.2 Documentation Access
1. Browses repository documentation same as Developer
2. Can view all generated docs and architecture diagrams
3. Cannot see `.docuai.yml` or configuration options

### 3.3 Q&A Usage
1. Uses Q&A assistant same as Developer
2. Subject to monthly query limit (100 on Free, unlimited on Pro)
3. Cannot configure settings or invite others

### 3.4 Limitations
- Cannot view raw source code
- Cannot configure repositories
- Cannot invite team members
- Cannot trigger documentation generation (only PR merges trigger it)
- Read-only access to all features

---

## 4. Cross-User Scenarios

### 4.1 Team Collaboration on a Feature
1. **Dev A** creates feature branch, makes commits
2. **Dev B** reviews, approves
3. **Dev A** merges PR → Action triggers
4. DocuAI generates docs → PR comment posted
5. **Dev B**, **Dev C** (another team member), and **Viewer X** see the comment and can browse full docs
6. **Viewer X** asks Q&A: "What new endpoints were added?" → Gets answer

### 4.2 Admin Upgrades from Free to Pro
1. Admin goes to Settings → Billing
2. Selects "Upgrade to Pro"
3. Immediately gains:
   - Unlimited repositories
   - Unlimited team members
   - Unlimited Q&A queries
   - Unlimited doc history
4. Existing repos and team continue unaffected

### 4.3 Repository Configuration Override
1. Repo has global default: English docs
2. **Developer X** adds `.docuai.yml` with `language: "Spanish"`
3. Next PR merge → generates Spanish docs
4. Other repos continue in English
5. No conflict or re-generation needed

---

## Summary Table

| Action | Admin | Developer | Viewer |
|--------|-------|-----------|--------|
| Connect repos | ✓ | ✗ | ✗ |
| Invite users | ✓ | ✗ | ✗ |
| Configure settings | ✓ | ✓ (.docuai.yml only) | ✗ |
| Browse docs | ✓ | ✓ | ✓ |
| Use Q&A | ✓ | ✓ | ✓ |
| Manage billing | ✓ | ✗ | ✗ |
| View PR comments | ✓ | ✓ | ✓ (if public) |
