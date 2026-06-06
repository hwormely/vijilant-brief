# Vijilant — UI/UX Roadmap by Role

Every Vijilant screen, organized by who sees it. Source: `~/Padrino/09_Dev/SCREENS.md` (canonical inventory), `~/Padrino/09_Dev/MOCKUP_HANDOFF.md` (v5 design tokens — LOCKED), `~/vigilant/AGENTS.md` (locked v5 details), and the architecture map.

**Design system:** v5 — teal `#3ECFB2` · navy `#1B2A4A` · coral `#F4714F`. DM Sans body + DM Mono wordmark/KPIs. Tier ramp navy→teal→coral→red (T1 Stable=navy → T4 Critical=red). Conic Recovery Score ring. Glassy KPI cards on a navy→teal hero. v5 is the **only** design system — pre-v5 (green-token / purple-retheme / v2–v4) are purged.

**Five roles:** Care Manager (CM) · CM Lead · Supervisor · Admin · Survivor.

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
| **Messages** | with clients + team | with CMs + clients | org-wide thread | org-wide | with own CM only |
| **AI Call Notation** | record + review | review | review | review | — |
| **Metrics** | own | own + per-CM ⚠ | org | org + financial | — |
| **Audit log** | own actions | own + CMs ⚠ | org | full org | — |
| **Profile** | ✓ | ✓ | ✓ | ✓ | ✓ |
| **Team management** | — | view assigned CMs ⚠ | invite + edit | full + role swap | — |
| **Org Setup / Billing** | — | — | — | full | — |
| **Operator Console** | — | — | — | platform-admin only | — |

**Legend:** ⚠ = surface exists in code but the **CM Lead role itself is not yet first-class** — today, CM Leads are running through Supervisor permissions. Phase 1 dependency from HAMPSTER: stand up CM Lead as a real role (spec exists at `vijilant-cm-lead-workflow-spec.md`).

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

### CM Lead UI rules
- All shared with Supervisor: tier badges navy→red, recovery score rings 72/36 px, gradient buttons.
- Lead-only: a "lead chip" (small DM Mono pill, coral-on-warm-bg) on every screen header so the user always knows they're in Lead view, not CM view.
- Double-tap-to-top on bottom-nav (same as CM).

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

### Team page
- All staff (CM + Lead + Supervisor + Admin) with role chip, last sign-in, caseload count, "Invite Member" CTA.
- Per-row [Edit] modal → change role, change permitted_roles, deactivate.

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

### Admin design rules
- Same v5 tokens.
- "Admin chip" (DM Mono pill) on Billing, BAA, Org Setup, Operator Console — visual cue that the action is org-wide and audit-logged.
- Destructive actions (deactivate user, cancel subscription) require typed confirmation.

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

### Survivor design rules — non-negotiable
- **Reading level: 6th grade.** No "Recovery Score," no "T4," no "domain." Use "Where you are," "Critical / Needs attention / Stable / Strong."
- **One CTA per screen** (the action items card on the home is the only exception, and it's max 3 items).
- **Plain-language tier labels:** "Critical · Complex · Moderate · Stable" — never numeric.
- **Don't show the 0–1000 score number.** Show the ring + the band ("Stable") + a sentence ("You're in a strong place. Keep going.").
- **Language switcher in the header** — start with EN + ES (per `multilingual` skill spec). Voice input/output as a v1.1 addition.
- **Trauma-informed copy** — no exclamation points, no "Great job!", no urgent red unless something is actually urgent.

---

## Cross-role surfaces

### Messaging (Stage 7) — unified inbox
- Conversations + notifications in one inbox (the standalone bell was removed).
- Web: `/messages`. Mobile: header chat icon on Dashboard.
- **Survivor containment:** survivor sees only their own `dm_survivor` thread (RLS-enforced).
- **Realtime** via Supabase publication; polling fallback.
- **PHI-free notification trigger** at the DB layer — the notification row says "you have a new message" without disclosing message content.

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
