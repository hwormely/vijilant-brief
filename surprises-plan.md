# Vijilant — Plan to Close the 5 Architecture Surprises

The architecture map at `/architecture-map.html` surfaced five named drift items in the codebase. This document is the executable plan to close each one before the stand-up window closes.

**Effort scale:** S = ≤2h · M = ≤1 day · L = 1–3 days.
**Risk:** describes what breaks if the fix goes wrong.

---

## Surprise #1 — Dead code in `lib/systemFormTemplates.ts`

### Finding
`apps/api/src/lib/systemFormTemplates.ts` is 276 lines with full passing tests but **zero production callers**. Verified by grep — only `systemFormTemplates.test.ts` imports it. This was intended as Phase 2 step 7 ("system template seed") but was never wired up.

### Plan — two paths, pick one

**Path A — Delete (recommended for the stand-up window).** Move the file + its test into `archive/` with a single-paragraph note explaining the intent. Keep the git history; future-you can find it.
- Effort: S (15 min).
- Risk: zero. Anything depending on it would already be broken.
- Steps:
  1. `git mv apps/api/src/lib/systemFormTemplates.ts archive/lib/systemFormTemplates.ts`
  2. `git mv apps/api/src/lib/systemFormTemplates.test.ts archive/lib/systemFormTemplates.test.ts`
  3. Add `archive/README.md` if not already there with a one-line index.
  4. Commit: `chore(api): archive unused systemFormTemplates (Phase 2 step 7 not wired)`

**Path B — Land step 7.** Wire it as a one-time seed callable from the Operator Console: "Seed system form templates for org X." Useful only if you anticipate Phase 2 form builder shipping this quarter.
- Effort: M.
- Risk: low. New seed path; idempotent.
- Steps:
  1. Add `POST /api/operator/orgs/:id/seed-system-templates` (platform-admin only).
  2. Wire the existing functions through.
  3. Test against a staging org.
  4. Document in the Operator console UI.

### Pick
**Path A.** Step 7 isn't on the 7-day stand-up critical path; the dead weight in the bundle is.

---

## Surprise #2 — Mobile error reporting is uninstrumented

### Finding
`apps/mobile/package.json` lists `@honeybadger-io/js` but the imports were explicitly removed from `app/_layout.tsx:25-29` because the **web SDK** breaks on React Native. The React Native SDK (`@honeybadger-io/react-native`) is not installed. Production mobile crashes go nowhere.

### Plan
Install the RN SDK and wire it to the same Honeybadger project (so web + API + mobile crashes converge on one inbox).

1. `pnpm --filter mobile add @honeybadger-io/react-native`
2. Remove `@honeybadger-io/js` from `apps/mobile/package.json` (it's web-only — keep it only where it works).
3. In `apps/mobile/app/_layout.tsx`:
   ```ts
   import Honeybadger from '@honeybadger-io/react-native';
   Honeybadger.configure({
     apiKey: process.env.EXPO_PUBLIC_HONEYBADGER_API_KEY,
     environment: __DEV__ ? 'development' : 'preview',
     beforeNotify: (notice) => {
       // PHI scrub — strip request bodies, headers, query strings that may contain ids
       if (notice?.request?.params) notice.request.params = '[scrubbed]';
       return notice;
     },
   });
   ```
4. Wrap the root tree with `<Honeybadger.ErrorBoundary>` so render-time errors are captured.
5. Add a smoke test: a hidden "force crash" button in dev builds that throws — verify the crash lands in Honeybadger.
6. Trigger a new EAS preview build → confirm.

- Effort: M (couple of hours, plus one new EAS build).
- Risk: **PHI exposure if `beforeNotify` is wrong.** Mitigation: every Vijilant Honeybadger config (API, web, mobile) shares the same scrub policy. Verify against the same payload contract.
- Cost: $0 incremental (Honeybadger Solo $39/mo covers all environments).

---

## Surprise #3 — Schema drift on `resource_matches`

### Finding
`resource_matches` is queried by 6 code paths (web hooks + mobile hooks + scripts) but its `CREATE TABLE` lives in `supabase/migrations/BACKFILL_resource_matches.sql`, not in the numbered migration sequence. A fresh apply that only runs the numbered files would skip the table.

### Plan
Promote the BACKFILL to a numbered migration, keep the schema identical so existing live data is untouched.

1. Open `supabase/migrations/BACKFILL_resource_matches.sql`.
2. Copy the `CREATE TABLE` into `supabase/migrations/053_resource_matches.sql` (next number — verify highest current).
3. Wrap in `IF NOT EXISTS` so re-running against the live DB is a no-op.
4. Add RLS policies that match the implicit policy live data is already operating under (verify by querying `pg_policies` on live before writing).
5. Delete `BACKFILL_resource_matches.sql` (or move to `archive/migrations/`).
6. Run `supabase db diff` locally against a fresh schema to confirm parity.
7. Apply migration 053 to staging → smoke-test the 6 query sites.

- Effort: M (1 day including verification).
- Risk: **catastrophic if RLS policy is wrong on the new migration.** Mitigation: read live policies first; the new migration replicates them exactly. Bastion review before merge.

---

## Surprise #4 — AI barrel intent is only partially enforced

### Finding
`lib/ai/index.ts` was meant to be the single seam for every Claude call (the place where prompt caching, retries, and observability live). In practice only `lib/triageEngine.ts` imports the barrel. Four routes (`routes/briefs`, `routes/supervisorBrief`, `routes/aiDrafts`, `routes/survivorSignup`) import the underlying `lib/ai/*` files directly, bypassing the barrel.

### Plan
Route every AI call through the barrel. This is mostly mechanical — replace direct imports with barrel imports — but it requires expanding the barrel's exports.

1. Audit the four bypassing routes. List every named export they pull from `lib/ai/*`:
   - `briefs.ts` → `generateClientBrief`
   - `supervisorBrief.ts` → `generateSupervisorBrief`
   - `aiDrafts.ts` → `draftCaseNote`, `prefillAidApplication`, `proposePostAssessment`, `draftSurvivorMessage`
   - `survivorSignup.ts` → `verifyGovIdImage`
2. Re-export each from `lib/ai/index.ts`.
3. Replace direct imports across the four routes (typescript-aware find/replace).
4. Add an eslint rule: `no-restricted-imports` pattern `lib/ai/**` (except inside `lib/ai/` itself) → forces future code through the barrel.
5. Land an empty observability layer in the barrel (no-op today but the seam for tomorrow's prompt caching + retry + telemetry). Concretely: a `withAiObservability(fn)` higher-order function the barrel wraps every export in.
6. Confirm no behavior change (every route renders identical output for the same input).

- Effort: M (1 day).
- Risk: **circular import** if the eslint rule is added before the imports are fixed. Mitigation: fix the imports first, then add the rule.
- Payoff: prompt caching can ship in one PR after this lands — saves materially on Anthropic spend.

---

## Surprise #5 — Quadruple mount on `/api/clients`

### Finding
`apps/api/src/index.ts:108-111` mounts **four** routers on the same `/api/clients` prefix:
- `assessmentsRouter`
- `clientsRouter`
- `briefsRouter`
- `aiDraftsRouter`

Express delegates to the first router whose handler matches. The four files maintain disjoint sub-paths today, but any overlap is invisible drift. Index.ts has a long comment block warning about this.

### Plan — three options, pick one

**Option A — Document, don't merge (recommended for the stand-up window).** Add an `OWNED_PATHS` constant + JSDoc header to each of the four routers listing exactly which sub-paths they own. CI lint that fails the build if any of the four declare a sub-path that another also declares.
- Effort: M.
- Risk: low. Pure documentation + linter.
- Steps:
  1. Each of the four routers gets a header constant:
     ```ts
     export const OWNED_PATHS = ['/', '/:id/notes', '/:id/followups', …] as const;
     ```
  2. Write a script `scripts/check-clients-mount-disjoint.ts` that loads all four `OWNED_PATHS` arrays and asserts their intersection is empty.
  3. Wire into `pnpm check:v5` or its own `pnpm check:mounts` step.
  4. Update the index.ts comment block to reference the check.

**Option B — Merge into a single router.** All four files combine into one `routes/clients/` directory with sub-files. Single `app.use('/api/clients', clientsRouter)`.
- Effort: L (2–3 days including re-test).
- Risk: medium. Lots of moved code; lots of imports to update.
- Payoff: zero ambiguity forever.

**Option C — Move the AI surfaces off `/api/clients`.** `aiDrafts` becomes `/api/ai-drafts`. `briefs` becomes `/api/briefs`. Only `clients` and `assessments` stay on `/api/clients` (and they're already cleanly disjoint).
- Effort: M (mostly a public API change).
- Risk: medium — every mobile + web caller that hits the AI sub-paths needs updating.

### Pick
**Option A.** Within the 7-day window the cost/benefit favors documentation + a linter. Option B is the right v1.1 move once the messaging+billing branch is on main.

---

## Sequence summary

```
Day 1   #1 archive systemFormTemplates                          (S, no risk)
Day 1   #3 promote BACKFILL_resource_matches to 053             (M, RLS review needed)
Day 2   #2 install RN Honeybadger + new EAS preview build       (M, $0 cost)
Day 3   #5 OWNED_PATHS + lint check                              (M, no behavior change)
Day 3–4 #4 route AI through barrel + observability seam          (M, payoff: cacheable next)
```

All five close inside 4 days. None blocks any other surface on the 7-day stand-up plan.

---

## Verification checklist (run before the chapter closes)

- [ ] `grep -rn systemFormTemplates apps/api/src` returns nothing.
- [ ] `apps/mobile` Honeybadger smoke test → crash appears in inbox within 60s.
- [ ] `supabase db diff --schema public` clean against the numbered migrations only.
- [ ] `grep -rn "from '\\.\\./lib/ai/" apps/api/src/routes` returns nothing (all go through `lib/ai/index.js`).
- [ ] `pnpm check:mounts` exits 0 and the four `OWNED_PATHS` constants exist.
- [ ] `pnpm build` + `mobile tsc --noEmit` + `pnpm check:v5` all clean.
