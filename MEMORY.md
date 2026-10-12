# MEMORY.md — shared agent memory

**Read this early** in any coding, review, or merge session.
**Update it** when the same failure class appears twice (prefer a test/sensor over a long essay).

Keep this file short so every agent loads it.

### Dual-write rule (mandatory)

Whenever an agent writes durable operational memory in a local runtime store,
**also write here** so other agents and machines share the same context.

### No CI loop for memory-only updates

Commits that touch **only** this file do **not** require typecheck, lint, test, build, or
product PRs. Land the lesson on the default branch quickly.

_Last updated: 2026-10-12_

---

## Operating notes

| Topic | Rule |
|-------|------|
| Claims | Open a PR with `Active-work claim: <slug>` in the PR body; add `agent-claimed` label |
| Green gate | `npm run typecheck && npm test` — see `.agents/gate.json` |
| Deploy | Vercel project `weather-scrapper-api`; host `vercel-fetch-noaa.vercel.app`; main → production |
| Migrations | `npm run migrate` — idempotent; applied against Neon (`NEON_POSTGRESS_CONN_STR`) |

---

## Critical invariants (do not violate)

- **This is a read-through cache, not a proxy.** Resolve date range → query Neon → fetch
  only gaps upstream → upsert → merge. Never refetch immutable days (>3 days old).
- **Missing data is never zero-filled.** Days absent from a daily response are absent, not `0`.
  Observations failing NOAA's QC flag are dropped.
- `meta.cache` and `meta.upstream_calls` must reflect what a response actually cost.
- Vercel CDN does **not** cache authenticated requests — every request reaches a function and
  Neon regardless of `s-maxage`.
- `NEON_POSTGRESS_CONN_STR` is **intentionally misspelled** (matches deployed variable name).
  Do not "fix" the spelling.
- Precipitation units: **inches per hour**. Timestamps are naive local time; timezone
  dropped on validation.
- `clients/ts/precip-client.ts` and `clients/py/precip_client.py` are the canonical consumer
  clients. Changing the API contract means updating both and re-copying into consumers
  (`fog-light/src/lib/precip-client.ts`, `rainhedge/src/rainhedge/precip_client.py`).

---

## Lessons (append-only)

### 2026-10-12 — adoption-seed

- Repo adopted Gibson agent-memory conventions (AGENTS memory block + this file).
- Pure memory commits skip the product green gate; dual-write is mandatory.
