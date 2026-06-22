# Medica — Recovery Flows

A standalone companion to the build specification, data model, lifecycle state machines, and unhappy-paths spec. The unhappy-paths document (`Medica_Unhappy_Paths.md`) designs the screen a user *sees* when something goes wrong; this document designs the **resolution** — the actor-by-actor flow that gets the entity back to a good state. Where an unhappy path hands off to another actor (ops, pharmacy, doctor, finance, CS), that handoff is named here as an `RFLOW-`.

It draws every step, API, component, and entity from the same sources as the build spec: STEP ids and the `actor · role/permission · api · comp · data ← →` format come from `Medica_Build_and_Technical_Specification.md`; entities come from `Medica_Data_Model.md`; the `⚠` transitions and states come from `Medica_Lifecycle_State_Machines.md`.

Every patient- or provider-facing notification raised by a recovery flow (reschedule, dunning, recall, replacement, side-effect) is rendered in the **recipient's** `preferred_locale` via the per-locale `notif.template`; `[clinical]` recall and emergency wording is approved per locale by the clinical-safety spec (`Medica_Internationalization.md`).

---

## How to read this document

Each recovery is written as:

```
RFLOW-NN — name · resolves UP-XXX (+ ⚠ transition) · [classification] · [clinical]?
  owner:   the actor/surface that owns the resolution
  trigger: the unhappy-path entry / state transition that opens it
  steps:   the resolution as STEP-style steps (reusing or adding STEP/API/CMP ids)
  closes:  the state the entity lands in once resolved
  decision: business/clinical policy this flow needs but does not assume
```

Steps reuse the build-spec block:
```
STEP-ID — description · serves F-xxx · [status]
  actor: role (scope) · requires: permission(s)
  api  · API-ID · METHOD /path · service · what it does
    data · ← fields sent · → fields received (serializer allowlist)
  comp · CMP-ID · Name · mfe · what it shows/does
```

**Process diagrams (BPMN):** several recoveries are drawn as swimlanes — `P5_coldchain_replacement.bpmn` (`RFLOW-01`), `P6_batch_recall.bpmn` (`RFLOW-02`), `P7_urgent_side_effect.bpmn` (`RFLOW-05`), `P8_payment_subscription.bpmn` (`RFLOW-12`), and the doctor-review/Rx recoveries inside `P1_onboarding_eligibility_subscription.bpmn` (`RFLOW-18`) and `P2_visit_external_prescription.bpmn` (`RFLOW-19`). `P4_nurse_safety_sequence.bpmn` shows where `RFLOW-01`/`RFLOW-03` are triggered.

### Classification (how each recovery is delivered for the <50-patient pilot)

| tag | meaning |
|---|---|
| `[impl]` | dedicated software we write; built in the month it is placed under. Reserved for safety-critical recoveries. |
| `[manual→M_]` | resolved by a human using **existing** admin tools until the noted month, then formalized onto a tool already on the roadmap. The default for the pilot. |
| `[reuse]` | leans entirely on existing STEP/API/CMP ids already in the build spec; no new build. |

`⚠` marks a transition owned by the lifecycle spec; `[clinical]` marks a rule deferred to the future clinical-safety spec; `[int·party→M_]` marks a third-party dependency. New ids introduced here are folded back into the data model and build spec — see §6.

---

## 1. Classification & timeline summary

Every recovery, its owner, how it is delivered for the pilot, and the month its build (if any) lands. **New build is concentrated in Month 5** beside the pharmacy/vial/batch/cold-chain work; everything else reuses existing ids or is run by hand with existing admin tools.

| RFLOW | name | resolves | owner | class | month | reuse / new |
|---|---|---|---|---|---|---|
| 01 | Cold-chain breach → replacement dose | UP-NUR-01 ⚠ `[clinical]` | pharmacy + ops | `[impl]` | **M5** | **new** `API-FIELD-050/051`, `CMP-ADM-041`; reuse `API-PHARM-006/004`, `API-FIELD-033/021` |
| 02 | Batch recall → reassign affected patients | UP-PHM-02 ⚠ `[clinical/ops]` | ops + pharmacy | `[impl]` | **M5** | **new** `API-PHARM-010/011`, `CMP-ADM-042`; reuse RFLOW-01, `API-NOTIF-*`, `incident` |
| 03 | Wrong / mismatched vial | UP-NUR-02 ⚠ `[clinical]` | nurse → ops | `[reuse]` | M4 / M5 | reuse `API-FIELD-007`, RFLOW-01 |
| 04 | Hub cold-chain quarantine | UP-PHM-01 `[clinical]` | pharmacy + ops | `[manual→M5]` | M5 | reuse `API-PHARM-005/006`, `incident` |
| 05 | Urgent side-effect clinical escalation | UP-PAT-14 `[clinical]` | doctor + ops | `[reuse]` | M4 | reuse `API-PATIENT-030`, `API-VISIT-008`, `incident` |
| 06 | Nurse safety escalation | UP-NUR-05 | ops / CS | `[reuse]` | M4 (→ M9 CS) | reuse `API-FIELD-010/011` |
| 07 | No nurse available / no capacity | UP-ADM-04 | ops | `[manual→M5]` | M4 manual → M5 | reuse `API-FIELD-033` (F-706), `API-PROV-022` |
| 08 | Missed-home-visit reschedule | UP-PAT-11 / UP-NUR-04 ⚠ | ops / nurse / patient | `[manual→M8]` | M4 reuse → M8 tool | reuse `API-FIELD-021` (F-422), `API-GW-022` (F-619) |
| 09 | Stock-out / shortage at pickup | UP-PHM-04 / UP-NUR-09 | pharmacy + ops | `[manual→M5]` | M5 | reuse `API-FIELD-041`, `API-PHARM-004` |
| 10 | Patient-not-home abort | UP-NUR-04 ⚠ | nurse → ops | `[reuse]` | M4 | reuse `API-FIELD-009/010`, RFLOW-08 |
| 11 | Provider reassignment / in-flight care | UP-DOC-04 | ops | `[manual→M5]` | M5 | reuse `API-PROV-022` (AssignmentBoard) |
| 12 | Payment-failure resolution | UP-PAT-07 ⚠ | self → CS / finance | `[reuse]` | M3 | reuse `API-PAY-001/002`, `API-PAY-006` |
| 13 | Past_due dunning | UP-PAT-08 ⚠ | self → ops | `[manual→M3]` | M3 | reuse `API-PAY-006`, `API-NOTIF-003`, `API-SUB-003` |
| 14 | Expired-mid-treatment reactivation | UP-PAT-09 ⚠ `[clinical]` | ops | `[manual→M3]` | M3 | reuse `API-SUB-002`, `incident` |
| 15 | Refund / refund-pending | UP-PAT-13 / UP-ADM-02 | finance / CS | `[reuse]` | M4 | reuse `API-PAY-007` (F-147) |
| 16 | Chargeback reconciliation | UP-ADM-02 | finance | `[manual→M8]` | M8 | reuse `API-PAY-030` (F-633) |
| 17 | Settlement-run failure | UP-ADM-01 ⚠ | finance | `[reuse]` | M6 | reuse `API-PAY-024` (idempotent rerun) |
| 18 | Ineligible / borderline → paid doctor review | UP-PAT-05 / UP-PAT-06 `[clinical]` | doctor (via ops / CS) | `[manual→M3]` | M2 manual → M3 | reuse `API-VISIT-002`, `API-PAY-001` |
| 19 | Prescription rejected by TUMS → reissue | UP-DOC-01 ⚠ `[int·TUMS]` | doctor | `[reuse]` | M5 | reuse `API-VISIT-005/006` |
| 20 | KYC manual review | UP-PAT-03 `[int·Shahkar]` | ops / CS | `[manual→M9]` | M1 manual → M9 CS | reuse `API-AUTH-007`, `ticket` |

Edge / rare cases resolve as a **generic CS ticket** and are listed in §4.E without a full flow.

---

## 2. Safety-critical recoveries (build)

These are the only recoveries that get dedicated software for the pilot. Both land in Month 5 with the pharmacy/vial/batch model. During the brief supervised Month 4 pilot window — before the M5 software exists — a cold-chain breach or recall is handled by hand by ops (dispatch a replacement dose, phone affected patients); the steps below are the automation of that manual process.

### RFLOW-01 — Cold-chain breach → replacement dose · resolves UP-NUR-01 (nurse_visit → blocked ⚠) · `[impl]` · `[clinical]`
- owner: pharmacy + ops · trigger: temperature out of range at delivery; injection hard-blocked.
- closes: original `nurse_visit` → `aborted`; `replacement_order` → `fulfilled` when the replacement injection records.
- decision needed: replacement SLA (how fast a replacement must reach the patient); whether the patient is ever charged for the lost dose (default: no) `[clinical]`.

**STEP-R01-01 — Record breach** · serves F-433 · `[clinical]`
- actor: nurse (assigned) · requires: `nurse_visit:write:assigned`
- api · `API-FIELD-005` · POST /nurse-visits/{id}/coldchain · field · record temperature *(reuse)*
  - data · ← `temp_c, source` · → `coldchain_reading{ temp_c, breach, recorded_at }`
- comp · `CMP-NUR-005` · ColdChainEntry · mfe-nurse · breach blocks the injection step
- → `nurse_visit` → `blocked` (lifecycle). The threshold that sets `breach` is `[clinical]`.

**STEP-R01-02 — Report the affected vial damaged** · serves F-506 · `[clinical]`
- actor: nurse (self) · requires: `inventory:write:self`
- api · `API-PHARM-006` · POST /me/pharmacy/damage · pharm · damage report *(reuse)*
  - data · ← `vial_id, reason` · → `damage_report{ id }`
- comp · `CMP-NUR-022` · InventoryManager · mfe-nurse · mark held vial damaged
- → `vial` → `damaged`; `nurse_inventory.status` → `damaged`.

**STEP-R01-03 — Abort the blocked visit** · serves (nurse_visit → aborted ⚠)
- actor: nurse (assigned) · requires: `nurse_visit:write:assigned`
- api · `API-FIELD-009` · POST /nurse-visits/{id}/submit · field · finalize as aborted (reason = coldchain) *(reuse)*
  - data · ← `outcome=aborted, reason` · → `nurse_visit{ id, status }`
- comp · `CMP-NUR-009` · VisitSubmit · mfe-nurse · abort with reason

**STEP-R01-04 — Open a replacement order** · serves F-433 (recovery) · `[impl]`
- actor: ops_manager (any), or system on abort-reason=coldchain · requires: `ops:manage:any`
- api · `API-FIELD-050` · POST /replacements · field · create replacement order **(new)**
  - data · ← `nurse_visit_id, original_vial_id, reason` · → `replacement_order{ id, reason, status }`
- comp · `CMP-ADM-041` · ReplacementQueue · mfe-admin · queue of open replacements **(new)**
- also raises an `incident` (gw) — must-alert.

**STEP-R01-05 — Dispense a fresh vial** · serves F-504
- actor: pharmacy (self) · requires: `pharmacy:write:self`
- api · `API-PHARM-004` · POST /me/pharmacy/dispense · pharm · dispense replacement *(reuse)*
  - data · ← `vial_barcode, nurse_id, signature_ref` · → `dispense_record{ id }, vial{ status }`
- comp · `CMP-PHM-003` · DispenseToNurse · mfe-pharmacy · scan + signature
- → writes `replacement_order.replacement_vial_id`.

**STEP-R01-06 — Schedule the replacement visit** · serves F-706 / F-422
- actor: ops_manager (any) · requires: `assignment:write:any` / `nurse_visit:write:assigned`
- api · `API-FIELD-033` · POST /logistics/assign · field · push a new visit (or `API-FIELD-021` reschedule) *(reuse)*
  - data · ← `visit_id, nurse_id` · → `nurse_visit{ id, nurse_id, status }`
- comp · `CMP-NUR-023` · EmergencyAssignment · mfe-nurse · push visit
- → spawns a new `assigned` `nurse_visit`; patient notified via `API-NOTIF-002/003`.
- `replacement_order` → `fulfilled` when that visit records an injection.

---

### RFLOW-02 — Batch recall → reassign affected patients · resolves UP-PHM-02 (batch.recall_status → recalled ⚠) · `[impl]` · `[clinical/ops]`
- owner: ops + pharmacy · trigger: supplier/admin recall of a lot.
- closes: `recall` → `closed` when every affected scheduled visit has a fulfilled replacement and every already-injected patient has a logged clinical follow-up.
- decision needed: patient-facing communication wording when a recall affects already-scheduled or already-administered doses (UP open-decision #11) `[clinical]`.

**STEP-R02-01 — Flag and recall the batch (cascade)** · serves F-505 (recovery) · `[impl]` · `[clinical/ops]`
- actor: admin / ops_manager (any) · requires: `pharmacy:write:any` / `ops:manage:any`
- api · `API-PHARM-010` · POST /batches/{id}/recall · pharm · set recall + cascade **(new)**
  - data · ← `reason` · → `recall{ id, batch_id, status }, affected_count`
- comp · `CMP-ADM-042` · RecallManager · mfe-admin · recall control + progress **(new)**
- cascade (event-driven, no cross-service FK):
  - in-hub vials of the batch → `vial.status = damaged`;
  - dispensed / nurse-held vials → event to `field` → `nurse_inventory.status = damaged`, surfaced with a recalled flag in `CMP-NUR-022 InventoryManager`.
- → `batch.recall_status` → `recalled`; `recall` → `cascaded`.

**STEP-R02-02 — Compute affected patients** · serves F-618 (recovery) · `[impl]`
- actor: ops_manager (any) · requires: `ops:manage:any`
- api · `API-PHARM-011` · GET /recalls/{id}/affected · pharm · affected read model **(new)**
  - data · → `(derived){ scheduled[]{ nurse_visit_id, patient_identity_id }, administered[]{ injection_id, patient_identity_id, administered_at } }`
- comp · `CMP-ADM-042` · RecallManager · mfe-admin · affected list (scheduled vs already-administered)
- `scheduled[]` = upcoming `nurse_visit`s that would draw a recalled vial; `administered[]` = `injection_record`s whose `vial_id` belongs to the recalled batch.

**STEP-R02-03 — Reassign scheduled doses** · serves F-433 (recovery)
- per affected scheduled visit → open **RFLOW-01** with `reason = recall` (re-dispense a non-recalled vial + reschedule). *(reuse RFLOW-01)*

**STEP-R02-04 — Clinical follow-up for already-administered doses** · serves F-618 · `[clinical]`
- actor: ops_manager (any) → doctor (assigned) / cs_agent · requires: `ops:manage:any`
- api · `API-GW-021` · GET /ops/dashboard · gw · incident + outreach queue *(reuse; manual before M8)*
  - data · → `(derived){ incident[]{ type, severity, ref } }`
- comp · `CMP-ADM-031` · OpsDashboard · mfe-admin · triage
- each already-administered patient → `incident` + doctor/CS outreach (`API-NOTIF-*`, `ticket`); guidance is `[clinical]`.

---

## 3. Logistics / field recoveries (manual, then existing admin tools)

### RFLOW-03 — Wrong / mismatched vial · resolves UP-NUR-02 · `[reuse]` · `[clinical]`
- owner: nurse → ops · trigger: scanned vial doesn't match prescription/patient, or is expired/recalled, at injection.
- steps: nurse rescans the correct vial (`API-FIELD-007`, `CMP-NUR-007 InjectionRecorder`) and proceeds — no escalation if a usable vial is in hand. If none is usable, report it (`API-PHARM-006` if damaged) and enter **RFLOW-01** with `reason = mismatch`.
- closes: injection records normally, or original visit → `aborted` and a `replacement_order` opens.
- decision needed: the match rules (drug / dose / recall) are `[clinical]`.

### RFLOW-04 — Hub cold-chain quarantine · resolves UP-PHM-01 · `[manual→M5]` · `[clinical]`
- owner: pharmacy + ops · trigger: hub fridge out of range (`coldchain_log.breach`).
- steps: pharmacy sees the breach (`API-PHARM-005`, `CMP-PHM-004 ColdChainMonitor`), quarantines the exposed stock by marking those vials damaged (`API-PHARM-006`, `CMP-PHM-005 DamageReport`), and raises an `incident` (gw) for ops triage. If quarantine leaves a scheduled visit without stock → **RFLOW-09**.
- closes: exposed vials → `damaged`; incident resolved.
- decision needed: what counts as "exposed stock" and what happens to it is `[clinical]`.

### RFLOW-05 — Urgent side-effect clinical escalation · resolves UP-PAT-14 · `[reuse]` · `[clinical]`
- owner: doctor + ops · trigger: `side_effect_report.urgent = true` at side-effect submission.
- steps: patient submits the urgent report (`API-PATIENT-030`, `CMP-PAT-056 SideEffectReporter`) → must-alert to the assigned doctor + ops via `incident`. Doctor reviews in patient management (`API-VISIT-021`, `CMP-DOC-011`) and may raise an emergency visit (`API-VISIT-008`) or message the patient. Patient sees an emergency-line CTA (F-210).
- closes: incident resolved by doctor disposition.
- note: during the M4 pilot the on-call doctor handles this by hand (F-741); formal CS routing lands at M9 (`STEP-9-09`). Guidance wording is `[clinical]`.

### RFLOW-06 — Nurse safety escalation · resolves UP-NUR-05 · `[reuse]`
- owner: ops / CS · trigger: nurse presses the safety button, or escalates an emergency on-site.
- steps: nurse triggers a personal safety alert (`API-FIELD-011`, `CMP-NUR-010 SafetyButton`) → `safety_alert` row, immediate must-alert; or an emergency escalation (`API-FIELD-010`) → `emergency_escalation{ routed_to }`. Ops handles it by hand during the pilot; at M9 `CMP-NUR-010` re-points to the CS console (`STEP-9-09`, `API-FIELD-010`).
- closes: `safety_alert.resolved_at` set.

### RFLOW-07 — No nurse available / no capacity · resolves UP-ADM-04 · `[manual→M5]`
- owner: ops · trigger: no nurse can cover a visit's area/time.
- steps: the unassignable visit is spotted by hand from the schedule in the pilot (surfaced on `CMP-ADM-031 OpsDashboard` from M8). Ops assigns via the AssignmentBoard (`API-PROV-022`, `CMP-ADM-012`) or pushes an on-demand assignment (`API-FIELD-033`, F-706). If there is truly no capacity → **RFLOW-08** reschedule + notify.
- closes: visit assigned, or rescheduled.
- decision needed: capacity-gap lookahead and the patient-facing message when a visit can't be staffed (UP open-decision #12).

### RFLOW-08 — Missed-home-visit reschedule · resolves UP-PAT-11 / UP-NUR-04 (nurse_visit → rescheduled ⚠) · `[manual→M8]`
- owner: ops / nurse / patient · trigger: visit missed or aborted.
- steps: patient self-reschedules (`API-FIELD-021`, `CMP-PAT-053 VisitTimeManager`, F-422), spawning a new `assigned` visit; ops reschedules by hand in the pilot, and via the RescheduleTool from M8 (`API-GW-022`, `CMP-ADM-032`, F-619). Patient notified (`API-NOTIF-*`).
- closes: a new `assigned` `nurse_visit`.
- decision needed: missed-home-visit fee policy and cap (UP open-decision #7).

### RFLOW-09 — Stock-out / shortage at pickup · resolves UP-PHM-04 / UP-NUR-09 · `[manual→M5]`
- owner: pharmacy + ops · trigger: no usable vial for a scheduled visit, or a count mismatch at nurse pickup.
- steps: nurse reports the shortage/damage and raises a restock request (`API-FIELD-041`, `CMP-NUR-022 InventoryManager`); pharmacy fulfills the restock or re-dispenses (`API-PHARM-004`). If the hub is also out → ops sources stock or reschedules (**RFLOW-08**). Must-alert ops (risks a missed dose) via `incident`.
- closes: restock fulfilled, or visit rescheduled.
- decision needed: substitution and stock-out-prediction policy (UP open-decision #12).

### RFLOW-10 — Patient-not-home abort · resolves UP-NUR-04 (nurse_visit → aborted ⚠) · `[reuse]`
- owner: nurse → ops · trigger: nurse on-site, can't complete.
- steps: nurse marks the visit aborted with a structured reason (`API-FIELD-009`, `CMP-NUR-009 VisitSubmit`). A safety reason → **RFLOW-06**; any other reason → **RFLOW-08** reschedule.
- closes: `nurse_visit` → `aborted`, then a new visit via RFLOW-08.
- decision needed: which abort reasons are billable and which auto-escalate (UP open-decision #9).

### RFLOW-11 — Provider reassignment / in-flight care · resolves UP-DOC-04 · `[manual→M5]`
- owner: ops · trigger: a provider's `assignment` goes `inactive` while they were treating a patient.
- steps: ops reassigns the patient to a new provider on the AssignmentBoard (`API-PROV-022`, `CMP-ADM-012`); the new `active` assignment grants `:assigned`. Both providers and the patient are notified.
- closes: a new `active` `assignment`.
- decision needed: are in-flight visits grandfathered when an assignment is removed (UP open-decision #10)? If yes, ops keeps the prior assignment active until the in-flight visit completes.

---

## 4. Payment / finance, clinical, and edge recoveries

### 4.A Payment / finance

#### RFLOW-12 — Payment-failure resolution · resolves UP-PAT-07 (payment → failed ⚠) · `[reuse]`
- owner: self → CS / finance · trigger: gateway callback failure or session expiry at checkout.
- steps: patient retries, switches method, or pays from wallet (`API-PAY-001/002` checkout, `API-PAY-006` wallet-pay; `CMP-PAT-031 CheckoutFlow`). Checkout is idempotent so a retry can't double-charge. Persistent failure → CS ticket (M9) or a manual finance check during the pilot.
- closes: `payment` → `paid`; subscription unlocked.
- decision needed: auto-retry cadence, and whether the subscription is held in `pending_payment` or released on failure (UP open-decision #4).

#### RFLOW-13 — Past_due dunning · resolves UP-PAT-08 (active → past_due ⚠) · `[manual→M3]`
- owner: self → ops · trigger: a renewal payment fails.
- steps: amber "Payment due" banner + Pay-now (`API-PAY-006`, `CMP-PAT-033 WalletView` / `CMP-PAT-032 SubscriptionManager`); dunning reminders scheduled (`API-NOTIF-003`). If the grace window elapses → **RFLOW-14**.
- closes: `past_due` → `active` on payment.
- decision needed: the dunning schedule and what features degrade while `past_due` (UP open-decision #5).

#### RFLOW-14 — Expired-mid-treatment reactivation · resolves UP-PAT-09 (past_due → expired ⚠) · `[manual→M3]` · `[clinical]`
- owner: ops · trigger: grace window elapsed mid-treatment.
- steps: access is limited; patient resubscribes (`API-SUB-002`, `CMP-PAT-030 PackagePicker` / `CMP-PAT-031 CheckoutFlow`). Because stopping GLP-1 mid-course has clinical implications, expiry raises an `incident` for ops and the messaging encourages talking to a doctor rather than only "pay to unlock."
- closes: re-onboard to `active`.
- decision needed: grace-window length, whether an in-flight `nurse_visit` is honored after expiry, and clinical messaging (UP open-decision #6) `[clinical]`.

#### RFLOW-15 — Refund / refund-pending · resolves UP-PAT-13 / UP-ADM-02 · `[reuse]`
- owner: finance / CS · trigger: a refund is owed (medication/service issue) or a refund is pending/failed.
- steps: admin/finance issues the refund (`API-PAY-007`, `CMP-ADM-006 RefundAction`, F-147) → `wallet_transaction{ type=refund }`; the pending row shows in the wallet (`CMP-PAT-033 WalletView`). A failed refund → **RFLOW-16**.
- closes: `wallet_transaction` → settled.

#### RFLOW-16 — Chargeback reconciliation · resolves UP-ADM-02 · `[manual→M8]`
- owner: finance · trigger: an external chargeback arrives, or a refund exceeds the wallet balance.
- steps: the chargeback surfaces on the Finance dashboard (`API-PAY-030`, `CMP-ADM-033 FinanceDashboard`, F-633); finance reconciles by hand.
- closes: discrepancy reconciled.
- decision needed: refund policy when the wallet has been spent down, and the chargeback workflow (UP open-decision #13).

#### RFLOW-17 — Settlement-run failure · resolves UP-ADM-01 (settlement_run → failed ⚠) · `[reuse]`
- owner: finance · trigger: a settlement batch errors or partially fails.
- steps: per-item results show in the SettlementRunner (`API-PAY-024`, `CMP-ADM-020`); finance retries only the failed items. The rerun is idempotent so it can't double-pay.
- closes: all `settlement_item`s settled.
- decision needed: are partial settlements committed, or all-or-nothing per run (UP open-decision #13)?

### 4.B Clinical / eligibility / prescription

#### RFLOW-18 — Ineligible / borderline → paid doctor review · resolves UP-PAT-05 / UP-PAT-06 · `[manual→M3]` · `[clinical]`
- owner: doctor (via ops / CS) · trigger: eligibility returns `ineligible` or `borderline`.
- steps: the result screen shows a **"our support team will call you"** pending state (`API-PATIENT-021`, `CMP-PAT-021 EligibilityResult`) — **no patient-facing booking/payment**; the pending status is persisted on the assessment and re-shown on every login until resolved. CS / concierge contacts the patient and, when appropriate, **unlocks** the review booking (`API-VISIT-002`, `CMP-PAT-040 VisitBooking`) + payment (`API-PAY-001`) — these remain hidden from the patient until CS contact. The doctor then reviews the EMR (`API-PATIENT-010`, `CMP-DOC-002`) and may proceed or override.
- closes: an eligible path opened by the doctor, or a respectful, non-dead-end close.
- decision needed: which reasons are shown, and whether the review is ultimately paid, plus its price — arranged by CS on the call (UP open-decision #3) `[clinical]`.

#### RFLOW-19 — External prescription rejected → re-issue + re-link · resolves UP-DOC-01 (prescription → rejected ⚠) · `[reuse]` · `[int·RxProvider]`
- owner: doctor · trigger: the external e-prescription provider returns a rejected/invalid status.
- steps: the rejection reason shows on `CMP-DOC-004 PrescriptionLink`; the doctor **re-issues the prescription in the external Rx software** (keyed by the patient's national ID, outside Medica), then re-links the new `external_rx_id` (`API-VISIT-005`) and Medica re-fetches the status (`API-VISIT-006`). The provider-unreachable variant (UP-DOC-02) auto-retries the fetch.
- closes: prescription → `active`.

#### RFLOW-20 — KYC manual review · resolves UP-PAT-03 · `[manual→M9]` · `[int·Shahkar]`
- owner: ops / CS · trigger: Shahkar mismatch sets `kyc_status` → `failed`.
- steps: patient retries or edits details (`API-AUTH-007`, `CMP-PAT-004 KycVerification`); after the retry limit, ops reviews by hand during the pilot, and via a CS new-patient panel `ticket` from M9 (`API-CRM-011`, `CMP-CS-006 NewPatientPanel`).
- closes: `kyc_status` → `verified`, or a documented manual override.
- decision needed: KYC retry count before manual review, and whether onboarding proceeds while pending (UP open-decision #2).

### 4.C Edge / rare cases — generic CS ticket (no dedicated flow)

These resolve through the standard CS ticket queue (`API-CRM-010/011`, `CMP-CS-001…007`, M9; by hand before that). They are named here only so the unhappy-paths `resolver:` lines have a target.

- **Account reinstatement** (UP-PAT-15) → CS ticket → admin toggles identity status (`API-GW-020`, `CMP-ADM-030 UserManagement`).
- **Affiliate commission dispute / clawback** (UP-AFF-02) → CS/ops ticket; clawback rule on early churn/refund is a `[Decision needed]` (UP open-decision #14).
- **Affiliate application rejected** (UP-AFF-03) → CS ticket → ops.
- **Geocode failed for an address** (UP-ADM-03) → ops sets a manual map pin inline (`CMP-ADM-012 AssignmentBoard` consuming `API-FIELD-030`); writes back `address.geo_lat/lng`. No RFLOW.
- **GPS check-in override** (UP-NUR-03) → nurse manual-confirms with a logged reason (audited); the override allowance is a `[Decision needed]` (UP open-decision #8). No RFLOW.

### 4.D Self-service / platform behavior (no handoff, no RFLOW)

Resolved by the user on the same screen or by platform behavior; the unhappy-paths entry already designs the exit. Listed for completeness: UP-PAT-01, -02, -04, -12, -16; UP-NUR-07, -08, -10; UP-PHM-03, -05, -06; UP-DOC-02 (auto-retry), -03, -06; UP-ADM-06, -07; UP-AFF-01; UP-X-01…07. Offline behavior (UP-NUR-06, UP-X-01) remains deferred — the nurse-offline spec was dropped — so writes queue where safe and otherwise block.

---

## 5. Resolver index

The owner and recovery target for every unhappy-path entry. **These are the `resolver:` lines to add to `Medica_Unhappy_Paths.md`** (one per entry).

| UP entry | resolver |
|---|---|
| UP-PAT-01 OTP wrong/expired | self-service (resend / change number) — no RFLOW |
| UP-PAT-02 Login locked out | self-service (wait / reset) — no RFLOW |
| UP-PAT-03 KYC failed | ops / CS · RFLOW-20 |
| UP-PAT-04 Lab upload failed | self-service (retry / skip) — no RFLOW |
| UP-PAT-05 Ineligible | doctor (via ops/CS) · RFLOW-18 |
| UP-PAT-06 Borderline → review | doctor (via ops/CS) · RFLOW-18 |
| UP-PAT-07 Payment failed | self → CS/finance · RFLOW-12 |
| UP-PAT-08 Subscription past_due | self → ops · RFLOW-13 |
| UP-PAT-09 Expired mid-treatment | ops · RFLOW-14 |
| UP-PAT-10 Visit no-show (patient) | self-service (rebook `API-VISIT-002`) — no RFLOW |
| UP-PAT-11 Nurse arrived, patient not home | ops / nurse · RFLOW-08 |
| UP-PAT-12 Live tracking unavailable | self-service (graceful degradation) — no RFLOW |
| UP-PAT-13 Wallet refund pending/failed | finance / CS · RFLOW-15 |
| UP-PAT-14 Urgent side-effect | doctor + ops · RFLOW-05 |
| UP-PAT-15 Account suspended/banned | CS → admin · generic CS ticket (§4.C) |
| UP-PAT-16 Empty states | n/a |
| UP-NUR-01 Cold-chain breach | pharmacy + ops · RFLOW-01 |
| UP-NUR-02 Wrong/mismatched vial | nurse → ops · RFLOW-03 |
| UP-NUR-03 GPS check-in fails | nurse (logged override) → ops · generic (§4.C) |
| UP-NUR-04 Patient not home / unsafe | nurse → ops · RFLOW-10 (safety → RFLOW-06) |
| UP-NUR-05 Safety button | ops / CS · RFLOW-06 |
| UP-NUR-06 Offline mid-visit | deferred (offline spec dropped) — no RFLOW |
| UP-NUR-07 Route unavailable / Optime down | self-service (work list manually) — no RFLOW |
| UP-NUR-08 Signature capture fails | self-service (OTP fallback) — no RFLOW |
| UP-NUR-09 Inventory shortage at pickup | pharmacy + ops · RFLOW-09 |
| UP-NUR-10 Empty states | n/a |
| UP-DOC-01 External prescription rejected/invalid | doctor · RFLOW-19 |
| UP-DOC-02 External Rx provider unreachable | self-service (auto-retry / refresh) — no RFLOW |
| UP-DOC-03 Patient record incomplete | doctor self / patient — no RFLOW |
| UP-DOC-04 Assignment lost mid-care | ops · RFLOW-11 |
| UP-DOC-05 Provider deactivated | ops / admin · generic CS ticket (§4.C) |
| UP-DOC-06 Empty states | n/a |
| UP-PHM-01 Hub cold-chain breach | pharmacy + ops · RFLOW-04 |
| UP-PHM-02 Batch recall | ops + pharmacy · RFLOW-02 |
| UP-PHM-03 Barcode scan fails | self-service (manual entry, logged) — no RFLOW |
| UP-PHM-04 Stock-out / cannot fulfill | pharmacy + ops · RFLOW-09 |
| UP-PHM-05 Expired stock detected | pharmacy self (quarantine/record) — no RFLOW |
| UP-PHM-06 Empty states | n/a |
| UP-ADM-01 Settlement run fails/partial | finance · RFLOW-17 |
| UP-ADM-02 Refund / chargeback edge | finance · RFLOW-15 / RFLOW-16 |
| UP-ADM-03 Geocode failed | ops (manual pin) · generic (§4.C) |
| UP-ADM-04 No nurse available | ops · RFLOW-07 |
| UP-ADM-05 Incident / cold-chain alert | ops (triage hub) — destination for must-alert RFLOWs, not its own RFLOW |
| UP-ADM-06 Role/permission misconfig | admin self — no RFLOW |
| UP-ADM-07 Bulk action partial failure | admin self — no RFLOW |
| UP-AFF-01 Referral link invalid | self-service (proceed without code) — no RFLOW |
| UP-AFF-02 Commission dispute | ops / finance · generic CS ticket (§4.C) |
| UP-AFF-03 Affiliate app rejected | ops · generic CS ticket (§4.C) |
| UP-MKT-01 BMI invalid input | self-service — no RFLOW |
| UP-MKT-02 Public eligibility not-a-fit / borderline `[clinical]` | self-service → signup; post-signup → RFLOW-18 |
| UP-MKT-03 Public eligibility preview unavailable | self-service (still allow signup) — no RFLOW |
| UP-X-01…07 Cross-cutting | self-service / platform — no RFLOW |

---

## 6. Changes folded back into the other docs

New ids introduced here are minimal — four APIs, two components, two entities, all for the two safety-critical builds — and fold back as follows.

### Into `Medica_Data_Model.md`

**`field` service — new entity `replacement_order`** · serves F-433 (recovery) · STEP-R01-04
- fields: `nurse_visit_id` (→ field.nurse_visit), `original_vial_id` (→ pharm.vial, by id), `replacement_vial_id` (→ pharm.vial, by id; nullable until dispensed), `reason` enum(coldchain/mismatch/recall), `status` `enum (states in lifecycle spec)`, `opened_by` (identity_id)

**`pharm` service — new entity `recall`** · serves F-505/F-618 (recovery) · STEP-R02-01
- fields: `batch_id` (→ batch), `reason`, `affected_count` (int), `status` `enum (states in lifecycle spec)`, `initiated_by` (identity_id)
- reuses existing `batch.recall_status` (none/flagged/recalled) and `nurse_inventory.status = damaged` for the cascade; adds no fields to `vial` or `nurse_inventory`.

**§13 cross-service reference map — add:**
- `field.replacement_order.nurse_visit_id` → `field.nurse_visit.id` (same service); `original_vial_id` / `replacement_vial_id` → `pharm.vial.id`.
- `pharm.recall.batch_id` → `pharm.batch.id`; recall cascade flags `pharm.vial` and, by event, `field.nurse_inventory`.

**§14 open questions — cross-reference:** the recall cascade is the consumer of the batch→vial traceability noted in `pharm` §7; no new open question.

### Into `Medica_Lifecycle_State_Machines.md`

Two compact utility machines (the `⚠` transitions they react to already exist and are unchanged):

| entity | states | transitions / notes |
|---|---|---|
| replacement_order · field | requested → dispatched → fulfilled ⊗ / cancelled ⊗ | requested on STEP-R01-04; dispatched on dispense (STEP-R01-05); fulfilled when the replacement visit records an injection; internal (ops sees status in ReplacementQueue). |
| recall · pharm | initiated → cascaded → closed ⊗ | initiated + cascaded on STEP-R02-01 (flags vials in hub **and** nurse inventory); closed when all affected scheduled doses are replaced and all administered doses have a logged follow-up. internal. |

### Into `Medica_Build_and_Technical_Specification.md`

Add to **Month 5** the recovery steps STEP-R01-04, STEP-R01-05/06 (assembly), and STEP-R02-01/02 with the new ids:
- `API-FIELD-050` POST /replacements (field) · `API-FIELD-051` GET /replacements (field, ReplacementQueue read)
- `API-PHARM-010` POST /batches/{id}/recall (pharm, cascade) · `API-PHARM-011` GET /recalls/{id}/affected (pharm, read model)
- `CMP-ADM-041` ReplacementQueue (mfe-admin) · `CMP-ADM-042` RecallManager (mfe-admin)
- recalled/replacement state is surfaced in the existing `CMP-NUR-022 InventoryManager` (no new nurse component).

Add to **§7 (new data introduced by this layer):** `replacement_order` (field) and `recall` (pharm) as the two genuinely-new entities; `API-PHARM-011`'s affected list is a `(derived)` read model consistent with data-model §13.

### Into `Medica_Unhappy_Paths.md`

Add the §5 resolver line to each `UP-` entry, and a one-line pointer in the closing notes that recovery flows are designed in `Medica_Recovery_Flows.md`. Cross-reference, from the open-decisions list (§I), the items an RFLOW now operationalizes: #3 (RFLOW-18), #5 (RFLOW-13), #6 (RFLOW-14), #7 (RFLOW-08), #9 (RFLOW-10), #10 (RFLOW-11), #11 (RFLOW-02), #12 (RFLOW-07/09), #13 (RFLOW-16/17), #14 (§4.C). The decisions themselves remain open; the flows carry the mechanism with the value marked `Decision needed`/`[clinical]`.

---

## 7. Planning impact by month

The recovery layer adds build to exactly one month and formalizes manual recoveries onto tools already on the roadmap.

- **Month 1** — KYC failure is handled by hand (RFLOW-20, manual); no build.
- **Month 2** — Ineligible/borderline review is concierge by hand (RFLOW-18, manual); no build.
- **Month 3** — Payment-failure, dunning, and expired-mid-treatment reactivation (RFLOW-12/13/14) ride on the subscription/payment steps already built this month; no new build beyond idempotent checkout, which the unhappy-paths spec already flags.
- **Month 4 (pilot launch)** — All non-safety recoveries run by hand: refunds via `CMP-ADM-006 RefundAction`, reschedules via patient self-serve (`API-FIELD-021`) + direct ops action, no-nurse and reassignment by hand, urgent side-effect and nurse safety via on-call ops. A cold-chain breach or recall during the supervised pilot is dispatched manually until the M5 software lands.
- **Month 5 — the only month that gains real build.** RFLOW-01 (replacement dose) and RFLOW-02 (recall cascade): ~4 new APIs (`API-FIELD-050/051`, `API-PHARM-010/011`), 2 new admin components (`CMP-ADM-041/042`), 2 new entities (`replacement_order`, `recall`). These sit beside the M5 pharmacy/vial/batch/cold-chain work and reuse its dispense, damage-report, assignment, and reschedule ids. RFLOW-04/07/09/11 become tool-assisted here too (AssignmentBoard, on-demand assign, restock).
- **Month 6** — Settlement-run failure (RFLOW-17) relies on the idempotent rerun built into `API-PAY-024`; no extra build.
- **Month 8** — Chargeback reconciliation (RFLOW-16) and the missed-visit reschedule (RFLOW-08) formalize onto the FinanceDashboard, RescheduleTool, and the OpsDashboard incident feed (`API-GW-021`), which becomes the triage hub for every must-alert RFLOW.
- **Month 9** — Ticket-based recoveries (KYC review, account reinstatement, affiliate disputes, persistent payment failures) move from by-hand to the CS panels (`API-CRM-010/011`, `CMP-CS-001…007`).

No new month is introduced and nothing is pulled forward beyond the two Month-5 safety-critical builds.
