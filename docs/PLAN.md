# Plan

## Goal

A portfolio project that shows I can build a full product (Go backend, Postgres, React frontend) **and** make an LLM feature reliable: guarded, evaluated and cost-controlled.

The finished project is:
- a live demo
- this repo, with an eval report in the README
- a write-up: *"How I took text-to-SQL accuracy from X% to Y%, and what didn't help."*

Time budget: 5–10 hours a week, so about **6–8 weeks** in total.

## Architecture

```
React dashboard ──► Go API ──► PostgreSQL
   │                  │          ├─ app schema       (raw tables, generator writes here)
   │                  │          └─ analytics schema (curated views; the only thing the agent sees)
   └─ "Ask" panel ──► Agent loop ──► Claude API
                        │
                        └─ tools: list_views · describe_view · run_query
                                     │
                                     └─ guard: read-only role, allowlist, timeout, row cap, cost check
```

One Go binary serves the API and the built frontend. The eval runner is a second binary that uses the same agent code.

## Data model

**Raw tables** (`app` schema), filled by the generator:
- `plans`: name, billing interval, price
- `customers`: signup date, country, acquisition channel, company size
- `subscriptions`: customer, plan, start and end dates, status, cancellation reason
- `subscription_events`: new, upgrade, downgrade, churn, reactivation, with the MRR change
- `invoices`: amount, paid or failed, issue date

**Analytics views** (`analytics` schema), the agent's whole world:
- `mrr_monthly`: total MRR by month
- `mrr_movements`: new, expansion, contraction, churned and reactivated MRR by month
- `churn_monthly`: customer churn and revenue churn rates
- `cohort_retention`: retention by signup month
- `plan_mix`: customers and MRR by plan over time
- `acquisition`: signups and retention by channel

No personal data (emails, names) is exposed in the analytics views, and the evals test that the agent can't get to it.

## Planted events (the ground truth for "why" questions)

The generator simulates about two years of a SaaS business, with these events built in:

| When | Event | Visible effect |
|------|-------|----------------|
| Mar 2025 | Pro plan price increase | Churn spike on Pro for ~2 months; some downgrades to Basic |
| Jul 2025 | Payment processor outage | Failed invoices, then involuntary churn the following month |
| Sep 2025 | Partner campaign | Signup spike from the "partner" channel; that cohort retains poorly |
| Every Dec | Seasonal slowdown | Fewer signups |

Each event becomes a group of eval questions with a known answer.

## Agent

- **Tools:** `list_views`, `describe_view` (columns plus plain-English metric definitions), `run_query`.
- **Loop:** Claude decides which tools to call. The final answer includes the number or explanation, the SQL that was run, and the result rows.
- **Model:** Claude Sonnet as the default, with Haiku and Opus compared in the evals.

## Guardrails (defense in depth)

1. **Database permissions do the real work.** The agent connects as a role that can only `SELECT` from the `analytics` schema. Even if everything else fails, it can't write data or read raw tables.
2. **A single-statement check:** only one `SELECT` or `WITH` query per call.
3. **Limits:** `statement_timeout`, a row cap on results, and an `EXPLAIN` cost ceiling before running.
4. **Prompt-injection and abuse tests in the eval suite:** "drop the table", "show me customer emails", "ignore your instructions".

## Evals

- **Cases:** YAML files with a question, an expected answer and a grader. Graders:
  - `numeric`: a value with a tolerance, compared against a reference SQL query
  - `rows`: an expected set of rows
  - `explains`: for "why" questions, an LLM judge with a rubric based on the planted cause
  - `refuses`: for abuse cases, the agent must decline and no guardrail may be bypassed
- **Scores:** accuracy, guardrail violations (must be zero), latency (p50/p95), tokens and cost per question.
- **CI:** runs on every pull request against a seeded Postgres, posts a report, and fails if accuracy drops below the baseline or any guardrail is violated.

## Public demo safeguards

- A per-IP rate limit and a global daily token budget; when the budget runs out, the demo says so politely.
- Cached answers for the suggested example questions.
- Hosting: Go on Fly.io (or similar), with managed Postgres such as Neon. Exact choice made in milestone 6.

## Milestones

Each milestone is roughly a week at 5–10 hours.

1. **Data.** Repo layout, migrations, the generator with planted events, analytics views, and tests that check the metric math. *Done when:* `make seed` produces two years of data and the views return believable numbers.
2. **Dashboard.** Go API endpoints and a React dashboard: MRR, MRR movements, churn, cohorts. *Done when:* the dashboard runs locally against seeded data.
3. **Agent v1.** Tool loop, guardrails, and an "Ask" panel that shows the answer, the SQL and the rows. *Done when:* it answers simple questions correctly and refuses the abuse cases.
4. **Evals.** About 40 cases (including abuse cases), the runner, CI, and the first baseline report. *Done when:* every pull request gets a score.
5. **Make it better.** A metric glossary, example queries, better view descriptions, and a model comparison. Every change is measured against the baseline. *Done when:* there's a clear accuracy improvement with numbers behind it.
6. **Ship.** Deploy, add cost controls, polish the README, write the article, and add the project to martinschvartz.github.io. *Done when:* the live demo link works for anyone.

## Repo layout

```
cmd/api      HTTP API, also serves the built frontend
cmd/seed     data generator
cmd/evals    eval runner
internal/    db, generator, metrics, agent, guard
migrations/  SQL schema and views
evals/cases  eval cases (YAML)
web/         React + TypeScript dashboard
docs/        plan and write-up drafts
```
