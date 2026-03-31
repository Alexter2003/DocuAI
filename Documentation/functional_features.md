# Functional Features

Functional features describe what DocuAI actually *does* for its users.

## 1. Authentication & Onboarding
- F1.1: Users can log in via GitHub OAuth.
- F1.2: Users can grant the platform access to specific repositories.

## 2. GitHub Integration (Core Engine)
- F2.1: Listen for Pull Request `merged` events via Webhooks or GitHub Actions.
- F2.2: Scan the repository diffs to understand what files were added, modified, or deleted.
- F2.3: Generate summarized code documentation using AI.
- F2.4: Post a PR comment containing the summary, diagrams (Mermaid.js), and a link to the deep documentation.

## 3. Documentation Platform (Dashboard)
- F3.1: Display a structured, beautiful view of the generated documentation per repository.
- F3.2: Provide a global search across an organization's documentation.
- F3.3: Offer an Interactive Q&A Assistant (Chatbot) that reads the documentation (via Vector Embeddings) and answers developer questions exactly.

## 4. Configuration & Rules
- F4.1: Allow users to provide their own AI Provider API Key (BYOK).
- F4.2: Parse `.docuai.yml` from the user's repository root to apply custom rules (target language, folder exclusions, custom prompt style).

## 5. User Roles & Access Control
- F5.1: **Admin** can create organizations, connect repositories, manage team members, and configure global settings.
- F5.2: **Developer** can view documentation, use Q&A, and optionally configure `.docuai.yml` in their repositories.
- F5.3: **Viewer** can browse documentation and use Q&A but cannot modify settings or invite users (no GitHub OAuth required).
- F5.4: Admins can revoke team member access at any time.
- F5.5: Viewers have email-based access (magic link login) instead of GitHub OAuth.

## 6. Pricing Tiers
- F6.1: **Free Tier** provides 3 connected repositories, 1 team member (owner only), Gemini free AI model, 30-day doc history, and 100 Q&A queries per month.
- F6.2: **Pro Tier** provides unlimited repositories, unlimited team members, BYOK or hosted Gemini Pro, unlimited doc history, and unlimited Q&A queries with priority support.
- F6.3: Admins can upgrade from Free to Pro anytime via the dashboard billing page.
