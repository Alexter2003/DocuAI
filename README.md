# DocuAI

Automated documentation platform that generates deep technical documentation from pull requests using AI, with semantic search and Q&A capabilities.

## Overview

DocuAI connects to your GitHub repositories and triggers on PR merges. AI processing runs on your own GitHub Actions, while the platform handles webhook ingestion, vector storage, and Q&A.

- **Trigger:** PR merge into `main`/`master`
- **Output:** PR comment + structured documentation stored in PostgreSQL with pgvector
- **Q&A:** Semantic search over your codebase documentation

## Tech Stack

- **Backend:** NestJS + TypeScript
- **Database:** PostgreSQL + Prisma + pgvector
- **AI:** Google Gemini (free tier) with BYOK support
- **Auth:** GitHub OAuth

## Project Setup

```bash
pnpm install
```

## Running the App

```bash
# development
pnpm run start

# watch mode
pnpm run start:dev

# production
pnpm run start:prod
```

## Tests

```bash
# unit tests
pnpm run test

# e2e tests
pnpm run test:e2e

# coverage
pnpm run test:cov
```

## Environment Variables

Copy `.env.example` to `.env` and fill in the required values.

## Documentation

Full project documentation is available in the [`/Documentation`](./Documentation) folder:

- [System Flow](./Documentation/system_flow.md)
- [Architecture Decisions](./Documentation/architecture_decisions.md)
- [Functional Features](./Documentation/functional_features.md)
- [Non-Functional Features](./Documentation/non_functional_features.md)
- [User Journeys](./Documentation/user_journeys.md)
- [Error Flows & Edge Cases](./Documentation/error_flows.md)
