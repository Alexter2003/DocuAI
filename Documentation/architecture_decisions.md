# Architecture Decisions (ADR)

This document tracks all the architectural decisions made for DocuAI.

## 1. System Structure
- **Decision:** Hybrid Execution Model.
- **Why:** To prevent exhausting our API limits. The platform provides a web dashboard, but the actual AI processing runs distributed via GitHub Actions on the user's repository servers.

## 2. Tech Stack
- **Backend:** NestJS + TypeScript (Handles API requests, Webhooks, DB logic).
- **Frontend:** Next.js (React) + TailwindCSS (Dashboard for viewing docs and configuring settings).
- **Database:** Supabase (PostgreSQL) + Prisma ORM.

## 3. Storage & AI Capabilities
- **Vector Storage:** `pgvector` inside Supabase. Stores chopped-up documentation embeddings to power the Q&A Assistant without needing a separate vector database like Pinecone.
- **AI Engine:** Google Gemini (Free Tier) with strong custom prompting. Supports "Bring Your Own Key" (BYOK).

## 4. Configuration Layer
- **Decision:** Hybrid Configuration.
- **Why:** Account billing/API keys are configured on the web dashboard. Project-specific rules (`ignore` lists, prompts, language) are configured via a `.docuai.yml` file tracked in version control.

## 5. GitHub Triggers & Storage Strategy
- **Trigger:** AI documentation executes *only* when a Pull Request is Merged.
- **Delivery:** Generates deep documentation and pushes it to our PostgreSQL DB. Simultaneously leaves a PR comment containing a high-level summary, Mermaid architecture diagrams, and a link to the full DocuAI platform version.
