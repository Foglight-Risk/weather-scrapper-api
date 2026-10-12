## Autonomous development contract

This repo is developed by both humans and coding agents. Any agent or harness working here
reads this section first and obeys it. It states repo facts, not process doctrine — your
harness supplies the process.

Machine-readable companion: `.agents/gate.json` (same commands, parseable).

### Shared agent memory

- **Read** `MEMORY.md` at session start before mutating work.
- **Dual-write:** durable lessons go in `MEMORY.md`, not only in a local runtime store.
- **No CI loop for pure memory commits:** commits touching only `MEMORY.md` skip the gate.
  Land the lesson on the default branch quickly.
- **Append-only** by default; rebase immediately before writing.

### Gate commands (the definition of "green" here)

```bash
npm run typecheck   # tsc --noEmit
npm test            # vitest run
```

Both must pass before any **product** commit, with **zero new failures versus the branch
point**. Pure memory-only commits are exempt.

### Ground rules for machine contributors

1. **Never mutate the shared checkout.** One task = one branch = one git worktree of your own.
2. **Claim before you touch.** Add an `Active-work claim: <slug>` line to an open PR body and
   a row in `docs/active-work.md`; add the `agent-claimed` label. Overlapping claim → stop and
   coordinate, never race.
3. **Never review or approve your own work.** An independent reviewer inspects the exact
   committed head, read-only.
4. **Acceptance criteria are the contract**, verified by a sensor, never by assertion.
5. **Report failures verbatim.** Never mark done what you did not verify.
6. **Clean up:** remove the worktree, delete the branch, release the claim.

### Hot files (single claimant, own PR)

| File | Rule |
|------|------|
| `migrations/*.sql` | Additive-only; never drop columns or tables; name the migration in the claim |
| `clients/ts/precip-client.ts` | Consumer-facing contract — changes here require updating both consumer copies |
| `clients/py/precip_client.py` | Same as above |
| `package.json` | No casual deps; own commit |

### Critical invariants

- **Read-through cache, not a proxy.** Resolve date range → query Neon → fetch only gaps →
  upsert → merge. Days older than 3 days are immutable and never refetched.
- **Missing data is never zero-filled.** Days absent from a response are absent, not `0`.
  Observations failing NOAA QC are dropped.
- `meta.cache` and `meta.upstream_calls` must accurately reflect what a response cost.
- `NEON_POSTGRESS_CONN_STR` is **intentionally misspelled** — do not correct it.
- Vercel CDN does **not** cache authenticated requests.
- Precipitation: **inches per hour**; timestamps naive local time; timezone dropped on
  validation.
- API contract changes must be propagated to both vendored clients and both consumer repos
  (`fog-light` and `rainhedge`), in separate repos/commits.

### Deployment truth (verified 2026-10-12)

- Host/project: Vercel — `weather-scrapper-api` (dmessing)
- Remote: `git@github.com:dmessing/weather-scrapper-api.git`
- Production URL: `vercel-fetch-noaa.vercel.app`
- Production branch: `main`
- Merge to `main` → Vercel auto-deploys; preview on every PR branch
- Migrations: applied manually with `npm run migrate` against Neon prod

### Stop and ask a human for

- **Money:** Vercel plan changes, NOAA CDO token rotation, new API keys
- **Schema:** any change to `migrations/*.sql` that is not purely additive
- **Auth / rate limits:** changes to `CLIENT_TOKENS` mechanism or per-consumer attribution
- **Secret values:** `NEON_POSTGRESS_CONN_STR`, `NOAA_CDO_TOKEN`, `CLIENT_TOKENS`
- **Public launches:** first exposure of a new endpoint

### Commit and PR conventions

- `git commit -s` (DCO — `Signed-off-by` required on every commit)
- Branch naming: `<type>/<slug>`
- Agents may **not** merge their own reviewed PRs without explicit human instruction
