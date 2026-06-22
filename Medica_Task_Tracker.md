# Medica — Task Tracker

Actionable tasks pulled from the open items across the specification set. Each task carries: **source** (the `ID` / doc section it comes from), **owner** (role responsible), **priority + milestone**, and **done-when** (acceptance criterion).

Priority key: **P0** = pilot blocker — must be signed off before the M4 supervised pilot treats real patients (README §6); **P1** = needed for the build month it's tied to; **P2** = can follow the pilot.

Owner key: Clinician · Product/Ops · Backend · Frontend · Designer · Platform/DevOps · Compliance/Security.

---

## 1. Clinical safety — pilot-gating (P0, before M4)

The smallest set that must be approved before real patients (Clinical Safety §16). Until each is signed off, its path runs under direct human supervision or stays disabled. The Clinical-Safety Spec is a **template awaiting the clinician** — these fill its `[clinical]` values/wording.

- [ ] **Cold-chain breach thresholds** — set temperature/time limits and breach action.
  `source: Clinical_Safety §1, §16 · drives RFLOW-01 / P5 / STEP-4A-06` · owner: Clinician · priority: **P0 (M4 gate)** · done-when: approved settings sheet signed; no dose administered outside the threshold.
- [ ] **Eligibility ruleset** — define eligible / borderline / ineligible criteria.
  `source: Clinical_Safety §3 · drives STEP-2-07, UP-PAT-05, RFLOW-18 / P1` · owner: Clinician · priority: **P0 (M4 gate)** · done-when: assessment logic + reasons signed off.
- [ ] **Titration schedule** — dose progression rules.
  `source: Clinical_Safety §6 · drives STEP-3B-07 Rx / P2` · owner: Clinician · priority: **P0 (M4 gate)** · done-when: schedule approved; no Rx issued without it.
- [ ] **Vial-match + pre-injection assessment rules** — injection safety gates.
  `source: Clinical_Safety §7–8 · drives STEP-4A-07/08, RFLOW-03 / P4` · owner: Clinician · priority: **P0 (M4 gate)** · done-when: match + assessment rules signed.
- [ ] **Urgent side-effect classification + emergency guidance** — post-injection safety.
  `source: Clinical_Safety §9 · drives STEP-4B-08, RFLOW-05 / P7` · owner: Clinician · priority: **P0 (M4 gate)** · done-when: urgency triage + guidance copy approved per locale.
- [ ] **Informed-consent content** — required to enroll.
  `source: Clinical_Safety §13 · drives STEP-2-08 / P1` · owner: Clinician + Compliance · priority: **P0 (M4 gate)** · done-when: consent text approved per locale.
- [ ] **`[clinical]` copy set** — ineligible reasons, side-effect, recall, public eligibility wording.
  `source: Clinical_Safety; Design_System §11; Unhappy_Paths §I` · owner: Clinician + i18n · priority: **P0** · done-when: per-locale (fa-IR, en) message keys filled, placeholders removed.

---

## 2. Build / engineering tasks

### 2.1 Safety-critical recovery software (the two recoveries that get code for the pilot)

- [ ] **Cold-chain replacement order** — build replacement-order flow + admin queue.
  `source: STEP-R01-04 · Build_spec Flow R (M5); RFLOW-01 / P5` · owner: Backend + Frontend · priority: P1 (M5) · done-when: `API-FIELD-050/051` + `CMP-ADM-041 ReplacementQueue` live; raises must-alert incident; order → fulfilled on replacement injection.
- [ ] **Batch recall cascade** — recall a batch and cascade to vials/inventory.
  `source: STEP-R02-01 · Build_spec Flow R (M5); RFLOW-02 / P6` · owner: Backend + Frontend · priority: P1 (M5) · done-when: `API-PHARM-010` + `CMP-ADM-042 RecallManager` cascade in-hub + held vials to `damaged`; `batch.recall_status=recalled`.
- [ ] **Recall affected-patient read model** — compute scheduled vs administered.
  `source: STEP-R02-02 · Build_spec Flow R (M5); P6` · owner: Backend · priority: P1 (M5) · done-when: `API-PHARM-011` returns derived affected list; scheduled → reassign, administered → incident + follow-up.
- [ ] **Side-effect monitor (ops oversight)** — read + assign/route, no clinical write.
  `source: CMP-ADM-043 · Build_spec M5; RFLOW-05 / P7` · owner: Frontend · priority: P1 (M4–M5) · done-when: monitor lists urgent escalations, assigns to the doctor who performs disposition.

### 2.2 Data-model fold-backs (new entities/fields surfaced by the build spec)

- [ ] **Add `nurse_note` entity** to the `field` service.
  `source: Build_spec §7 (STEP-5-11, F-453)` · owner: Backend · priority: P1 (M5) · done-when: `nurse_note{ nurse_id, patient_id, body, at }` added to data model + migration.
- [ ] **Confirm `replacement_order` + `recall` entities** in the data model.
  `source: Build_spec §7 (STEP-R01-04, STEP-R02-01)` · owner: Backend · priority: P1 (M5) · done-when: both entities present in `field` / `pharm`; affected list documented as `(derived)`.
- [ ] **Add `identity.preferred_locale`** to `auth` + expose on `/auth/me`.
  `source: Build_spec §7; Internationalization` · owner: Backend · priority: P1 · done-when: field returned by `API-AUTH-008`, carried as non-authoritative JWT `locale` hint; gateway resolves effective locale.

---

## 3. Business / clinical policy decisions (Unhappy Paths §I)

Policy calls the specs deliberately do not assume. The **mechanism** exists (config keys in Configuration Registry §5); the **value** is the decision below.

- [ ] **OTP / login limits + lockout** — set attempt caps and lockout durations.
  `source: UP-PAT-01/02 · config: auth.otp.*, auth.login.*` · owner: Product + Security · priority: P1 · done-when: values set in Configuration Registry; tested against lockout.
- [ ] **KYC retries + onboard-while-pending** — retry count and whether onboarding proceeds.
  `source: UP-PAT-03 · config: auth.kyc.max_retries, auth.kyc.allow_onboarding_while_pending` · owner: Product + Compliance · priority: P1 · done-when: policy chosen and config set.
- [ ] **Ineligible: reasons shown + paid override review** `[clinical]`.
  `source: UP-PAT-05, RFLOW-18 · config: clinical.eligibility.show_reasons, .offer_paid_review` · owner: Clinician + Product · priority: **P0 (M4 gate)** · done-when: reasons policy + whether a paid doctor-override exists, signed off.
- [ ] **Payment auto-retry + hold vs release** on failure.
  `source: UP-PAT-07 · config: billing.payment.max_auto_retries, .retry_cadence_minutes, .hold_subscription_on_failure` · owner: Product/Finance · priority: P1 (M3) · done-when: cadence + hold policy set.
- [ ] **Dunning schedule + degraded features** while `past_due`.
  `source: UP-PAT-08, RFLOW-13 · config: billing.dunning.schedule_days, .degraded_features` · owner: Product/Finance · priority: P1 · done-when: schedule + degraded set defined.
- [ ] **Grace length + honor in-flight visits** `[clinical]` on `past_due → expired`.
  `source: UP-PAT-09, RFLOW-14 · config: billing.grace_window_days, billing.expired.honor_inflight_visit` · owner: Product + Clinician · priority: P1 · done-when: grace window + in-flight rule signed.
- [ ] **Missed-visit fees** — online + home-visit fee policy.
  `source: UP-PAT-10/11, RFLOW-08 · config: billing.missed_online_visit.fee_minor, .missed_home_visit.fee_minor, .fee_cap_per_month` · owner: Product/Finance · priority: P2 · done-when: fees + monthly cap set.
- [ ] **Manual GPS-override allowance + audit**.
  `source: UP-NUR-03 / P4 · config: logistics.gps.allow_manual_override` · owner: Product/Ops · priority: P1 (M4) · done-when: allowed-or-not + audit trail decided.
- [ ] **Billable + auto-escalation abort reasons**.
  `source: UP-NUR-04 · config: billing.abort.billable_reasons` · owner: Product/Ops · priority: P1 · done-when: reason taxonomy classified billable/escalating.
- [ ] **In-flight care when an assignment is removed**.
  `source: UP-DOC-04, RFLOW-11 · (flow decision, not a single config)` · owner: Product + Clinician · priority: P2 · done-when: handover rule defined in recovery flow.
- [ ] **Recall patient-communication** for affected scheduled doses `[clinical]`.
  `source: UP-PHM-02, RFLOW-02 / P6 · config: clinical.recall.patient_comms_template_id` · owner: Clinician + Product · priority: P1 (with recall build) · done-when: comms template approved.
- [ ] **Substitution + stock-out prediction; no-capacity surfacing**.
  `source: UP-PHM-04, UP-ADM-04, RFLOW-07/09 · config: logistics.substitution.policy, .stockout.predict_days, .capacity.lookahead_days` · owner: Product/Ops · priority: P2 · done-when: policies + lookahead windows set.
- [ ] **Settlement model + refund/chargeback workflow**.
  `source: UP-ADM-01/02, RFLOW-16/17 · config: billing.settlement.partial_commit, billing.refund.allow_after_spend` · owner: Finance · priority: P2 · done-when: partial-vs-all-or-nothing + refund-after-spend chosen.
- [ ] **Commission clawback** on early churn/refund.
  `source: UP-AFF-02 · config: crm.commission.clawback_window_days, .clawback_on` · owner: Finance · priority: P2 · done-when: clawback window + triggers set.
- [ ] **Offline-queue policy** — which writes are safe to queue per surface.
  `source: UP-X-01 (→ nurse-offline spec) · (architecture decision)` · owner: Backend + Product · priority: P2 (deferred) · done-when: per-surface queue allowlist defined in offline spec.

---

## 4. PHI access policy (PHI Matrix §8)

- [ ] **CS medical-panel scope** — which clinical fields the CS sub-panel exposes.
  `source: PHI §8.1 (STEP-9-02)` · owner: Compliance + Product · priority: P1 · done-when: field set fixed (recommendation: `Dv` problem-summary, not full EMR).
- [ ] **Break-glass procedure** — logged, time-boxed unmasked read of `national_id`/`bank_iban`, or declare it never permitted.
  `source: PHI §8.2` · owner: Compliance/Security · priority: P1 · done-when: procedure documented + audited, or formally disallowed.
- [ ] **`birth_date` exposure** to CS and finance — masked vs full.
  `source: PHI §8.4` · owner: Compliance · priority: P2 · done-when: masking rule confirmed.
- [ ] **Other-provider visibility of `nurse_note` / `private_note`** — confirm author-only.
  `source: PHI §8.5` · owner: Compliance · priority: P2 · done-when: confirmed (recommendation: author-only).
- [ ] **Retention + withdrawal** — which reads are revoked when `consent.withdrawn_at` is set.
  `source: PHI §8.6` · owner: Compliance · priority: P1 · done-when: revocation/restriction matrix defined.

---

## 5. Routing & deployment (Site Map §6)

- [ ] **Path-prefix vs subdomains** — one origin with `/doctor`, `/nurse`… vs per-surface subdomains.
  `source: Site_Map §6.1` · owner: Platform/DevOps · priority: P1 (before frontend deploy) · done-when: origin model chosen; health-route location set (SCR→path map unaffected).
- [ ] **Rendering strategy per surface** — SSR/SSG for marketing/content, CSR for panels.
  `source: Site_Map §6.3` · owner: Frontend + Platform · priority: P1 · done-when: per-surface rendering confirmed.
- [ ] **Nurse login path** — keep `/nurse/login` or share one `/login` + route by role.
  `source: Site_Map §6.4 (SCR-NUR-01)` · owner: Frontend · priority: P2 · done-when: login routing decided.
  *(Site Map §6.2 patient-at-root is already resolved — no task.)*

---

## 6. Information architecture (Screen Inventory §8)

Proposed groupings the specs imply but leave to design.

- [ ] **Patient navigation grouping** — IA for the 38 patient screens (e.g. Home / Treatment / Wallet / Content / Support bottom-nav).
  `source: Screen_Inventory §8.1` · owner: Designer · priority: P1 (design handoff) · done-when: nav model agreed.
- [ ] **Onboarding wizard vs task dashboard** — `SCR-PAT-01 → 12` as linear flow or dashboard.
  `source: Screen_Inventory §8.2` · owner: Designer · priority: P1 · done-when: pattern chosen.
- [ ] **Nurse visit step-flow** — `SCR-NUR-04 → 10` as an enforced sequence (check-in → cold-chain → injection).
  `source: Screen_Inventory §8.3` · owner: Designer + Clinician · priority: P1 · done-when: enforced order confirmed against lifecycle.
- [ ] **Admin panel grouping** — sections across the 21 admin screens (patients/providers/logistics/finance/config).
  `source: Screen_Inventory §8.4` · owner: Designer · priority: P2 · done-when: section model agreed.
- [ ] **CS sub-panel routing** — how `SCR-CS-02 → 07` surface from the queue `SCR-CS-01`.
  `source: Screen_Inventory §8.5` · owner: Designer · priority: P2 · done-when: routing pattern agreed.

---

## 7. Design decisions (Design System §10)

- [ ] **Final palette + type scale** — lock §2 starting direction; confirm AA contrast.
  `source: Design_System §10.1, §2` · owner: Designer · priority: P1 (design handoff) · done-when: values locked + contrast verified.
- [ ] **Light/dark mode** — support dark or light-only for the pilot.
  `source: Design_System §10.2` · owner: Designer + Product · priority: P2 · done-when: scope decided.
- [ ] **Motion language** — where motion serves vs adds noise.
  `source: Design_System §10.4` · owner: Designer · priority: P2 · done-when: motion guidelines drafted.
- [ ] **Jalali date-picker component** — choose Persian-calendar picker, themed to tokens.
  `source: Design_System §10.5` · owner: Designer + Frontend · priority: P1 · done-when: component selected + themed.
- [ ] **Iconography + illustration set** — coherent set for empty states + marketing.
  `source: Design_System §10.6` · owner: Designer · priority: P2 · done-when: set defined.

---

## 8. Design-handoff checklist (README §7)

Confirm before handing to Figma / Claude Design. (Items already ✅ in the specs are not repeated as tasks.)

- [ ] **Final palette, type scale, IA groupings** — the designer's call (see §6, §7 above).
  `source: README §7` · owner: Designer · priority: P1 · done-when: locked.
- [ ] **`[clinical]` copy in designs** — replace placeholders once the Clinical-Safety Spec is ready.
  `source: README §7; Design_System §11` · owner: Clinician + Designer · priority: P0 (before real-patient screens) · done-when: placeholders replaced with approved copy.

---

## Summary — what gates the M4 pilot (P0)

The pilot cannot treat real patients until §1 (all 6 clinical sign-offs + `[clinical]` copy) and the two P0 policy/copy items (ineligible reasons review, `[clinical]` copy in designs) are signed off. Everything else can run manually or follow.
