# Vijilant — H Must-Do Checklist

The consolidated list of items only H can complete to fully stand Vijilant up at the enterprise bar. Compiled 2026-06-06 after the 5-PR architecture cleanup session.

**Read order:** P1 (blocking) → P2 (within 14 days) → P3 (within 30 days). Each item lists what H does, who supports it on the MITA side, and where the dependency unblocks.

---

## P1 — blocks first paying customer or first Enterprise/Government deal

### 1. Counsel-approved BAA, MSA, DPA, NPP text
- **Why:** `apps/api/src/lib/baaTemplate.ts` is a labeled placeholder. Stripe must stay TEST until counsel signs off. No real org can sign the BAA today.
- **What H does:** schedule counsel time this week. Provide the existing placeholder text + the pricing PRD as context. Get back signed text for all four documents.
- **Routes to:** MITA-03 Counsel (review + bump `BAA_TEMPLATE_VERSION` on text change).
- **Unblocks:** Stripe LIVE keys · first paid Starter/Pro signup · in-app BAA signing flow · HIPAA Covered Entity orgs.

### 2. Stripe LIVE keys in Render
- **Why:** Stripe is in TEST mode today. Cannot accept real payments.
- **What H does:** in the Render dashboard, set:
  - `STRIPE_SECRET_KEY=sk_live_…`
  - `STRIPE_WEBHOOK_SECRET=whsec_…`
  - Three `STRIPE_*_PRICE_ID=price_…` values (from the seeded products — `seed-stripe-products.mjs` is idempotent, re-run if needed).
- **Prerequisite:** counsel BAA + MSA + DPA + NPP signed (item #1) so customers can actually transact.
- **Routes to:** MITA-09 Ledger (reconciliation cadence) + MITA-03 Counsel (sign-off).
- **Unblocks:** real-money signups.

### 3. Merge 5 open PRs in `hwormely/vigilant`
PR review + merge is yours alone. Diff is small and well-scoped for each:
- [PR #1](https://github.com/hwormely/vigilant/pull/1) — `chore: OWNED_PATHS lint for /api/clients quadruple-mount`. **Risk: none.** Pure additive constants + check script. 6 files, +190 lines.
- [PR #2](https://github.com/hwormely/vigilant/pull/2) — `refactor: route every Claude call through lib/ai/index.ts barrel`. **Risk: none — pure refactor.** 7 files, +128 / -10. Trivial conflict with PR #1 in `package.json` (both add a sibling line).
- [PR #3](https://github.com/hwormely/vigilant/pull/3) — `fix(db): promote BACKFILL_resource_matches.sql → 053_resource_matches.sql`. **Risk: none against live DB** (all guards `IF NOT EXISTS`). 4 files.
- [PR #4](https://github.com/hwormely/vigilant/pull/4) — `fix(mobile): install @honeybadger-io/react-native + wire crash reporting`. **Risk: TS clean; mobile validation requires a new EAS preview build.** 4 files, +47 / -35.
- [PR #5](https://github.com/hwormely/vigilant/pull/5) — `docs(mita): canonical Vijilant ↔ MITA ↔ Padrino integration map`. **Risk: none — docs only.** 2 files, +150.

### 4. New EAS preview build for Honeybadger mobile validation
- **After PR #4 merges.** Set `EXPO_PUBLIC_HONEYBADGER_API_KEY` in the EAS preview env (point at the same Honeybadger project the API + web report to). Then:
  ```
  eas build -p ios --profile preview
  eas build -p android --profile preview
  ```
- **Smoke test on the new build:** trigger a known throw → verify it appears in Honeybadger with the PHI-scrub applied (no params/context/session/breadcrumbs).
- **Routes to:** MITA-02 Keeper (the observability surface).
- **Unblocks:** mobile production crashes have an inbox.

### 5. Supabase Pro upgrade — turn on HIBP leaked-password protection
- **Why:** HIPAA-adjacent SaaS without a leaked-password check is a procurement red flag. Pro-plan-gated feature.
- **What H does:** upgrade Supabase project `ezcaausjenhlylbhlmtd` to Pro → enable Attack Protection → re-run the security advisor (should show clean).
- **Routes to:** MITA-11 Bastion (sign-off after re-run).
- **Cost:** +$25/mo on the project (from Pro tier).

---

## P2 — needed within 14 days to claim enterprise readiness

### 6. Domain + DNS hardening
- **CAA records** on `vijilant.io` restricting which CAs can issue certs (Let's Encrypt only, or whichever CA Vercel/Render uses).
- **SPF + DKIM + DMARC** records for outbound mail (Resend uses your domain). DMARC at `p=quarantine` minimum.
- **Registrar 2FA** — YubiKey, not SMS.
- **Routes to:** MITA-02 Keeper.

### 7. Reconcile the 9 remaining drift migrations
- `~/vigilant/supabase/migrations/DRIFT_NOTE.md` lists 9 migrations applied live but missing from the repo: `cleanup_profiles`, `resources_visibility`, `resource_intelligence`, `escalations`, `client_intake`, `documents`, `badges`, `duplicate_flags`, `024_intake_workflow`.
- **What H does:** dump the live DDL for each (`pg_dump --schema-only` against the relevant tables) and either re-author each as a numbered migration OR accept that the DRIFT_NOTE remains the source of truth and update it with the per-migration DDL inlined.
- **Routes to:** MITA-01 Builder (do the work) + MITA-11 Bastion (review RLS implications).

### 8. SOC 2 Type I scoping call
- 60-day project. Type I is the right starting point — Type II requires 6 months of observed controls.
- **What H does:** schedule a discovery call with an auditor (Vanta · Drata · A-LIGN · Secureframe — all have HIPAA SaaS templates). Get pricing (typical: $15–30k for Type I + automation tooling).
- **Routes to:** MITA-03 Counsel + MITA-11 Bastion.

### 9. Anthropic Zero-Data-Retention (ZDR) opt-in
- **Why:** PHI flows to Anthropic via every Claude call. ZDR + Anthropic BAA together make Anthropic a sound PHI processor.
- **What H does:** request ZDR via Anthropic's enterprise contact form. Sign the BAA Anthropic offers for HIPAA customers.
- **Routes to:** MITA-03 Counsel.

### 10. WorkOS account — **OR** commit to in-house SAML (your call from earlier)
- You picked "build SAML in-house, defer SCIM" in the planning round. That's a 3–5 engineer-week effort tracked in the enterprise-gap doc. **What H does:** confirm priority for next session (build now vs. defer until first Enterprise lead actually lands).
- **Routes to:** MITA-01 Builder.

### 11. Deepgram BAA — for AI Call Notation live audio
- Notation works in test mode today (synthetic transcript → Claude bilingual draft → real case note). Live audio is gated until the Deepgram BAA is signed AND the mobile `expo-audio` install lands.
- **What H does:** initiate Deepgram BAA negotiation. Once signed, set `DEEPGRAM_API_KEY` in Render → flip `NOTATION_ENABLED=true`.
- **Routes to:** MITA-03 Counsel + MITA-01 Builder.

### 12. PRD.md brand-spelling + v5 tier color sweep
- `~/vigilant/PRD.md` uses "Vigilant" (G) throughout in user-facing prose and references the pre-v5 tier colors (`colors.green` instead of the v5 ramp). 50+ replacements.
- **What H does:** approve a sweep PR. Could be done by a Claude session in 30 minutes once approved.
- **Routes to:** MITA-06 Atlas (doc steward) + MITA-05 Studio (design).

---

## P3 — within 30 days for credible Enterprise + Government conversations

### 13. Customer-facing status page
- `status.vijilant.io`. Better Stack or Statuspage. $29/mo. Procurement always asks.
- **Routes to:** MITA-02 Keeper.

### 14. PagerDuty (or Better Stack) for on-call
- H is single point of failure today. Set up a rotation (even if it's just H + one other for now) + paging.
- **Routes to:** MITA-02 Keeper.

### 15. Public API + OpenAPI spec + at least one SDK
- Enterprise customers want to push intake data from their CRM. The schema additions (api_keys, webhook_subscriptions) are designed in `enterprise-gap-analysis.md`.
- **Routes to:** MITA-01 Builder.

### 16. HIBP for survivor passwords (post-Pro upgrade)
- Item #5 unlocks this. After Supabase Pro lands, the actual flip is a config toggle. Verify by trying a password from a known breach list — should be rejected.
- **Routes to:** MITA-11 Bastion.

### 17. Quarterly DR drill
- Restore a Supabase snapshot to a staging project. Verify integrity. Document the runbook in `~/vigilant/RUNBOOK.md` (doesn't exist yet — create on next session).
- **Routes to:** MITA-02 Keeper.

### 18. LLP migration dress rehearsal
- See `migration-roadmap.md` for the 21-day plan. The single biggest external dependency is QW delivering a clean LLP data export.
- **What H does:** send Ria a formal export request this week. Cite the schema mapping in `migration-roadmap.md`.
- **Routes to:** MITA-15 Hearth + MITA-01 Builder.

### 19. Investor / brief PDF refresh
- `Vijilant_Executive_Summary_v2.pdf` + `Vijilant_Seed_Investor_Deck_v2.pdf` + the two case-study PDFs are dated 2026-05-23 / 2026-05-24. Missing ~10 days of progress (Stage 8 Phase 4 + Stage 9 + Spanish i18n + Android EAS fix + the 5 architecture cleanup PRs).
- **What H does:** schedule a refresh session — python-pptx rebuild matching the v5 design system (already noted in `~/Padrino/09_Dev/CLAUDE.md` memory).
- **Routes to:** MITA-07 Signal + MITA-05 Studio.

---

## Decisions waiting on H (no specific timeline)

These aren't time-critical but matter for the next 60 days:

- **Token key naming in `@vigilant/tokens`.** Pre-v5 keys (`green`, `amber`) survived with v5 values. Either migrate keys to semantic names (`tier1Color`, etc. — L effort, touches every consumer) or add a clarifying comment block to `colors.ts` (S effort). Raised in `document-audit.md`.
- **Architecture map's quadruple mount on `/api/clients`.** PR #1 documents the disjointness via `OWNED_PATHS`. A v1.1 option is to merge the four files into one `routes/clients/` directory with sub-files. Not urgent.
- **Multi-language scope.** Spanish demo lands on main as of `b0470fc`. What's the next priority — finish Spanish across every screen, or add a second language (French · Mandarin · Vietnamese · Korean)? Drives the `multilingual` skill prioritization.

---

## What this session shipped (not on H's list — already done)

For completeness — the work that was done on this session that does NOT need H action:

- 5 PRs in `hwormely/vigilant` (numbered above as P1 item #3).
- 1 commit in `~/Padrino` — Padrino-side MITA ↔ Vijilant integration map.
- 9 new docs in `hwormely.github.io/vijilant-brief/` (architecture map · feature roadmap · UI/UX roadmap · migration roadmap · surprises plan · security hardening · enterprise gap analysis · document audit · this checklist).
- 4 investor + case-study PDFs hosted in the brief repo.
- 1 Cowork handoff: `VIJILANT_Overhead_Margins_Handoff.md`.
- Memory updated: flipped the stale "no notifications table" claim.

---

## How to track

This checklist is a living document. As H closes items, move them to a "Done" section at the bottom rather than deleting — keeps the audit trail intact. New items get added to the appropriate priority section with a one-line cite-back.

When P1 is empty, Vijilant is shippable at the enterprise bar.
