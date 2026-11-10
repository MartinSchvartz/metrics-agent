# metrics-agent

A subscription metrics dashboard with an agent you can ask questions in plain English, like *"Why did churn go up in March?"* The agent answers by querying the data, and always shows the SQL it ran.

The interesting part isn't the chat box. It's making text-to-SQL safe and measurably correct:

- **Guardrails:** the agent can only read a curated set of analytics views, through a read-only database role, with timeouts, row limits and a query cost check.
- **Evals:** a suite of questions with known answers runs in CI and scores accuracy, safety, latency and cost, so prompt or model changes can't silently make things worse.
- **Data with known answers:** a generator simulates a SaaS business and plants real events (a price change, a payment outage, a low-quality acquisition channel), so "why" questions have verifiable answers.

**Status:** planning. See [docs/PLAN.md](docs/PLAN.md).

**Stack:** Go · PostgreSQL · React + TypeScript · Claude API · GitHub Actions
