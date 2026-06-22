# Medica — PHI & Sensitive-Field Access Matrix

A standalone companion to the build specification, data model, lifecycle state machines, unhappy-paths, and recovery-flows specs. It is the single authority for **which accessor may read, write, or see a masked version of every sensitive field** — protected health information (PHI), personal identifiers, location, financial data, and secrets.

It sits between two mechanisms that already exist but neither of which answers this question on its own:

- The `resource:action:scope` permission model (build spec §4) decides whether a role may touch a *resource* at all. It is coarse — per-table, not per-field.
- The per-step serializer allowlists (the `→` lists in the build spec) decide which fields a *single step* returns. They are scattered across every step and nothing reconciles them.

**This matrix is the source of truth those allowlists are derived from and tested against.** Any step's `→` list must be a subset of what this matrix permits for that step's accessor relationship. Field names come from `Medica_Data_Model.md`.

---

## 1. How to read this document

### Access levels (cell values)

| symbol | meaning |
|---|---|
| `RW` | read and write the field |
| `R` | read the full value |
| `R~` | read a **masked / partial** value (e.g. IBAN last 4, phone partially hidden) |
| `Dv` | **derived only** — sees a transformed projection, never the raw field (e.g. the nurse sees *this visit's dose*, not the full prescription) |
| `Wo` | **write-once** — may set it, never read it back (e.g. national_id, password) |
| `A` | **audit / break-glass only** — never in normal serializer output; appears only in the audit log or a logged, exceptional access |
| `—` | no access |

### Accessor relationships (columns)

Access is determined by the accessor's **relationship to the specific record**, not by which roles they hold — see §5.

- **Self** — JWT `sub` equals the record's subject; reached via `/me/*`; `:self` permissions.
- **Asg. doctor / Asg. nurse** — an `active` row in `prov.assignment` links this provider to this patient; `:assigned` scope.
- **Pharmacy** — a hub operator with no patient relationship; supply-chain data only.
- **CS** — a `cs_agent` on `:any` support scope; operational support, clinical access gated (§4.C note).
- **Finance** — `finance:read:any`; money only.
- **Ops/Admin** — `:any` scope; broad and **fully audited** (`S-011`).
- **Affiliate** — omitted from every table below: an affiliate sees only referral-funnel stage counts and their own commission, and **zero** patient PHI/PII (§4.G).

### Standing rules

- Every cross-subject read (`:assigned` or `:any`) is audited (`S-011`, `gw.audit_log`).
- `(enc)` fields are encrypted at rest (`S-002`); most are never returned in plaintext to anyone.
- Where this matrix and a step's `→` list disagree, the matrix wins and the step is corrected (§6).

---

## 2. Sensitive-data classification

| tier | examples | default posture |
|---|---|---|
| **Secret** | `password_hash`, `refresh_token_hash`, `code_hash`, `payment.gateway_session` | never exposed to anyone, any role |
| **Hard identifier (enc)** | `national_id`, `bank_iban`, `credential` files | write-once or masked; raw value never travels |
| **Clinical PHI** | health, history, labs, eligibility, SOAP, prescription, injection, vitals, side-effects | self + the assigned clinician with a care relationship; minimum-necessary for nurses |
| **Location PII** | `address.line`, `geo_*`, `nurse_visit.checkin_geo` | self + whoever must physically deliver/route |
| **Financial PII** | wallet, payments, earnings, invoices, settlements | self (own money) + finance |
| **Engagement** | weight goal, reminders, foodnoise, adherence | self + the assigned clinician; not operational roles |

---

## 3. Credentials & secrets (`auth`)

| field (entity) | Self | Asg. doctor | Asg. nurse | Pharmacy | CS | Finance | Ops/Admin |
|---|---|---|---|---|---|---|---|
| `password_hash` (identity) | — | — | — | — | — | — | — |
| `refresh_token_hash` (session) | — | — | — | — | — | — | — |
| `code_hash` (otp_challenge) | — | — | — | — | — | — | — |
| `national_id` (kyc_verification, enc) | `Wo` | — | — | — | — | — | `A` |
| `shahkar_ref` | — | — | — | — | — | — | `R`(A) |
| `kyc_status` / `status` (identity) | `R` | `R~` | `R~` | — | `R` | — | `R` |

National ID is captured once (`STEP-1-07`) and **never returned** — the build spec already notes this; the matrix ratifies it as `Wo`. Any unmasked read by an admin is break-glass and logged.

---

## 4. Patient data

### 4.A Identity & contact (`patient`, `auth`)

| field | Self | Asg. doctor | Asg. nurse | Pharmacy | CS | Finance | Ops/Admin |
|---|---|---|---|---|---|---|---|
| `phone` (identity) | `RW` | `R` | `R~` (via nurse-verify token) | — | `R` | `R~` | `R` |
| `email` (identity) | `RW` | — | — | — | `R` | — | `R` |
| `full_name` (patient_profile) | `RW` | `R` | `R` | — | `R` | `R~` | `R` |
| `birth_date` | `RW` | `R` | `R` | — | `R~` | — | `R` |
| `gender` | `RW` | `R` | `R` | — | — | — | `R` |
| `address.line`, `geo_lat/lng` (address) | `RW` | — | `R` (delivery) | — | `R` (logistics) | — | `R` |
| `preferred_locale` (identity) | `RW` | `R` | `R` | — | `R` | — | `R` |

The assigned **doctor** has no need for the home address (telehealth) → `—`; the assigned **nurse** and CS-logistics do.

### 4.B Clinical record (`patient`)

| field | Self | Asg. doctor | Asg. nurse | Pharmacy | CS | Finance | Ops/Admin |
|---|---|---|---|---|---|---|---|
| `health_record` (height/weight/bmi) | `RW` | `R` | `R~` (current only) | — | — | — | `R` |
| `medical_history` (conditions/allergies/meds) | `RW` | `R` | `Dv` (allergies + current_meds only) | — | — | — | `R` |
| `lab_result` (file_ref/kind) | `RW` | `R` | — | — | — | — | `R` |
| `eligibility_assessment.result` / `recommendation` | `Dv` | `R` | — | — | — | — | `R` |
| `eligibility_assessment.result` (+ recommendation) | `R` (own) | `R` | — | — | — | — | `R` |
| `eligibility_assessment.answers` (raw questionnaire) | `R` (own) | `R` | — | — | — | — | — |
| `consent` | `R` | `R` | — | — | `R` | — | `R` |
| `side_effect_report` (symptoms/severity/urgent) | `RW` (own report) | `RW` (review/disposition) | `Dv` (recent only) | — | — | — | `R` |
| `side_effect_report.assigned_doctor_id` (routing) | — | `R` | — | — | — | — | `RW` (ops assigns) |
| `weight_goal` / `reminder` / `foodnoise_entry` | `RW` | `R` | — | — | — | — | `R` |

The assigned nurse sees **only allergies + current dose context**, matching `STEP-4A-05` (`patient_summary{ full_name, allergies }` + `prescription{ dose }`) — not the full history, labs, or eligibility. That step is the compliant baseline for every nurse-facing `→` list.

> **CS medical sub-panel note.** `STEP-9-02` grants the CS medical/new-patient panel `patient:read:any`. Define exactly what that panel exposes: the recommendation is a `Dv` problem-summary (status, flags, last visit), **not** the full EMR. Listed as an open decision (§7).

### 4.C Visit & prescription (`visit`)

| field | Self | Asg. doctor | Asg. nurse | Pharmacy | CS | Finance | Ops/Admin |
|---|---|---|---|---|---|---|---|
| `soap_note` (S/O/A/P) | — | `RW` | — | — | — | — | `R` |
| `prescription.items` (drug/dose/titration) | `R` | `R` (fetched, read-only) | `Dv` (this visit's dose + titration) | — | — | — | `R` |
| `prescription.external_rx_id` | — | `RW` (links it) | — | — | — | — | `R` |
| `prescription.pdf_ref` | `R` | `R` | `R` | — | — | — | `R` |
| `private_note` (doctor's note) | — | `RW` (own) | — | — | — | — | `A` |
| `visit_chat_message` | `RW` (own) | `RW` | — | — | `R` | — | `R` |
| `visit_survey` | `RW` (own) | `R` | — | — | — | — | `R` |

`private_note` is the doctor's private working note — not visible to the patient, other providers, or CS; ops/admin only via audit.

### 4.D Field / injection (`field`)

| field | Self | Asg. doctor | Asg. nurse | Pharmacy | CS | Finance | Ops/Admin |
|---|---|---|---|---|---|---|---|
| `injection_record` (dose/site/vial) | `R` | `R` | `RW` | — | — | — | `R` |
| `assessment_record` (vitals) | `R~` | `R` | `RW` | — | — | — | `R` |
| `coldchain_reading` (temp/breach) | — | — | `RW` | `R` (hub) | — | — | `R` |
| `nurse_visit.address_snapshot` / `checkin_geo` | `R` (own) | — | `RW` | — | `R` (logistics) | — | `R` |
| `service_receipt.signature_ref` | `R` (own) | — | `RW` | — | — | — | `R~` |
| `nurse_note` (nurse's note about patient) | — | — | `RW` (own) | — | — | — | `A` |

---

## 5. Provider & financial data

### 5.A Provider sensitive (`prov`) — adds a **Self (provider)** column

| field | Self (provider) | Finance | Ops/Admin | everyone else |
|---|---|---|---|---|
| `bank_iban` (provider_profile, enc) | `R~` (masked) | `R~` (for settlement) | `A` | — |
| `credential.file_ref` (license / national_id_card) | `R` (own) | — | `R` (to verify) | — |
| `background_check_status` | `R~` (own: pass/fail) | — | `R` | — |

`bank_iban` is owned by `prov` and read masked by `pay` (matching `STEP-6-04/05` → `bank_iban(masked)`); the raw value never leaves `prov`.

### 5.B Financial (`pay`) — **Self (party)** = the patient or provider who owns the money

| field | Self (party) | Asg. doctor/nurse | CS | Finance | Ops/Admin |
|---|---|---|---|---|---|
| `wallet.balance` / `wallet_transaction` | `R` (own) | — | `R~` (status only) | `R` | `R` |
| `payment.gateway_session` | — | — | — | `A` | `A` |
| `payment` (amount/status) | `R` (own) | — | `R~` | `R` | `R` |
| `provider_earning` / `invoice` | `R` (own) | `R` (own) | — | `R` | `R` |
| `settlement_run` / `settlement_item` | — | — | — | `RW` | `R` |

### 5.C Affiliate (`crm`)

An affiliate sees **only**: `referral.stage` counts (aggregate funnel), `referral_link`, their own `affiliate_commission`, and their own `discount_code`. The affiliate **never** sees `referral.referred_identity_id` or any patient field — `STEP-9-11` already returns `referral[]{ stage, converted_at }` without the identity, and the matrix makes that a hard rule.

---

## 6. Reconciliation with existing steps

Auditing the build spec's `→` lists against this matrix surfaces one correction and ratifies the rest:

1. **Eligibility has no numeric score.** Earlier drafts carried an `eligibility_assessment.score` field and a rule to withhold it from the patient. Eligibility is a **categorical** determination (`eligible/borderline/ineligible`) from the clinical ruleset — there is no score. → **Applied:** the `score` field is removed from the data model, the build spec, and this matrix. `STEP-2-07` returns `result, recommendation, assessed_at`; the patient sees their own `result` + `recommendation`, the assigned doctor sees `result` + `answers`, ops sees `result`.
2. `STEP-1-07` never returns `national_id` — ratified as `Wo`.
3. `STEP-6-04/05` return `bank_iban` masked — ratified as `R~`.
4. `STEP-4A-05` gives the nurse allergies + dose only — ratified as the nurse minimum-necessary baseline.
5. `STEP-9-11` excludes `referred_identity_id` from the affiliate view — ratified as a hard rule.

> Note to the reader: every new step added to the build spec must have its `→` list checked against this matrix before merge. The serializer allowlist is the *implementation*; this matrix is the *policy*.

---

## 7. Cross-cutting rules

- **Relationship, not role.** A field's accessor is decided by the subject's relationship to *this record* — self, an `active` assignment, an operational ticket — enforced by the `/me` binding and the `assignment` table, **not** by the set of roles the identity holds. A nurse who also holds `cs_agent` cannot reach a patient's clinical record through the CS path, and a doctor+patient identity sees their own EMR via `/me` but only assigned patients via `/{id}`. This is what makes the multi-role design safe.
- **Minimum necessary.** Each relationship is granted the least it needs for its task; `Dv` and `R~` exist so a role can do its job without seeing the raw sensitive value.
- **Encrypted fields stay in their service.** `national_id`, `bank_iban`, and credential files never travel in plaintext; they are set write-once or read masked.
- **Break-glass is logged.** The only path to an unmasked hard identifier for ops/admin is an explicit, audited access (`A`), never a normal serializer field.
- **Audit everything cross-subject.** Any `:assigned`/`:any` read writes to `gw.audit_log` (`S-011`).
- **No machine translation of clinical free-text.** `preferred_locale` is non-sensitive routing data (governs which catalog/template renders), but free-text PHI — history notes, SOAP, side-effect `free_text`, nurse notes, chat — is stored and shown verbatim and is **never** machine-translated, for safety (`Medica_Internationalization.md` §2).
- **External prescription provider (egress).** Prescriptions are authored in **external** e-prescription software; the doctor uses the patient's national ID **in that external app, outside Medica**, and Medica links/fetches by `external_rx_id` via `[int·RxProvider]`. Medica does not transmit `national_id` to the provider for the fetch; if a future provider requires it, that is a new PHI egress and must be flagged and approved here.
- **Public landing inputs are anonymous and unstored.** The BMI calculator (`STEP-1-09`) and the public eligibility check (`STEP-2-10`) take height/weight/answers from an anonymous visitor, compute client-side or non-persisting results, and store **no row and no PHI** against any identity. PHI begins only after signup, when the user re-enters data under their authenticated profile (`health_record`, `eligibility_assessment`).

---

## 8. Open decisions

Genuine policy calls this document surfaces but does not assume:

1. **CS medical-panel scope** — exactly which clinical fields the CS medical/new-patient sub-panel (`STEP-9-02`, `patient:read:any`) exposes. Recommendation: a `Dv` problem-summary, not the full EMR.
2. **Break-glass procedure** — define the logged, time-boxed process by which an admin may read an unmasked `national_id`/`bank_iban`, or declare it never permitted.
4. **`birth_date` exposure** to CS and finance — confirm masked vs. full.
5. **Other-provider visibility of `nurse_note` / `private_note`** — confirm these stay author-only (recommendation: yes).
6. **Retention & withdrawal** — when `consent.withdrawn_at` is set, which of the above reads are revoked or restricted (ties to the compliance work the unhappy-paths spec defers).
