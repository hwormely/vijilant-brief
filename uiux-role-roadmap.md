# Vijilant — UI/UX Roadmap by Role

Every Vijilant screen, organized by who sees it. Source: `~/Padrino/09_Dev/SCREENS.md` (canonical inventory), `~/Padrino/09_Dev/MOCKUP_HANDOFF.md` (v5 design tokens — LOCKED), `~/vigilant/AGENTS.md` (locked v5 details), `~/Padrino/09_Dev/vijilant-v6-features-handoff.md` (the six v6 feature sets, locked 2026-06-12), and the architecture map.

**Design system:** v5 — teal `#3ECFB2` · navy `#1B2A4A` · coral `#F4714F`. DM Sans body + DM Mono wordmark/KPIs. Tier ramp navy→teal→coral→red (T1 Stable=navy → T4 Critical=red). Conic Recovery Score ring. Glassy KPI cards on a navy→teal hero. v5 is the **only** design system — pre-v5 (green-token / purple-retheme / v2–v4) are purged.

**Scope:** v6 (locked 2026-06-12) expands Vijilant from **16 → 38 screens** by adding six feature sets on top of the shipped v5 base — Advocacy + Unmet Needs (15–19) · Calendar (20–24) · Team / Profiles (25–28) · Maps & Directions (29–32) · Messages & Notifications (33–35) · Case Consultation Tracker (36–38). It also reworks the leadership nav (see "v6 nav change" below). All of it is documentation here — no app code.

**Five roles (code: six `profiles.role` values + token portal):** Care Manager (`case_manager`) · CM Lead (`cm_lead`) · Resource Lead (`resource_lead`) · Supervisor (`supervisor`) · Admin (`admin`, +ED via admin/supervisor) · Survivor (token-based portal, not a `profiles` role). **Visibility ladder** (consistent across Metrics, Caseload Health, Consultations): CM sees own; Lead/Admin/ED see all; peers see nothing; Resource Lead sees aggregate only (no individual PHI); survivors never.

### v6 nav change (LOCKED)
- **Leadership sidebar:** `Dashboard · Calendar · Residents · Resources · Team · Messages · Metrics · [Org Setup — Admin] · Profile`.
- **"Care Managers" tab REMOVED** for all leadership roles → replaced by **"Team"** (the Team tab leads with the leadership roster overview; see §Team). **"All Clients" RENAMED → "Residents".**
- **Mobile tab bar:** `Dashboard · Clients · + · Resources · Messages`. **Metrics was REMOVED from the mobile bar → replaced by Messages.**
- **No standalone "Consultations" nav item.** Two pathways instead (see §Case Consultation Tracker): (1) Team → click a CM → Metrics → Consultations tab; (2) Residents → survivor case file → consult form (client-bound).

### Ring semantics (v6, MOCKUP_HANDOFF §18a — easy to get wrong)
- **Conic Recovery Score ring = SURVIVORS ONLY** (their own 0–1000 recovery score).
- **Caseload Health = navy-core ring** (hero) / **segmented bar** (lists) — the CM aggregate. NEVER the conic ring. Composite of avg client recovery + follow-up timeliness + On-Track share from consults + overdue-consult count.

---

## Per-role coverage matrix

|  | CM | CM Lead | Supervisor | Admin | Survivor |
|---|---|---|---|---|---|
| **Sign in** | ✓ | ✓ | ✓ | ✓ | — (token link) |
| **Onboarding / Accept Invite** | ✓ | ✓ | ✓ | ✓ | ✓ |
| **Dashboard** | "My Day" | "Lead Console" ⚠ | "Org Overview" | "Org Rollup" ⚠ | "My Case" |
| **Caseload list** | My Cases | All assigned to my CMs ⚠ | All Clients | All Clients | — |
| **Client Detail** | full 7-tab | full + reassign | full + reassign | full + reassign + audit | — |
| **Assessment** | run | review | review | review | — |
| **Recovery Plan** | edit | edit + approve | review | review | view |
| **Notes** | write | review batch ⚠ | review | review | — |
| **Follow-ups** | own | overdue queue ⚠ | overdue queue | overdue queue | — |
| **Applications** | manage | manage + co-sign | manage | manage | view + sign |
| **Documents** | upload | manage | manage | manage | upload + view |
| **Resources** | search + match | search + match | search + match + edit | full CRUD | — |
| **Intake Submissions** | — | verify ⚠ | verify + assign | verify + assign + reject | — |
| **Messages** (v6 hub) | DMs + channels + My Survivors | DMs + channels + survivors | org-wide channels | org-wide | with own CM only |
| **Notifications** (v6, Profile-only) | own | own | own | own | — (case-ID push only) |
| **AI Call Notation** | record + review | review | review | review | — |
| **Metrics** (3-tab home) | own | own + per-CM ⚠ | org | org + financial | — |
| **Caseload Health** (v6, §18a) | own (navy ring) | all CMs | all CMs | all CMs | — |
| **Consultations** (v6) | own cases | all (review/adjust) | all | all | — |
| **Calendar** (v6) | own + org events | all CMs + org | org | org | — |
| **Needs + Advocacy** (v6, tabs 8–9) | edit | edit + lead_review | review | review | view (own, plain-language) |
| **Maps & Directions** (v6) | own clients + resources | all clients | all clients | all clients + org sites | own home + CM card |
| **Team** (v6 tab) | directory (view) | roster overview + directory | roster overview + directory | roster + directory + role/site edit | "Your CM" card only |
| **Audit log** | own actions | own + CMs ⚠ | org | full org | — |
| **Profile** (v6: photo/bio/specializations) | ✓ + VJ-assist | ✓ | ✓ | ✓ + role/site edit | ✓ (CM card view) |
| **Org Setup / Billing** | — | — | — | full | — |
| **Operator Console** | — | — | — | platform-admin only | — |

**Legend:** ⚠ = surface exists in code but the **CM Lead role itself is not yet first-class** — today, CM Leads are running through Supervisor permissions. Phase 1 dependency from HAMPSTER: stand up CM Lead as a real role (spec exists at `vijilant-cm-lead-workflow-spec.md`). The old standalone "Team management" row is folded into the v6 **Team** row above (the nav tab now leads with the leadership roster overview).

---

## Care Manager (CM) — the daily workhorse

The CM is the single most important user. They run the floor — daily intake, assessments, notes, follow-ups, recovery plan execution, resource referrals.

### Screen 4 — CM Dashboard "My Day"
- **Hero strip (mobile):** navy→teal gradient with glassy KPI cards (white numerals) — *signature look, don't flatten to white cards*.
- **KPI strip (5 cards):** Active Clients · Notes This Week · Follow-ups Done · Assessments · Aid Apps.
- **Module row (3-col):** Overdue Follow-ups · Score Drop Alerts (≥100 pts drop in 30d) · New Resources (last 14d).
- **Full-width:** Recent Activity feed.
- **VJ launcher** (gradient FAB matching achievement tiles) → bottom sheet for VJ chat.
- **Locked-v5 details:** mobile nav active = teal→coral gradient via SVG defs; resource links = navy not teal; FABs = grad-accent (teal→coral).

### Screen 6 — My Cases
- Initials chip · name · case ID · **tier badge** (navy→red ramp) · **36px Recovery Score ring** · last contact · status.
- Mobile filter pills: All / T1 / T2 / T3 / Closed.
- LLP-style border accents: red = no longer seeking, coral = active outreach, gray = standard.
- Desktop: full data-table with column toggles. Search + sort by every column.

### Screen 7 — Client Detail (the case file)
- **Hero:** gradient strip · 72px Recovery Score ring · name · tier badge · disaster event tag · Edit + Log Contact buttons.
- **7 locked tabs (in order):** Notes · Assessments · Recovery Plan · Follow-ups · Applications · Documents · Resources.
- **FAB gradient circles** (grad-accent) for the two main actions: new AID application · draft message.
- **AI brief at the top** (cached 5 min): 3 parts — current state · next action · risk.
- **Locked v5:** double-tap-to-top on the bottom-nav.

### Screen 8 — Assessment Form
- **Domain tabs:** Housing · Employment · Mental Health · Financial · Legal.
- Completed domains: Q&A summarized in `→ [answer_label]` format + a domain summary pill.
- Active domain: text-only radio options (never numeric prefixes — CMs answer in plain language; the math is computed server-side).
- Domain progress bar AND overall progress bar.
- Bottom CTAs: ← Previous / Next Question → (gradient on white).

### Screen 9 — New Client Intake / Quick Entry
- **4 steps:** Personal Info → Disaster & Housing → Needs Assessment → Consent & Signature.
- Desktop: gradient left panel showing step progress + form on right.
- Each step: gradient CTA, Save Draft option.
- Generates Client ID immediately so the CM can build the record even before all 12 sections are complete.

### Screen 10 / 10a — Resources Directory + Detail
- Category filter pills: All / Housing / Financial / Mental Health / Employment / Legal.
- Each resource card: category · name · description · [+ Add to client] gradient CTA · [Details →].
- Resource Detail (desktop): 58% left pane (resource info + application form) / 42% right pane (matching clients).
- Resource Detail (mobile): 2 tabs — Resource Detail / Matching Clients [N].
- **Match criteria** (the magic): Area Needs tag intersection · domain score below Stable · Recovery Score ≤ 749 · "Already Applied" guard so the same survivor doesn't get the same app twice.
- Match reasons in plain language (never numeric — "Housing disrupted, in Eaton fire zone, eligible for HAP" instead of "score = 23").

### Screen 12 — Metrics (CM-scoped)
- KPI strip (6 cards): Notes Logged · Follow-ups Done · New Assessments · Aid Apps Submitted · Resources Approved · Funding Approved.
- Recovery Score trend chart — filled gradient area, green=up / red=down.
- Top Movers — Improving + Top Movers — Needs Attention.
- Scope param: `?cmId=` (this CM only).

### Screen 14 — Profile
- Profile card · Org card · Achievements (SVG icons, no emoji; earned = gradient bg + white icon, locked = gray bg + muted) · Security (Face ID toggle) · Notification preferences · Sign Out (red text, confirmation modal).
- **Face ID** is biometric-only, no passcode fallback (HIPAA).

### v6 — CM-facing additions

The CM's case file grows from 7 tabs to **9** (Needs + Advocacy added), and the CM picks up Calendar, Maps, the v6 Messages hub, a richer Profile, and the client-bound consult form. These are the v6 screens the CM touches:

#### Screens 15–19 — Needs + Advocacy (on the case file)
- **Tab 8 — Needs:** per-client unmet-needs ledger. Each need: title · category · est. cost · cost breakdown · fair-price evidence (doc refs) · vendor + W-9 status (`not_requested / requested / received`) · status (`identified → vetted → presented → funded / partially_funded / denied / withdrawn`) · funded amount + funder.
- **Tab 9 — Advocacy** (moved 8→9): package builder. Section 6 "The Ask" pulls selected needs via `package_needs`; supporting docs (photos/invoices/inspections/W-9) attach via `package_attachments` and render in an export section. AI generation is PHI-stripped (reads notes/aid-apps/follow-ups/goals/assessments — never names/addresses/DOB).
- **Package lifecycle:** `cm_review → lead_review → approved` — `lead_review` is REQUIRED for the `ltrg_unr` audience, optional for `external_funder`. Export preview produces the funder-facing package.

#### Screens 20–24 — Calendar
- **CM desktop dashboard:** Hero Calendar Strip (Variant B). **Mobile:** day-stack agenda reached via dashboard.
- **Views:** Day / Week / Month (all devices); Quarter deadline-horizon (desktop only). Gradient-encased TODAY date everywhere.
- **Auto-Event Engine** (projection, not duplication): saving a note's Next-Steps date, a goal target, an aid-app/referral deadline, or a follow-up upserts a calendar event keyed on (source_table, source_id). Auto-events are edited **at the source**; only manual + org events edit in a sheet over the dimmed Day view.
- Six event-category colors (locked palette, never color-only). External sync is one-way + PHI-masked (pushed title = `"{Type} · {Case ID}"`, no names).

#### Screens 29–32 — Maps & Directions (Esri ArcGIS, BAA-signed)
- **Tappable addresses:** Client Detail hero + Current State, Quick Entry, resource cards + Resource Detail.
- **Survivor map:** Esri basemap · perimeter polygon overlay · home pin with structure-status badge · dashed route from CM's on-demand location + drive time.
- **ONE CTA "Start Directions"** → hands off to the device maps app (audit-logged; CM location requested on open, used for route, discarded — never stored).
- **"Resources Near Client"** = map centered on the home, catalog plotted, list by distance (distances computed server-side from stored coords — no added PHI exposure).

#### Screens 33–35 — Messages hub
- **Messages = humans→you.** CM lands on the v6 hub (the CM Board's v5 home): channels (#general, #resources, #site-…), DMs, and **My Survivors** threads.
- Case IDs OK in any channel; **names NEVER** — VJ nudges the sender if a client name is detected in a draft.
- **Mobile: Messages is Tab 5** (replaced Metrics) + a dashboard Messages card + push. Survivor push payloads carry **case ID only**, never message content.
- **Notifications** (system→you) live in a Profile module only — see Screen 28 below.

#### Screens 25–28 — Profile (v6 expansion)
- **Profile edit gains VJ-assist:** VJ drafts the **bio** from the CM's activity (tenure, caseload composition, closed-case types) and suggests **specializations** from caseload patterns (CM accepts/dismisses chips). Certifications stay **MANUAL** (compliance record — never AI-inferred).
- **Self-edit:** photo · bio · phone · languages · specializations · certifications · experience. **Admin-only:** role · site.
- **Metrics Snapshot card** = navy-core **Caseload Health** ring (NOT the conic recovery ring) + tier-mix chips + 4 stats. Tap → tabbed Metrics home (Performance / Caseload Health / Consultations).
- **Notifications module** (Screen 28 area): per-type push/in-app toggles. Badge sits over the avatar (mobile) / Profile nav item (desktop). No global bell.

#### Screens 36–38 — Consult form (client-bound, from the case file)
- The CM reaches consultation via **Residents → survivor case file → consult form** (Pathway 2). The form is bound to the client; it is NOT a standalone nav item.
- New consultation **auto-prefills** tier, recovery_phase, last contact, next follow-up, and current needs from the live case file — the CM/Lead reviews/adjusts, does not re-key.
- Action items are real tasks: each one creates a calendar event (`event_type='manual'`) + a notification to the responsible party.

### CM-only background features
- 30-min idle auto-logoff (mobile + web).
- Encrypted persisted query cache for offline.
- Durable offline mutation replay (notes, follow-ups, goals queue and replay after restart).

---

## CM Lead — first-class role, NOT YET shipped

CM Leads (Marisol Espino + Sara Potter at LLP) review the field. They verify intake, assign clients to CMs, oversee caseload health across CMs, escalate, support CMs in real-time.

**Current state:** they run via the Supervisor permission tier. The CM Lead role itself is not yet stamped into `profiles.role` or `permitted_roles` enum, and four screens are missing.

### Spec → roadmap

Source: `~/Padrino/09_Dev/vijilant-cm-lead-workflow-spec.md`.

1. **CM Lead Dashboard "Lead Console"** — NEW screen. KPIs: Pending Intake · CMs in My Group · T4 Critical Queue · Notes Needing Review · Caseload Imbalance Flag. Module row: Score Drop Alerts (org-wide) · CMs Over Capacity (≥35) · Overdue Follow-ups across all CMs.
2. **CM Roster** — list of every CM in the Lead's group with caseload count, last sign-in, T4 count, overdue follow-up count, note-review queue depth, badge of "over capacity" if ≥35.
3. **Caseload by CM** — drilldown from CM Roster. All clients of a single CM. Same Client List shape but filtered, with a "Reassign" CTA per row.
4. **Note Review Queue** — CMs author, CM Leads review. Each row: client · CM · note excerpt · timestamp · [Approve · Request Edit · Flag]. The flag option creates an in-platform message back to the CM.
5. **Escalation Inbox** — incoming escalations from CMs (e.g., "this client needs a Supervisor's eyes"). State machine: new → claimed → resolved.

### v6 — CM Lead consultation + roster (the leadership view)

The two v6 surfaces that matter most for the Lead:

1. **Consultation Pathway 1 — Team → click a CM → Metrics → Consultations tab.** This is the per-CM rollup + cadence flags. From here the Lead reviews each CM's consultations, the overdue-for-consultation list (flagged by tier cadence, e.g. T1 every 2 weeks), and adjusts the auto-prefilled consult fields. (Pathway 2 — the client-bound consult form off a survivor case file — is also available to the Lead.)
2. **Leadership roster overview — at the TOP of the Team tab** (Lead/Admin/ED only). This is the **former standalone Screen 38 consult dashboard, merged into Team**: every CM's **Caseload Health bars** + cadence flags + a **program-issues feed**, sitting above the staff card grid. Build it as part of the Team screen, not a separate route.

The consult itself (Screens 36–38) feeds three things: **action items → calendar events + notifications** to the responsible party; **System Issues → `program_issues`** (ED/Admin review surface, types: workflow/resource_gap/training_need/technology/partner/policy/communication); and **On-Track share → the Caseload Health composite**. Coaching that lands lifts the score — a case moving Escalation → On Track raises it. Caseload Health is the navy-core ring (hero) / health bars (roster), never the conic recovery ring.

### CM Lead UI rules
- All shared with Supervisor: tier badges navy→red, recovery score rings 72/36 px, gradient buttons.
- Lead-only: a "lead chip" (small DM Mono pill, coral-on-warm-bg) on every screen header so the user always knows they're in Lead view, not CM view.
- Double-tap-to-top on bottom-nav (same as CM).
- **v6 nav:** the Lead's sidebar is the leadership set — `Dashboard · Calendar · Residents · Resources · Team · Messages · Metrics · Profile` (no Org Setup; that's Admin-only). The removed "Care Managers" tab is replaced by **Team**.

---

## Supervisor

Supervises the whole org. Sees all clients, all CMs, all metrics. Reviews supervisor brief. Assigns clients. Manages team.

### Screen 5 — Supervisor Dashboard "Org Overview"
- **KPI strip (5 cards):** Active Clients · CMs Active · T1+T2 Cases · Funding Approved · Open Escalations.
- **Module row (3-col equal):** Score Drop Alerts · CMs Over Capacity (≥35) · New Resources.
- **Full-width:** Intake Pipeline (Pending → Verified → Assigned → Follow-up counts).
- **Supervisor brief at top** (the 3-part AI summary: caseload state · top priority · concerns).

### Screen 13 — Intake Submissions
- **4 pipeline tabs with count badges:** Pending · Verified · Assigned · Follow-up.
- **Pending → Supervisor:** [Verify ✓] [Flag] [Reject].
- **Verified → CM Lead:** [Assign to CM →] [Flag] [Reject].
- **Assigned → CM:** [Quick Entry → Create Client].
- **Follow-up → Supervisor:** [Resolve & Verify ✓] [Reject].
- Candidate card shows: submission date · household size · language · damage type · needs chips · contact info · photo-ID warning if missing.
- Flags: `no_id` · `ambiguous_match` · `out_of_zone`.
- Assign modal: CMs sorted by caseload (least loaded first), ⚠ over-capacity flag at ≥35.

### Team tab (Screens 25–28, v6) — replaces the removed "Care Managers" tab
The Team tab is the consolidation point. It has two stacked regions:

1. **Leadership roster overview (TOP, leadership-only):** every CM's **Caseload Health bars** + cadence flags + a **program-issues feed**. This is the former standalone consult dashboard, merged in. Leads/Supervisors/Admin/ED see it; CMs and Resource Leads do not.
2. **Staff directory (below):** Resource-card grid of all staff — CM + Lead + Resource Lead + Supervisor + Admin + ED — with role chip, photo, last sign-in, caseload count, "Invite Member" CTA. Per-row [Edit] modal → change role, change permitted_roles, change site, deactivate.

Each staff card opens a **profile detail** (photo · bio · phone_work · languages · specializations · certifications · experience · featured badges as avatar medallions). The **Metrics Snapshot** card on a profile (navy-core Caseload Health ring + tier-mix chips + 4 stats) is visible to the CM themselves + Lead/Admin/ED only; peers don't see the card at all (omitted, not greyed). Tap → the 3-tab Metrics home.

### v6 — Consultation pathways (Supervisor)
The Supervisor reaches consultations the same two ways as the Lead — no standalone "Consultations" nav item:
1. **Team → click a CM → Metrics → Consultations tab** (per-CM rollup + cadence flags).
2. **Residents → survivor case file → consult form** (client-bound).

### v6 — Calendar + Maps
- **Calendar:** org-scope of the auto-event engine — sees all CMs' projected events plus org events; can create org events (community-facing, full detail, no PHI) via the new-org-event modal.
- **Maps:** all clients' survivor maps + org-sites map; Org Settings address is tappable.

### Audit Log
- Filterable by user · action · table · date range.
- Export (signed CSV) — **GAP today**, see enterprise-gap-analysis.

### Metrics — Supervisor scope
- KPI strip same as CM but org-wide.
- "View per-CM" drilldown that scopes Metrics to a single CM with the `?cmId=` param.

---

## Admin

Org owner. Everything a Supervisor sees, plus billing, BAA status, org settings, plan/seat management, and the comp'd-org case.

### Admin-only additions
- **Billing page** — current plan + add-ons + Stripe customer portal link + invoice history. Annual prepay decision. Seat usage vs cap.
- **BAA page** — current BAA status (signed / not_required / pending) + signer info + signed-PDF download + re-sign on template version bump.
- **Org Setup** — name · type · HIPAA role (CE / BA) · EIN if 501c3 · default disaster zone · primary domain · branding (white-label addon).
- **Audit Log Export** — long-running export job for compliance / counsel requests. **PARTIAL today**.
- **Team CRUD** — invite, edit, role swap, deactivate. Already shipped.

### v6 — Admin additions
- **Org Sites:** the `org_sites` table (name · address · phone · lat/lng) is managed from Org Setup; addresses are tappable into the Maps surface, and site chips appear on staff profiles. lat/lng are populated by the Esri geocoder.
- **Team tab — full control:** Admin (and ED via admin/supervisor) see the leadership roster overview and the staff directory, and are the only roles that can edit **role** and **site** on a profile.
- **Program-issues review:** `program_issues` raised from consults (System Issues field) surface to ED/Admin for review — types: workflow / resource_gap / training_need / technology / partner / policy / communication.
- **Org unmet-needs rollup:** aggregate section on the Metrics screen (Supervisor + Resource Lead too) — aggregate only, no individual PHI.
- **Nav:** Admin is the only role with `[Org Setup]` in the leadership sidebar.

### Admin design rules
- Same v5 tokens.
- "Admin chip" (DM Mono pill) on Billing, BAA, Org Setup, Operator Console — visual cue that the action is org-wide and audit-logged.
- Destructive actions (deactivate user, cancel subscription) require typed confirmation.

---

## Resource Lead — aggregate-only (v6 makes this role explicit)

`resource_lead` is now a first-class `profiles.role` value. The Resource Lead runs the catalog and reads outcomes in aggregate — **never individual survivor PHI**.

- **Resources:** search + match + edit on the catalog (full CRUD is Admin).
- **Metrics / unmet-needs rollup:** sees the org unmet-needs rollup and Caseload Health **in aggregate only** — no per-survivor drill-down, no individual recovery scores.
- **Team tab:** appears in the staff directory; does **not** see the leadership roster overview (that's Lead/Admin/ED).
- **Messages:** #resources channel is auto-fed by `catalog_notifications` as system posts — the Resource Lead's catalog updates broadcast here.
- **Consultations / case files:** none. The visibility ladder bars individual PHI for this role.

---

## Survivor — the most critical role to get right

Survivors are disaster-recovery clients. They are vulnerable, under stress, often on slow connections, often in temporary housing. The survivor portal must be the simplest, calmest, most direct surface in the product. No CM jargon. Plain language. Big tap targets.

### Access
- Secure link sent by CM (no login credentials required for v1).
- v1.1: optional account creation via gov-ID + magic-link if the survivor wants persistent access.

### Screen S1 — Survivor Portal Home
- **Hero:** gradient strip with 72px Recovery Score ring (their own score, plain-language band — "Stable" not "T1"), client name, case ID, tier badge, CM info card with photo + name + "Message CM" CTA.
- **Action items card:** "Upload your insurance docs" · "Sign pending application" · "Reply to your CM" — chunky cards, one tap per action.
- **Latest message from CM** — preview + "Read all messages" link.
- **4-tab mobile nav (no + button):** My Case · Plan · Apps · Messages.
- **Desktop sidebar:** My Case · Recovery Plan · Applications · Messages · Documents.

### Screen S2 — Survivor Case File (4 tabs)
- **My Plan:** read-only view of the recovery plan tasks. Categorized: housing · financial · health · documentation · support_network. Each task: title · status · target date · "Mark Done" (creates an in-platform message back to the CM).
- **Applications:** read-only list + signature where required. Autofill fields shown with green border + "Filled" chip. Signature pad (tappable/clickable) → "Sign & Authorize Submission" gradient CTA.
- **Messages:** thread between survivor and their assigned CM. Real-time via Supabase publication. PHI-free notification trigger so the CM gets pinged.
- **Documents:** upload interface for docs requested by CM (gov ID · insurance claim · lease · FEMA application). Camera-first on mobile; drag-drop on desktop.

### v6 — Survivor touchpoints
- **"Your CM" card upgrade:** now shows the CM's **photo + bio snippet + languages + featured badges** (avatar medallions) — but **NO phone/email**; the survivor reaches the CM only through portal messaging.
- **Maps:** the survivor sees their own home location + CM card context; no other clients, no staff locations. Survivor coordinates are PHI (strict RLS, never logged, never to AI).
- **Needs / Advocacy:** read-only, plain-language view of their own needs only — no cost-evidence internals, no funder vocabulary, no other survivors.
- **Push / notifications:** survivor push payloads carry **case ID only, never message content** (lock-screen visibility). Survivor sees no Notifications module — the in-portal "Latest message from CM" + Messages tab covers it.
- **Not changed:** survivors never see Team, Calendar (staff), Consultations, Caseload Health, or any aggregate.

### Survivor design rules — non-negotiable
- **Reading level: 6th grade.** No "Recovery Score," no "T4," no "domain." Use "Where you are," "Critical / Needs attention / Stable / Strong."
- **One CTA per screen** (the action items card on the home is the only exception, and it's max 3 items).
- **Plain-language tier labels:** "Critical · Complex · Moderate · Stable" — never numeric.
- **Don't show the 0–1000 score number.** Show the ring + the band ("Stable") + a sentence ("You're in a strong place. Keep going.").
- **Language switcher in the header** — start with EN + ES (per `multilingual` skill spec). Voice input/output as a v1.1 addition.
- **Trauma-informed copy** — no exclamation points, no "Great job!", no urgent red unless something is actually urgent.

---

## Cross-role surfaces

### Messaging + Notifications (Stage 7 → v6 Screens 33–35) — now SEPARATED
v6 splits the old unified inbox into two surfaces — **nothing appears in both**:
- **MESSAGES = humans→you.** Unified hub at sidebar **"Messages"** (all roles, own unread badge) — the CM Board's v5 home. Channels (`public / site / dm / case_consult`): #announcements (Admin/ED post), #general, #resources (auto-fed by `catalog_notifications` as system posts), #site-pasadena / #site-altadena (auto-membership). DMs = two-member channels. **Mobile: Messages is Tab 5** (replaced Metrics on the bar) + dashboard card + push.
- **NOTIFICATIONS = system→you.** Live in a **module on the Profile screen ONLY** — no global bell. Badge over the Profile nav item (desktop) / avatar (mobile). Per-type push/in-app toggles. (`notifications` + `notification_preferences` tables already exist — extended, not recreated.)
- **Survivor containment:** survivor sees only their own `dm_survivor` thread (RLS-enforced); no channels.
- **Realtime** via Supabase publication; polling fallback.
- **PHI-free + name-free:** case IDs OK in any channel, names NEVER (VJ nudges the sender on a detected client name). Survivor push payloads carry case ID only, never content.

### AI Call Notation (Stage 9)
- Staff-only. Survivors never see this surface.
- Consent script v0.1 must be accepted by the survivor before recording starts.
- CM Records → STT (Deepgram, dormant) → Claude summarize (bilingual EN+ES) → DRAFT case note → CM reviews side-by-side with transcript → Approve writes a real case note (Discard wipes the transcript).
- **Today:** Test mode (synthetic transcript → real Claude bilingual draft → real case note). Live audio gated on Deepgram BAA + `expo-audio` on mobile.

### VJ chat (the in-app assistant)
- Available to CM, CM Lead, Supervisor, Admin. Not to Survivor (yet).
- Role-conditional system prompt — different tools available to different roles.
- SSE streaming, tool-use loop (max 10 iterations), session-only history (no DB persistence).
- Gradient FAB matches the rest of the design language.

---

## Mobile vs. desktop split

| Screen | Mobile | Desktop | Notes |
|---|---|---|---|
| Dashboard (CM) | Primary use case · glassy KPI hero | Same data, denser layout | CMs work mostly from phones in the field |
| Dashboard (Supervisor/Admin) | Available but secondary | Primary use case | 3-column layouts assume desktop |
| Client Detail | All 7 tabs · stacked | All 7 tabs · side-by-side panes | The case file is the most-used screen |
| Assessment | One section at a time | One section at a time, wider columns | Long form — both surfaces need progress affordance |
| Resources Directory | Card stack | Card grid · 3-col | Equally important both surfaces |
| Resource Detail | 2 tabs (info / matching) | Dual-pane (58/42) | Desktop is better for batch matching |
| Intake Submissions | Stack | 4-pipeline-column board | Lead/Supervisor task — desktop primary |
| Operator Console | — | Desktop only | Platform-admin, low-frequency |
| Survivor Portal | Primary | Available | Survivors are mobile-first |
| AI Call Notation | Capture only | Capture + review | Review needs side-by-side transcript |
| Calendar (v6) | Day-stack agenda via dashboard | Hero strip + Day/Week/Month/Quarter | Quarter deadline-horizon is desktop-only |
| Needs + Advocacy (v6) | Stacked tabs 8–9 on case file | Tabs 8–9 + package builder + export preview | Export preview is desktop-primary |
| Maps & Directions (v6) | Map + "Start Directions" handoff | Map + Resources-Near-Client list | Handoff to device maps app on mobile |
| Messages hub (v6) | Tab 5 (replaced Metrics) | Sidebar "Messages" | The CM Board's v5 home |
| Team tab (v6) | Via Profile screen | Roster overview + staff grid | Leadership roster overview is desktop-primary |
| Consult form (v6) | From case file, stacked | From case file + per-CM Consultations tab | Client-bound; reached via Residents or Team→CM→Metrics |

---

## Phase plan for the UI/UX surfaces still missing

**Day 1–2:** CM Lead Dashboard ("Lead Console") · CM Roster · Caseload by CM. These three close the biggest role gap.

**Day 2–3:** Survivor portal polish — plain-language tier labels everywhere, "My Data" download CTA, language switcher in header.

**Day 3–4:** Admin rollup view — caseload + outcomes + capacity rollups (the level above Supervisor brief).

**Day 4–5:** Onboarding tour for new CMs · empty-state copy across every screen (today some are blank) · skeletons for slow connections.

**Day 5–6:** Accessibility pass — WCAG 2.1 AA. Color contrast on the tier coral check, focus rings on every interactive element, keyboard nav on the assessment, screen-reader labels on the rings.

**Day 6–7:** Settings — notification granularity (per-event-type), language preference, time-zone preference. Survivor consent revocation surface.

---

## Anti-drift rules (LOCKED — never break)

These are H's named v5 parity locks from `~/vigilant/AGENTS.md`. They're stamped in code via the `pnpm check:v5` CI guard.

1. **Tier badges & filter pills:** always `tierColors()` from `@vigilant/tokens`. Ramp **navy → teal → coral → red**. NO local tier color maps.
2. **Domain strength pills:** always `domainStrengthColors()` — same ramp (Disrupted=red, Partial=coral, Stable=teal, Strong=navy, None=grey).
3. **Mobile nav icons:** active = grad-accent gradient (teal→coral) via SVG Defs + `url(#…)`; inactive = grey. Never solid teal active.
4. **Resource links → navy**, not teal. Card name + website + mobile contactLink.
5. **Mobile Dashboard hero:** teal-dominant navy→teal gradient with GLASSY KPI cards (white numerals) — the signature look. Don't flatten to white cards.
6. **Mobile client-detail FABs** (aid app + draft message): grad-accent gradient circles + white icons.
7. **Double-tap-to-top:** first tap on a bottom-nav button → navigate; second tap (already focused) → scroll-to-top. Via `hooks/useScrollToTopOnTabPress.ts` built on `expo-router`'s `useNavigation` (NEVER `@react-navigation/native`).
8. **Brand spelling:** "VIJILANT" (J) in every user-visible string, screen title, wordmark, and settings copy. Code paths keep `vigilant` for compatibility.
9. **v6 nav (LOCKED 2026-06-12):** Leadership sidebar = `Dashboard · Calendar · Residents · Resources · Team · Messages · Metrics · [Org Setup—Admin] · Profile`. "Care Managers" → **Team**; "All Clients" → **Residents**. Mobile tab bar = `Dashboard · Clients · + · Resources · Messages` (Metrics removed from the bar → Messages). **No standalone "Consultations" item** — two pathways only (Team→CM→Metrics→Consultations tab; Residents→case file→consult form).
10. **v6 ring semantics (§18a):** conic Recovery Score ring = SURVIVORS ONLY; **Caseload Health = navy-core ring (hero) / segmented bar (lists)** for the CM aggregate — never the conic ring for staff.
11. **v6 Messages/Notifications split:** Messages (humans→you) = sidebar hub; Notifications (system→you) = Profile module only. No global bell. Nothing in both.
12. **v6 names-never rule:** case IDs OK in channels and external calendar pushes; client names NEVER in any channel, push payload, or synced calendar title.
