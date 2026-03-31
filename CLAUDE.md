# DocuAI - Project Context

This document serves as the entry point for understanding DocuAI's product definition, architecture, and features.

## 📋 Documentation Index

All project documentation is organized in the `/Documentation` folder:

### 1. [**System Flow** (`Documentation/system_flow.md`)](./Documentation/system_flow.md)
   - Complete lifecycle from user onboarding through GitHub Actions execution to vector storage and Q&A
   - User auth flow, PR merge triggers, AI processing pipeline, webhook ingestion, and dashboard Q&A loop

### 2. [**Architecture Decisions** (`Documentation/architecture_decisions.md`)](./Documentation/architecture_decisions.md)
   - **Tech Stack:** NestJS backend, Next.js frontend, Supabase (PostgreSQL + Prisma), pgvector for embeddings
   - **Hybrid Model:** AI processing runs on user's GitHub Actions, platform handles webhooks and Q&A
   - **Storage:** pgvector for semantic search without external vector DB
   - **AI Engine:** Google Gemini (free tier) with BYOK support
   - **Config:** Dashboard for account settings, `.docuai.yml` for per-repo rules

### 3. [**Functional Features** (`Documentation/functional_features.md`)](./Documentation/functional_features.md)
   - GitHub OAuth authentication
   - PR merge triggers → AI documentation generation
   - Dashboard with structured doc viewing, global search, Q&A chatbot
   - `.docuai.yml` configuration (language, folder exclusions, custom prompts)
   - BYOK support for AI providers

### 4. [**Non-Functional Features** (`Documentation/non_functional_features.md`)](./Documentation/non_functional_features.md)
   - **Cost Efficiency:** Free tier AI, distributed compute via GitHub Actions
   - **Performance:** 3-second Q&A response target using pgvector indexing
   - **Security:** Source code not stored permanently, API keys encrypted at rest
   - **DX:** Config-as-code with `.docuai.yml`, premium dashboard with dark mode

### 5. [**User Journeys** (`Documentation/user_journeys.md`)](./Documentation/user_journeys.md)
   - Step-by-step flows for Admin, Developer, and Viewer roles
   - Onboarding, repository connection, team management
   - Documentation browsing and Q&A usage
   - Configuration workflows and team collaboration scenarios

### 6. [**Error Flows & Edge Cases** (`Documentation/error_flows.md`)](./Documentation/error_flows.md)
   - Webhook delivery retries and timeout handling
   - AI generation failures (auth, rate limits, empty diffs)
   - Vector embedding fallbacks and Q&A graceful degradation
   - Configuration validation and permission checks
   - Storage quota management and auto-cleanup

---

## 📊 Tier Comparison

| Feature | Free | Pro |
|---------|------|-----|
| Repositories | 3 | Unlimited |
| Team members | 1 (owner) | Unlimited |
| AI Model | Gemini Free | BYOK or Gemini Pro |
| Doc history | 30 days | Unlimited |
| Q&A queries/month | 100 | Unlimited |
| Support | Community | Priority |

---

## 🎯 Quick Reference

- **Trigger Point:** PR merge into main/master
- **Output:** PR comment + deep documentation stored in PostgreSQL
- **Q&A Backend:** Vector embeddings + semantic search via pgvector
- **Primary Users:** Development teams needing automated documentation
