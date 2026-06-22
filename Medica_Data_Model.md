# Medica — Data Model

A standalone companion to the build specification. It defines the persistent entities behind every API, organized by the service that owns them. Each microservice has its own database; this document defines what lives in each.

---

## 1. Conventions

**Database isolation.** Every service owns a separate PostgreSQL database. There are **no foreign keys across services**. A reference to data in another service is stored as a plain id (`identity_id` or a resource UUID), validated at the application layer, and kept consistent through domain events (the event backbone is its own concern, defined separately). Foreign keys are used freely *within* a service.

**The universal user handle is `identity_id`.** `auth` owns the identity (one login). Every other service keys its per-user rows by `identity_id` — e.g. a patient's profile, a doctor's profile, and a nurse-visit all reference the same person by their `identity_id`. Internal table primary keys are per-table UUIDs; the cross-service handle is always `identity_id`.

**Common to every table (not repeated below):**
- `id` — UUID v7 (time-ordered) primary key.
- Audit columns — `created_at`, `created_by`, `updated_at`, `updated_by`.
- Soft delete — `deleted_at` (nullable; rows are never hard-deleted).

**Other rules:**
- **Money** is stored as an integer in **minor units (Rial)** with a `currency` code — never a float.
- **Timestamps** are UTC; Jalali conversion happens at presentation only.
- **Localization.** User-visible strings are never stored as literals: static UI text resolves from frontend message catalogs (`pkg-i18n`), and dynamic/admin content is stored per-locale via a `locale` column on its row (`content`, `template`, and — to be localized — the eligibility questionnaire, `consent`, package display labels, `protocol_template`). Enum *values* are locale-neutral; their labels are resolved at render. The locale model is owned by `Medica_Internationalization.md`.
- **PHI fields** (national id, health data) are encrypted at rest (`S-002`); encrypted columns are marked `(enc)`.
- **Status fields** are enums; their *allowed transitions* are defined in the lifecycle specification (`Medica_Lifecycle_State_Machines.md`) and here marked `enum (states defined in lifecycle spec)`.
- Field lists below omit the common columns above and show only entity-specific fields.

---

## 2. `auth` service

Owns identity, credentials, roles, permissions, and KYC. The source of `identity_id`.

**identity** — the account · serves F-101, F-102, F-105 · STEP-1-01..07
- fields: `phone` (text, unique), `email` (text, nullable), `password_hash` (text), `phone_verified` (bool), `status` `enum (states in lifecycle spec)`, `kyc_status` enum(unverified/pending/verified/failed), `preferred_locale` (text, BCP-47; default = `i18n.default_locale`)
- refs: 1–N to `identity_role`, `session`, `kyc_verification`

**role** — a named bundle of permissions · serves F-655
- fields: `code` enum(patient/doctor/nurse/pharmacy/cs_agent/affiliate/ops_manager/finance/admin), `name`, `description`

**permission** — a single grant · serves F-655
- fields: `code` (text, e.g. `patient:read:assigned`), `resource`, `action`, `scope` enum(self/assigned/any), `description`

**role_permission** — join (role ⇄ permission) · serves F-655
- fields: `role_id` (→ role), `permission_id` (→ permission)

**identity_role** — join (identity ⇄ role); **this is what enables one person to hold many roles** · serves F-655 · STEP-4C-04
- includes `assigned_by` (identity_id). **Providers (doctor/nurse) have no self-signup** — an admin creates the identity and assigns the role; the provider login screens state "no account? contact Medica admin."
- fields: `identity_id` (→ identity), `role_id` (→ role), `assigned_by` (identity_id), `assigned_at`, `revoked_at` (nullable)

**session** — issued token pair · serves S-005
- fields: `identity_id` (→ identity), `refresh_token_hash`, `expires_at`, `revoked` (bool), `device`

**otp_challenge** — one-time codes · serves F-101, F-102, F-103, F-104
- fields: `phone`/`identity_id`, `code_hash`, `purpose` enum(signup/login/reset), `expires_at`, `consumed` (bool)

**kyc_verification** — national-ID check · serves F-105 · [int·Shahkar]
- fields: `identity_id` (→ identity), `national_id` (text, **enc**), `shahkar_ref`, `status`, `verified_at`

---

## 3. `patient` service

Owns the patient profile, EMR, eligibility, and engagement data. Keyed by `identity_id` (the patient).

**patient_profile** · serves F-110 · STEP-2-01
- fields: `identity_id` (→ auth.identity), `full_name`, `birth_date`, `gender` enum, `primary_address_id` (→ address)

**health_record** — one row per measurement; the series is the weight history · serves F-111, F-116, F-117 · STEP-2-02, 7-06
- fields: `patient_id` (identity_id), `height_cm`, `weight_kg`, `bmi` (computed), `source` enum(self/nurse/visit), `recorded_at`

**medical_history** · serves F-112 · STEP-2-03
- fields: `patient_id`, `conditions` (jsonb), `allergies` (jsonb), `current_meds` (jsonb)

**lab_result** · serves F-113 · STEP-2-04 · [int·ArvanCloud]
- fields: `patient_id`, `file_ref` (object-storage key), `kind`, `uploaded_at`

**address** · serves F-114, F-115 · STEP-2-05, 7-06
- fields: `patient_id`, `label`, `line` (text), `geo_lat`, `geo_lng` (nullable; set by geocode), `is_default` (bool)

**eligibility_assessment** · serves F-121, F-122, F-123, F-125, F-126 · STEP-2-06, 2-07
- fields: `patient_id`, `answers` (jsonb), `result` enum(eligible/borderline/ineligible), `recommendation`, `review_status` enum(none/review_pending/in_review/resolved), `assessed_at`
- *eligibility is a **categorical determination** from the clinical ruleset (`[clinical]`), **not a numeric score** — the outcome is `eligible/borderline/ineligible` (borderline / ineligible → doctor review, RFLOW-18).*
- *`borderline`/`ineligible` set `review_status = review_pending`: the result screen shows a **"support will call you"** state with no in-app book/pay, and the patient lands back on it on every re-login until CS moves it to `in_review`/`resolved`. CS unlocks the review booking + payment off-app.*
- *the public landing eligibility check (`STEP-2-10`, F-097) is anonymous and **non-persisting** — it computes a non-binding indicator and stores no row here and no PHI; the authoritative assessment is created only after signup.*

**consent** · serves F-124 · STEP-2-08 · S-012
- fields: `patient_id`, `type`, `version`, `signed_at`, `ip`, `withdrawn_at` (nullable)

**side_effect_report** · serves F-177, F-178 · STEP-4B-08
- fields: `patient_id`, `symptoms` (jsonb: `[{ code, severity? }]` — `code` from a clinician-owned symptom vocabulary; per-item `severity` optional/future), `free_text`, `severity` enum (overall, patient self-rated), `onset_at`, `urgent` (bool), `media_ref[]` (photos, e.g. injection site) `[deferred→M5]`, `reported_via` enum(patient/nurse/cs), `status` `enum (states in lifecycle spec)`, `assigned_doctor_id` (identity_id, nullable — ops routes the report to this doctor), `reviewed_by` (identity_id, nullable), `reviewed_at`, `outcome`, `reported_at`
- *A report relates only to the **patient** — it carries no visit or dose link; a patient reports a side effect from anywhere, at any time. `urgent` is set by the clinical-safety ruleset off the symptom `code`s present (red-flag list owned by clinical spec §9), not free-typed (`[clinical]`). Per-item `severity` is intentionally optional — the jsonb shape keeps it open without building it for the pilot. The review fields (`status`, `reviewed_by`, `reviewed_at`, `outcome`) close the must-alert loop opened by `urgent=true` → RFLOW-05.*

**weight_goal** · serves F-188 · STEP-7-02
- fields: `patient_id`, `start_weight`, `target_weight`, `target_date`, `status` enum(active/met/abandoned)

**reminder** · serves F-189 · STEP-7-03
- fields: `patient_id`, `type` enum(water/activity/sleep/injection), `schedule` (cron/rrule), `enabled` (bool)

**foodnoise_entry** · serves F-223, F-224 · STEP-7-05
- fields: `patient_id`, `craving_level` (int), `note`, `logged_at`

**content** — catalog, not per-patient · serves F-195, F-196, F-197, F-199 · STEP-7-04
- fields: `type` enum(article/video/faq/guide), `title`, `body`/`url`, `locale`, `tags` (jsonb)
- *open question: content may warrant its own CMS service later (see §13).*

> `adherence` (F-185, F-186) is **derived** (computed from injection_record + reminders) — exposed as a read model, not a base table.

---

## 4. `prov` service

Owns provider (doctor + nurse) profiles, credentials, schedules, and the assignment table.

**provider_profile** · serves F-358, F-403, F-404, F-477, F-606, F-609 · STEP-4C-07
- fields: `identity_id` (→ auth.identity), `type` enum(doctor/nurse), `full_name`, `title`/`specialty`, `photo_ref`, `bio`, `status` `enum (states in lifecycle spec)`, `bank_iban` (**enc**)
- note: `bank_iban` lives here (provider data); `pay` reads it by `identity_id`.

**credential** · serves F-403, F-404 · STEP-4C-08
- fields: `provider_id` (→ provider_profile), `type` enum(license/national_id_card), `file_ref`, `verified` (bool), `verified_by`, `verified_at`, `background_check_status` enum

**schedule_slot** · serves F-340, F-341 · STEP-6-03
- fields: `provider_id` (doctor), `start`, `end`, `status` enum(open/blocked/booked)

**assignment** — **powers the `:assigned` permission scope** · serves F-610 · STEP-4C-10
- fields: `patient_id` (identity_id), `provider_id` (identity_id), `provider_type` enum(doctor/nurse), `active` (bool), `assigned_by`, `assigned_at`, `unassigned_at` (nullable)
- the `:assigned` check is: a row exists where `provider_id` = caller's `sub`, `patient_id` = target, `active = true`.

**nurse_verify_token** — patient-side nurse identity check · serves F-221 · STEP-4B-02
- fields: `nurse_visit_id` (→ field, by id), `provider_id`, `code`, `photo_ref`, `valid_until`

> Provider performance/summary (F-408, F-485, F-486) are **derived** read models from visits + injections, not base tables.

---

## 5. `visit` service

Owns online visits, clinical notes, and prescriptions.

**visit** · serves F-150, F-151, F-153, F-157, F-331 · STEP-3B-01, 3B-09
- fields: `patient_id` (identity_id), `doctor_id` (identity_id), `type` enum(initial/follow_up/emergency), `scheduled_at`, `channel` enum(phone/video/chat), `status` `enum (states in lifecycle spec)`
- *Booking precedes payment: a patient with no active subscription books into `status = pending_payment`; the subscription purchase (`STEP-3A-02`) confirms it to `scheduled`. The visit stays locked (no join) while `pending_payment`. (User Flows `PF-D.U3 → R-SUBSCRIBE-GATE`.)*

**soap_note** · serves F-328 · STEP-3B-06
- fields: `visit_id` (→ visit), `subjective`, `objective`, `assessment`, `plan`, `author_id` (doctor identity_id)

**prescription** · serves F-327, F-329, F-330, F-155, F-159 · STEP-3B-07, 3B-08
- fields: `visit_id` (→ visit), `patient_id`, `doctor_id`, `external_rx_id` (id returned by the external e-prescription software), `source` enum(external), `items` (jsonb: drug, dose, **titration_step** — fetched read-only from the provider; clinical content owned by the external Rx software), `pdf_ref`, `status` `enum (states in lifecycle spec)`, `external_ref`, `fetched_at`
- *the Rx is authored in **external** prescription software (keyed by the patient's national ID, outside Medica), which returns `external_rx_id`; Medica links + fetches via `[int·RxProvider]` (`STEP-3B-07/08`). Medica does not author or submit prescriptions itself.*

**visit_rating** — patient satisfaction for a doctor visit (backs the dashboard NPS, F-303) · serves F-303 · STEP-6-01 · [deferred→M6]
- fields: `visit_id` (→ visit), `patient_id`, `score` (1–5), `comment`, `rated_at`
- *optional; added so F-303/NPS has a backing entity. Distinct from `visit_survey` (the home-service survey, F-156). Drop if NPS is not built for the pilot.*

**private_note** — doctor's private notes · serves F-317 · STEP-6-02
- fields: `patient_id`, `doctor_id`, `body`

**visit_chat_message** · serves F-154 · STEP-7-08
- fields: `visit_id`, `sender_id`, `body`, `sent_at`

**visit_survey** · serves F-156 · STEP-7-08
- fields: `visit_id`, `responses` (jsonb)

**message** — doctor → patient via app/CS · serves F-319 · STEP-6-02
- fields: `from_id`, `to_patient_id`, `channel`, `body`, `sent_at`

---

## 6. `field` service

Owns nurse visits, injections, field cold-chain, routing, nurse inventory, and safety.

**nurse_visit** · serves F-170, F-171, F-181, F-415, F-416, F-418, F-430, F-442 · STEP-4A-03..10, 4B-01
- fields: `patient_id` (identity_id), `nurse_id` (identity_id), `prescription_id` (→ visit, by id), `schedule_id` (→ injection_schedule, nullable — the standing weekly slot this visit was generated from), `scheduled_at`, `address_snapshot` (jsonb, copied from patient.address), `checkin_geo`, `checkin_at`, `status` `enum (states in lifecycle spec)`

**injection_schedule** · serves F-181 · STEP-4B-10
- fields: `patient_id` (identity_id), `weekday` enum(sat/sun/mon/tue/wed/thu/fri), `window_start` (time), `window_end` (time), `status` enum(active/paused), `created_at`, `updated_at`
- *the patient's standing **weekly** injection slot. Smart-routing generates each `nurse_visit` into this window (`STEP-5-08`) and assigns a nurse; **editing** the slot reschedules the whole future series, while a **single-session** change (`STEP-4B-04`, F-171) overrides one `nurse_visit.scheduled_at` without touching the schedule.*

**injection_record** · serves F-437, F-175 · STEP-4A-08
- fields: `nurse_visit_id` (→ nurse_visit), `patient_id`, `nurse_id`, `vial_id` (→ pharm.vial, by id), `dose`, `site` (rotation code), `administered_at`
- *the My-Injections weight chart (F-179) overlays the patient's own `health_record` weight series (F-116/F-117) onto these injection dates — no weight is stored on the injection itself.*

**assessment_record** · serves F-434 · STEP-4A-07
- fields: `nurse_visit_id`, `vitals` (jsonb), `recent_side_effects`, `recorded_at`

**coldchain_reading** — field temperature at delivery · serves F-433 · STEP-4A-06 · [auto int·Elitech→M5]
- fields: `nurse_visit_id`, `temp_c`, `source` enum(manual/datalogger), `breach` (bool), `recorded_at`
- *the "breach ⇒ do not administer" rule is defined in the clinical spec.*

**service_receipt** · serves F-174, F-222, F-441 · STEP-4A-09, 4B-03, 4B-05
- fields: `nurse_visit_id`, `signature_ref`, `otp_confirmed` (bool), `med_received` (bool), `confirmed_at`

**education_record** · serves F-440 · STEP-5-16
- fields: `nurse_visit_id`, `topics` (jsonb), `notes`, `recorded_at`

**nurse_note** — a nurse's private note about a patient · serves F-453 · STEP-5-11
- fields: `nurse_id` (identity_id), `patient_id` (identity_id), `body`, `at`

**nurse_inventory** · serves F-460, F-461, F-462, F-465 · STEP-5-12
- fields: `nurse_id` (identity_id), `vial_id` (→ pharm.vial, by id), `status` enum(in_stock/used/returned/damaged), `expiry_date`, `picked_up_at`

**restock_request** · serves F-463 · STEP-5-12
- fields: `nurse_id`, `items` (jsonb), `status` enum(requested/approved/fulfilled), `requested_at`

**route** · serves F-415, F-703, F-704, F-705 · STEP-4A-03, 5-08 · [int·Optime]
- fields: `nurse_id`, `date`, `ordered_visit_ids` (jsonb array), `optime_ref`, `status`

**nurse_status** · serves F-407 · STEP-4A-02
- fields: `nurse_id`, `status` enum(ready/en_route/in_visit/break), `updated_at`

**safety_alert** · serves F-497 · STEP-4A-12
- fields: `nurse_id`, `geo`, `raised_at`, `resolved_at` (nullable)

**emergency_escalation** · serves F-443 · STEP-4A-11, 9-09
- fields: `nurse_visit_id`, `raised_by`, `reason`, `routed_to`, `status`

**replacement_order** — tracks a dose that must be re-dispensed and re-delivered after a cold-chain breach, vial mismatch, or recall · serves F-433 (recovery) · STEP-R01-04 (see `Medica_Recovery_Flows.md`)
- fields: `nurse_visit_id` (→ nurse_visit), `original_vial_id` (→ pharm.vial, by id), `replacement_vial_id` (→ pharm.vial, by id; nullable until dispensed), `reason` enum(coldchain/mismatch/recall), `status` `enum (states in lifecycle spec)`, `opened_by` (identity_id)

> Live tracking (F-172, F-707, F-708) is proxied to Optime; only `optime_ref` is persisted (on `route`/`nurse_visit`).

---

## 7. `pharm` service

Owns the medication supply chain: hubs, batches, vials, dispensing, and hub cold-chain. The batch→vial model gives per-unit traceability and recall support.

**pharmacy_hub** · serves F-501, F-653 · STEP-5-01
- fields: `operator_identity_id` (→ auth.identity), `name`, `location` (geo), `status`

**batch** — a received lot · serves supply traceability (supports recalls)
- fields: `drug`, `lot_number`, `expiry_date`, `quantity_received`, `supplier`, `received_at`, `recall_status` enum(none/flagged/recalled)

**vial** — a single administrable unit · serves F-504, F-461 · STEP-5-04
- fields: `batch_id` (→ batch), `hub_id` (→ pharmacy_hub), `barcode` (unique), `status` enum(in_hub/dispensed/used/damaged/expired), `current_holder` (hub id or nurse identity_id)

**supplier_receipt** · serves F-503 · STEP-5-02
- fields: `hub_id`, `batch_id`, `quantity`, `received_at`

**dispense_record** · serves F-504 · STEP-5-04
- fields: `hub_id`, `vial_id` (→ vial), `nurse_id` (identity_id), `barcode_scanned`, `signature_ref`, `dispensed_at`

**coldchain_log** — hub storage temperature · serves F-505 · STEP-5-03 · [int·Elitech]
- fields: `hub_id`, `temp_c`, `source` enum(datalogger/manual), `breach` (bool), `recorded_at`

**damage_report** · serves F-506 · STEP-5-05
- fields: `vial_id`/`hub_id`, `reason`, `reported_by`, `reported_at`

**recall** — a batch recall and its cascade · serves F-505, F-618 (recovery) · STEP-R02-01 (see `Medica_Recovery_Flows.md`)
- fields: `batch_id` (→ batch), `reason`, `affected_count` (int), `status` `enum (states in lifecycle spec)`, `initiated_by` (identity_id)
- reuses existing `batch.recall_status` (none/flagged/recalled) and `nurse_inventory.status = damaged` for the cascade; adds no fields to `vial` or `nurse_inventory`.

> Inventory audit (F-515) is the per-service event log of all vial movements, surfaced as a read view.

---

## 8. `pay` service

Owns wallets, payments, invoices, settlements, and earnings. References parties by `identity_id`; reads `bank_iban` from `prov`.

**wallet** · serves F-141 · STEP-3A-05
- fields: `identity_id` (→ auth.identity), `balance_minor` (int), `currency`

**wallet_transaction** · serves F-142, F-143, F-144, F-146, F-147 · STEP-3A-05, 4C-06
- fields: `wallet_id` (→ wallet), `type` enum(topup/payment/refund/cashback/payback), `amount_minor`, `balance_after_minor`, `ref_type`, `ref_id`, `gateway_ref`, `status`

**payment** · serves F-131, F-142 · STEP-3A-02 · [int·ZarinPal/Venda]
- fields: `identity_id`, `amount_minor`, `gateway` enum(zarinpal/venda), `gateway_session`, `ref_type`, `ref_id`, `status` `enum (states in lifecycle spec)`, `paid_at`

**provider_earning** — drives finance panels · serves F-350–F-354, F-470–F-474, F-507, F-508 · STEP-6-04..07
- fields: `party_type` enum(doctor/nurse/pharmacy/affiliate), `party_id` (identity_id), `source` enum(visit/injection/commission/bonus), `amount_minor`, `period`, `ref_id`

**invoice** · serves F-356, F-476, F-513, F-632 · STEP-6-04, 8-04
- fields: `party_type`, `party_id`, `period`, `amount_minor`, `lines` (jsonb), `pdf_ref`, `status`, `issued_at`

**settlement_run** · serves F-627, F-628, F-629 · STEP-6-08
- fields: `period`, `party_type`, `total_minor`, `status`, `run_by`, `run_at`

**settlement_item** · serves F-627, F-628, F-629 · STEP-6-08
- fields: `settlement_run_id` (→ settlement_run), `party_id`, `amount_minor`, `breakdown` (jsonb), `status`

> Finance dashboards / MRR / ARR (F-625, F-626) are **derived** read models, populated from payment + subscription events.

---

## 9. `sub` service

Owns subscription packages and patient subscriptions.

**package** · serves F-130, F-132 · STEP-3A-01
- fields: `tier` enum(basic/standard/premium), `price_minor`, `included_services` (jsonb), `discount_rules` (jsonb)

**subscription** · serves F-131, F-133, F-139 · STEP-3A-02..04
- fields: `identity_id` (patient), `package_id` (→ package), `status` `enum (states in lifecycle spec)`, `auto_renew` (bool), `started_at`, `renews_at`, `cancelled_at`

**subscription_event** — lifecycle log · serves F-133, F-139
- fields: `subscription_id` (→ subscription), `type` enum(created/renewed/cancelled/lapsed/reactivated), `at`

**exit_interview** · serves F-139 · STEP-3A-04
- fields: `subscription_id`, `reasons` (jsonb), `free_text`, `at`

**discount** — first-month / promo · serves F-132 · STEP-7-09
- fields: `code`, `percent`, `applies_to`, `valid_from`, `valid_to`

---

## 10. `notif` service

Owns outbound messaging, templates, and campaigns.

**notification** · serves F-205, F-152, F-176 · STEP-2-09, 3B-02, 4B-07 · [int·Kavenegar/Pushe]
- fields: `identity_id`, `channel` enum(sms/push/email), `template_id` (→ template), `payload` (jsonb), `status` enum(queued/sent/failed), `provider_ref`, `scheduled_at`, `sent_at`

**template** · serves F-657 · STEP-8-07
- fields: `code`, `channel`, `locale`, `body`, `vars` (jsonb)

**campaign** · serves F-644 · STEP-8-06
- fields: `name`, `segment` (jsonb), `template_id`, `channel`, `status`, `scheduled_at`

---

## 11. `crm` service

Owns customer-service tickets and the affiliate program.

**ticket** · serves F-700–F-761 · STEP-9-01..07
- fields: `patient_identity_id`, `channel`, `category` enum(logistics/marketing/affiliate/medical/new_patient/payment), `priority`, `status` enum(open/in_progress/resolved/closed), `assigned_agent_id`

**call_log** · serves F-702 · STEP-9-01
- fields: `ticket_id` (→ ticket), `agent_id`, `notes`, `duration_s`, `at`

**cs_message** · serves F-770 · STEP-9-08
- fields: `ticket_id`, `from_agent_id`, `body`, `at`

**affiliate** · serves F-800, F-807 · STEP-9-10, 9-13
- fields: `identity_id` (→ auth.identity), `status`, `commission_rate`, `approved_by`

**referral_link** · serves F-802 · STEP-9-11
- fields: `affiliate_id` (→ affiliate), `code` (unique), `utm` (jsonb)

**referral** · serves F-803 · STEP-9-11
- fields: `referral_link_id` (→ referral_link), `referred_identity_id`, `stage` enum(signup/first_visit/first_injection/retained), `converted_at`

**affiliate_commission** · serves F-804, F-805 · STEP-9-12
- fields: `affiliate_id`, `period`, `amount_minor`, `status` (paid out via `pay` settlement)

**discount_code** — affiliate codes · serves F-808 · STEP-9-13
- fields: `affiliate_id`, `code`, `percent`, `valid_from`, `valid_to`

---

## 12. `gw` / config service

The gateway owns no domain data; system configuration and central audit live here.

**api_key** · serves F-656 · STEP-4C-05
- fields: `name`, `key_hash`, `scopes` (jsonb), `integration`

**protocol_template** — Rx protocols · serves F-651 · STEP-4C-03
- fields: `name`, `body` (jsonb), `version`

**setting** · serves F-652, F-653, F-657 · STEP-8-07
- fields: `key`, `value` (jsonb), `category` enum(rates/hubs/templates/auth/billing/clinical/logistics/retention/i18n/marketing)
- the full catalog of keys — type, default, guardrail bounds, who may edit, and which rule each parameterizes — is defined in `Medica_Configuration_Registry.md`. `clinical`-category keys additionally require the clinical-safety spec to set their values and safe bounds.

**incident** · serves F-618 · STEP-8-02
- fields: `type`, `severity`, `ref_type`, `ref_id`, `status`

**audit_log** — central audit · serves S-011, F-663 · STEP-8-08
- fields: `identity_id`, `action`, `resource_type`, `resource_id`, `scope`, `ip`, `at`
- fed by audit events from every service; each service also keeps its own local audit trail.

> `rpt` (reporting/BI) owns **derived read models only** — growth metrics, MRR/ARR cohorts, BI aggregates (F-625, F-626, F-640, F-641, F-660, F-662, F-663) — populated from domain events, not authoritative OLTP tables.

---

## 13. Cross-service reference map

Because databases are isolated, these references are by id only (validated in app code, kept consistent by events):

- `patient.*`, `visit.*`, `field.*`, `pay.*`, `sub.*`, `crm.*` → reference a person by **`identity_id`** (owned by `auth`).
- `field.injection_record.vial_id`, `field.nurse_inventory.vial_id` → **`pharm.vial.id`**.
- `field.nurse_visit.prescription_id` → **`visit.prescription.id`**.
- `prov.nurse_verify_token.nurse_visit_id` → **`field.nurse_visit.id`**. `patient.side_effect_report.reviewed_by` and `assigned_doctor_id` → **`identity_id`** (the only cross-references; the report itself links solely to its `patient_id`).
- `prov.assignment` (patient_id, provider_id) → both **`identity_id`**; this table is the authority for `:assigned` scope.
- `pay.*` reads `bank_iban` from **`prov.provider_profile`** by `identity_id`.
- `field.replacement_order.nurse_visit_id` → **`field.nurse_visit.id`** (same service); `original_vial_id`, `replacement_vial_id` → **`pharm.vial.id`**.
- `visit.visit_rating.visit_id` → **`visit.visit.id`** (same service); `patient_id` → **`identity_id`**.
- `pharm.recall.batch_id` → **`pharm.batch.id`**; the recall cascade flags `pharm.vial` and, by event, **`field.nurse_inventory`** (sets `status = damaged`).

## 14. Open questions to resolve before schema build

1. **Titration schedule** — `visit.prescription.items.titration_step` structure is a placeholder; its rules belong to the clinical spec (a later companion).
2. **Cold-chain breach rule** — `coldchain_reading.breach` is recorded but the "do not administer" consequence is undefined here; clinical spec.
3. **Content/CMS** — `patient.content` may graduate to its own service if editorial volume grows.
4. **Derived vs stored** — adherence, provider performance, finance dashboards, BI are modeled as derived read models; confirm whether any must be materialized tables for performance.
5. **Email** — `identity.email` is nullable (the spec made phone primary); confirm whether email is ever required.
6. **Localization** — `identity.preferred_locale` is added and `content`/`template` carry `locale`; confirm which rows still need a `locale` column (questionnaire, `consent`, package display labels, `protocol_template`) and the supported-locale set. Owned by `Medica_Internationalization.md`.
