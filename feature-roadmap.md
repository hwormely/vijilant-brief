# Vijilant — Built vs. Remaining

A clear-eyed read of what is in production today, what is committed but not deployed, and what is still ahead. Compiled 2026-06-06 against the live `~/vigilant` monorepo and all build handoffs in `~/Claude CoWork/Outputs/Vigilant/Business/`.

Stand-up window: **target = 14 days** (extended from the original 7-day spec, per H 2026-06-06).

---

## What shipped 2026-06-06 (this session)

Five PRs against `hwormely/vigilant`. All open for H to review + merge.

- [PR #1](https://github.com/hwormely/vigilant/pull/1) — `chore: OWNED_PATHS + check:mounts for /api/clients quadruple-mount` — closes Surprise #5
- [PR #2](https://github.com/hwormely/vigilant/pull/2) — `refactor: route every Claude call through lib/ai/index.ts barrel` — closes Surprise #4
- [PR #3](https://github.com/hwormely/vigilant/pull/3) — `fix(db): promote BACKFILL_resource_matches.sql → 053_resource_matches.sql` — closes Surprise #3
- [PR #4](https://github.com/hwormely/vigilant/pull/4) — `fix(mobile): install @honeybadger-io/react-native + wire crash reporting` — closes Surprise #2
- [PR #5](https://github.com/hwormely/vigilant/pull/5) — `docs(mita): canonical Vijilant ↔ MITA ↔ Padrino integration map` — wires the 16-agent MITA system to the codebase

Surprise #1 (archive `lib/systemFormTemplates.ts`) turned out to be a **false positive** — file IS used by `apps/api/scripts/seed.ts:49`. Architecture map updated.

Bonus: significant prior architecture work was found ALREADY on main — Stage 7 messaging, Stage 8 billing/BAA/onboarding, Stage 8 Phase 4 pricing portal, Stage 9 AI Call Notation, Spanish i18n foundation, mobile mic capture, Metrics KPI gradient fixes. The roadmap status table below reflects this.

Full owner list of remaining items: see `h-must-do.md` for the consolidated P1 / P2 / P3 H-checklist.

---

## Status legend
- **SHIPPED** — committed to `main`, deployed to web (Vercel) and/or built into the iOS preview build, end users can touch it.
- **STAGED** — committed but not yet merged to `main`, or merged but not yet behind a preview build.
- **BUILT, GATED** — code complete and merged, behind an env flag waiting on a vendor or counsel dependency.
- **PARTIAL** — surface exists, key sub-feature missing.
- **PLANNED** — has a spec / handoff, not started.
- **GAP** — known gap, no spec yet.

---

## Foundation (Stages 1–3)

| # | Feature | Status | Notes |
|---|---|---|---|
| 1 | Express 5 API server · helmet · CORS · rate limiting · cron | **SHIPPED** | `apps/api/src/index.ts:1-176`. Render-hosted. |
| 2 | Supabase Auth (JWT) · profile + org bootstrap | **SHIPPED** | `middleware/auth.ts`. Multi-role via `permitted_roles`. |
| 3 | RLS-protected schema, 51 numbered migrations applied | **SHIPPED** | One drift item: `resource_matches` lives in a `BACKFILL_` file, not numbered. |
| 4 | 12-section assessment form (`section_data` JSONB) | **SHIPPED** | Form on mobile + web. AI auto-triage feeds tier. |
| 5 | Deterministic Recovery Score 0–1000 · 5 visible domains + 2 silent modifiers | **SHIPPED** | `lib/computeDomainScores.ts:1-507`. LOCKED H 2026-05-24. |
| 6 | Triage Engine · band-based tier · Claude rationale | **SHIPPED** | `lib/triageEngine.ts`. Crisis flag forces T4. |
| 7 | Recovery Plan generator (Claude Opus 4.7) | **SHIPPED** | Best-effort; assessment commits regardless. |
| 8 | Client CRUD · notes · follow-ups · goals · archive · reassign · tier | **SHIPPED** | `routes/clients.ts` (970 lines, 14 endpoints). Refactor candidate. |
| 9 | Per-client AI brief (cached 5 min) | **SHIPPED** | `routes/briefs.ts`. Invalidated on writes. |
| 10 | Per-org supervisor brief (cached 5 min) | **SHIPPED** | `routes/supervisorBrief.ts`. Supervisor/admin only. |
| 11 | Reports + exports (.docx case summary, .xlsx caseload) | **SHIPPED** | `routes/exports.ts`. Strength labels for external docs (CLAUDE rule 4). |
| 12 | Intake portal (token-gated, public) | **SHIPPED** | `/intake/:token`. |
| 13 | AI draft drawer (note · message · AID prefill · proposal) | **SHIPPED** | `routes/aiDrafts.ts`. |
| 14 | Daily overdue follow-up email cron | **SHIPPED** | `cron 0 8 * * * UTC`. Honors per-user prefs. |
| 15 | Audit log (INSERT-only) + per-write `logAudit()` | **SHIPPED** | DB-level write block + code-level explicit attribution. |
| 16 | Honeybadger error reporting (API + web) | **SHIPPED** | beforeNotify scrubs PHI. |

## HIPAA + security backbone

| # | Feature | Status | Notes |
|---|---|---|---|
| 17 | RLS on every PHI table | **SHIPPED** | Verified by Bastion review 2026-05-25. |
| 18 | Migration 035 — views flipped to `security_invoker=on` | **SHIPPED** | Closed the HIGH cross-org PHI read. |
| 19 | 30-min idle auto-logoff (mobile + web) | **SHIPPED** | HIPAA §164.312(a)(2)(iii). |
| 20 | 12-char + complexity password policy | **SHIPPED** | `lib/password.ts`. Used by AcceptInvite + ResetPassword. |
| 21 | AES-256 encrypted persisted query cache (mobile) | **SHIPPED** | Key in expo-secure-store. |
| 22 | Durable offline mutation replay (mobile) | **SHIPPED** | Notes, follow-ups, goals replay after restart. |
| 23 | DB-level BAA gate trigger (`enforce_baa_signed_before_phi`) | **SHIPPED** | 10 PHI tables. Blocks unsigned-org PHI inserts. |
| 24 | Audit triggers on `client_documents` + `recovery_score_events` | **PLANNED** | H sign-off pending. Migration 037 partially landed. |
| 25 | HIBP leaked-password check | **GAP** | Pro-plan-gated on Supabase. Phase 2 close item. |

## Stage 7 — messaging

| # | Feature | Status | Notes |
|---|---|---|---|
| 26 | Unified inbox (conversations + notifications) | **STAGED** | Branch `stage7-messaging-v5-parity`, commit `7fdfcea`. iOS preview build `8ebff352`. NOT yet on main. |
| 27 | Realtime via Supabase publication | **STAGED** | Migration `046_messaging_realtime.sql` applied. Polling fallback in place. |
| 28 | Survivor containment (`dm_survivor` thread) | **STAGED** | RLS-enforced. |

## Stage 8 — billing / BAA / onboarding

| # | Feature | Status | Notes |
|---|---|---|---|
| 29 | Operator console (platform-admin) | **STAGED** | Commit `8d07efc` + `625bcf2`. Branch not merged. |
| 30 | Self-serve org signup (Starter + Pro tiers) | **STAGED** | 5-step wizard. Captures HIPAA role, legal entity, MSA + DPA acceptance. |
| 31 | In-app BAA e-signature | **STAGED** | Records ESIGN evidence + generates signed PDF. Template text is a labeled placeholder — counsel sign-off required before go-live. |
| 32 | Stripe checkout + webhook (signature-verified, idempotent) | **STAGED** | TEST keys only until counsel BAA + Terms. Stripe sees no PHI. |
| 33 | Public lead capture (Enterprise / Government / RAP / 501c3) | **STAGED** | Operator queue with 501c3 decision flow. |
| 34 | Plans + addons catalog (4 tiers + 3 addons + RAP discount) | **STAGED** | Pricing PRD canonical: Starter $299 · Pro $799 · Enterprise & Gov custom · annual = 20% off = 10× monthly · whitelabel +$500 · API +$200 · SLA +$300 · RAP 20% off monthly. |

## Stage 9 — AI Call Notation

| # | Feature | Status | Notes |
|---|---|---|---|
| 35 | Consent-gated session lifecycle | **SHIPPED** | Commit `604b1ed` to main 2026-06-05. |
| 36 | Bilingual EN+ES draft case note via Claude (forced tool) | **SHIPPED** | `lib/notation/summarize.ts`. claude-sonnet-4-5. |
| 37 | Deepgram Nova-3 STT integration | **BUILT, GATED** | Off until `DEEPGRAM_API_KEY` set + BAA signed. Test stub works today. |
| 38 | Web review surface (consent → record → draft + transcript review → approve) | **SHIPPED** | `/notation`. MediaRecorder or synthetic transcript. |
| 39 | Mobile capture screen | **PARTIAL** | UI shipped; `expo-audio` not yet added → no real mic capture today. Adding it = new EAS build. |
| 40 | NOTATION_ENABLED env gate | **BUILT, GATED** | H to flip in Render dashboard to turn on. |

---

## What's left to stand up in the next 7 days

These are the items that block "elite enterprise SaaS" stand-up. Ordered by sequence — each unlocks the next.

### Day 1–2 · Merge & deploy

1. **Merge `stage7-messaging-v5-parity` to `main`.** Bundles Stage 7 messaging + all of Stage 8 onboarding/billing/BAA into production. Once this lands, an org can sign themselves up, sign their BAA, get a Stripe checkout, and start using the app.
2. **Wire Render env vars for billing + Stripe.** H sets `STRIPE_SECRET_KEY`, `STRIPE_WEBHOOK_SECRET`, `STRIPE_*_PRICE_ID` (already seeded by `seed-stripe-products.mjs`).
3. **Ship the next iOS preview build** (post-merge). Share Expo install link. Mobile sees Stage 7 messaging + Stage 8 onboarding.

### Day 2–3 · Counsel + content blockers

4. **Counsel BAA text.** `lib/baaTemplate.ts` is a LABELED PLACEHOLDER. Counsel approves binding text → bump `BAA_TEMPLATE_VERSION` → re-deploy. Without this, Stripe stays TEST and no real customer can sign.
5. **Counsel MSA + DPA + Order Form text.** Currently captured as acceptance records but the text on the survivor / org-admin screens is placeholder.
6. **HIPAA Notice of Privacy Practices (NPP).** Required for any org that's a Covered Entity. Currently not surfaced anywhere in the product.

### Day 3–4 · Plumbing fixes (the 5 surprises from the architecture map)

7. **Delete `lib/systemFormTemplates.ts`** or land Phase 2 step 7 (system template seed). 276 lines of dead code today.
8. **Install `@honeybadger-io/react-native`** and re-instrument mobile entry. Production mobile crashes go nowhere today.
9. **Promote `BACKFILL_resource_matches.sql` to a numbered migration.** Fresh-environment migration apply would skip the table otherwise.
10. **Route every AI call through `lib/ai/index.ts`.** Currently only `triageEngine.ts` uses the barrel — 4 routes still bypass it. Without this, prompt caching + retry + telemetry can't land in one place.
11. **Document the quadruple-mount on `/api/clients`.** Add a header comment in each of `clients.ts`, `assessments.ts`, `briefs.ts`, `aiDrafts.ts` listing the sub-paths each one owns. Cheap insurance.

### Day 4–5 · Enterprise gaps (priority order — see `enterprise-gap-analysis.html`)

12. **SSO / SAML / SCIM** for Enterprise + Government tiers. Today only email+password. Government counties will require this on day one.
13. **Org-level audit log export** (signed CSV, by date range, by user, by table). Today there's an audit log read endpoint but no export.
14. **Disaster Recovery / Backup verification protocol.** Supabase has daily PITR but no documented restore test cadence. Bastion will flag.
15. **Data Subject Access Request (DSAR) endpoint.** "Give me everything you have on me" — required by GDPR, CPRA. Today: no endpoint.
16. **Survivor data export** ("My data") on the survivor portal. Same legal driver as DSAR plus user-facing transparency.

### Day 5–6 · UI/UX polish (per the role roadmap)

17. **CM Lead workflow surfaces** — explicitly defined in `vijilant-cm-lead-workflow-spec.md` but not all sub-screens are wired. Caseload-by-CM review, batch reassignment, escalation handoff.
18. **Resource Lead workflow surfaces** — `vijilant-resource-lead-workflow-spec.md`. Rocsii's pattern — bulk add, retire, version, in-app notify.
19. **Admin dashboard** — caseload + outcomes + capacity rollups. The supervisor brief exists; the Admin-tier rollup view does not.
20. **Onboarding tour** — first-run guide for a new CM. Today users land in an empty Dashboard.

### Day 6–7 · Stand-up dress rehearsal

21. **Migration dress rehearsal** for HOLD Dena → Vijilant (see `migration-roadmap.html`). One pilot CM transferred end-to-end before LLP cutover.
22. **Document audit pass** — see `document-audit.html`. Brand spelling (Vijilant J in all user-visible text), tier ramp consistency, recovery score range (0–1000 not 0–100), and stale memory notes.
23. **Adversarial security pass** — see `security-hardening.html`. SSRF / IDOR / Stripe replay / SAST scan / RLS bypass attempts.

---

## What's missing from the schema (Apple-team review)

Pulled forward from `enterprise-gap-analysis.html` so it lives next to the build status:

- **`sso_connections`** — per-org SSO provider config (entity ID, metadata XML, JIT provisioning rules). Not modeled.
- **`api_keys`** — per-org Vijilant API tokens with scopes + rotation history. Today: no public API.
- **`webhook_subscriptions`** — outbound webhooks so org systems can listen to client events. Not modeled.
- **`audit_export_jobs`** — long-running audit log exports with signed S3 deliverables. Not modeled.
- **`dsar_requests`** — Data Subject Access Requests with state machine (received → fulfilled → delivered → 30-day expiry). Not modeled.
- **`org_data_retention_policies`** — per-org retention windows (case notes 6yr, transcripts 18mo, audit 10yr). Today: global only.
- **`legal_holds`** — court-ordered case freeze (blocks delete/archive on a specific client + cascades to docs + notes). Not modeled.
- **`incident_reports`** — security incident state machine for breach response. Not modeled.
- **`feature_flags`** — per-org / per-cohort feature gating. Today: only env-level `NOTATION_ENABLED`.
- **`outbox` / `idempotency_keys`** for at-least-once side effects (Stripe-style replay safety). Today: only on stripe_webhook_events.

---

## Critical path through the 7 days

A simplified Gantt-style read of the dependency chain:

| Day | Build | Decision | Test |
|---|---|---|---|
| 1 | Merge Stage 7+8 to main, deploy web | H confirms TEST keys vs LIVE Stripe | Stage 7 messaging in iOS preview |
| 2 | Counsel sends BAA text · NPP draft · MSA/DPA | H approves BAA text → `BAA_TEMPLATE_VERSION` bump | First pilot org signs BAA in TEST |
| 3 | Delete dead code · install RN Honeybadger · promote BACKFILL migration | — | Mobile crash test (force-throw + verify capture) |
| 4 | SSO/SAML scaffold for Enterprise · audit export endpoint | H decides which SSO IdP first (Okta vs Azure AD vs Google) | First Enterprise pilot — operator console provision |
| 5 | DSAR endpoint · CM Lead screens · Admin rollup view | H signs off on CM Lead screens against `vijilant-cm-lead-workflow-spec.md` | LLP CM Lead Marisol validates |
| 6 | HOLD Dena migration dress rehearsal | H + Marisol + Sara approve pilot client | One pilot client end-to-end |
| 7 | Final security pass · audit alignment | H sign-off on launch readiness | LLP cutover ready |

---

## Risks that could push past 7 days

1. **Counsel turnaround on BAA / MSA / DPA / NPP.** Single biggest external dependency. If counsel slips, Stripe stays TEST and no paying org signs.
2. **SSO is bigger than it looks.** SAML metadata exchange + SCIM provisioning + role mapping + JIT account creation is a 3–5 day engineering job by itself. May need to slip to v1.1 with a manual-provision fallback for Enterprise.
3. **Android EAS build stability.** First Android success was 2026-06-05 (commit `acdd9e32`). Need a second clean build to confirm reproducibility before LLP CMs on Android phones can install.
4. **HOLD Dena export format.** QW has not yet committed to a CSV/JSON export. If they don't, migration is manual or via screen-scrape — both add days.
5. **NOTATION_ENABLED rollout.** Deepgram BAA + `expo-audio` install for mobile = two parallel paths. Either can slip without blocking the rest of stand-up.

---

## Definition of "stood up"

For the next 7 days to count as stood up, every line below must be true on day 8:

- A net-new organization can self-serve sign up at `/get-started`, complete the 5-step wizard, sign the BAA in-app, and pay via Stripe (LIVE keys).
- A net-new case manager can log in, run a full 12-section assessment, see their Recovery Score, see the recovery plan auto-generated by Claude Opus, and write a case note (with or without the AI draft drawer).
- A survivor can be invited via intake link, fill the form, get verified by the CM Lead, get assigned to a CM, see their own portal at `/portal`, and message their CM.
- A platform admin can provision a comp'd org via `/operator`, approve a 501c3 application, and resolve a Recovery Access Program lead.
- A LLP CM (Marisol's team) has been migrated from HOLD Dena to Vijilant — caseload, notes, follow-ups, applications, all in their new home with audit trail preserved.
- Crash + error reporting is live across API, web, AND mobile.
- Audit log exports work, DSAR endpoint works, BAA gate enforced at the DB.
- The five architecture-map surprises are closed (dead code deleted · mobile Honeybadger installed · resource_matches migration numbered · AI barrel adopted · quadruple-mount documented).
