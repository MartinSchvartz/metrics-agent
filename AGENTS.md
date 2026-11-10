# AGENTS.md

Context for AI coding agents working in this repo. Read this first. Keep it up to date: when a decision changes or a milestone finishes, update this file in the same change.

## What this project is

`metrics-agent` is a portfolio project by Martin Schvartz (Senior Software Engineer, Buenos Aires). It's a subscription metrics dashboard with an agent that answers plain-English questions by querying Postgres and always shows the SQL it ran.

The point of the project is to show that LLM features can be made **safe and measurably correct**: guardrails, evals in CI, and cost controls. Hiring managers and engineers are the audience. The README, the eval report and a write-up matter as much as the code.

Full plan: [docs/PLAN.md](docs/PLAN.md). Other project ideas live outside the repo, in `~/Desktop/portfolio-project-ideas.md`.

## Current status

- **Milestone:** 0, planning done. Next up is milestone 1 (data: migrations, generator, analytics views).
- Update this section when a milestone starts or finishes.

## Decisions already made (don't re-ask)

- **Repo:** `github.com/MartinSchvartz/metrics-agent`, public from day one.
- **Data:** a synthetic generator with planted events (see PLAN.md). Stripe test mode was considered and rejected for now, because synthetic data gives known answers for evals.
- **Backend:** Go. **Database:** PostgreSQL. **Frontend:** React + TypeScript. **LLM:** Claude API (Sonnet as the default; Haiku and Opus compared in evals).
- **Guardrails:** database permissions are the primary control. The agent's role can only `SELECT` from the `analytics` schema. Application checks (single statement, timeout, row cap, `EXPLAIN` cost limit) are a second layer, never the only one.
- **No personal data** in the `analytics` schema.
- **Time budget:** 5–10 hours a week, about 6–8 weeks.

## How to work with Martin

- **Plan first:** for any non-trivial change, show a short plan and wait for an OK before writing code.
- **Ask before pushing** to GitHub, and before anything outward-facing (deploys, publishing, spending money).
- **Verify before claiming it works:** run the build, the tests and the evals, and report what actually happened, including failures.
- **Copy** (README, UI text, write-ups): short, plain and human. No buzzwords, no marketing tone.
- **Keep the scope tight:** suggest extras, don't build them unasked.

## Local environment

- macOS, Go 1.27, Node 26, PostgreSQL 18 via Homebrew (running as a service on `localhost:5432`). Docker is not installed.
- The repo lives in `~/code/metrics-agent`. Don't put projects on the Desktop: it's synced to iCloud, which creates " 2" duplicate files that break builds.
- Secrets (like `ANTHROPIC_API_KEY`) go in `.env`, which is git-ignored. Never commit secrets. Keep `.env.example` current.

## Commands

Fill these in as they're created.

| Task | Command |
|------|---------|
| Create and seed the local database | _milestone 1_ |
| Run the API and dashboard | _milestone 2_ |
| Run tests | `go test ./...` |
| Run evals | _milestone 4_ |

## Conventions

- **Layout:** `cmd/` holds binaries (api, seed, evals), `internal/` holds packages, `migrations/` has plain SQL, `web/` has the frontend and `evals/cases/` has the YAML eval cases.
- **Go:** standard library first. Keep dependencies few and justified. `pgx` for Postgres. Errors are wrapped with context. Table-driven tests.
- **SQL:** migrations are plain numbered `.sql` files. Analytics views get a comment explaining each metric's definition, because the agent reads them.
- **Evals:** any change to prompts, tools, view descriptions or the model must be run against the eval suite, with the before/after numbers in the pull request description.
- **Commits:** small and focused, with a message that says why.
