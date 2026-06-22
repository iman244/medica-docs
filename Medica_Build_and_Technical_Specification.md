# Medica — Build & Technical Specification

A complete, self-contained specification for building Medica. It defines the product's flows as discrete steps; for each step it states **who** performs it (actor, role, permission), **what** it serves (feature), **how** it is realized (backend API and frontend component), and **which data** the API exchanges with the component.

---

## 1. What Medica is

Medica is a telehealth weight-management service. A patient signs up, is assessed for eligibility for GLP-1 therapy, subscribes, has an online doctor visit, and receives the medication through a nurse who visits their home and administers the injection. The product spans a patient app, a doctor panel, a nurse app, a pharmacy-hub panel, an admin/operations panel, a customer-service panel, and an affiliate panel.

The first launch is a supervised pilot of fewer than 50 patients. At that scale some operations are run by hand and automated later; those are tagged in the steps.

## 2. How to read this document

Each step is written as:
```
STEP-ID — description · serves F-xxx · [status]
  actor: role (scope)
  requires: permission(s)
  api  · API-ID · METHOD /path · service · what it provides
    data · ← fields the component sends · → fields the component receives
  comp · CMP-ID · Name · mfe · what it shows/does
```

- **STEP_ID** = `STEP-<month><flow-letter?>-<nn>` (e.g. `STEP-3B-07`).
- **API_ID** = `API-<service>-<nnn>`. Services: `auth, patient, prov, visit, field, pay, pharm, sub, notif, crm, rpt, gw`.
- **COM_ID** = `CMP-<mfe>-<nnn>`. Microfrontends: `PAT, DOC, NUR, PHM, ADM, CS, AFF`; shared `SHL`.
- **status** — omitted means built in our own code. Otherwise: `[int·party→M_]` third-party integration, `[manual→M_]` done by hand until the noted month.
- **data line** — `←` lists the fields the component sends (request body). `→` lists the fields the API returns to the component. **The `→` list is the serializer allowlist for that step**: any entity field not listed is intentionally withheld (sensitive or unused). Field names come from the entities in the companion data model; an entity shown as `entity{ … }` means a row/object of that entity with the listed fields. `(derived)` marks a read model computed from events rather than a stored row.

### Process diagrams (BPMN)

The flows below are also drawn as end-to-end BPMN swimlanes; each diagram task is labelled with the `STEP-`/`API-` id it depicts. See the diagram alongside the spec when implementing a flow:

- `P1_onboarding_eligibility_subscription.bpmn` — Month 1–3 sign-up → eligibility → subscribe (`STEP-1A-*`, `STEP-2-*`, `STEP-3A-*`).
- `P2_visit_external_prescription.bpmn` — Month 3 Flow B online visit + external Rx (`STEP-3B-*`, `API-VISIT-005/006`).
- `P3_home_visit_fulfillment.bpmn` — Month 4–6 dose order → nurse → settlement (`STEP-5-*`, `STEP-4A/4B-*`, `STEP-6-*`).
- `P4_nurse_safety_sequence.bpmn` — Month 4 Flow A nurse safety sequence (`STEP-4A-04..10`).
- `P5_coldchain_replacement.bpmn` / `P6_batch_recall.bpmn` — Month 5 Flow R safety-critical recoveries (`RFLOW-01`, `RFLOW-02`).
- `P7_urgent_side_effect.bpmn` — Month 4 Flow B side-effect report → escalation (`STEP-4B-08`, `RFLOW-05`).
- `P8_payment_subscription.bpmn` — Month 3 Flow A checkout + renewal (`STEP-3A-02`, `RFLOW-12`).

## 3. Architecture

**Backend** — microservices in FastAPI, kept deliberately coarse (about ten services from one shared template). Services: `auth` (identity, roles, KYC), `patient` (profile, EMR, eligibility, engagement), `prov` (doctor + nurse profiles, schedules, credentials, assignments), `visit` (online visits, prescriptions), `field` (nurse visits, injections, routing, inventory), `pay` (wallet, gateway, invoices, settlements), `pharm` (hub inventory, cold-chain), `sub` (subscriptions), `notif` (SMS/push/email), `crm` (customer service, affiliate), `rpt` (reporting/BI), `gw` (gateway/admin).

**Frontend** — microfrontends via Module Federation, composed by a host `shell`. Independently deployable remotes: `mfe-patient`, `mfe-doctor`, `mfe-nurse`, `mfe-pharmacy`, `mfe-admin`, `mfe-cs`, `mfe-affiliate`, and `mfe-marketing` (the **public** landing/marketing site — pre-auth, SSG/SSR-friendly for SEO). Shared packages: `pkg-design` (design system), `pkg-auth` (session SDK), `pkg-api` (typed clients), `pkg-i18n` (locale resolution, per-remote + shared message catalogs, RTL/LTR direction switching, and Jalali/Gregorian + numeral/currency formatting; ships `fa-IR` source + `en`, extensible by config — see `Medica_Internationalization.md`). Components call `pkg-api` → gateway → owning service; never a service directly.

**DevOps** — Kubernetes (ArvanCloud), GitLab CI + ArgoCD, logging to Loki, metrics to VictoriaMetrics with Grafana, tracing via OpenTelemetry — baked into the service template so every service is observable from its first deploy. Each deployable exposes health probes for Kubernetes: FastAPI services from the shared template, and each Next.js frontend via `GET /api/health` (liveness) + `GET /api/health/ready` (readiness) — unauthenticated, PHI-free, defined in `Medica_Site_Map.md`.

**Cross-cutting (built Month 1, used by every step):** `CMP-SHL-001 AppShell`, `CMP-SHL-002 SessionProvider`, `CMP-SHL-003 DesignSystem`, `CMP-SHL-006 LocaleProvider`, `CMP-SHL-007 LocaleSwitcher`; `API-GW-001` (gateway, incl. locale resolution), `API-AUTH-008 GET /auth/me`, `API-AUTH-010 POST /auth/token/refresh`. The `pkg-i18n` framework, the `fa-IR` source catalog, and `identity.preferred_locale` are foundational in Month 1 so every later screen is authored against message keys from day one (`Medica_Internationalization.md`).

**Data model.** Each microservice owns its own database (no cross-service foreign keys; services reference each other by `identity_id` or resource UUID). The persistent entities behind these APIs are defined in the companion `Medica_Data_Model.md`; the `data ·` lines below draw their field names from it. The states behind every `status` field, their legal transitions, and each state's UI treatment are defined in the companion `Medica_Lifecycle_State_Machines.md`.

## 4. Access model

### Identity and roles
One **identity** is one login (one phone number + password). An identity can hold **one or more roles**, and effective permissions are the union of all its roles. Supported cases include: an operations manager who is also a patient (`[ops_manager, patient]`), a doctor who uses the service themselves (`[doctor, patient]`), or a nurse who also handles support (`[nurse, cs_agent]`).

The JWT carries `sub` (identity id), `roles[]`, and the resolved `perms[]`. An optional `act` claim records which role the user is currently acting as for UI purposes; it never widens access beyond `perms[]`.

### Roles
`patient` · `doctor` · `nurse` · `pharmacy` (hub operator) · `cs_agent` (customer service) · `affiliate` · `ops_manager` · `finance` · `admin`.

### Permissions: `resource:action:scope`
- **resource** — patient, profile, eligibility, subscription, wallet, visit, prescription, nurse_visit, route, inventory, pharmacy, provider, assignment, finance, settlement, report, ops, content, support, cs, affiliate, config, role.
- **action** — read, write, create, conduct, issue, submit, run, manage.
- **scope** — `self` (only the authenticated subject's own data) · `assigned` (data linked to the actor by an assignment) · `any` (all data; admin/ops).

### Path conventions
- **`/me/...`** operates on the authenticated subject. The id comes from the JWT `sub` and is never in the path. Guarded by a `:self` permission. Used whenever a user acts on their own data.
- **`/{type}/{id}/...`** operates on another subject. The id is in the path. Guarded by a `:assigned` permission (with an assignment check) or `:any`. Used by providers and admins.

A single logical operation may be exposed at both routes, sharing one API_ID handler; each step shows the route and permission that apply to its actor.

### Enforcement
1. The **gateway** (`API-GW-001`) validates the JWT and checks the route's required permission is present in `perms[]`.
2. The **service** enforces scope: `self` requires the subject to equal `sub`; `assigned` looks up the assignment table; `any` is allowed.
3. Every cross-subject access is **audited** (`S-011`).

### Role → permission catalog (representative)

| Role | Key permissions |
|---|---|
| `patient` | `profile:write:self`, `eligibility:write:self`, `subscription:write:self`, `wallet:write:self`, `visit:create:self`, `visit:read:self`, `nurse_visit:read:self`, `content:read` |
| `doctor` | `patient:read:assigned`, `visit:conduct:assigned`, `prescription:issue:assigned`, `prescription:submit:assigned`, `finance:read:self` |
| `nurse` | `nurse_visit:write:assigned`, `route:read:self`, `inventory:write:self`, `finance:read:self` |
| `pharmacy` | `pharmacy:write:self`, `inventory:write:self`, `finance:read:self` |
| `cs_agent` | `cs:manage:any`, `support:write:any`, `patient:read:any` |
| `affiliate` | `affiliate:read:self`, `affiliate:write:self` |
| `ops_manager` | `ops:manage:any`, `assignment:write:any`, `provider:manage:any`, `report:read:any` |
| `finance` | `finance:read:any`, `settlement:run:any` |
| `admin` | `config:manage:any`, `role:manage:any`, plus all `:any` |

A `:self` permission only ever exposes that identity's own data, via `/me`. Cross access requires `:assigned` or `:any`. Roles are assigned at `STEP-4C-04`; assignments powering `:assigned` are created at `STEP-4C-10`.

---

# Month 1 — Foundation + identity

**STEP-1-01 — Enter phone + password, request OTP** · serves F-101
- actor: anonymous (pre-auth) · requires: none (public)
- api · `API-AUTH-001` · POST /auth/register · auth · create account, trigger OTP
  - data · ← `phone, password` · → `identity{ id, phone, phone_verified }, otp_sent`
- comp · `CMP-PAT-001` · SignUpForm · mfe-patient · phone + password + request OTP

**STEP-1-02 — Verify phone OTP** · serves F-101
- actor: anonymous (pre-auth) · requires: none (OTP-bound)
- api · `API-AUTH-002` · POST /auth/otp/verify · auth · verify OTP, issue tokens
  - data · ← `phone, otp_code` · → `tokens{ access, refresh }, identity{ id, phone_verified }`
- comp · `CMP-PAT-001` · SignUpForm · mfe-patient · OTP entry

**STEP-1-03 — Log in (phone + password)** · serves F-102
- actor: anonymous (pre-auth) · requires: none (credential check)
- api · `API-AUTH-003` · POST /auth/login · auth · phone + password → tokens
  - data · ← `phone, password` · → `tokens{ access, refresh }, identity{ id, roles }`
- comp · `CMP-PAT-002` · LoginForm · mfe-patient · phone + password

**STEP-1-04 — Log in (phone + OTP)** · serves F-102
- actor: anonymous (pre-auth) · requires: none (OTP-bound)
- api · `API-AUTH-003` · POST /auth/login · auth · phone + OTP → tokens
  - data · ← `phone, otp_code` · → `tokens{ access, refresh }, identity{ id, roles }`
- comp · `CMP-PAT-002` · LoginForm · mfe-patient · OTP toggle

**STEP-1-05 — Recover password** · serves F-103
- actor: anonymous (pre-auth) · requires: none (SMS-bound)
- api · `API-AUTH-004/005` · POST /auth/password/reset/{request,confirm} · auth · SMS reset + set password
  - data · ← `phone` (request) / `reset_code, new_password` (confirm) · → `ok`
- comp · `CMP-PAT-003` · PasswordResetFlow · mfe-patient · request + confirm

**STEP-1-07 — Verify identity (KYC)** · serves F-105 · [int·Shahkar]
- actor: patient (self) · requires: `profile:write:self`
- api · `API-AUTH-007` · POST /me/kyc/verify · auth · submit national ID, return status
  - data · ← `national_id` · → `kyc_verification{ status, verified_at }`  *(national_id is never returned)*
- comp · `CMP-PAT-004` · KycVerification · mfe-patient · national-ID capture + result

## Public landing (marketing) · actor: **anonymous (public)**

**STEP-1-08 — Landing home** · serves F-095 · [public]
- actor: anonymous (public) · requires: none (public)
- api · (none for chrome; marketing copy is localized static/SSG. Editable sections may later read a public content endpoint.)
- comp · `CMP-MKT-001` · LandingHome · mfe-marketing · hero, "how it works," section composition, CTAs into signup; sections are config-gated (`marketing.enabled_sections`)

**STEP-1-09 — BMI calculator** · serves F-096 · [public]
- actor: anonymous (public) · requires: none (public)
- api · (none — computed client-side; nothing persisted)
- comp · `CMP-MKT-002` · BmiCalculator · mfe-marketing · height/weight → BMI instantly; no storage; CTA to the eligibility check / signup

Security configuration S-001–S-018 is platform-level and has no step or UI.

---

# Month 2 — Patient profile + eligibility
*Actor for all: **patient (self)**; paths are `/me/*`, subject from JWT.*

**STEP-2-01 — Complete basic profile** · serves F-110
- actor: patient (self) · requires: `profile:write:self`
- api · `API-PATIENT-001` · PUT /me/profile · patient · save profile
  - data · ← `full_name, birth_date, gender` · → `patient_profile{ id, full_name, birth_date, gender }`
- comp · `CMP-PAT-010` · ProfileForm · mfe-patient · basic profile

**STEP-2-02 — Enter health data, see BMI** · serves F-111
- actor: patient (self) · requires: `profile:write:self`
- api · `API-PATIENT-002` · PUT /me/health · patient · save measurement, compute BMI
  - data · ← `height_cm, weight_kg` · → `health_record{ height_cm, weight_kg, bmi, recorded_at }`  *(bmi is computed, read-only)*
- comp · `CMP-PAT-011` · HealthDataForm · mfe-patient · height/weight + BMI

**STEP-2-03 — Enter medical history** · serves F-112
- actor: patient (self) · requires: `profile:write:self`
- api · `API-PATIENT-003` · PUT /me/history · patient · save history
  - data · ← `conditions[], allergies[], current_meds[]` · → `medical_history{ conditions, allergies, current_meds }`  *(arrays → badge lists)*
- comp · `CMP-PAT-012` · MedicalHistoryForm · mfe-patient · history

**STEP-2-04 — Upload lab results** · serves F-113 · [int·ArvanCloud]
- actor: patient (self) · requires: `profile:write:self`
- api · `API-PATIENT-004` · POST /me/labs · patient · store lab file
  - data · ← `file, kind` · → `lab_result{ id, file_ref, kind, uploaded_at }`
- comp · `CMP-PAT-013` · LabUploader · mfe-patient · upload + thumbnails

**STEP-2-05 — Set delivery address** · serves F-115
- actor: patient (self) · requires: `profile:write:self`
- api · `API-PATIENT-005` · PUT /me/address · patient · save delivery address
  - data · ← `label, line, is_default` · → `address{ id, label, line, geo_lat, geo_lng, is_default }`  *(geo set later by geocode)*
- comp · `CMP-PAT-014` · AddressForm · mfe-patient · address entry

**STEP-2-06 — Answer eligibility questionnaire** · serves F-121
- actor: patient (self) · requires: `eligibility:write:self`
- api · `API-PATIENT-020` · GET /eligibility/questionnaire · patient · question set
  - data · → `question[]{ id, text, type, options }`
- comp · `CMP-PAT-020` · EligibilityWizard · mfe-patient · questionnaire

**STEP-2-07 — Compute eligibility, show result** · serves F-122, F-123, F-125, F-126
- actor: patient (self) · requires: `eligibility:write:self`
- api · `API-PATIENT-021` · POST /me/eligibility/answers · patient · rule engine + persist
  - data · ← `answers` · → `eligibility_assessment{ result, recommendation, assessed_at }`  *(`result` is a **categorical** outcome from the clinical ruleset — `eligible/borderline/ineligible`, `[clinical]`; there is **no numeric score**)*
- comp · `CMP-PAT-021` · EligibilityResult · mfe-patient · result + recommendation

**STEP-2-08 — Sign informed consent** · serves F-124
- actor: patient (self) · requires: `eligibility:write:self`
- api · `API-PATIENT-022` · POST /me/consent · patient · record consent
  - data · ← `type, version, accept` · → `consent{ id, type, version, signed_at }`
- comp · `CMP-PAT-022` · ConsentForm · mfe-patient · consent capture

**STEP-2-09 — Receive confirmations** · serves F-205 · [int·Kavenegar/Pushe]
- actor: system → patient · requires: n/a (internal)
- api · `API-NOTIF-001/002` · POST /notify/{sms,push} · notif · send message
  - data · → `notification{ id, channel, status }`
- comp · `CMP-SHL-005` · NotificationCenter · shell · in-app + push

**STEP-2-10 — Public eligibility check (preliminary)** · serves F-097 · `[clinical]` · [public]
- actor: anonymous (public) · requires: none (public)
- api · `API-PATIENT-023` · GET /public/eligibility/preview-questionnaire · patient · public question subset (localized)
  - data · → `question[]{ id, text, type, options }`
- api · `API-PATIENT-024` · POST /public/eligibility/preview · patient · anonymous preliminary indicator
  - data · ← `answers` · → `(derived){ indicator, message }`  *(non-binding; **nothing persisted**, no PHI stored)*
- comp · `CMP-MKT-003` · EligibilityCheck · mfe-marketing · public questionnaire → preliminary indicator → CTA to sign up for the full assessment (`STEP-2-06/07`)
- `[clinical]`: which questions form the public subset, the result framing, and the non-binding disclaimer are owned by the clinical-safety spec; the preview never returns a definitive verdict.


**STEP-2-11 — Patient home** · serves F-100
- actor: patient (self) · requires: `patient:read:self`
- api · `API-PATIENT-025` · GET /me/home · patient · home summary (next visit, treatment status, quick actions, alerts)
  - data · → `(derived){ next_visit, subscription_status, next_injection, alert[], quick_actions[] }`
- comp · `CMP-PAT-000` · PatientHome · mfe-patient · post-login hub (the patient's main screen; precedes notifications and other patient sections)

---

# Month 3 — Subscription, payment, online visit + doctor

## Flow A — subscribe & pay · actor: **patient (self)**

**STEP-3A-01 — View packages** · serves F-130
- actor: patient (self) · requires: `subscription:read:self`
- api · `API-SUB-001` · GET /subscriptions/packages · sub · package catalog
  - data · → `package[]{ id, tier, price_minor, included_services, discount_rules }`
- comp · `CMP-PAT-030` · PackagePicker · mfe-patient · package cards

**STEP-3A-02 — Purchase & pay** · serves F-131 · [int·ZarinPal/Venda]
- actor: patient (self) · requires: `subscription:write:self`, `wallet:write:self`
- api · `API-SUB-002` · POST /me/subscriptions · sub · create subscription
  - data · ← `package_id` · → `subscription{ id, status, renews_at }`
- api · `API-PAY-001/002` · POST /payments/{checkout,callback} · pay · gateway session + verify
  - data · ← `amount_minor, ref_type, ref_id` · → `payment{ id, status, gateway_session }, redirect_url`
- comp · `CMP-PAT-031` · CheckoutFlow · mfe-patient · gateway redirect + result

**STEP-3A-03 — Manage / stop renewal** · serves F-133
- actor: patient (self) · requires: `subscription:write:self`
- api · `API-SUB-003` · PATCH /me/subscriptions/{subId}/renewal · sub · stop/resume
  - data · ← `auto_renew` · → `subscription{ id, auto_renew, renews_at }`
- comp · `CMP-PAT-032` · SubscriptionManager · mfe-patient · renewal controls

**STEP-3A-04 — Cancel + exit interview** · serves F-139
- actor: patient (self) · requires: `subscription:write:self`
- api · `API-SUB-004` · POST /me/subscriptions/{subId}/cancel · sub · cancel + interview
  - data · ← `reasons[], free_text` · → `subscription{ id, status, cancelled_at }`
- comp · `CMP-PAT-034` · ExitInterview · mfe-patient · cancellation survey

**STEP-3A-05 — Wallet (balance, top-up, pay)** · serves F-141, F-142, F-143, F-146
- actor: patient (self) · requires: `wallet:read:self`, `wallet:write:self`
- api · `API-PAY-003/004/005/006` · GET /me/wallet, /me/wallet/transactions, POST /me/wallet/{topup,pay} · pay
  - data · ← `amount_minor` (topup/pay) · → `wallet{ balance_minor, currency }, wallet_transaction[]{ type, amount_minor, balance_after_minor, ref_type, status, created_at }`
- comp · `CMP-PAT-033` · WalletView · mfe-patient · balance + top-up + transactions

## Flow B — online visit & doctor

**STEP-3B-01 — Patient books visit** · serves F-150, F-151
- actor: patient (self) · requires: `visit:create:self`
- api · `API-VISIT-001` · GET /visits/availability · visit · open slots
  - data · → `slot[]{ id, start, end }`
- api · `API-VISIT-002` · POST /me/visits · visit · book visit
  - data · ← `slot_id, type` · → `visit{ id, scheduled_at, type, channel, status }`
- comp · `CMP-PAT-040` · VisitBooking · mfe-patient · slot picker

**STEP-3B-02 — Patient gets reminder** · serves F-152 · [int·Kavenegar/Pushe]
- actor: system → patient · requires: n/a
- api · `API-NOTIF-003` · POST /notify/schedule · notif · queue reminders
  - data · → `notification{ scheduled_at, channel }`
- comp · `CMP-PAT-041` · VisitReminderBanner · mfe-patient · banner

**STEP-3B-03 — Phone visit** · serves F-153 · [manual]
- actor: doctor ↔ patient (operational) · requires: n/a — no API/component

**STEP-3B-04 — Doctor opens their patient list** · serves F-310
- actor: doctor (assigned) · requires: `patient:read:assigned`
- api · `API-PROV-001` · GET /me/patients · prov · patients assigned to caller
  - data · → `patient_summary[]{ identity_id, full_name, status, last_visit_at, next_visit_at }` *(derived projection; see data-model §14)*
- comp · `CMP-DOC-001` · PatientList · mfe-doctor · assigned patients

**STEP-3B-05 — Doctor opens a patient profile + EMR** · serves F-312
- actor: doctor (assigned) · requires: `patient:read:assigned` (+ assignment check)
- api · `API-PATIENT-010` · GET /patients/{id} · patient · full profile + EMR
  - data · → `patient_profile{ full_name, birth_date, gender }, health_record[], medical_history{ conditions, allergies, current_meds }, lab_result[], eligibility_assessment{ result, recommendation }`
- comp · `CMP-DOC-002` · PatientProfilePanel · mfe-doctor · profile + EMR

**STEP-3B-06 — Doctor writes SOAP** · serves F-328
- actor: doctor (assigned) · requires: `visit:conduct:assigned`
- api · `API-VISIT-004` · POST /visits/{id}/soap · visit · save SOAP
  - data · ← `subjective, objective, assessment, plan` · → `soap_note{ id }`
- comp · `CMP-DOC-003` · VisitConsole · mfe-doctor · SOAP editor

**STEP-3B-07 — Doctor links external prescription** · serves F-327, F-329 · [int·RxProvider→M5]
- actor: doctor (assigned) · requires: `prescription:link:assigned`
- api · `API-VISIT-005` · POST /visits/{id}/prescription/link · visit · attach external Rx id + fetch details from the e-prescription provider
  - data · ← `external_rx_id` · → `prescription{ id, external_rx_id, items, pdf_ref, status, fetched_at }`  *(the doctor authors the Rx in the **external** prescription software using the patient's national ID — outside Medica — and pastes back the returned id; items/PDF are fetched read-only)*
- comp · `CMP-DOC-004` · PrescriptionLink · mfe-doctor · enter external Rx id + view fetched prescription

**STEP-3B-08 — Confirm prescription status** · serves F-330 · [int·RxProvider→M5]
- actor: doctor (assigned) · requires: `prescription:read:assigned`
- api · `API-VISIT-006` · GET /visits/{id}/prescription/status · visit · refresh status from the e-prescription provider
  - data · → `prescription{ status, external_ref, fetched_at }`
- comp · `CMP-DOC-004` · PrescriptionLink · mfe-doctor · status

**STEP-3B-09 — Auto-schedule next visit** · serves F-331
- actor: doctor (assigned) · requires: `visit:conduct:assigned`
- api · `API-VISIT-007` · POST /visits/{id}/next · visit · schedule next
  - data · ← `scheduled_at` · → `visit{ id, scheduled_at, status }`
- comp · `CMP-DOC-005` · NextVisitScheduler · mfe-doctor · set next

**STEP-3B-10 — Patient views & downloads prescription** · serves F-155, F-159
- actor: patient (self) · requires: `visit:read:self`
- api · `API-VISIT-003` · GET /me/visits/{visitId} · visit · own visit detail
  - data · → `visit{ id, scheduled_at, status }, prescription{ items, pdf_ref, issued_at }`
- comp · `CMP-PAT-042` · PrescriptionView · mfe-patient · show Rx + download PDF

**STEP-3B-11 — Patient raises emergency request** · serves F-157
- actor: patient (self) · requires: `visit:create:self`
- api · `API-VISIT-008` · POST /me/visits/emergency · visit · raise emergency
  - data · ← `reason` · → `visit{ id, type, status }`
- comp · `CMP-PAT-043` · EmergencyRequestButton · mfe-patient · raise emergency

Doctor scheduling F-340 `[manual→M6]`; refund F-147 `[manual→M4]`; F-301, F-313, F-316, F-154 `[deferred→M6/M7]`.

---

# Month 4 — Nurse + home visit + tracking → pilot launch

## Flow A — nurse home visit · actor: **nurse**

**STEP-4A-01 — Nurse logs in** · serves F-401
- actor: nurse (self) · requires: none (credential check) → session carries `nurse` role
- api · `API-AUTH-003` · POST /auth/login · auth · phone + password/OTP
  - data · ← `phone, password` · → `tokens, identity{ id, roles }`
- comp · `CMP-NUR-000` · NurseLogin · mfe-nurse · login

**STEP-4A-02 — Nurse dashboard & status** · serves F-405, F-407
- actor: nurse (self) · requires: `route:read:self`
- api · `API-FIELD-002` · POST /me/status · field · set own status
  - data · ← `status` · → `nurse_status{ status, updated_at }`
- comp · `CMP-NUR-001` · NurseDashboard · mfe-nurse · status + summary

**STEP-4A-03 — View route & schedule (today / tomorrow / week)** · serves F-415, F-416, F-417, F-418, F-409 · [order manual→M5]
- actor: nurse (self) · requires: `route:read:self`
- api · `API-FIELD-001` · GET /me/route?range=today|tomorrow|week · field · own visit list for the selected horizon (default `today`; navigation/ETA active for `today` only — future ranges are read-only planning views)
  - data · → `route{ range, from, to }, nurse_visit[]{ id, patient_name, address_snapshot, scheduled_at, eta, status }`
- comp · `CMP-NUR-002` · DailyRouteList · mfe-nurse · list + ETA + nav link, with a today/tomorrow/week range selector

**STEP-4A-04 — GPS check-in** · serves F-430
- actor: nurse (assigned) · requires: `nurse_visit:write:assigned`
- api · `API-FIELD-003` · POST /nurse-visits/{id}/checkin · field · GPS check-in
  - data · ← `checkin_geo` · → `nurse_visit{ id, status, checkin_at }`
- comp · `CMP-NUR-003` · VisitCheckin · mfe-nurse · check-in

**STEP-4A-05 — View patient profile + dose** · serves F-431, F-432
- actor: nurse (assigned) · requires: `patient:read:assigned`, `nurse_visit:read:assigned`
- api · `API-FIELD-004` · GET /nurse-visits/{id} · field · profile + Rx + dose
  - data · → `patient_summary{ full_name, allergies }, prescription{ items, dose, titration_step }, vial{ barcode, expiry_date }`
- comp · `CMP-NUR-004` · PatientVisitCard · mfe-nurse · profile + dose

**STEP-4A-06 — Record cold-chain temp** · serves F-433 · [manual entry; auto int·Elitech→M5]
- actor: nurse (assigned) · requires: `nurse_visit:write:assigned`
- api · `API-FIELD-005` · POST /nurse-visits/{id}/coldchain · field · record temperature
  - data · ← `temp_c, source` · → `coldchain_reading{ temp_c, breach, recorded_at }`  *(breach drives a "do not administer" block — clinical spec)*
- comp · `CMP-NUR-005` · ColdChainEntry · mfe-nurse · temp input

**STEP-4A-07 — Pre-injection assessment** · serves F-434
- actor: nurse (assigned) · requires: `nurse_visit:write:assigned`
- api · `API-FIELD-006` · POST /nurse-visits/{id}/assessment · field · vitals
  - data · ← `vitals{ … }, recent_side_effects` · → `assessment_record{ id, recorded_at }`
- comp · `CMP-NUR-006` · PreInjectionAssessment · mfe-nurse · vitals form

**STEP-4A-08 — Record injection site** · serves F-437
- actor: nurse (assigned) · requires: `nurse_visit:write:assigned`
- api · `API-FIELD-007` · POST /nurse-visits/{id}/injection · field · site, dose, vial
  - data · ← `dose, site, vial_barcode` · → `injection_record{ id, administered_at }`
- comp · `CMP-NUR-007` · InjectionRecorder · mfe-nurse · site chart + vial

**STEP-4A-09 — Patient signs** · serves F-441
- actor: nurse (assigned, captures patient signature) · requires: `nurse_visit:write:assigned`
- api · `API-FIELD-008` · POST /nurse-visits/{id}/signature · field · patient signature
  - data · ← `signature_image` · → `service_receipt{ id, confirmed_at }`
- comp · `CMP-NUR-008` · SignaturePad · mfe-nurse · signature

**STEP-4A-10 — Submit visit** · serves F-442
- actor: nurse (assigned) · requires: `nurse_visit:write:assigned`
- api · `API-FIELD-009` · POST /nurse-visits/{id}/submit · field · submit + package to finance
  - data · → `nurse_visit{ id, status }`
- comp · `CMP-NUR-009` · VisitSubmit · mfe-nurse · finalize

**STEP-4A-11 — Escalate emergency** · serves F-443 · [manual→M9]
- actor: nurse (self) · requires: `nurse_visit:write:assigned`
- api · `API-FIELD-010` · POST /nurse-visits/{id}/emergency · field · escalate
  - data · ← `reason` · → `emergency_escalation{ id, status, routed_to }`
- comp · `CMP-NUR-010` · SafetyButton · mfe-nurse · escalation

**STEP-4A-12 — Safety button** · serves F-497
- actor: nurse (self) · requires: `route:read:self`
- api · `API-FIELD-011` · POST /me/safety · field · personal safety alert
  - data · ← `geo` · → `safety_alert{ id, raised_at }`
- comp · `CMP-NUR-010` · SafetyButton · mfe-nurse · safety alert

## Flow B — patient tracks & reports · actor: **patient (self)**

**STEP-4B-01 — See schedule + nurse profile** · serves F-170, F-220
- actor: patient (self) · requires: `nurse_visit:read:self`
- api · `API-FIELD-020` · GET /me/nurse-visits · field · own schedule + nurse
  - data · → `nurse_visit[]{ id, scheduled_at, status }, provider_profile{ full_name, photo_ref, title }`
- comp · `CMP-PAT-050` · NurseVisitSchedule · mfe-patient · schedule + nurse

**STEP-4B-02 — Verify nurse identity** · serves F-221
- actor: patient (self) · requires: `nurse_visit:read:self`
- api · `API-PROV-010` · GET /me/nurse-visits/{id}/nurse-verify · prov · nurse code/photo
  - data · → `nurse_verify_token{ code, photo_ref, valid_until }`
- comp · `CMP-PAT-051` · NurseIdentityVerify · mfe-patient · verify nurse

**STEP-4B-03 — Confirm medication received** · serves F-222
- actor: patient (self) · requires: `nurse_visit:write:self`
- api · `API-FIELD-022` · POST /me/nurse-visits/{id}/confirm-receipt · field · medication receipt
  - data · ← `med_received` · → `service_receipt{ med_received, confirmed_at }`
- comp · `CMP-PAT-052` · MedReceiptConfirm · mfe-patient · confirm meds

**STEP-4B-04 — Confirm / change time** · serves F-171
- actor: patient (self) · requires: `nurse_visit:write:self`
- api · `API-FIELD-021` · POST /me/nurse-visits/{id}/reschedule · field · confirm/change time
  - data · ← `new_time` · → `nurse_visit{ scheduled_at, status }`
- comp · `CMP-PAT-053` · VisitTimeManager · mfe-patient · confirm/change

**STEP-4B-05 — Confirm service received** · serves F-174
- actor: patient (self) · requires: `nurse_visit:write:self`
- api · `API-FIELD-022` · POST /me/nurse-visits/{id}/confirm-receipt · field · service receipt
  - data · ← `signature_or_otp` · → `service_receipt{ confirmed_at }`
- comp · `CMP-PAT-054` · ServiceReceipt · mfe-patient · confirm service

**STEP-4B-06 — See injection history** · serves F-175
- actor: patient (self) · requires: `nurse_visit:read:self`
- api · `API-FIELD-023` · GET /me/injections · field · own injection history
  - data · → `injection_record[]{ administered_at, dose, site, vial_id }`
- comp · `CMP-PAT-055` · InjectionHistory · mfe-patient · history

**STEP-4B-07 — Next-injection reminder** · serves F-176 · [int·Kavenegar/Pushe]
- actor: system → patient · requires: n/a
- api · `API-NOTIF-003` · POST /notify/schedule · notif · schedule reminder
  - data · → `notification{ scheduled_at }`
- comp · `CMP-SHL-005` · NotificationCenter · shell · reminder

**STEP-4B-08 — Report side effect (urgent → doctor)** · serves F-177, F-178
- actor: patient (self) · requires: `side_effect:write:self`
- api · `API-PATIENT-030` · POST /me/side-effects · patient · report; `urgent` set by clinical ruleset `[clinical]`
  - data · ← `symptoms[], onset_at, free_text, severity, media_ref[]` · → `side_effect_report{ id, urgent, status, reported_at }`
- comp · `CMP-PAT-056` · SideEffectReporter · mfe-patient · report

**STEP-4B-09 — Track nurse live** · serves F-172 · [int·Optime→M5]
- actor: patient (self) · requires: `nurse_visit:read:self`
- api · `API-FIELD-034` · GET /me/nurse-visits/{id}/track · field · live position + ETA
  - data · → `track{ geo, eta }`
- comp · `CMP-PAT-057` · NurseTracker · mfe-patient · live map

## Flow C — minimal admin ops · actor: **admin / ops_manager (any)**

**STEP-4C-01 — List patients** · serves F-601
- actor: admin (any) · requires: `patient:read:any`
- api · `API-GW-010` · GET /patients · gw · patient list
  - data · → `patient[]{ identity_id, full_name, status, created_at }`
- comp · `CMP-ADM-001` · PatientTable · mfe-admin · list

**STEP-4C-02 — Patient detail + status** · serves F-602, F-603
- actor: admin (any) · requires: `patient:read:any`, `patient:write:any`
- api · `API-GW-011/012` · GET /patients/{id}, PATCH /patients/{id}/status · gw · profile + status
  - data · ← `status` · → `patient_profile{ … }, subscription{ status }, visit[]{ scheduled_at, status }`
- comp · `CMP-ADM-002` · PatientDetail · mfe-admin · profile + status

**STEP-4C-03 — Edit Rx protocols** · serves F-651
- actor: admin (any) · requires: `config:manage:any`
- api · `API-GW-013` · CRUD /admin/protocols · gw · Rx templates
  - data · ← `name, body, version` · → `protocol_template{ id, name, version }`
- comp · `CMP-ADM-003` · ProtocolEditor · mfe-admin · templates

**STEP-4C-04 — Manage roles & permissions** · serves F-655
- actor: admin (any) · requires: `role:manage:any`
- api · `API-GW-014` · CRUD /admin/roles, POST /identities/{id}/roles · gw · define + assign roles
  - data · ← `identity_id, role_id` · → `role[], permission[], identity_role[]{ identity_id, role_id }`
- comp · `CMP-ADM-004` · RoleManager · mfe-admin · roles + assign (multi-role)

**STEP-4C-05 — Manage API keys** · serves F-656
- actor: admin (any) · requires: `config:manage:any`
- api · `API-GW-015` · CRUD /admin/api-keys · gw · integration keys
  - data · ← `name, scopes, integration` · → `api_key{ id, name, scopes, integration }`  *(key_hash never returned; raw key shown once on create)*
- comp · `CMP-ADM-005` · ApiKeyManager · mfe-admin · keys

**STEP-4C-06 — Issue refund** · serves F-147
- actor: admin / finance (any) · requires: `finance:read:any`, `wallet:write:any`
- api · `API-PAY-007` · POST /wallet/{patientId}/refund · pay · admin-issued refund
  - data · ← `amount_minor, reason, ref_id` · → `wallet_transaction{ type, amount_minor, status }`
- comp · `CMP-ADM-006` · RefundAction · mfe-admin · issue refund

**STEP-4C-07 — Onboard provider (doctor / nurse)** · serves F-606, F-609
- actor: ops_manager / admin (any) · requires: `provider:manage:any`, `role:manage:any`
- api · `API-PROV-020` · POST /providers/onboard · prov · onboard + assign role
  - data · ← `identity_id, type, full_name, title` · → `provider_profile{ id, type, status }`
- comp · `CMP-ADM-010` · ProviderOnboarding · mfe-admin · onboard — **covers both doctors and nurses**; required in the pilot since both see/serve patients from M4

**STEP-4C-08 — Verify credentials** · serves F-403, F-404
- actor: ops_manager / admin (any) · requires: `provider:manage:any`
- api · `API-PROV-021` · POST /providers/{id}/verify · prov · credential verification
  - data · ← `credential_id, verified` · → `credential{ verified, background_check_status, verified_at }`
- comp · `CMP-ADM-011` · CredentialVerify · mfe-admin · verify

**STEP-4C-09 — Monitor side-effect reports + assign reviewer** · serves F-620
- actor: ops_manager / admin (any) · requires: `side_effect:read:any`, `side_effect:assign:any`
- api · `API-PATIENT-031` · GET /admin/side-effects · patient · list incoming reports (urgent first)
  - data · → `side_effect_report[]{ id, patient, symptoms, urgent, status, assigned_doctor_id, reported_at }`
- api · `API-PATIENT-032` · POST /admin/side-effects/{id}/assign · patient · assign a reviewing doctor (routing only)
  - data · ← `doctor_id` · → `side_effect_report{ assigned_doctor_id }`
- comp · `CMP-ADM-043` · SideEffectMonitor · mfe-admin · ops oversight of the urgent-escalation path (RFLOW-05) — **read + assign/route only, no clinical write**; the **assigned** doctor performs review & disposition

**STEP-4C-10 — Assign patient to provider** · serves F-610
- actor: ops_manager / admin (any) · requires: `assignment:write:any`
- api · `API-PROV-022` · POST /assignments · prov · create assignment (drives `:assigned`)
  - data · ← `patient_id, provider_id, provider_type` · → `assignment{ id, active, assigned_at }`
- comp · `CMP-ADM-012` · AssignmentBoard · mfe-admin · assign — needed in the pilot so ops can assign/reassign doctors and nurses to patients from M4

Until automated in Month 5, the following run by hand with no component: medication dispensing, route ordering. *(Provider onboarding + credential verification moved into Month 4 — `STEP-4C-07/08` — so doctors and nurses can be provisioned before the pilot's first visits; side-effect monitoring added as `STEP-4C-09` for pilot safety oversight.)*

---

# Month 5 — Pharmacy + smart routing + cold-chain + nurse completion

**STEP-5-01 — Pharmacy inventory dashboard** · serves F-502
- actor: pharmacy (self) · requires: `pharmacy:read:self`
- api · `API-PHARM-002` · GET /me/pharmacy/inventory · pharm · own hub inventory
  - data · → `vial[]{ id, barcode, status, batch{ drug, lot_number, expiry_date } }`
- comp · `CMP-PHM-001` · InventoryDashboard · mfe-pharmacy · inventory

**STEP-5-02 — Record supplier receipt** · serves F-503
- actor: pharmacy (self) · requires: `pharmacy:write:self`
- api · `API-PHARM-003` · POST /me/pharmacy/receipts · pharm · supplier receipt
  - data · ← `drug, lot_number, expiry_date, quantity, supplier` · → `batch{ id }, supplier_receipt{ id, received_at }`
- comp · `CMP-PHM-002` · SupplierReceipt · mfe-pharmacy · log stock

**STEP-5-03 — Monitor cold chain** · serves F-505 · [int·Elitech]
- actor: pharmacy (self) · requires: `pharmacy:read:self`
- api · `API-PHARM-005` · GET /me/pharmacy/coldchain · pharm · temp log
  - data · → `coldchain_log[]{ temp_c, breach, recorded_at }`
- comp · `CMP-PHM-004` · ColdChainMonitor · mfe-pharmacy · temp + alerts

**STEP-5-04 — Dispense to nurse** · serves F-504
- actor: pharmacy (self) · requires: `pharmacy:write:self`
- api · `API-PHARM-004` · POST /me/pharmacy/dispense · pharm · dispense (barcode + signature)
  - data · ← `vial_barcode, nurse_id, signature_ref` · → `dispense_record{ id, dispensed_at }, vial{ status }`
- comp · `CMP-PHM-003` · DispenseToNurse · mfe-pharmacy · scan + signature

**STEP-5-05 — Report damage** · serves F-506
- actor: pharmacy (self) · requires: `pharmacy:write:self`
- api · `API-PHARM-006` · POST /me/pharmacy/damage · pharm · damage report
  - data · ← `vial_id, reason` · → `damage_report{ id }`
- comp · `CMP-PHM-005` · DamageReport · mfe-pharmacy · report

**STEP-5-06 — View audit log** · serves F-515
- actor: pharmacy (self), admin (any) · requires: `pharmacy:read:self` / `report:read:any`
- api · `API-PHARM-007` · GET /me/pharmacy/audit · pharm · transaction log
  - data · → `audit[]{ action, vial_id, actor, at }`
- comp · `CMP-PHM-006` · AuditLogView · mfe-pharmacy · log

**STEP-5-07 — Geocode patient address** · serves F-701, F-702 · [int·Optime]
- actor: system/ops · requires: `ops:manage:any`
- api · `API-FIELD-030` · POST /logistics/geocode · field · address → coords
  - data · ← `address_line` · → `geo{ lat, lng }`  *(written back to patient.address.geo_lat/lng)*
- comp · (backend; consumed by `CMP-ADM-012`)

**STEP-5-08 — Generate optimized route** · serves F-703, F-704, F-705 · [int·Optime]
- actor: system → nurse · requires: `route:read:self`
- api · `API-FIELD-031/032` · GET /me/route, POST /logistics/route/push · field · route + push
  - data · → `route{ ordered_visit_ids, optime_ref }, nurse_visit[]{ id, eta }`
- comp · `CMP-NUR-020` · RouteMapOptimized · mfe-nurse · route + position

**STEP-5-09 — On-demand emergency assignment** · serves F-706 · [int·Optime]
- actor: ops_manager (any) → nurse · requires: `assignment:write:any`
- api · `API-FIELD-033` · POST /logistics/assign · field · on-demand assignment
  - data · ← `visit_id, nurse_id` · → `nurse_visit{ id, nurse_id, status }`
- comp · `CMP-NUR-023` · EmergencyAssignment · mfe-nurse · push visit

**STEP-5-10 — Live tracking + ETA** · serves F-707, F-708, F-172 · [int·Optime]
- actor: nurse (self), patient (self) · requires: `route:read:self` / `nurse_visit:read:self`
- api · `API-FIELD-034` · GET /logistics/track/{visitId} · field · live position + ETA
  - data · → `track{ geo, eta }`
- comp · `CMP-NUR-020` · RouteMapOptimized · mfe-nurse · position
- comp · `CMP-PAT-057` · NurseTracker · mfe-patient · live map + ETA

**STEP-5-11 — Nurse assigned-patient list** · serves F-450, F-451, F-452, F-453, F-454
- actor: nurse (assigned) · requires: `patient:read:assigned`
- api · `API-FIELD-040` · GET /me/patients · field · assigned patients + search
  - data · → `patient_summary[]{ identity_id, full_name, status, next_visit_at }, nurse_note[]{ body, at }`
- comp · `CMP-NUR-021` · AssignedPatients · mfe-nurse · list + search + notes

**STEP-5-12 — Nurse inventory** · serves F-460–F-465 · [int·Elitech]
- actor: nurse (self) · requires: `inventory:write:self`
- api · `API-FIELD-041` · CRUD /me/inventory · field · own vial count, restock, cold log
  - data · ← `vial_id, status` (+ `items[]` for restock) · → `nurse_inventory[]{ vial_id, status, expiry_date }, restock_request{ id, status }`
- comp · `CMP-NUR-022` · InventoryManager · mfe-nurse · count + restock + cold log

**STEP-5-13 — Reorder / reschedule** · serves F-420, F-422
- actor: nurse (self/assigned) · requires: `route:read:self`, `nurse_visit:write:assigned`
- api · `API-FIELD-001` · GET /me/route (reorder) · field · reorder/reschedule
  - data · ← `ordered_visit_ids` · → `route{ ordered_visit_ids }`
- comp · `CMP-NUR-024` · VisitReorder · mfe-nurse · reorder

**STEP-5-14 — Nurse week/month summary** · serves F-408
- actor: nurse (self) · requires: `finance:read:self`
- api · `API-PROV-023` · GET /me/summary · prov · own week/month
  - data · → `(derived){ visit_count, income_minor, period }`
- comp · `CMP-NUR-025` · NurseSummary · mfe-nurse · summary

**STEP-5-15 — Support inbox** · serves F-410
- actor: nurse (self) · requires: authenticated nurse
- api · `API-NOTIF-002` · POST /notify/push · notif · support notifications
  - data · → `notification[]{ id, payload, status, sent_at }`
- comp · `CMP-NUR-026` · SupportInbox · mfe-nurse · notifications

**STEP-5-16 — Record patient education** · serves F-440
- actor: nurse (assigned) · requires: `nurse_visit:write:assigned`
- api · `API-FIELD-042` · POST /nurse-visits/{id}/education · field · log education
  - data · ← `topics[], notes` · → `education_record{ id }`
- comp · `CMP-NUR-027` · EducationRecorder · mfe-nurse · education

**STEP-5-20 — External e-prescription integration live** · serves F-330 · [int·RxProvider]
- actor: doctor (assigned) · requires: `prescription:read:assigned`
- api · `API-VISIT-006` · GET /visits/{id}/prescription/status · visit · live fetch from the e-prescription provider
  - data · → `prescription{ status, external_ref, fetched_at }`
- comp · `CMP-DOC-004` · PrescriptionLink · mfe-doctor · status

## Flow R — safety-critical recovery (cold-chain replacement + batch recall)
*The two recovery paths that get dedicated software for the pilot; every other recovery reuses existing steps or is run by hand. Full flows in `Medica_Recovery_Flows.md` (RFLOW-01, RFLOW-02).*

**STEP-R01-04 — Open a replacement order** · serves F-433 (recovery) · `[clinical]`
- actor: ops_manager (any), or system on a cold-chain/mismatch/recall abort · requires: `ops:manage:any`
- api · `API-FIELD-050` · POST /replacements · field · open a replacement order
  - data · ← `nurse_visit_id, original_vial_id, reason` · → `replacement_order{ id, reason, status }`
- api · `API-FIELD-051` · GET /replacements · field · open-replacement queue
  - data · → `replacement_order[]{ id, nurse_visit_id, reason, status }`
- comp · `CMP-ADM-041` · ReplacementQueue · mfe-admin · queue + action open replacements
- *also raises an `incident` (gw) — must-alert. Re-dispense reuses `API-PHARM-004`; reschedule reuses `API-FIELD-033`/`API-FIELD-021`. `replacement_order` → fulfilled when the replacement visit records an injection.*

**STEP-R02-01 — Recall a batch (cascade)** · serves F-505, F-618 (recovery) · `[clinical/ops]`
- actor: admin / ops_manager (any) · requires: `pharmacy:write:any` / `ops:manage:any`
- api · `API-PHARM-010` · POST /batches/{id}/recall · pharm · set recall + cascade to vials/inventory
  - data · ← `reason` · → `recall{ id, batch_id, status }, affected_count`
- comp · `CMP-ADM-042` · RecallManager · mfe-admin · recall control + progress
- *cascade: in-hub vials → `vial.status=damaged`; dispensed/nurse-held vials → event to `field` → `nurse_inventory.status=damaged`, surfaced with a recalled flag in `CMP-NUR-022 InventoryManager`. → `batch.recall_status=recalled`.*

**STEP-R02-02 — Compute affected patients** · serves F-618 (recovery)
- actor: ops_manager (any) · requires: `ops:manage:any`
- api · `API-PHARM-011` · GET /recalls/{id}/affected · pharm · affected read model
  - data · → `(derived){ scheduled[]{ nurse_visit_id, patient_identity_id }, administered[]{ injection_id, patient_identity_id, administered_at } }`
- comp · `CMP-ADM-042` · RecallManager · mfe-admin · affected list (scheduled vs administered)
- *scheduled doses → reassign via STEP-R01-04 (reason=recall); administered doses → `incident` + doctor/CS follow-up `[clinical]`.*

---

# Month 6 — Doctor completion + financial panels + settlements

**STEP-6-01 — Doctor dashboard** · serves F-301–F-305
- actor: doctor (self) · requires: `patient:read:assigned`, `finance:read:self`
- api · `API-VISIT-020` · GET /me/dashboard · visit · own counts, NPS, alerts, income
  - data · → `(derived){ active_patients, new_this_week, nps, alert[]{ type, ref }, income_minor }`
- comp · `CMP-DOC-010` · DoctorDashboard · mfe-doctor · dashboard

**STEP-6-02 — Patient management** · serves F-311, F-313, F-314, F-315, F-317, F-319
- actor: doctor (assigned) · requires: `patient:read:assigned`, `visit:conduct:assigned`
- api · `API-VISIT-021` · GET /patients/{id}/visit-history · visit · history + nurse report
  - data · → `visit[]{ scheduled_at, type, status }, injection_record[]{ administered_at, dose }`
- api · `API-PATIENT-040` · GET /patients/{id}/charts · patient · weight/BMI series
  - data · → `health_record[]{ weight_kg, bmi, recorded_at }`
- api · `API-VISIT-022/023` · CRUD /patients/{id}/notes, POST /patients/{id}/messages · visit · notes + message
  - data · ← `body` · → `private_note{ id, body, created_at }, message{ id, sent_at }`
- comp · `CMP-DOC-011` · PatientManagement · mfe-doctor · history, charts, notes, messaging

**STEP-6-03 — Doctor scheduling** · serves F-340
- actor: doctor (self) · requires: `visit:conduct:self`
- api · `API-PROV-030` · CRUD /me/schedule · prov · own slots + block time
  - data · ← `start, end, status` · → `schedule_slot[]{ id, start, end, status }`
- comp · `CMP-DOC-012` · DoctorScheduleManager · mfe-doctor · slots

**STEP-6-00 — Doctor login** · serves F-102
- actor: anonymous (pre-app) · requires: none
- api · `API-AUTH-002/003` · POST /auth/login, /auth/otp · auth · password or OTP login
  - data · ← `phone, password` / `phone, otp` · → `session{ access, refresh }`
- comp · `CMP-DOC-000` · DoctorLogin · mfe-doctor · login — **no self-signup**; providers are admin-provisioned (`identity_role.assigned_by = admin`); the screen states "no account? contact Medica admin"

**STEP-6-09 — Doctor visits list** · serves F-301, F-313
- actor: doctor (assigned) · requires: `visit:read:assigned`
- api · `API-VISIT-026` · GET /me/visits?range=&status= · visit · the doctor's own visits (today + history)
  - data · → `visit[]{ id, patient, scheduled_at, type, status }`
- comp · `CMP-DOC-014` · DoctorVisitsList · mfe-doctor · Visits tab

**STEP-6-10 — Doctor prescriptions list** · serves F-329, F-330
- actor: doctor (assigned) · requires: `prescription:read:assigned`
- api · `API-VISIT-024` · GET /me/prescriptions · visit · the doctor's linked prescriptions (read)
  - data · → `prescription[]{ id, patient, external_rx_id, status, fetched_at }`
- comp · `CMP-DOC-015` · DoctorPrescriptionsList · mfe-doctor · Prescriptions tab

> Doctor panel chrome: `CMP-DOC-020 DoctorShell` (sidebar nav + top bar) wraps all doctor tabs — Dashboard (`STEP-6-01`), My Patients (`STEP-6-02`-linked list), Visits (`STEP-6-09`), Prescriptions (`STEP-6-10`), Time plan (`STEP-6-03`).

**STEP-6-04 — Doctor financial panel** · serves F-350–F-356, F-358
- actor: doctor (self) · requires: `finance:read:self`
- api · `API-PAY-020` · GET /me/finance · pay · own income + invoices
  - data · → `provider_earning[]{ source, amount_minor, period }, invoice[]{ period, amount_minor, pdf_ref }, bank_iban(masked)`
- comp · `CMP-DOC-013` · DoctorFinancePanel · mfe-doctor · finance

**STEP-6-05 — Nurse financial panel** · serves F-470–F-477
- actor: nurse (self) · requires: `finance:read:self`
- api · `API-PAY-021` · GET /me/finance · pay · own income, bonuses, invoices
  - data · → `provider_earning[]{ source, amount_minor, period }, invoice[]{ period, amount_minor, pdf_ref }, bank_iban(masked)`
- comp · `CMP-NUR-030` · NurseFinancePanel · mfe-nurse · finance

**STEP-6-06 — Nurse performance + hotline** · serves F-485, F-486, F-495
- actor: nurse (self) · requires: `finance:read:self`
- api · `API-PAY-022` · GET /me/performance · pay · own NPS, counts
  - data · → `(derived){ nps, visit_count, period }`
- comp · `CMP-NUR-031` · NursePerformance · mfe-nurse · performance
- comp · `CMP-NUR-032` · EmergencyHotline · mfe-nurse · hotline

**STEP-6-07 — Pharmacy commission** · serves F-507, F-508, F-512, F-513
- actor: pharmacy (self) · requires: `finance:read:self`
- api · `API-PAY-023` · GET /me/commission · pay · own commission + invoice
  - data · → `provider_earning[]{ source, amount_minor, period }, invoice[]{ period, amount_minor, pdf_ref }`
- comp · `CMP-PHM-010` · CommissionPanel · mfe-pharmacy · commission

**STEP-6-08 — Run monthly settlements** · serves F-627, F-628, F-629
- actor: finance / admin (any) · requires: `settlement:run:any`
- api · `API-PAY-024` · POST /settlements/run · pay · settle providers/pharmacies
  - data · ← `period, party_type` · → `settlement_run{ id, total_minor, status }, settlement_item[]{ party_id, amount_minor, status }`
- comp · `CMP-ADM-020` · SettlementRunner · mfe-admin · run settlements

---

# Month 7 — Patient engagement + content + extras
*Actor for all unless noted: **patient (self)**, paths `/me/*`.*

**STEP-7-01 — Adherence chart + streak** · serves F-185, F-186
- actor: patient (self) · requires: `profile:read:self`
- api · `API-PATIENT-050` · GET /me/adherence · patient · adherence + streak
  - data · → `(derived){ adherence_pct, streak_count, series[]{ period, taken } }`
- comp · `CMP-PAT-060` · AdherenceChart · mfe-patient · chart

**STEP-7-02 — Weight goal** · serves F-188
- actor: patient (self) · requires: `profile:write:self`
- api · `API-PATIENT-051` · CRUD /me/goals · patient · weight goal + progress
  - data · ← `target_weight, target_date` · → `weight_goal{ start_weight, target_weight, target_date, status }`
- comp · `CMP-PAT-061` · WeightGoal · mfe-patient · goal

**STEP-7-03 — Daily reminders** · serves F-189
- actor: patient (self) · requires: `profile:write:self`
- api · `API-PATIENT-052` · CRUD /me/reminders · patient · reminders
  - data · ← `type, schedule, enabled` · → `reminder[]{ id, type, schedule, enabled }`
- comp · `CMP-PAT-062` · RemindersSettings · mfe-patient · reminders

**STEP-7-04 — Content library** · serves F-195, F-196, F-197, F-199
- actor: patient (self) · requires: `content:read`
- api · `API-PATIENT-053` · GET /content · patient · articles, videos, FAQ, guides
  - data · → `content[]{ type, title, url, body, tags }`
- comp · `CMP-PAT-063` · ContentLibrary · mfe-patient · content

**STEP-7-05 — Foodnoise tracking** · serves F-223, F-224
- actor: patient (self) · requires: `profile:write:self`
- api · `API-PATIENT-054` · CRUD /me/foodnoise · patient · track + chart
  - data · ← `craving_level, note` · → `foodnoise_entry[]{ craving_level, logged_at }`
- comp · `CMP-PAT-064` · FoodnoiseTracker · mfe-patient · log + chart

**STEP-7-06 — Multiple addresses + weight history** · serves F-114, F-116, F-117
- actor: patient (self) · requires: `profile:write:self`
- api · `API-PATIENT-055` · CRUD /me/addresses · patient · addresses + weight history
  - data · ← `label, line, is_default` (+ `weight_kg` for manual entry) · → `address[]{ id, label, line, is_default }, health_record[]{ weight_kg, recorded_at }`
- comp · `CMP-PAT-065` · AddressBook · mfe-patient · addresses + history

**STEP-7-07 — Support chat + FAQ + emergency line** · serves F-207, F-209, F-210
- actor: patient (self) ↔ cs_agent · requires: `support:write:self` / `support:write:any`
- api · `API-CRM-001` · CRUD /me/support/chat · crm · live support
  - data · ← `body` · → `ticket{ id, status }, cs_message[]{ from_agent_id, body, at }`
- comp · `CMP-PAT-066` · SupportChat · mfe-patient · support + FAQ

**STEP-7-08 — Visit chat + survey + history PDF** · serves F-154, F-156, F-158
- actor: patient (self) · requires: `visit:read:self`
- api · `API-VISIT-025` · CRUD /me/visits/{id}/chat · visit · chat + survey + history PDF
  - data · ← `body` / `responses` · → `visit_chat_message[]{ sender_id, body, sent_at }, visit_survey{ responses }, history_pdf_ref`
- comp · `CMP-PAT-067` · VisitChat · mfe-patient · chat + survey + PDF

**STEP-7-09 — Pricing perks** · serves F-132, F-144
- actor: patient (self) · requires: `subscription:read:self`, `wallet:read:self`
- api · `API-SUB-010` · POST /me/subscriptions/discount · sub · first-month discount
  - data · ← `code` · → `discount{ percent, applies_to }`
- api · `API-PAY-025` · POST /me/wallet/cashback · pay · cashback
  - data · → `wallet_transaction{ type, amount_minor }`
- comp · `CMP-PAT-068` · PricingPerks · mfe-patient · discount + cashback

---

# Month 8 — Admin completion + finance/BI
*Actor for all: **admin / ops_manager / finance (any)**.*

**STEP-8-01 — User management** · serves F-604, F-605, F-607, F-608
- actor: admin (any) · requires: `provider:manage:any`, `patient:write:any`
- api · `API-GW-020` · CRUD /providers, PATCH /identities/{id}/status · gw · lists + activate
  - data · ← `status` · → `provider_profile[]{ identity_id, type, full_name, status }`
- comp · `CMP-ADM-030` · UserManagement · mfe-admin · users + activate

**STEP-8-02 — Operations dashboard** · serves F-615, F-616, F-617, F-618
- actor: ops_manager (any) · requires: `ops:manage:any`
- api · `API-GW-021` · GET /ops/dashboard · gw · visits, live map, cold-chain, incidents
  - data · → `(derived){ today_visits, nurse_position[]{ nurse_id, geo }, coldchain_alert[], incident[]{ type, severity, status } }`
- comp · `CMP-ADM-031` · OpsDashboard · mfe-admin · ops view

**STEP-8-03 — Reschedule tool** · serves F-619
- actor: ops_manager (any) · requires: `ops:manage:any`
- api · `API-GW-022` · POST /ops/reschedule · gw · admin reschedule
  - data · ← `visit_id, new_time` · → `nurse_visit{ scheduled_at, status }`
- comp · `CMP-ADM-032` · RescheduleTool · mfe-admin · reschedule

**STEP-8-04 — Finance dashboard** · serves F-625, F-626, F-631, F-632, F-633
- actor: finance / admin (any) · requires: `finance:read:any`
- api · `API-PAY-030` · GET /finance/dashboard · pay · finance + BI
  - data · → `(derived){ income_minor, cost_minor, ebitda_minor, mrr_minor, arr_minor }, invoice[], chargeback[]`
- comp · `CMP-ADM-033` · FinanceDashboard · mfe-admin · finance

**STEP-8-05 — Growth dashboard** · serves F-640, F-641
- actor: ops_manager / admin (any) · requires: `report:read:any`
- api · `API-RPT-001` · GET /growth · rpt · CAC/LTV, channels
  - data · → `(derived){ cac_minor, ltv_minor, channel[]{ source, count, conversion } }`
- comp · `CMP-ADM-034` · GrowthDashboard · mfe-admin · growth

**STEP-8-06 — Campaign manager** · serves F-644 · [int·Kavenegar]
- actor: ops_manager / admin (any) · requires: `ops:manage:any`
- api · `API-NOTIF-010` · POST /campaigns/sms · notif · SMS campaign
  - data · ← `name, segment, template_id` · → `campaign{ id, status, scheduled_at }`
- comp · `CMP-ADM-035` · CampaignManager · mfe-admin · campaigns

**STEP-8-07 — Settings** · serves F-652, F-653, F-657
- actor: admin (any) · requires: `config:manage:any`
- api · `API-GW-023` · CRUD /admin/settings · gw · rates, hubs, templates, and other tunables
  - data · ← `key, value, category` · → `setting[]{ key, value, category }`
- comp · `CMP-ADM-036` · SettingsPanel · mfe-admin · settings
- *the editable key catalog (type, default, guardrail bounds, editor, owning rule) is `Medica_Configuration_Registry.md`; `clinical`-category keys are write-guarded to their clinician-set safe bounds.*

**STEP-8-08 — Reports + export** · serves F-660, F-662, F-663
- actor: admin / finance (any) · requires: `report:read:any`
- api · `API-RPT-002` · GET /reports · rpt · scheduled reports, export, audit log
  - data · → `report[]{ id, name, period }, export_file_ref, audit_log[]{ action, resource_type, at }`
- comp · `CMP-ADM-037` · ReportsExport · mfe-admin · reports

---

# Month 9 — Customer service + affiliate + hardening

**STEP-9-01 — CS call console + routing** · serves F-700–F-703
- actor: cs_agent (any) · requires: `cs:manage:any`
- api · `API-CRM-010` · CRUD /cs/calls · crm · hotline, queue, routing, tickets
  - data · ← `notes, category` · → `ticket[]{ id, category, status, priority, patient_identity_id }, call_log{ id }`
- comp · `CMP-CS-001` · CallConsole · mfe-cs · queue + ticket

**STEP-9-02 to 9-07 — Six CS sub-panels** · serves F-710–F-761
- actor: cs_agent (any), scoped per panel · requires: `cs:manage:any` (+ `patient:read:any` for medical/new-patient, `finance:read:any` for payment)
- api · `API-CRM-011` · CRUD /cs/panels/{logistics|marketing|affiliate|medical|new-patient|payment} · crm · per-domain queue + actions
  - data · → `ticket[]{ id, status, priority }` + per-domain refs (e.g. `nurse_visit{ status }` for logistics, `payment{ status }` for payment)
- comp · `CMP-CS-002…007` · {Logistics,Marketing,Affiliate,Medical,NewPatient,Payment}Panel · mfe-cs · per-domain panel

**STEP-9-08 — CS messaging + KPI** · serves F-770, F-771
- actor: cs_agent (any) · requires: `cs:manage:any`
- api · `API-CRM-012` · POST /cs/message, GET /cs/kpi · crm · messaging + KPIs
  - data · ← `body` · → `cs_message[]{ from_agent_id, body, at }, (derived){ resolution_time, satisfaction }`
- comp · `CMP-CS-008` · CSDashboard · mfe-cs · KPI + messaging

**STEP-9-09 — Emergency escalation in-software** · serves F-443
- actor: nurse (self) → cs_agent · requires: `nurse_visit:write:assigned` / `cs:manage:any`
- api · `API-FIELD-010` · POST /nurse-visits/{id}/emergency · field · escalate to CS
  - data · ← `reason` · → `emergency_escalation{ id, status, routed_to }`
- comp · `CMP-NUR-010` · SafetyButton · mfe-nurse · escalation (now to CS)

**STEP-9-10 — Affiliate dashboard** · serves F-800, F-801
- actor: affiliate (self) · requires: `affiliate:read:self`
- api · `API-CRM-020` · GET /me/affiliate/dashboard · crm · own performance
  - data · → `(derived){ leads, conversions, income_minor }`
- comp · `CMP-AFF-001` · AffiliateDashboard · mfe-affiliate · dashboard

**STEP-9-11 — Referral links + conversion** · serves F-802, F-803
- actor: affiliate (self) · requires: `affiliate:write:self`
- api · `API-CRM-021` · POST /me/affiliate/links · crm · links + funnel
  - data · ← `utm` · → `referral_link{ code, utm }, referral[]{ stage, converted_at }`
- comp · `CMP-AFF-002` · ReferralLinks · mfe-affiliate · links + funnel

**STEP-9-12 — Commission + payment** · serves F-804, F-805
- actor: affiliate (self, read), finance (any, pay) · requires: `affiliate:read:self` / `settlement:run:any`
- api · `API-CRM-022` · GET /me/affiliate/commission · crm · own commission
  - data · → `affiliate_commission{ period, amount_minor, status }`
- comp · `CMP-AFF-003` · CommissionPanel · mfe-affiliate · commission

**STEP-9-13 — Affiliate onboarding + codes + audit** · serves F-807, F-808, F-809
- actor: admin (onboard, any), affiliate (codes, self) · requires: `role:manage:any` / `affiliate:write:self`
- api · `API-CRM-023` · CRUD /affiliates, /me/affiliate/codes · crm · onboarding, codes, audit
  - data · ← `identity_id` (onboard) / `code, percent` (codes) · → `affiliate{ id, status }, discount_code[]{ code, percent, valid_to }`
- comp · `CMP-AFF-004` · DiscountCodes · mfe-affiliate · codes
- comp · `CMP-ADM-040` · AffiliateOnboarding · mfe-admin · approve

**STEP-9-14 — A/B experiments** · serves Q-011, Q-012
- actor: ops_manager (any) configures, patient experiences · requires: `config:manage:any`
- (shared) `pkg-experiment` · shell · flags + variant assignment
- comp · consumed by `mfe-patient`

---

## 5. Multi-role behavior

- **Manager who is also a patient** (`[ops_manager, patient]`): in `mfe-patient` they call `/me/*` and see their own patient data (`:self` from the patient role); in `mfe-admin` they call `/patients/{id}`, `/ops/*` (`:any` from ops_manager). The two never bleed — `/me` is bound to their identity, the `{id}` routes are bound to a permission check.
- **Doctor who is also a patient** (`[doctor, patient]`): `/me/profile` returns their own health record; `/patients/{id}` returns a patient they're assigned to (assignment check). They cannot read an unassigned patient even though they hold the doctor role.
- **Nurse acting as a patient for themselves**: `/me/nurse-visits` returns visits where they are the *patient*; their nurse work lives under `/me/route` and `/nurse-visits/{id}` (assigned). The service distinguishes by which relationship the subject holds on each record.

Roles are assigned and revoked at `STEP-4C-04` (RoleManager). Assignments that power `:assigned` scope are created at `STEP-4C-10` (AssignmentBoard).

## 6. Traceability

Every screen traces in one line: `STEP → F (feature) → API (backend, with its data) + CMP (frontend)`, qualified by `actor` and `requires`. The API's `→` data list is what the component renders and the serializer's allowlist for that step; its `←` list is what the component submits. Use the STEP_ID as the unit of work in tickets and tests.

## 7. New data introduced by this layer

The data lines surfaced a small number of read shapes not present as base tables in the data model. They are **derived projections / read models**, not new stored fields, and are noted here so the data model's open-questions list stays accurate:
- `patient_summary{ identity_id, full_name, status, last_visit_at, next_visit_at }` — list projection for doctor/nurse/admin patient lists (`STEP-3B-04`, `5-11`, `4C-01`). Composed from `patient_profile` + `assignment` + `visit`.
- `nurse_note{ body, at }` — per-patient nurse notes (`STEP-5-11`, feature F-453). **This is genuinely new and should be added to the `field` service** as a `nurse_note` entity (`nurse_id`, `patient_id`, `body`). Flagged for the data model.
- `replacement_order` (`field`) and `recall` (`pharm`) — the two genuinely-new entities introduced by the safety-critical recovery layer (`STEP-R01-04`, `STEP-R02-01`); added to the data model. `API-PHARM-011`'s affected list is a `(derived)` read model, not a stored table, consistent with data-model §13. Full design in `Medica_Recovery_Flows.md`.
- `identity.preferred_locale` is added to `auth` and returned by `API-AUTH-008 GET /auth/me` (and carried as a non-authoritative JWT `locale` hint); the gateway resolves the effective locale per request. Static UI strings live in frontend message catalogs (`pkg-i18n`), not the DB; dynamic content is localized via existing `locale` columns. Full design in `Medica_Internationalization.md`.
- All `(derived)` dashboards (doctor, ops, finance, growth, performance, adherence, CS KPI) are read models populated from events, consistent with data-model §13.
