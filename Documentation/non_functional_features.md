# Non-Functional Features

Non-Functional Features (NFRs) define system attributes such as performance, usability, and constraints.

## 1. Cost Efficiency (Critical)
- **Constraint:** The system must maximize the use of the AI provider's free tier.
- **Solution:** Execute heavy AI generation inside GitHub Actions (distributed compute & IPs) and restrict execution triggers to PR merges only.
- **Tier Constraint:** Free tier users are limited to 3 repositories and no premium API features; this limits daily AI calls and keeps infrastructure costs minimal.
- **Constraint:** Documentation history must be managed to control storage costs.
- **Solution:** Free tier docs auto-delete after 30 days; Pro tier has unlimited retention.

## 2. Performance & Scalability
- **Constraint:** The Q&A Assistant must respond within 3 seconds.
- **Solution:** Use Supabase `pgvector` indexed searches to quickly retrieve documentation context.
- **Scaling:** The NestJS backend handles mostly lightweight CRUD and Webhook parsing. Heavy lifting is offloaded to the user's GitHub Actions runner.

## 3. Usability & Developer Experience (DX)
- **Constraint:** Developers should not have to leave their terminal/code editor to update documentation constraints.
- **Solution:** The `.docuai.yml` config-as-code pattern ensures DX is frictionless.
- **Constraint:** The dashboard UI must feel premium, modern, and developer-friendly.
- **Solution:** Next.js + Tailwind + minimal styling with dark mode by default.

## 4. Security & Privacy
- **Constraint:** Source code must not be permanently stored if not requested.
- **Solution:** DocuAI only stores the *generated documentation*, not the raw source code of the user.
- **Constraint:** User API keys must be securely encrypted at rest in PostgreSQL.
- **Constraint:** Role-based access must be enforced to prevent unauthorized access to documentation or settings.
- **Solution:** Backend validates user role on every API request; frontend redirects to permitted areas.
- **Constraint:** Free tier and Pro tier must be isolated (quota enforcement, feature availability).
- **Solution:** Backend checks tier on every request that has quota or feature limits (Q&A queries, repo count, doc history).
