# Vijilant — Document Audit (alignment · comprehensiveness · continuity)

A structured audit across ~66 Vijilant documents that orbit the build: `~/Padrino/09_Dev/` (engineering source-of-truth), `~/vigilant/` (in-repo handoffs + CLAUDE.md), `~/Claude CoWork/Outputs/Vigilant/Business/` (Cowork outputs + handoffs), `~/Padrino/HAMPSTER/` (LLP × QW partnership), and the brief-pages folder this doc sits in.

The aim isn't exhaustive — it's high-confidence flagging of the **drift that would actually mislead a future reader or break a future build**.

**Compiled:** 2026-06-06. Methodology: structured spot-check + grep for known-drift markers.

---

## Audit categories

1. **Brand spelling drift** (Vijilant J vs Vigilant G)
2. **Tier color / design-system drift** (pre-v5 vs v5)
3. **Recovery Score range drift** (0–100 vs 0–1000)
4. **Schema drift** (memory or doc claims that don't match live)
5. **Status drift** (built/shipped status that doesn't match reality)
6. **Cross-document continuity** (one doc references another that has moved or changed)
7. **Stale memory entries**

For each finding: **document · line/section · what's wrong · severity (Low/Med/High) · fix**.

---

## Category 1 — Brand spelling

**Rule** (`~/Padrino/09_Dev/CLAUDE.md`): User-visible strings, screen titles, wordmarks, and settings copy must read **VIJILANT (J)**. Code file names and variable names may retain `vigilant` (G) for path compatibility.

| Doc | Line / section | Issue | Severity | Fix |
|---|---|---|---|---|
| `~/vigilant/PRD.md` | Throughout | Uses "Vigilant" (G) in user-facing copy. PRD predates the v5 brand-spelling lock. | **High** | Sweep PRD: every prose mention of the product = "Vijilant"; every `~/vigilant/` path = unchanged. ~50 replacements estimated. |
| `~/vigilant/CLAUDE.md` | Throughout | Mostly correct (uses "Vigilant" in path context only) — verify on the next pass. | Low | Spot check. |
| `~/vigilant/PRODUCT.md` | Throughout | Predates the lock. Same pattern as PRD. | Med | Sweep. |
| `~/vigilant/SCREENS.md` | Throughout | Predates the lock. | Med | Sweep. |
| New brief-pages docs (mine, this session) | Verified | Use "Vijilant" in user-visible text consistently. Code paths use `~/vigilant/`. | OK | — |
| Cowork handoffs in `Business/` | Mixed | Older handoffs say "Vigilant", newer ones "Vijilant". Drift varies. | Med | Sweep before any external share. |

**Single source of truth:** `~/Padrino/09_Dev/CLAUDE.md` is the lock. Every other doc reconciles to it.

**Recommended fix command** (review every diff manually before applying):
```
# safe pattern — replaces only prose context, not paths
sed -i.bak 's/\bVigilant\b/Vijilant/g' <doc>
# THEN restore any false-positive in path strings like "~/vigilant"
```

---

## Category 2 — Tier color / design-system drift

**Rule** (`~/Padrino/09_Dev/CLAUDE.md` + `~/vigilant/AGENTS.md`): v5 is the ONLY design system. Tier ramp is **navy → teal → coral → red** (T1 Stable=navy → T4 Critical=red). Pre-v5 (green-token / purple-retheme / v2–v4) is purged.

| Doc | Line / section | Issue | Severity | Fix |
|---|---|---|---|---|
| `~/vigilant/PRD.md` | Lines 710, 763–782 | Maps tier 1 (Low) → `colors.green` (the pre-v5 mapping). Says "Critical (0–249) · High · Medium · Low" — band labels are old; v5 labels are "Critical · Complex · Moderate · Stable" (per `~/vigilant/AGENTS.md`). | **High** | Update PRD tier color + label mappings to v5. |
| `~/vigilant/CLAUDE.md` | Line 105–137 | Lists `green`, `greenMid`, `greenLight`, `amber`, etc. as canonical token keys. Per the v5 retheme the KEYS survive (backwards compatibility) but their VALUES are teal-family. A reader of CLAUDE.md alone could not infer that. | Med | Add a v5-retheme note at top of `~/vigilant/CLAUDE.md` design section pointing to AGENTS.md "Locked v5 details". Or migrate token keys to `tier1`, `tier2`, `tier3`, `tier4`. |
| `~/Padrino/09_Dev/vijilant-assessment-workflow-spec.md` | Line 471–514 | Score band colors named `colors.red / colors.amber / colors.greenMid / colors.green`. Same backwards-compat trap as above. | Med | Update to `tier{1,2,3,4}` semantic names OR add a clarifying note. |
| `~/Padrino/09_Dev/SCREENS.md` | Throughout | v5-aligned. References "tier badges navy→red ramp" correctly. | OK | — |
| `~/Padrino/09_Dev/MOCKUP_HANDOFF.md` | Throughout | v5 LOCKED tokens. Authoritative. | OK | — |
| `~/Padrino/09_Dev/DESIGN.md` + `DESIGN_Nike.md` / `_Notion.md` / `_Shopify.md` / `_Stripe.md` | All | Inspiration / reference docs. May reference pre-v5 design. | Low | Confirm these are reference-only (won't be read as source-of-truth) and leave. |

**The deeper issue:** the v5 retheme changed *values* without changing *keys* in `@vigilant/tokens`. This is the source of every visible drift in this category. Two cleanup options:

- **A.** Migrate token keys to semantic names: `tier1Color`, `tier2Color`, …, `domainStableColor`, `domainPartialColor`, etc. **L effort.** Touches every page that uses tokens.
- **B.** Leave keys; add giant comment block at the top of `colors.ts` explaining the semantic-vs-historical naming and the canonical ramp. **S effort.**

Pick A for v1.1 polish; pick B for the 7-day window.

---

## Category 3 — Recovery Score range

**Rule** (LOCKED 2026-05-24): Recovery Score is 0–1000. The 0–100 number is **per-domain component** (e.g., housing component = 0–100). 1–5 is the **answer_value**. These three scales should never be conflated.

| Doc | Line / section | Issue | Severity | Fix |
|---|---|---|---|---|
| `~/Padrino/09_Dev/vijilant-assessment-workflow-spec.md` | Line 471 ("0–100. Computed server-side…") | This refers to the per-domain component score, not the Recovery Score — context-correct but easy to misread. | Low | Add a one-line clarifier: "Per-domain component (0–100), not the Recovery Score (0–1000)." |
| `~/Padrino/09_Dev/vigilant-claude-code-handoff.md` | Section 5.2 | Pre-spec; describes the OLD slider-based 5×0–100 score model. The spec doc explicitly overrides this. | Med | Header banner already exists. Add bold "SUPERSEDED — see vijilant-assessment-workflow-spec.md §5.2". |
| `~/vigilant/PRD.md` | Lines 692–701 | Correctly says 0–1000 for the Recovery Score. **OK.** | OK | — |
| Newer build handoffs (Stage 7/8/9) | All | Reference 0–1000 correctly. **OK.** | OK | — |

---

## Category 4 — Schema drift

**Rule:** Doc claims about the schema must match the live numbered migrations.

| Doc | Section | Issue | Severity | Fix |
|---|---|---|---|---|
| `~/.claude/projects/-Users-wrmly-Padrino/memory/vigilant-architecture-facts.md` | "No `notifications` table — the notifications route is a daily Resend cron" | **STALE.** Migration 031 ADDED the notifications table (2026-05-25). Migration 034 added notification_preferences. | High | Re-write memory: "Notifications table EXISTS (031). Notification prefs EXIST (034). The cron is still the daily overdue emailer but it now coordinates with the in-app table." |
| Architecture map (this brief) | DB cluster | Correctly says notifications + prefs + push_tokens exist. **OK.** | OK | — |
| `~/Padrino/09_Dev/vijilant-assessment-workflow-spec.md` | Schema section | Author-edited 2026-05-24 lock; aligns with migrations 033 and forward. **OK.** | OK | — |
| `~/Padrino/09_Dev/vigilant-claude-code-handoff.md` | Schema | Pre-lock document; references the OLD 5-column `recovery_assessments` shape, not the JSONB `section_data`. | Med | Already superseded; add a sharper SUPERSEDED banner. |
| `BACKFILL_resource_matches.sql` | Migration filename | Lives outside the numbered sequence. See `surprises-plan.html` § Surprise #3. | High | Promote to numbered migration. |
| `~/vigilant/AGENTS.md` | Throughout | Aligned with current schema. Maintains the v5 + Stage 7/8/9 lock notes. **OK.** | OK | — |

---

## Category 5 — Status drift (built / shipped / staged)

**Rule:** When a doc says "shipped" it must match a real deploy to main / a real preview build.

| Doc | Claim | Reality | Fix |
|---|---|---|---|
| `~/Claude CoWork/Outputs/Vigilant/Business/VIJILANT_Org_Signup_Portal_BUILD_Handoff.md` | "Built and committed" | True — but the branch `stage7-messaging-v5-parity` is NOT merged to main yet. End users on production main DO NOT have this. | Add a header line: "Branch built; NOT yet merged to main." |
| `~/vigilant/AGENTS.md` Stage 7 section | "iOS preview build 8ebff352 finished" | True. **OK.** | OK |
| `~/vigilant/AGENTS.md` Stage 8 + Phase 4 sections | "Built across DB+API+web+mobile, all green, committed 8d07efc + 625bcf2" | True for commits; NOT yet on main. | Reflect in the "What's left" doc + here. |
| HAMPSTER `PLATFORM_BUGS.md` | Last update 2026-06-03 reflects 6/4 deploy plan. Need a post-6/4 update entry to confirm what actually shipped. | Open | Add a 2026-06-05 entry to `PLATFORM_BUGS.md` after the QW deploy lands. (LLP-side action.) |

---

## Category 6 — Cross-document continuity

**Rule:** When doc A points at doc B, doc B should exist at the expected path.

| Citation | Target | Status |
|---|---|---|
| `~/Padrino/09_Dev/CLAUDE.md` → `vijilant-assessment-workflow-spec.md` | exists at `09_Dev/` | OK |
| `~/Padrino/09_Dev/CLAUDE.md` → `vigilant-claude-code-handoff.md` | exists at `09_Dev/` | OK |
| `~/Padrino/HAMPSTER/CLAUDE.md` → `CROSS_SYSTEM.md` | exists at `HAMPSTER/` | OK |
| `LLP_Operational_Workflow.md` → `09_Dev/vijilant-cm-lead-workflow-spec.md` | exists | OK |
| `LLP_Operational_Workflow.md` → `09_Dev/vijilant-resource-lead-workflow-spec.md` | needs to be verified | spot-check below |
| Architecture map → all 9 new docs in brief-pages | This doc is one of them | OK |

**Spot check:**
```
ls -la /Users/wrmly/Padrino/09_Dev/vijilant-resource-lead-workflow-spec.md
```
If missing → drift between `LLP_Operational_Workflow.md` and what actually exists. Document as a gap; create the spec in the Day 1–2 CM Lead surface work.

---

## Category 7 — Stale memory entries

`~/.claude/projects/-Users-wrmly-Padrino/memory/MEMORY.md` is loaded into every Claude session. If it's wrong, every session-start agent reasons from wrong premises.

| Entry | Status |
|---|---|
| `vigilant-architecture-facts` — "no notifications table" | **STALE.** Already noted. Fix as above. |
| `feedback-verify-vigilant-specs` — "verify against migrations before building" | Evergreen. **OK.** |
| `project-vigilant-worktree-cleaner` — "~/vigilant: untracked files get deleted once mid-session" | Spot-check: does the cleaner still exist? If yes, evergreen. |
| `project-vigilant-v5-parity-locks` — v5 design rules | **OK.** Authoritative. |
| `project-vijilant-deck-design` — python-pptx for decks | **OK.** Evergreen guidance. |

---

## Comprehensiveness — what's not yet documented

Documents that **should exist** to match an elite SaaS bar but don't yet:

| Doc | Why it's needed | Effort |
|---|---|---|
| `~/vigilant/RUNBOOK.md` | Subpoena response · DR drill · vendor-compromise response | M |
| `~/vigilant/BAAS.md` | Every vendor BAA: signed status · expiry · contact · ZDR status | S |
| `~/vigilant/SECURITY_ROADMAP.md` | 60-day follow-on plan after the 7-day stand-up | S |
| `~/vigilant/SLA.md` | Customer-facing uptime / response time commitments | M |
| `~/Padrino/09_Dev/vijilant-resource-lead-workflow-spec.md` | Rocsii's role spec (per LLP) | S |
| `~/Padrino/09_Dev/vijilant-admin-workflow-spec.md` | Admin (Dr. Syms) role spec, parallel to CM and CM Lead | S |
| `~/Padrino/09_Dev/vijilant-survivor-workflow-spec.md` | Survivor portal spec — the highest-stakes UX | M |
| API public docs (OpenAPI + Mintlify or similar) | Enterprise integration prerequisite | M |
| `docs.vijilant.io` public help center | Customer success prerequisite | M |

---

## Continuity — does the doc surface tell a coherent story?

Walk through the "newcomer experience":

1. **A new engineer sits down at `~/vigilant/CLAUDE.md`.** → Reads about v5 design. Reads `AGENTS.md` for locked details. Reads PRD.md for product spec. **Issue:** PRD.md predates v5 in places — tier color mapping reads old. The newcomer would absorb pre-v5 mental model.
2. **A new investor reads `Vijilant_Executive_Summary_v2.pdf`.** → Sees the pitch deck design system ("Confident step beyond") + the 5-year financial model. **Issue:** the deck's v2 may pre-date the latest Stage 8/9 announcements. Verify.
3. **A new Cowork session opens with `~/Claude CoWork/Outputs/Vigilant/Business/VIJILANT_Pricing_and_Signup_Portal_Handoff.md`.** → Canonical pricing PRD. **OK.** Use as authoritative reference everywhere downstream.
4. **A new LLP CM opens HAMPSTER.** → Reads `PHASE_1.md`, `WORKFLOW.md`, `KEY_PEOPLE.md`. Coherent. **OK.**
5. **A new partner reads the brief site.** → index.html + competitive-brief.html + (new) architecture-map.html + 7 new docs. After this audit + index update, **coherent.**

---

## Priority fix list (the next-7-days cleanup pass)

If H can fix only the items that materially mislead a future reader or build, in priority order:

1. **Sweep PRD.md for Vijilant spelling + tier color update.** (High · 2h)
2. **Update vigilant-architecture-facts memory** — flip the "no notifications table" line. (High · 5min)
3. **Promote BACKFILL_resource_matches.sql to numbered migration.** (High · per surprises-plan)
4. **Add SUPERSEDED banner to vigilant-claude-code-handoff.md** pointing at the assessment workflow spec. (Med · 5min)
5. **Add Vijilant ASCII banner / brand-spelling note at the top of every Cowork Business handoff.** (Med · 30min sweep)
6. **Create the three missing role specs:** Resource Lead, Admin, Survivor. (Med · 2h each)
7. **Create RUNBOOK.md + BAAS.md + SECURITY_ROADMAP.md skeletons.** (Med · 1h total)

---

## Continuous-alignment process — how to keep this clean

The biggest source of drift is undocumented decisions. Two cheap mechanisms:

1. **A weekly "doc-alignment 10-minute pass"** — H scans the deltas across CLAUDE.md, AGENTS.md, the active memory entries, and the latest build handoff. Anything contradictory gets one line of resolution.
2. **A "lock note" header on every spec** — if a doc supersedes another, both ends carry a mutual reference. PRD has a "this section is SUPERSEDED by X" header; X has a "this supersedes PRD section Y" header.
3. **One canonical pricing/architecture/security doc each** — when a fact moves, it moves in one place and the others link. Today pricing has `VIJILANT_Pricing_and_Signup_Portal_Handoff.md` (✓). Architecture has the new map (✓). Security has `security-hardening.md` (✓). The pattern is right; extend it to schema and brand spelling.

---

## Document index — v6 feature set (added 2026-06-12)

Six new VIJILANT feature sets were locked 2026-06-12. The build artifacts below were authored 2026-06-11/2026-06-12 and are appended here so the audit trail stays complete. All paths are under `~/Padrino/09_Dev/`.

**Handoff + spec docs:**

| Doc | Date | Description | Size | Status |
|---|---|---|---|---|
| `vijilant-v6-features-handoff.md` | 2026-06-12 | Master build spec for the six v6 feature sets (Advocacy/Unmet Needs · Calendar · Team/Profiles · Maps & Directions · Messages & Notifications · Case Consultation Tracker). | 19 KB | LOCKED 2026-06-12 |
| `vijilant-v6-claude-code-prompts.md` | 2026-06-12 | The two-prompt Claude Code build set: Prompt 1 doc-update pass + Prompt 2 build pass. | 17 KB | LOCKED 2026-06-12 |
| `vijilant-advocacy-schema.md` | 2026-06-11 | Full SQL + workflow for the advocacy / unmet-needs tables. | 29 KB | Authored 2026-06-11 |

**Mockups (v5 design system, extending screens 1–14 + survivor portal + profile):**

| Doc | Date | Description | Screens | Size | Status |
|---|---|---|---|---|---|
| `vijilant-v5-advocacy-screens.html` | 2026-06-12 | Advocacy + Unmet Needs. | 15–19 | 30 KB | Authored 2026-06-12 |
| `vijilant-v5-calendar-screens.html` | 2026-06-12 | Calendar. | 20–24 | 84 KB | Authored 2026-06-12 |
| `vijilant-v5-dashboard-calendar-variants.html` | 2026-06-11 | Dashboard calendar-strip variants. | dashboard variants | 45 KB | Authored 2026-06-11 |
| `vijilant-v5-team-screens.html` | 2026-06-12 | Team / Profiles. | 25–28 | 63 KB | Authored 2026-06-12 |
| `vijilant-v5-maps-screens.html` | 2026-06-12 | Maps & Directions. | 29–32 | 50 KB | Authored 2026-06-12 |
| `vijilant-v5-messages-screens.html` | 2026-06-12 | Messages & Notifications. | 33–35 | 55 KB | Authored 2026-06-12 |
| `vijilant-v5-consultation-screens.html` | 2026-06-12 | Case Consultation Tracker. | 36–38 | 57 KB | Authored 2026-06-12 |

---

## Open question for H

There's one decision this audit can't answer:

**Token key naming in `@vigilant/tokens`.** Today keys are pre-v5 (`green`, `greenMid`, `amber`) but values are v5 (teal-family). Should we:
- (A) Migrate keys to semantic names (`tier1Color`, etc.) — clean, but L effort and touches every consumer.
- (B) Leave keys; add a giant comment block at the top of `colors.ts` explaining the historical-vs-semantic split — S effort.
- (C) Hybrid — alias `tier1Color = green` etc., new code uses the new names, old code keeps working.

This is a v1 → v1.1 decision. Worth raising at the next MITA-05 Studio review.
