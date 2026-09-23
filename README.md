[README.md](https://github.com/user-attachments/files/32561142/README.md)
# SeSha-AI-Marketing-SAAS
SeSha AI Marketing is a AI SAAS for digital marketing powered by AI 
# SeSha AI Marketing

An AI marketing platform for small and mid-sized businesses: marketing strategy, content, SEO and AEO audits on a first-party website crawler, structured data, competitor intelligence, advertising and campaign planning, leads, a marketing calendar, automations, analytics and reports — in one multi-tenant workspace.

Built and operated by **SeSha AI Marketing** · [seshaaimarketing.com](https://seshaaimarketing.com) · seshaaimarketing@gmail.com

**Next.js 15** (App Router) · **React 19** · **TypeScript** (strict, `noUncheckedIndexedAccess`) · **Tailwind CSS v4** · **PostgreSQL 16** · Node ≥ 20.11
**Four production dependencies:** `next`, `react`, `react-dom`, `clsx` + `tailwind-merge`. No ORM, no UI kit, no chart library, no PDF library, no payment SDK.

> ### Status: feature-complete, **not production ready**
>
> All nine development phases are done and 2,121 tests pass, but two critical blockers remain: **there is no PostgreSQL adapter** (the app runs on an in-memory store), and **the production build has never been run** (the project was developed without npm registry access). See [PRODUCTION-READINESS-REPORT.md](./PRODUCTION-READINESS-REPORT.md) for all five blockers, with severity, impact and the fix for each.

---

## Quick start

```bash
git clone https://github.com/<your-account>/sesha-ai-marketing.git
cd sesha-ai-marketing
rm -rf node_modules        # the archive ships offline stand-ins; they must go
npm install
cp .env.example .env.local # set AUTH_SECRET at minimum
npm run dev                # http://localhost:3000
```

Then, before anything else, run the gate that has never run:

```bash
npm run verify             # typecheck → lint → 2,121 tests → production build
```

Without `DATABASE_URL` the app uses an in-memory adapter: fine for development, and refused in production unless `ALLOW_MEMORY_DB=true` is set deliberately. With `DATABASE_URL` set, the app refuses to start until the PostgreSQL adapter exists — on purpose, rather than silently losing data.

## Scripts

| Command | What it does |
|---|---|
| `npm run dev` / `build` / `start` | Next.js development, production build, production server |
| `npm run verify` | typecheck + lint + tests + build — the real gate |
| `npm test` | 2,121 unit, service and gate tests (`node:test`, no browser) |
| `npm run test:e2e` | Playwright: 5 breakpoints plus Firefox, WebKit, Edge, Android, iOS |
| `npm run verify:mutation` | 21 deliberate breakages, each with the test that must catch it |
| `npm run sandbox:all` | The offline harness, for environments with no npm access |
| `node scripts/build-schema.mjs` | Regenerate `db/schema.sql` from the migrations |
| `node scripts/build-docs.mjs` | Regenerate `docs/API.md`, `docs/DATABASE.md`, `docs/ENVIRONMENT.md` |

## Database

PostgreSQL 16. 57 tables, 9 forward-only migrations, each one transaction, every foreign key indexed and row-level security on every tenant table.

```bash
psql "$DATABASE_URL" -v ON_ERROR_STOP=1 -f db/migrations/0001_init.sql
psql "$DATABASE_URL" -v ON_ERROR_STOP=1 -f db/migrations/0002_accounts.sql
psql "$DATABASE_URL" -v ON_ERROR_STOP=1 -f db/migrations/0003_ai_content.sql
psql "$DATABASE_URL" -v ON_ERROR_STOP=1 -f db/migrations/0004_crawl_audit.sql
psql "$DATABASE_URL" -v ON_ERROR_STOP=1 -f db/migrations/0005_schema_competitors_ops.sql
psql "$DATABASE_URL" -v ON_ERROR_STOP=1 -f db/migrations/0006_analytics_calendar_automation_reports.sql
psql "$DATABASE_URL" -v ON_ERROR_STOP=1 -f db/migrations/0007_subscriptions_usage_admin.sql
psql "$DATABASE_URL" -v ON_ERROR_STOP=1 -f db/migrations/0008_indexes_and_operations.sql
psql "$DATABASE_URL" -v ON_ERROR_STOP=1 -f db/migrations/0009_foreign_key_indexes.sql
```

Run `0007` as a whole file: it widens a `CHECK`, updates rows, then narrows it again, and that order matters. A test fails if this list ever drifts from the files on disk.

## Project layout

```
src/app/(marketing)   public website, 24 routes    src/lib/auth           sessions, RBAC, CSRF, lockout
src/app/(auth)        sign-in, sign-up, reset      src/lib/ai             providers, orchestrator, provenance
src/app/(app)         the customer workspace       src/lib/crawler        SSRF policy, fetcher, robots, parsing
src/app/(admin)       platform admin (staff only)  src/lib/db             repository interfaces + memory store
src/app/api           84 route handlers            src/lib/reliability    timeout, retry, circuit breaker
src/content           typed copy and config        src/lib/observability  redacting logger, metrics
src/components        UI and feature components    db/migrations          0001–0009
tests                 2,121 tests, incl. gates     scripts                offline harness, generators
```

## Deploying

| Target | Guide |
|---|---|
| GitHub + Netlify (`netlify.toml` and CI are in the repo) | [docs/NETLIFY-GITHUB-DEPLOY.md](./docs/NETLIFY-GITHUB-DEPLOY.md) |
| Any host, step by step, demo or real launch | [docs/GOING-LIVE.md](./docs/GOING-LIVE.md) |
| Docker / VPS topology, backups, monitoring | [docs/DEPLOYMENT.md](./docs/DEPLOYMENT.md), [docs/OPERATIONS.md](./docs/OPERATIONS.md) |

Every environment variable is documented in [`.env.example`](./.env.example) and [docs/ENVIRONMENT.md](./docs/ENVIRONMENT.md). No secret is ever committed; `.env*` files are git-ignored and a test enforces it.

## Documentation

[docs/README.md](./docs/README.md) indexes everything. The ones most people want:

- [ARCHITECTURE.md](./ARCHITECTURE.md) — the design record, Parts I–IX
- [PRODUCTION-READINESS-REPORT.md](./PRODUCTION-READINESS-REPORT.md) — status, blockers, known limitations
- [SECURITY-AUDIT.md](./SECURITY-AUDIT.md) — controls, findings and residual risks
- [docs/API.md](./docs/API.md) · [docs/DATABASE.md](./docs/DATABASE.md) — generated from the code
- [docs/TESTING.md](./docs/TESTING.md) · [docs/TROUBLESHOOTING.md](./docs/TROUBLESHOOTING.md) · [docs/ROADMAP.md](./docs/ROADMAP.md)
- [CLAUDE.md](./CLAUDE.md) — the rules any contributor (human or AI) follows

## What this product will not do

These are enforced by types and tests, not by good intentions:

- **No AI output without provenance.** A response that does not declare where each part came from is rejected, not displayed.
- **No invented numbers.** No fabricated analytics, search volume, ratings, reviews or prices. An unavailable metric renders its reason, never a zero.
- **No crawling without consent**, and the crawler cannot be pointed at a private network or a cloud metadata endpoint.
- **No message is ever sent and no ad money is ever spent.** Messaging, social and advertising are planning only.
- **No card data is storable.** There is no column in 57 tables that could hold one.
- **Cross-tenant requests return 404**, not 403, and an organization admin is not platform staff.

## Contributing

1. Branch from `main`, and read [CLAUDE.md](./CLAUDE.md) first — it is the contract for this codebase.
2. `npm run verify` and `npm run verify:mutation` must pass, and `node scripts/build-schema.mjs --check` and `node scripts/build-docs.mjs --check` must be clean.
3. A bug fix comes with a test that fails without it. A new security control comes with a mutation.
4. Open a pull request; the template lists the definition of done. CI runs the whole gate, plus the migrations against a real PostgreSQL 16 and Playwright on three engines.

## License

Copyright © 2026 SeSha AI Marketing. All rights reserved. This is proprietary software — see [LICENSE](./LICENSE). It is not open source, and no permission to use, copy, modify or distribute it is granted by its presence here.
