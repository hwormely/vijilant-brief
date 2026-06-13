# Vijilant — Enterprise Gap Analysis (Apple-team Review)

Written as if a senior product team from an elite company (think: Apple's Health + Enterprise org reviewing a vendor before recommending it to a Fortune 100 procurement) is doing the once-over on Vijilant before it ships to enterprise customers.

Tone: high standards. The bar isn't "does this work?" — it's "would a sophisticated buyer trust this with the most sensitive disaster-recovery PHI in their region?"

This document is the gap list. Closing it is the precondition to credibly selling Enterprise + Government tiers.

---

## Methodology

We walked through every Vijilant surface — schema, API, web, mobile, observability, billing, compliance, operations — and graded each against an elite SaaS bar. Findings are organized into 8 categories:

1. **Schema gaps** (data model holes)
2. **Identity & access** (SSO, RBAC, lifecycle)
3. **Compliance & legal** (HIPAA, GDPR, CPRA, SOC 2)
4. **Operations & observability** (alerting, runbooks, on-call)
5. **Developer experience** (API for partners, webhooks, docs)
6. **Customer success** (in-app, account ops)
7. **Internationalization & accessibility**
8. **Pricing & packaging integrity**

Each finding has: **what's missing · why an enterprise buyer cares · effort to close (S/M/L) · suggested owner · cost impact**.

---

## 1. Schema gaps

The data model carries most of the long-term cost of a SaaS. Missing tables now means painful migrations later.

### 1.1 `sso_connections` — per-org SSO config
**Missing.** No way to configure SAML or OIDC for an organization. Every user signs in via email+password.
**Why it matters:** Enterprise IT will not allow employees to maintain a separate Vijilant password. Government tier 100% requires this on day one. Without SSO, you can't sell to LA County, FEMA, or any state agency.
**Schema:**
```sql
create table sso_connections (
  id uuid primary key default gen_random_uuid(),
  org_id uuid not null references organizations(id) on delete cascade,
  provider text not null check (provider in ('saml','oidc')),
  entity_id text,
  sso_url text,
  certificate_pem text,
  metadata_xml text,
  jit_provisioning boolean default true,
  default_role text default 'case_manager',
  attribute_mapping jsonb default '{}',  -- email, full_name, role, etc.
  enabled boolean default false,
  created_at timestamptz default now(),
  unique (org_id, provider)
);
```
**Effort:** L (3–5 days for SAML; OIDC is shorter).
**Owner:** API + Operator Console UI.
**Cost:** $0 incremental.

### 1.2 `scim_provisioning` — automated user lifecycle
**Missing.** No way for an IT system to push user creates/updates/deletes into Vijilant.
**Why it matters:** Enterprise IT requires SCIM 2.0 for "user joined → account provisioned, user left → account deprovisioned within 24h." Compliance frameworks (SOC 2) audit on this.
**Schema:** SCIM tokens + provisioning event log.
**Effort:** L.

### 1.3 `api_keys` — per-org API tokens
**Missing.** No public API today. Customers can't integrate.
**Why it matters:** Enterprise buyers want to push intake data from their existing CRM, pull metrics into their data warehouse, automate aid app status updates.
**Schema:**
```sql
create table api_keys (
  id uuid primary key default gen_random_uuid(),
  org_id uuid not null references organizations(id),
  name text not null,
  prefix text not null,           -- visible part: vj_live_abc123
  hashed_secret text not null,    -- argon2id of the full key
  scopes text[] not null default '{}',  -- 'read:clients', 'write:assessments', etc.
  created_by uuid references profiles(id),
  created_at timestamptz default now(),
  last_used_at timestamptz,
  expires_at timestamptz,
  revoked_at timestamptz
);
```
**Effort:** M.

### 1.4 `webhook_subscriptions` — outbound webhooks
**Missing.** Customers have no way to listen to Vijilant events (client assigned, assessment completed, recovery score change, BAA signed, etc.).
**Why it matters:** Same as API keys — integration. Enterprise systems-of-record (Salesforce, Workday, ServiceNow) consume webhooks.
**Schema:** subscription + event log + delivery attempt table with retry state.
**Effort:** M.

### 1.5 `audit_export_jobs` — long-running exports
**Missing.** Audit log is queryable but bulk-exportable only with patience.
**Why it matters:** Compliance audits, subpoenas, counsel requests all need date-range exports of the audit log. Today that's a manual query.
**Schema:** job table with state machine (queued → running → ready → expired) + signed S3 URL + 7-day TTL.
**Effort:** M.

### 1.6 `dsar_requests` — Data Subject Access Requests
**Missing.** No surface for a survivor to say "give me everything you have on me."
**Why it matters:** Required by GDPR Article 15, CPRA, multiple state laws. Disaster survivors may live in any state; California is GDPR-adjacent under CPRA.
**Schema:** request table + state machine (received → fulfilling → delivered → expired) + signed S3 manifest.
**Effort:** M.

### 1.7 `org_data_retention_policies` — per-org retention
**Missing.** Retention is global today (case notes 6yr from migration 052 design, transcripts 18mo, etc.).
**Why it matters:** Some orgs have shorter retention requirements; some longer. Government orgs often have statutory retention.
**Schema:**
```sql
create table org_data_retention_policies (
  org_id uuid primary key references organizations(id),
  case_notes_months int default 72,         -- 6yr default
  follow_ups_months int default 72,
  audit_log_months int default 120,          -- 10yr
  notation_transcripts_months int default 18,
  applied_at timestamptz default now()
);
```
**Effort:** S (table) + L (the actual deletion workers and proof-of-deletion audit).

### 1.8 `legal_holds` — court-ordered case freeze
**Missing.** Today archive/delete is permitted on any client. A court-ordered hold has no enforcement.
**Why it matters:** Subpoena response. A CM accidentally archiving a held client is a litigation hold violation.
**Schema:**
```sql
create table legal_holds (
  id uuid primary key default gen_random_uuid(),
  client_id uuid references clients(id),
  org_id uuid references organizations(id),
  reason text not null,
  imposed_by uuid references platform_admins(user_id),
  imposed_at timestamptz default now(),
  released_at timestamptz,
  cascade_to_docs boolean default true,
  cascade_to_notes boolean default true
);
```
Plus a BEFORE DELETE/UPDATE trigger on `clients`, `case_notes`, `client_documents`.
**Effort:** M.

### 1.9 `incident_reports` — security incident state machine
**Missing.** No formal incident record. Breaches are tracked in informal docs.
**Why it matters:** HIPAA breach notification rule requires a documented response. SOC 2 audits this.
**Schema:** severity, scope, affected orgs, timeline, notification status, lessons learned.
**Effort:** M.

### 1.10 `feature_flags` — per-org / per-cohort feature gating
**Missing.** Only env-level gates today (`NOTATION_ENABLED`).
**Why it matters:** Enterprise often wants a feature for one org only (white-label customizations, beta access, regulated feature rollouts). Government may forbid a feature an Enterprise customer requires.
**Schema:** flag definition + per-org override.
**Effort:** S (table) + M (the flag-evaluation library).

### 1.11 `idempotency_keys` — at-least-once side effects
**Partial.** `stripe_webhook_events.id` is the dedupe key for Stripe replay. No general pattern for our own mutations.
**Why it matters:** Mobile retries are common on flaky cell signals. A double-tap on "Submit Assessment" should never create two assessment rows.
**Schema:** generic idempotency key store + helper middleware.
**Effort:** M.

### 1.12 `survivor_consent_revocations`
**Missing.** A survivor can revoke consent for AI Call Notation, for messaging, for sharing data with partner orgs. Today consent is captured but revocation isn't modeled.
**Why it matters:** HIPAA + state law + survivor trust.
**Schema:** revocation log with scope (per-feature) + effective date.
**Effort:** S.

### 1.13 `parent_org_hierarchies` — org-of-orgs
**Missing.** Today every org is flat. Government tier needs "State has many Counties" hierarchy.
**Why it matters:** State-level deployments (e.g., CalOES) need to roll up across all county-level participating orgs.
**Schema:** `organizations.parent_org_id` + a recursive view.
**Effort:** M (data model is small; the consequences on RLS and reports are not).

### 1.14 `case_transfer_log` — explicit transfer between orgs
**Missing.** A survivor can move between orgs (LLP → other partner org). Today the only path is archive+recreate.
**Why it matters:** Disaster survivors move during recovery. Their case file should follow them.
**Schema:** transfer log with consent record + receiving-org BAA verification.
**Effort:** L (legal scope, not just technical).

---

## 2. Identity & access

### 2.1 SSO / SAML / OIDC
See 1.1. **L.**

### 2.2 SCIM 2.0
See 1.2. **L.**

### 2.3 2FA for staff (TOTP minimum)
**Missing.** Supabase Auth supports TOTP — not yet exposed in the UI.
**Why it matters:** Insider compromise is the #1 attack on a CM account.
**Effort:** S to expose; M to enforce per role (require for admin, optional for CM).

### 2.4 2FA / FIDO2 for platform admins
**Missing.** Platform admins access cross-org data. They should authenticate with a YubiKey.
**Effort:** M.

### 2.5 Granular role permissions
**Partial.** Roles are: case_manager, supervisor, admin, platform_admin, survivor. CM Lead is a fifth role per LLP's workflow but not modeled.
**Why it matters:** Government orgs often have 4–6 distinct roles (intake officer, case manager, supervisor, program manager, auditor, admin). Today they'd squeeze into 3.
**Effort:** M (add CM Lead first-class; add a permissions matrix beyond roles).

### 2.6 Session management surface
**Missing.** No way for a user (or admin) to see "active sessions" or revoke a session.
**Why it matters:** Compromise response. If a phone is stolen, the user needs to kill that session from the web.
**Effort:** M.

### 2.7 Conditional access
**Missing.** Enterprise IT often wants "block sign-in from outside the corporate VPN" or "only allow specific geolocations."
**Why it matters:** Government deployments require this.
**Effort:** M (rules engine on `requireAuth`).

---

## 3. Compliance & legal

### 3.1 Notice of Privacy Practices (NPP)
**Missing.** Required for Covered Entity orgs under HIPAA §164.520.
**Effort:** S (counsel-drafted) + M (in-app surface for survivor acknowledgement).

### 3.2 DSAR endpoint
See 1.6.

### 3.3 Audit log export
See 1.5.

### 3.4 Legal hold mechanism
See 1.8.

### 3.5 Right to be forgotten / data deletion
**Missing.** Today archive is a soft delete. True deletion (for a DSAR delete request) isn't implemented.
**Effort:** L (deletion has to cascade across audit, notes, documents, applications, transcripts, etc., with proof of deletion).

### 3.6 SOC 2 Type I → Type II
**Status:** not started.
**Why it matters:** Enterprise procurement asks for SOC 2 Type II report. Type I is achievable in 60 days; Type II takes 6 months of observed controls.
**Effort:** L (multi-quarter project). $15k–30k for the auditor; ongoing engineering cost in policy/control implementation.

### 3.7 HITRUST certification
**Status:** not started. Optional but enterprise + government often expect for healthcare-adjacent SaaS.
**Effort:** L (year+ project; $40k+ for the assessor).

### 3.8 BAA tracker
**Partial.** Pieces of this exist in `hipaa-compliance` skill spec. Operationalize as `~/vigilant/BAAS.md` with every vendor + expiry + contact.
**Effort:** S.

### 3.9 Privacy Policy + Terms of Service
**Partial.** Counsel review pending per Stage 8 notes. Required before public signup launches.
**Effort:** S (counsel) + S (in-app surfaces — already partially wired).

### 3.10 Cookie consent / tracker disclosure
**Missing.** Even though Vijilant doesn't run third-party trackers today, GDPR/CPRA expects an explicit banner if any cookie is set.
**Effort:** S.

---

## 4. Operations & observability

### 4.1 Status page
**Missing.** No `status.vijilant.io`. When the API or web is down, customers have no signal.
**Why it matters:** Enterprise procurement asks for the URL. Government often requires uptime publication.
**Effort:** S (StatusGator, Statuspage, or self-hosted CRON-driven page).

### 4.2 SLA monitoring
**Missing.** No formal SLA today; no monitoring against one.
**Why it matters:** Enterprise + Government will demand 99.9% with credits on miss.
**Effort:** M (define SLA + UptimeRobot or Better Stack + customer-facing dashboard).

### 4.3 On-call rotation + paging
**Missing.** H is the only on-call today. Single-point-of-failure for a customer-impacting outage.
**Why it matters:** Procurement asks "who responds at 3am?" — current honest answer is "H."
**Effort:** L (cultural + contractual; PagerDuty for the tooling).

### 4.4 Runbooks
**Partial.** Some exist informally; no consolidated `~/vigilant/RUNBOOK.md`.
**Why it matters:** Anyone responding to an incident needs a script.
**Effort:** M.

### 4.5 Distributed tracing
**Missing.** No request tracing across web → API → Supabase → Anthropic. Debugging multi-system failures is hard.
**Why it matters:** Enterprise scale exposes performance issues that need tracing to debug.
**Effort:** M (OpenTelemetry SDK + Honeycomb or Datadog).

### 4.6 Anomaly alerting on key metrics
**Partial.** Honeybadger handles errors; nothing watches "sign-in success rate" or "export volume" or "anon traffic spike."
**Effort:** M.

### 4.7 Backup verification cadence
See security hardening doc § Attack 11. **L** including the drill.

### 4.8 Capacity planning
**Missing.** No documented "at what scale does Render need to upgrade" decision tree.
**Effort:** S.

---

## 5. Developer experience (the "open" surface)

### 5.1 Public API documentation
**Missing.** No public docs.
**Effort:** M (Stoplight, Mintlify, or Scalar against an OpenAPI spec auto-generated from the Express routes).

### 5.2 OpenAPI spec
**Missing.** No formal spec.
**Effort:** M.

### 5.3 SDKs (TypeScript, Python)
**Missing.** Required for serious integration partners.
**Effort:** M each (after OpenAPI lands; auto-generate).

### 5.4 Sandbox environment
**Missing.** Today there's prod + dev. No sandbox where a partner can build against fake data.
**Effort:** M.

### 5.5 Webhook signing / replay protection
See 1.4.

### 5.6 Versioning policy
**Missing.** No `/v1/` prefix on the API today. First public-API customer locks the surface.
**Effort:** S (decide; document).

---

## 6. Customer success surfaces

### 6.1 In-product onboarding tour
**Missing.** A new CM signs in and sees an empty dashboard.
**Why it matters:** Time-to-first-value matters for adoption. The CM's first 10 minutes determine the partnership's success.
**Effort:** M.

### 6.2 Empty-state copy across every screen
**Partial.** Some screens have it; many don't.
**Effort:** S (UX copy pass).

### 6.3 In-app changelog / what's new
**Missing.** Releases happen; users don't know.
**Effort:** S.

### 6.4 In-app help search
**Missing.** No help search bar.
**Effort:** M (Algolia DocSearch on public docs once docs exist).

### 6.5 In-app feedback (NPS / CSAT)
**Missing.** Today the team learns of friction from QW-call equivalents. Direct in-app feedback would shorten the loop.
**Effort:** S (Pendo or self-hosted).

### 6.6 Support ticket surface
**Missing.** Today: email Dr. Syms or H. No formal ticket system.
**Effort:** M (Intercom / Zendesk / self-hosted) — costs $39–$99/mo.

### 6.7 Customer health dashboard (internal)
**Missing.** No central read of which orgs are at risk of churn.
**Effort:** M.

---

## 7. Internationalization & accessibility

### 7.1 i18n infrastructure
**Partial.** `multilingual` skill spec exists. `react-i18next` wired in web `main.tsx`. Mobile partial.
**Effort:** L to ship a second language end-to-end (Spanish first, per HAMPSTER).

### 7.2 RTL support
**Missing.** Required for Arabic survivor populations (rare today but disaster recovery may serve them).
**Effort:** M.

### 7.3 Voice input/output (HIPAA-safe)
**Missing.** Per multilingual skill: Deepgram Nova-3 for STT + AWS Polly for TTS. Wired into Stage 9 notation, not into the survivor portal as accessibility.
**Effort:** L.

### 7.4 WCAG 2.1 AA compliance
**Partial.** Accessibility doc exists at `~/Padrino/09_Dev/vigilant-accessibility.md`. Audit results unknown.
**Effort:** M for an audit; L to remediate.

### 7.5 Plain-language survivor copy
**Partial.** Locked v5 details specify survivor sees plain-language tier labels; some screens drift toward CM jargon.
**Effort:** S (copy pass against `~/Padrino/09_Dev/vigilant-ux-copy.md`).

---

## 8. Pricing & packaging integrity

These are commercial-side gaps. Per the pricing PRD (`VIJILANT_Pricing_and_Signup_Portal_Handoff.md`):

### 8.1 Volume tiers
**Missing.** Starter caps at 5 CMs. What happens at 6? Today the schema enforces no cap. Soft-cap + nag + auto-upgrade prompt would be normal.
**Effort:** M.

### 8.2 Per-seat add-ons
**Partial.** Add-ons are org-level (white-label, API, SLA). Per-seat priced add-ons (e.g., "Premium AI" per CM/month) aren't modeled.
**Effort:** S (schema) + M (Stripe wiring).

### 8.3 Usage-based billing for AI
**Missing.** Heavy-AI orgs cost more in Anthropic tokens. Today Vijilant absorbs the variance.
**Why it matters:** Per the overhead handoff, AI is the biggest variable cost. At scale, "all-you-can-eat AI" margins compress.
**Effort:** M (token metering) + M (Stripe metered billing).

### 8.4 Free trial mechanism
**Missing.** Self-serve sign-up goes straight to paid.
**Effort:** S (Stripe handles trials natively).

### 8.5 Annual contract negotiation surface
**Missing.** Enterprise + Government often want custom terms. Today the contact-sales path emails a human.
**Effort:** Out of scope for self-service; sales-ops is correct.

### 8.6 Coupon / promo code surface
**Partial.** RAP 20%-off is wired as a Stripe coupon. Generic promo codes (e.g., for partner-channel discounts) aren't surfaced.
**Effort:** S.

---

## 9. v6 feature sets — what's v1 vs. what's enterprise-phase

The six v6 feature sets (locked 2026-06-12, `vijilant-v6-features-handoff.md`) are operational-depth surfaces, not the compliance/identity table-stakes in §1–§8. An enterprise reviewer cares about them for two reasons: (a) most of each set is v1 and arrives with the standard build, which *raises* the product floor a sophisticated buyer is grading against; and (b) two specific pieces are themselves enterprise-tier and carry **new third-party BAA/vendor dependencies** — exactly the kind of thing procurement diligence surfaces.

### 9.1 What ships in v1 (raises the floor — no enterprise gate)
Team/Profiles (directory · VJ-assisted profiles · `org_sites` · leadership roster overview), Advocacy + Unmet Needs (PHI-stripped AI packages · `unmet_needs` ledger · PDF/DOCX/PPTX export), internal auto-Calendar + ICS, Messages & Notifications (channels + DMs, extending the existing `notifications`/`notification_preferences`), single-event Maps, and the Case Consultation Tracker (action-items-as-tasks · Caseload Health 0–1000). All v1, all PHI-handled under the existing RLS + audit + PHI-stripped-AI conventions.

### 9.2 Maps multi-event + Esri layer manager — **enterprise-phase**
**What's deferred.** v1 seeds ONE `disaster_events` row ("Eaton Canyon Fire") with a synthetic / Catalyst-CA perimeter. `disaster_events` **multi-event** support and an admin-facing **Esri layer manager** are the enterprise build.
**Why an enterprise buyer cares.** An org running concurrent disasters across regions — or a Government-tier deployment spanning multiple events — needs multi-event modeling and self-serve layer management. A single-county nonprofit (LLP) does not.
**New dependency to call out:** **Esri ArcGIS BAA + live key.** LOCKED provider — Google Maps and Mapbox cloud are BANNED for survivor addresses because they won't sign a BAA. Plus **Catalyst California ArcGIS layer access** (confirm perimeter/DINS layers are public or secure a data-sharing agreement). Survivor coordinates are PHI — geocoding stays under BAA, never logged, never to AI.
**Effort:** M (multi-event schema + RLS) · M (layer manager UI). Vendor: Esri BAA — owner: Counsel + API.

### 9.3 Calendar external sync (Google / Microsoft 365) — **enterprise-phase**
**What's deferred.** v1 = internal auto-event projection engine + ICS feed (zero third-party account). One-way, PHI-masked push to Google Calendar / Microsoft 365 (Graph API, surfaces in Teams) is the enhancement.
**Why an enterprise buyer cares.** Enterprise staff live in Outlook/Google Calendar; pushing Vijilant deadlines into the corporate calendar is a real adoption lever — but it's also a new data-egress surface procurement will scrutinize.
**PHI posture (load-bearing for the diligence answer):** pushed event title = `"{Type} · {Case ID}"` — NO names, notes, or addresses; org events push full detail (no PHI); nothing syncs back in. Tokens encrypted at rest in `calendar_sync_accounts`.
**New dependency to call out:** **Google / Microsoft 365 OAuth + tenant admin consent.** No BAA needed (no PHI leaves), but enterprise IT must grant the connected-app consent. v1 ships without it.
**Effort:** L (two providers + token lifecycle + masking guarantees). Owner: API.

### 9.4 Net effect on the buyer read
The v6 v1 surfaces close real product-depth gaps a reviewer would otherwise flag (no team directory, no in-app messaging, no calendar, no consultation/coaching loop). The two enterprise-phase pieces are correctly sequenced behind the same kind of vendor/BAA gate as §1.1 SSO — they belong in the Enterprise + Government conversation, not the v1 build. Net: v6 strengthens the "strong foundation" line in the reviewer's closing note; it does not move the Tier 1 identity/compliance priorities below.

---

## Prioritization — what an Apple team would say if given 7 days

If we forced an elite product team to rank everything above and pick what HAS to ship before "elite enterprise SaaS" is a credible claim:

**Tier 1 — won't sell Enterprise without these:**
1. SSO/SAML (§1.1, §2.1) — pre-requisite for every Enterprise + Government deal.
2. NPP, DSAR, audit export (§3.1, §3.2, §3.3) — pre-requisite for HIPAA-adjacent procurement.
3. Status page + SLA monitoring (§4.1, §4.2) — pre-requisite for procurement checklists.
4. 2FA for staff + platform admins (§2.3, §2.4) — basic security hygiene.
5. Public BAAS.md + Vijilant's own BAA template counsel-approved (§3.8, intersects security doc).

**Tier 2 — will be asked about in a 6-month review:**
6. Public API + OpenAPI + webhooks (§5.1, §5.2, §1.4).
7. SCIM (§1.2).
8. Per-org retention policies + legal-hold (§1.7, §1.8).
9. Right to be forgotten (§3.5).
10. Onboarding tour + empty states (§6.1, §6.2).

**Tier 3 — fast-follow:**
11. Granular permissions / CM Lead first-class (§2.5).
12. Org hierarchy (§1.13).
13. Maps multi-event + Esri layer manager (§9.2) — pairs with org hierarchy for Government tier; gated on Esri BAA.
14. Calendar external sync to Google / Microsoft 365 (§9.3) — adoption lever for enterprise staff; gated on tenant admin consent.
15. Status of every other item above.

*(The v6 v1 surfaces in §9.1 are not on this list — they ship with the standard build, not as enterprise gap-closers.)*

---

## What this costs

**Engineering effort to close Tier 1:** ~3 engineer-weeks (one person, ~20 days of focused work).

**Engineering effort to close Tier 2:** ~6 engineer-weeks.

**Counsel cost:** $5–15k for NPP, BAA finalization, MSA/DPA, Terms, Privacy Policy.

**Annual external compliance cost:** SOC 2 Type II ($15–30k auditor + ~$20k engineering preparation) ; HITRUST optional ($40k+ assessor + significant prep).

**Vendor pieces** (incremental monthly):
- Better Stack / Statuspage: $29/mo
- PagerDuty: $25/user/mo (1 user)
- Intercom or equivalent: $39/mo starter
- Datadog or Honeycomb: ~$30/mo at our scale
- OpenAPI hosting (Scalar): free → $30/mo
- Total operational additions: ~$200/mo at current scale.

**Vendor pieces** (one-time):
- DNS-CAA setup: $0.
- ClamAV Lambda: $5/mo.
- S3 Object Lock storage: $5/mo at our scale.
- WorkOS (SSO/SCIM in a box, alternative to building it ourselves): starts $125/mo per active connection. For 5 Enterprise customers: ~$625/mo. **This is the highest-leverage decision in this doc** — buy WorkOS → ship SSO/SCIM in days instead of weeks.

---

## The single recommendation if H can only do one thing this week

**Buy WorkOS.** It bundles SSO (SAML + OIDC), SCIM, Directory Sync, Audit Logs, Magic Link auth, and an admin portal. Drop-in. Closes §1.1, §1.2, §2.1, §2.2, partially §3.3. ~$625/mo at 5 Enterprise customers. Free under 1M MAU for SSO alone.

Without it, the 5-Enterprise-customers-in-Year-1 line in the financial model is wishful thinking. With it, the first Enterprise deal closes in days instead of months.

---

## What an Apple-team reviewer would write at the bottom of this report

> Vijilant has a strong technical foundation, an unusually thoughtful UX language for a v1 SaaS, and a clear theory of the case. The schema is well-considered; the security backbone is real; the architecture map shows discipline.
>
> The gaps above are not a failure of vision — they're the gaps every B2B SaaS has at this stage. The question is sequence. Tier 1 is non-negotiable for the Enterprise + Government push and can be closed inside one engineering-month if SSO is bought rather than built. Tier 2 is a 90-day project.
>
> The single most important architectural decision in the next 30 days is the **buy-vs-build of identity infrastructure**. Buying it lets Vijilant ship enterprise on schedule; building it delays the first Enterprise deal by 2+ quarters.
>
> Ship Tier 1. Then turn around and run the SOC 2 Type II project in parallel with normal feature development. By the time the audit closes, the product is selling itself into deals you couldn't credibly chase today.

That's the read.
