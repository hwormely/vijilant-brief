# Vijilant — HOLD Dena → Vijilant Migration Roadmap (LLP cutover)

How LLP moves its survivor caseload from Quality Works' HOLD Dena (Recover Prepared) platform to Vijilant — without losing data, audit history, or operational continuity.

**Owner:** Horace Wormely (Tech Systems Liaison · LLP).
**Counterparts:** Dr. Syms (LLP ED) · Marisol Espino + Sara Potter (LLP CM Leads) · Ria Joseph + Romain Hunter + Nyran (Quality Works).
**Source files:** `~/Padrino/HAMPSTER/PHASE_1.md` · `~/Padrino/HAMPSTER/PLATFORM_BUGS.md` · `~/Padrino/HAMPSTER/PLATFORM_FEATURES.md` · `~/Padrino/HAMPSTER/LLP_Operational_Workflow.md`.

---

## Why this migration is being planned now

LLP has been running on HOLD Dena since the Eaton fire response. The partnership is real and producing — but the platform sits at the wrong center of gravity for what LLP is becoming. Specific signals driving the move:

1. **Persistent data-loss risk.** Bug #5 (Very High): deleting a Care Manager profile **destroys** the case notes tied to that CM. Bug #20: a HOLD Dena deploy routed the entire LLP team into the survivor onboarding flow — full work stoppage, resolved by rollback, not by a structural fix. Recurrence risk is unaddressed.
2. **Cumbersome data-entry layer** (Bug #4) is unfixed and slows every CM every day.
3. **Workflow centralization gaps.** Of LLP's 12 operational stages, **6 are partly or fully outside HOLD Dena** today (discovery, verification & assignment, resource routing, CM coordination, CM Lead oversight, Admin reporting). Vijilant is built for the full 12.
4. **Role coverage gap.** HOLD Dena's existing scope is centered on the CM layer. LLP's other three operational layers (Admin · CM Leads · Resource Lead) have less explicit support today. Vijilant ships first-class surfaces for all four.
5. **Vendor pace.** QW's monthly engineering allocation (133 hours) is shared across many partner orgs. LLP-specific feature requests sit in a queue.
6. **Strategic alignment.** LLP is Vijilant's anchor customer-of-record and the live case study. Bringing LLP onto the platform that LLP's data is shaping is the consistent next move.

This migration is not a HOLD Dena replacement campaign for other orgs — that's Vijilant's normal GTM. This is the LLP-specific cutover.

---

## What "migration complete" looks like

Day 0 = the day every active LLP survivor case lives in Vijilant, every CM is signed in via SSO, every active follow-up is queued in Vijilant, every resource referral routes through Vijilant's Resources Directory, every audit row from HOLD Dena's history is preserved in Vijilant's audit log, and HOLD Dena is set to read-only with the pointer to Vijilant printed on its login screen.

**Concretely:**
- All ~N active survivor records (count to be confirmed by Marisol + Sara) migrated with full field set + photo ID + case-note history + follow-up state + assigned-CM relationship + tier signal.
- All staff profiles (Admin · CM Leads · Resource Lead · CMs) created in Vijilant with correct role + permitted_roles set.
- All resources in Rocsii's Google Sheet imported into Vijilant's `resources` table with tags harmonized.
- Audit history exported from HOLD Dena and imported into Vijilant's `audit_log` (preserved with original timestamps, marked as `source=hold_dena`).
- BAA signed between LLP and WRMLY LLC (Vijilant), counsel-approved text.
- DSAR endpoint live so a survivor can request "everything Vijilant has on me" and get a counsel-approved export.
- HOLD Dena export bundle archived (encrypted) in cold storage for the 6-year HIPAA retention window — not deleted until that window closes.

---

## Pre-flight — what has to be true before Day 1

These are blocking dependencies. If any line is red on T-7, the migration window slips.

| # | Dependency | Owner | Status |
|---|---|---|---|
| 1 | Vijilant BAA approved by counsel and signed by LLP Admin | H + counsel + Dr. Syms | BAA template is labeled placeholder; counsel sign-off PENDING |
| 2 | Vijilant MSA signed between WRMLY LLC and LLP | H + Dr. Syms | Template ready; counsel review PENDING |
| 3 | Vijilant Notice of Privacy Practices published | H + counsel | NOT DRAFTED |
| 4 | Vijilant `/operator` provisions an LLP org with all 4 role tiers | H | Operator console STAGED (branch `stage7-messaging-v5-parity`) |
| 5 | All LLP CMs / Leads / Admin invited and able to sign in | H + Marisol | Pending org provision |
| 6 | HOLD Dena data export format agreed with QW | H + Ria | Not yet requested; this is the single biggest external dependency |
| 7 | Audit history export agreed with QW | H + Ria | Not yet requested |
| 8 | Photo-ID images exportable from HOLD Dena | H + Ria | Verify against HOLD Dena's intake schema |
| 9 | Stripe LIVE keys (only needed if LLP becomes the first paying org; LLP may be RAP-comped) | H | TEST today |
| 10 | Android EAS build stable (some LLP CMs use Android org phones) | H | First successful Android build 2026-06-05; needs a second clean to confirm |

---

## Migration architecture — three movement patterns

Different data types move differently. Use the right pattern per type.

### Pattern A — Snapshot-and-load (people, clients, applications)
Static-shape records. Export from HOLD Dena → transform to Vijilant schema → bulk insert with `source=hold_dena` + `migrated_at=…` columns. Re-runnable.

### Pattern B — Replay-with-attribution (case notes, follow-ups, audit log)
Time-ordered records where the original timestamp matters. Each row keeps `original_created_at` distinct from Vijilant's `created_at`, plus an `original_id_in_hold_dena` for traceability. The audit log row reads `imported_from_hold_dena · action=…`.

### Pattern C — Re-state (resources, tier signals, recovery plans)
Records that are derived or re-shapeable. Re-tag the resource catalog against Vijilant's tag taxonomy. Re-run Vijilant's Triage Engine over the imported assessment data to compute fresh tiers (preserving the original HOLD Dena tier in an `original_tier` column for diff). Recovery plans regenerate via the Claude Opus 4.7 engine.

---

## The 12 operational stages — migration responsibility per stage

Pulled directly from `LLP_Operational_Workflow.md`. Each row says: where this stage lives in HOLD Dena, where it lives in Vijilant, and what the migration step is.

| Stage | HOLD Dena today | Vijilant target | Migration step |
|---|---|---|---|
| 1. Discovery (QR / link) | Routes nowhere intentional | Public intake `/intake/:token` + lead capture `/api/leads` | New QR codes printed pointing to Vijilant's `/intake/:token` for LLP's invite tokens. Old codes redirect via a temporary fallback page. |
| 2. Verification & assignment | Manual triage outside HOLD Dena | `/intake-submissions` 4-pipeline board (Pending → Verified → Assigned → Follow-up) | Stephanie Alex's verification workflow re-trained on Vijilant. CM Leads (Marisol/Sara) assign in Vijilant. |
| 3. Quick Entry (12-field min) | Partially in HOLD Dena | `/cases/new` 4-step wizard | Side-by-side CM training: same 12 fields, faster form. Old Drive folder pattern retired. |
| 4. Full Assessment (12 sections) | Cumbersome (Bug #4) | Vijilant's assessment form, locked section keys | Snapshot-and-load (Pattern A) for assessment answers. Re-state (Pattern C) for tier + Recovery Score. |
| 5. Triage tier + Recovery plan | Manual outside HOLD Dena | Auto-Triage Engine + Claude Opus plan generator | Re-run engines over imported assessments. Original tier preserved. |
| 6. Active case management | In HOLD Dena, slowed by bugs | Vijilant client-detail 7-tab | Replay-with-attribution for case notes + follow-ups + applications. |
| 7. Resource routing | Google Sheet email blast | Vijilant `/resources` directory with in-app notify | Rocsii's sheet exported as CSV → mapped to Vijilant's tag taxonomy → bulk insert. Old email blast retired. |
| 8. CM team coordination | Slack / text / email | Vijilant Messages (Stage 7 unified inbox) | New team thread created at cutover. Historical Slack stays in Slack (out of scope for migration). |
| 9. CM Lead oversight | Spot-checks by hand | CM Lead "Lead Console" (Day 1–2 build) | Lead Console must ship before cutover or Leads work blind. |
| 10. Applications / referrals | Mixed (HOLD Dena + Drive + partner systems) | Vijilant `/aid-applications` with AI prefill | Per-application snapshot-and-load. Wylie/FEMA/SBA templates pre-seeded. |
| 11. Reports to Admin | Manual from spreadsheets | Admin rollup view + audit export | Admin rollup view must ship before cutover or Dr. Syms loses visibility. |
| 12. Closure / handoff | In HOLD Dena, with Bug #5 data-loss risk | Vijilant archive/unarchive with audit + case notes detached from CM | Pattern A. Archive history preserved. Bug #5 architecture (notes bound to client not CM) is already correct in Vijilant. |

---

## Timeline — 21-day plan from prep to cutover

```
Week 1 (T-21 → T-15) · PREP            Counsel + export contract + Vijilant gaps
Week 2 (T-14 → T-8)  · BUILD/MIRROR    Vijilant production-ready · staging mirror of HOLD Dena
Week 3 (T-7  → T-1)  · DRESS REHEARSAL One pilot client end-to-end · validate
Week 4 (T-0)         · CUTOVER         All active cases moved · HOLD Dena → read-only
Weeks 5–6            · STABILIZE       Live in Vijilant · daily standups · bug-fix turn
```

### Week 1 — Prep (T-21 → T-15)

**Counsel parallel track.** Draft BAA, MSA, DPA, NPP. Get all four signed by both sides.

**Export contract with QW.** H formally requests a full LLP data export from Ria. Concrete asks:
- CSV (or JSON) export of every survivor record with every field.
- CSV (or JSON) export of every case note with `original_timestamp`, `cm_id`, `client_id`.
- CSV (or JSON) export of every follow-up with state + due date + completion timestamp.
- CSV (or JSON) export of every aid application + status timeline.
- Signed S3 URLs for every photo ID and uploaded document.
- Audit log export covering the full LLP usage window.
- A meeting between Ria and H to confirm schema mappings and any field-level transformations.

**Vijilant gap close.** Anything from `feature-roadmap.html`'s "Day 4–5 enterprise gaps" that is required for LLP must be done this week: SSO, audit export, DSAR endpoint, Admin rollup, CM Lead surfaces.

**Pilot identification.** Marisol + Sara pick 2 clients to use as migration pilots — one straightforward, one complicated.

### Week 2 — Build & mirror (T-14 → T-8)

**Vijilant LLP org provisioned via Operator Console.** Name = "The Legacy Land Project · Eaton Recovery". HIPAA role = Covered Entity. RAP-comped (501c3). All staff invited.

**Resource import.** Rocsii's Google Sheet exported. Tags mapped to Vijilant's directory taxonomy (financial · housing · mental_health · employment · legal). Imported. Rocsii confirms by hand-checking 20 random rows.

**Staging mirror.** A dry-run import script runs nightly against a Vijilant staging environment. The output is a diff report against the prior night so we catch schema or content drift early.

**Migration script (`scripts/llp-migration.ts`).** Lives in `~/vigilant/scripts/`. Idempotent (re-runnable with no double-insert) thanks to `original_id_in_hold_dena` unique constraint. Phases:
1. Profiles + organizations (staff)
2. Clients + photo IDs (Pattern A)
3. Assessments + section_data (Pattern A) → re-run Triage Engine (Pattern C)
4. Case notes (Pattern B)
5. Follow-ups (Pattern B)
6. Aid applications + answers (Pattern A)
7. Documents + uploads
8. Audit log (Pattern B)

**Training plan drafted.** Three sessions: (a) CM hands-on (60 min, mobile + desktop), (b) CM Lead deep-dive on Lead Console + Intake Pipeline (90 min), (c) Admin walkthrough on Org Setup + Audit + Billing (60 min). Materials in `~/Padrino/HAMPSTER/Materials_For_LLP/` styled to thelegacylandproject.org aesthetic.

### Week 3 — Dress rehearsal (T-7 → T-1)

**Day T-7:** Run the migration script against the two pilot clients only. Marisol + Sara open Vijilant, walk through everything. Bugs logged + fixed.

**Day T-6:** Repeat with 10 random clients (anonymized for the dry run if needed).

**Day T-5:** Full-export rehearsal. Pull a complete export, run the full script, validate row counts match between source and target.

**Day T-4 / T-3:** Training sessions run in this window. CM team gets hands-on. Marisol + Sara approve CM Lead surfaces against `vijilant-cm-lead-workflow-spec.md`.

**Day T-2:** All-hands LLP meeting. Dr. Syms blesses cutover. Final go/no-go from Marisol + Sara.

**Day T-1:** Quiet day. Final export pulled. Stripe LIVE keys flipped (if needed). Anchor support staff scheduled for Day 0.

### Week 4 — Cutover (T-0)

**Morning (08:00 PT):** Final delta export from HOLD Dena pulled (covers anything written since the Week 3 full export). Migration script runs against this delta only.

**Mid-day (10:00 PT):** All-hands LLP meeting. H walks team through "today you sign in to Vijilant." CM team signs in. First case note written.

**Afternoon (13:00 PT):** HOLD Dena set to read-only (QW flip). Banner on HOLD Dena login: "LLP has moved to Vijilant — sign in at app.vijilant.io. HOLD Dena is read-only for historical reference."

**EOD (17:00 PT):** H + Marisol + Sara + Rocsii audit the day. Row-count reconciliation between source and target. Any drift → ticketed for Day 1.

### Weeks 5–6 — Stabilize

Daily 15-min standup at 09:00 PT. Bug triage same day. Anything in Vijilant that the team feels is missing relative to HOLD Dena gets logged + scoped by EOD.

---

## Data-mapping cheat sheet

The key field maps that the migration script has to get right.

| HOLD Dena field | Vijilant target | Transform |
|---|---|---|
| `member_id` | `clients.id` | UUID generated; `original_member_id_in_hold_dena` preserved |
| `member_name` | `clients.first_name` + `clients.last_name` | Split on first space (manual review for compound names) |
| `tier` (1–4) | `clients.triage_tier` | Pass-through. Re-run engine → store as `engine_tier`; keep original as `original_tier_from_hold_dena` until first re-assessment. |
| `case_notes.note_text` | `case_notes.body` | Pass-through. `case_notes.note_type` defaulted to `field_visit` unless specified. |
| `case_notes.created_at` | `case_notes.original_created_at` | Vijilant's `created_at` = NOW(); original preserved for chronology. |
| `applications.status` | `aid_applications.status` | Map: `pending → submitted`, `approved → approved`, `denied → denied`, `in_review → under_review`. |
| `resources` | `resources` | Rebuild from CSV; tag remap. |
| `audit_trail` | `audit_log` | Each row gets `source=hold_dena · imported_at=NOW()`. |

Any field HOLD Dena exposes that Vijilant doesn't have a target for is captured in a `migration_overflow` JSONB column on the relevant target table, so nothing is lost. Counsel + Dr. Syms decide post-cutover whether to surface it.

---

## Risk register

Ranked by impact × probability.

1. **QW doesn't deliver a clean export on schedule** — *impact: catastrophic · probability: medium*. Fallback: manual export by scraping HOLD Dena per-client (slow, error-prone, requires QW screen access). Mitigation: H asks Ria formally Day 1 of Week 1; escalate to Dr. Syms if not responsive by Day 7.

2. **Counsel turnaround on BAA/MSA/DPA/NPP slips** — *impact: high · probability: medium*. Cutover can't happen without signed BAA. Mitigation: parallel-track counsel from Day 1; pre-pay rush fees if needed.

3. **Photo-ID images can't be exported in bulk** — *impact: high · probability: medium*. Photo IDs are tied to a Supabase Storage bucket on Vijilant's side. If QW's storage isn't bulk-exportable, we may have to ask CMs to re-upload on first contact. Mitigation: explicit Day-1 ask of Ria on storage extraction.

4. **CM Lead surfaces not ready by T-3** — *impact: medium · probability: medium*. The CM Lead role is "exists in code but not first-class." Mitigation: build prioritized as Day 1–2 in `feature-roadmap.html`.

5. **Android EAS build instability** — *impact: medium · probability: low (now)*. First successful Android build landed 2026-06-05. Mitigation: ship a second build before T-14 to confirm reproducibility.

6. **A migrated client looks "off" to their CM and erodes trust** — *impact: medium · probability: medium*. A field misses; a note loses an attachment. Mitigation: the staging-mirror nightly diff + the 2-client pilot are designed to surface this in Week 3.

7. **LLP CM resistance** — *impact: medium · probability: low*. Most CMs welcome a faster tool, but switching mid-recovery is real friction. Mitigation: CM Leads on both sides of the change · training-first scheduling · keep HOLD Dena read-only so a CM can always check the source.

---

## What this migration teaches Vijilant

Every friction point captured here is also a competitive learning. Per the HAMPSTER protocol, each lesson cross-posts to `~/Padrino/_BRAIN/MITA/MITA_MEMORY.md`. A non-exhaustive list:

- **Bulk data export is a v1 feature**, not a v2. Migrations are the most common reason an org switches platforms.
- **Photo-ID/document portability** — every Vijilant org should be able to export the photo-ID bucket on demand.
- **Audit log fidelity is what unlocks counsel sign-off** for cutovers. Vijilant's audit log already preserves `original_created_at` patterns by design.
- **Resource-tag taxonomy harmonization** is harder than it looks across orgs. The taxonomy should be opinionated upstream (Vijilant chooses; orgs map their tags to ours) or it metastasizes.
- **CM Lead is a first-class role at every org we touch** — not just LLP. Build it once, properly.
- **The 12-stage workflow map is universal** for disaster recovery case management. Every org has all 12 stages; what differs is which ones are inside their current tool. Vijilant's pitch is "all 12 stages, one tool."

---

## Where the v6 feature sets land in the cutover

Six v6 feature sets were locked 2026-06-12 (`vijilant-v6-features-handoff.md`). They matter to this migration in a specific way: most of them are **additive surfaces that don't exist in HOLD Dena**, so there's nothing to migrate *into* them — but several are exactly the surfaces that pull LLP's currently-outside-HOLD-Dena workflow stages into one tool, which is half the reason for the move in the first place.

### What the v6 v1 sets do for the cutover
- **Team / Profiles** finally centralizes Stage 8 (CM team coordination) and Stage 9 (CM Lead oversight) — both partly-or-fully outside HOLD Dena today. The leadership roster overview (all CMs' Caseload Health bars + cadence flags) is the Lead Console story from Stage 9 of the migration map, now first-class. Staff profiles are created at provision time (Week 2), not migrated.
- **Calendar (internal + ICS)** absorbs follow-up due dates, aid-application deadlines, and goal targets that today live across HOLD Dena, Drive, and CMs' heads. The auto-event engine projects from records already being migrated (case notes' Next Steps, follow-ups, applications) — so it lights up automatically once Pattern A/B loads run. No separate migration step.
- **Case Consultation Tracker** is a net-new operating discipline (no HOLD Dena equivalent). It reads migrated `case_status` + tier, so it's usable from Day 0 with zero back-fill; historical consults simply start at cutover.
- **Advocacy + Unmet Needs**, **Messages & Notifications** — net-new; nothing to migrate. Messages replaces the Stage 8 Slack/text/email pattern (historical Slack stays in Slack, already out of scope).

**Net:** none of these v1 sets adds a migration data-movement step beyond what's already in the script's eight phases. They consume migrated data; they don't require their own export from QW.

### v6 pieces that are NOT cutover blockers (enterprise-phase)
Two v6 capabilities are deferred to the enterprise phase and **must not gate the LLP cutover**:
- **Maps multi-event + Esri layer manager.** v1 ships ONE seeded event ("Eaton Canyon Fire") with a synthetic / Catalyst-CA perimeter — which is exactly LLP's situation, so v1 Maps is sufficient for cutover. Multi-event support is later.
- **Calendar external sync to Google / Microsoft 365.** v1 is internal calendar + ICS; LLP CMs need nothing more to cut over. External push is a later enhancement.

### New v6 dependencies to track (do not let them slip the window)
These are real but should run on a **parallel track**, not the critical path — only the single-event Maps surface needs to be demo-walkable for LLP, and a synthetic perimeter covers that if the Catalyst feed isn't confirmed in time.
- **Esri ArcGIS BAA + live key** — LOCKED provider (Google Maps + Mapbox cloud banned for survivor addresses; won't sign a BAA). Survivor coordinates are PHI. Add to `BAAS.md` alongside the LLP↔WRMLY BAA. *Owner: H + counsel.*
- **Catalyst California ArcGIS layer access** — confirm the Eaton perimeter/DINS feature layers are public or secure LLP view access / a data-sharing agreement. If not confirmed by T-7, demo + cutover use a synthetic perimeter GeoJSON. *Owner: H.*
- **Google / Microsoft 365 OAuth consent** — enterprise-phase only; **not** needed for LLP cutover. Listed so it isn't mistaken for a blocker.

These slot under the Week 1 "Vijilant gap close" track, with the Esri BAA filed in the same counsel parallel-track as the LLP BAA/MSA/DPA/NPP.

---

## Open questions for next QW touchpoint (Friday 2026-06-06)

These were captured at the 2026-06-03 call and remain open per `PHASE_1.md`:

1. **HOLD Dena bulk-export API or admin tool?** Confirm with Ria.
2. **Audit log export format and date-range support?**
3. **Photo-ID and document storage location** — single bucket, per-client folder, public or private URLs?
4. **Bug #5 (CM delete → data loss) ship date?**
5. **Bug #20 structural fix** (vs. the rollback)?
6. **Reporting-metrics meeting date** — Romain mentioned Friday June 12 or 13.

H raises all six on the Friday call.
