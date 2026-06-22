# Medica — User Flows (single source of truth)

Every actor's flows, end to end. **Happy steps** (`.H#`) are the success path; **unhappy branches** (`.U#`) are what can go wrong, each anchored at a happy step and pointing to a **recovery** in the shared Recovery Library at the end. Recoveries (`R-…`) are defined **once** and reused by every branch that needs them.

**The only external reference in this document is the feature id (`F-…`).** Flow ids (`PF-A.H1`, `PF-A.U1`, `R-RESCHEDULE`) are this document's own. Each item lists the feature(s) it serves; `F: —` means no specific feature.

ID scheme: `PF-A` flow · `PF-A.H3` happy step 3 · `PF-A.U2` unhappy branch 2 (at a happy step) · `R-NAME` a shared recovery with its own numbered steps. Families: `PF` patient · `MF` public · `DF` doctor · `NF` nurse · `PH` pharmacy · `OP` admin/ops · `CS` customer service · `AF` affiliate · `XF` cross-cutting. Symbols: `→` uses recovery · `⤴` rejoin · `⤓` exit.

---

# PF — PATIENT

## PF-A · Account & login

**Happy**
- `PF-A.H1` Enter phone + password, request an OTP · F-101
- `PF-A.H2` Verify the phone OTP · F-101 · may trigger: `PF-A.U1`
- `PF-A.H3` Log in (phone + password, or phone + OTP) · F-102 · may trigger: `PF-A.U2`, `PF-A.U3`
- `PF-A.H4` Recover password · F-103
- `PF-A.H5` Verify identity / KYC · F-105 · may trigger: `PF-A.U4`

**Unhappy**
- `PF-A.U1` **OTP wrong or expired** (at H2) → `R-RESEND-OTP` · F-101
- `PF-A.U2` **Locked out** (at H3) → `R-COOLDOWN` · F-102
- `PF-A.U3` **Account suspended / banned** (at H3) → `R-BLOCKED-STATE` · F-102
- `PF-A.U4` **KYC fails or mismatch** (at H5) → `R-KYC-REVIEW` · F-105

**Recoveries**

### `R-RESEND-OTP` — Resend a fresh OTP · F-101 · **used by:** PF-A.U1
1. Show an inline error. 2. Offer "resend" and issue a new OTP. — ⤴ retry.

### `R-COOLDOWN` — Lockout cooldown · F-102 · **used by:** PF-A.U2
1. Enforce a cooldown window. 2. Offer password recovery as the faster route. — ⤴ retry after cooldown.

### `R-KYC-REVIEW` — KYC manual review · F-105 · **used by:** PF-A.U4
1. Patient retries or edits their details.
2. After the retry limit, an ops/CS agent reviews the case by hand.
3. Agent verifies the patient, or records a documented manual override.
- ⤴ verified → continue · ⤓ fail → onboarding paused.


## PF-B · Onboarding (profile → eligibility → consent)

**Happy**
- `PF-B.H1` Complete basic profile · F-110
- `PF-B.H2` Enter height/weight, see BMI · F-111
- `PF-B.H3` Enter medical history · F-112
- `PF-B.H4` Upload lab results · F-113 · may trigger: `PF-B.U1`
- `PF-B.H5` Set delivery address · F-115
- `PF-B.H6` Answer eligibility questionnaire · F-121
- `PF-B.H7` Compute eligibility, show result · F-122, F-123, F-125, F-126 · may trigger: `PF-B.U2`, `PF-B.U3`
- `PF-B.H8` Sign informed consent · F-124
- `PF-B.H9` Receive confirmations · F-205
- `PF-B.H10` Land on patient home, ready to subscribe · F-100

**Unhappy**
- `PF-B.U1` **Lab upload rejected** (at H4) → `R-INLINE-VALIDATE` · F-113
- `PF-B.U2` **Borderline eligibility** (at H7) → `R-ELIG-REVIEW` · F-125
- `PF-B.U3` **Ineligible** (at H7) → `R-ELIG-REVIEW` · F-123

**Recoveries**

### `R-ELIG-REVIEW` — Doctor eligibility review (support-led) · F-123, F-125 · **used by:** PF-B.U2, PF-B.U3
1. Result screen shows a **"our support team will call you"** state — no patient-facing book/pay action. The pending status persists: on every re-login the patient sees the same message until it's resolved.
2. CS / concierge contacts the patient and **unlocks** the doctor-review booking + payment when appropriate (booking/payment still exist but stay hidden from the patient until CS makes contact).
3. Doctor reviews the record and decides eligible or not (may override).
- ⤴ approved → consent · ⤓ otherwise → not enrolled (respectful close).


## PF-C · Subscription & payment

**Happy**
- `PF-C.H1` View packages · F-130
- `PF-C.H2` Purchase & pay · F-131 · may trigger: `PF-C.U1`
- `PF-C.H3` Manage / stop renewal · F-133 · may trigger: `PF-C.U2`, `PF-C.U3`
- `PF-C.H4` Cancel + exit interview · F-139
- `PF-C.H5` Wallet — balance, top-up, pay · F-141, F-142, F-143, F-146 · may trigger: `PF-C.U4`

**Unhappy**
- `PF-C.U1` **Payment failed / gateway unreachable** (at H2) → `R-PAY-RETRY` · F-131
- `PF-C.U2` **Renewal failed → past due** (at H3) → `R-DUNNING` · F-133
- `PF-C.U3` **Expired mid-treatment** (at H3) → `R-REACTIVATE` · F-133
- `PF-C.U4` **Refund / cashback pending or failed** (at H5) → `R-REFUND` · F-147, F-144

**Recoveries**

### `R-PAY-RETRY` — Payment-failure resolution · F-131 · **used by:** PF-C.U1
1. Patient retries, switches method, or pays from wallet (idempotent — no double-charge).
2. Persistent failure → finance/CS check.
- ⤴ success → activate · ⤓ final failure → no subscription.

### `R-DUNNING` — Past-due dunning · F-133 · **used by:** PF-C.U2
1. Show an amber "Payment due" banner with pay-now.
2. Schedule dunning reminders; degrade some features while past due.
- ⤴ pay → active; if grace elapses → `R-REACTIVATE`.

### `R-REACTIVATE` — Expired-mid-treatment reactivation · F-133 · **used by:** PF-C.U3
1. Limit access; prompt resubscribe.
2. Because stopping GLP-1 mid-course is clinically risky, encourage talking to a doctor and raise an ops incident.
- ⤴ resubscribe → reactivated.

### `R-REFUND` — Refund / refund-pending · F-147, F-144 · **used by:** PF-C.U4
1. Finance issues the refund to the wallet.
2. Show the pending entry; a failed refund → `R-CHARGEBACK`.
- ⤴ settled to wallet.


## PF-D · Online doctor visit (patient side)

**Happy**
- `PF-D.H1` Book a visit · F-150, F-151
- `PF-D.H2` Get a reminder · F-152
- `PF-D.H3` Join the phone visit · F-153 · may trigger: `PF-D.U1`
- `PF-D.H4` View & download the prescription · F-155, F-159 · may trigger: `PF-D.U2`
- `PF-D.H5` Raise an emergency request · F-157

**Unhappy**
- `PF-D.U1` **Patient no-show** (at H3) → `R-RESCHEDULE` · F-153
- `PF-D.U2` **Prescription not ready** (at H4) → `R-RXFIX` · F-330

## PF-E · Home nurse visit & tracking (patient side)

**Happy**
- `PF-E.H1` See schedule + nurse profile · F-170, F-220
- `PF-E.H2` Verify nurse identity on arrival · F-221
- `PF-E.H3` Confirm or change the time · F-171 · may trigger: `PF-E.U1`
- `PF-E.H4` Confirm medication received · F-222
- `PF-E.H5` Confirm service received · F-174
- `PF-E.H6` Track the nurse live · F-172 · may trigger: `PF-E.U2`
- `PF-E.H7` See injection history · F-175
- `PF-E.H8` Next-injection reminder · F-176

**Unhappy**
- `PF-E.U1` **Patient not home / unreachable** (at H3) → `R-RESCHEDULE` · F-171
- `PF-E.U2` **Live tracking unavailable** (at H6) → `R-TRACK-FALLBACK` · F-172

## PF-F · Side-effect report

**Happy**
- `PF-F.H1` Report a side effect · F-177, F-178 · may trigger: `PF-F.U1`

**Unhappy**
- `PF-F.U1` **Urgent side effect** (at H1) → `R-CLINICAL-ESC` · F-178

**Recoveries**

### `R-CLINICAL-ESC` — Urgent side-effect clinical escalation · F-178 · **used by:** PF-F.U1
1. Must-alert the assigned doctor and ops immediately.
2. Doctor reviews and may raise an emergency visit or message the patient.
3. Patient sees an emergency-line option.
- ⤴ closed by clinician disposition.


## PF-G · Engagement & support

**Happy**
- `PF-G.H1` Adherence chart + streak · F-185, F-186
- `PF-G.H2` Weight goal · F-188
- `PF-G.H3` Daily reminders · F-189
- `PF-G.H4` Content library · F-195, F-196, F-197, F-199
- `PF-G.H5` Foodnoise tracking · F-223, F-224
- `PF-G.H6` Multiple addresses + weight history · F-114, F-116, F-117
- `PF-G.H7` Support chat + FAQ + emergency line · F-207, F-209, F-210
- `PF-G.H8` Visit chat + survey + history PDF · F-154, F-156, F-158
- `PF-G.H9` Pricing perks · F-132, F-144

**Unhappy**
- `PF-G.U1` **Nothing to show yet** (any step) → `R-EMPTY-STATE` · F: —

---

# MF — PUBLIC (pre-auth)

## MF-A · Landing & self-check

**Happy**
- `MF-A.H1` Landing home · F-095
- `MF-A.H2` BMI calculator · F-096 · may trigger: `MF-A.U1`
- `MF-A.H3` Public eligibility check (preliminary) · F-097 · may trigger: `MF-A.U2`, `MF-A.U3`

**Unhappy**
- `MF-A.U1` **Invalid BMI input** (at H2) → `R-INLINE-VALIDATE` · F-096
- `MF-A.U2` **Not a fit / borderline** (at H3) → `R-GUIDANCE-CTA` · F-097
- `MF-A.U3` **Preview unavailable** (at H3) → `R-PREVIEW-RETRY` · F-097

**Recoveries**

### `R-GUIDANCE-CTA` — Supportive guidance + sign-up CTA · F-097 · **used by:** MF-A.U2
1. Show supportive guidance. 2. Offer a sign-up CTA for the full check.

### `R-PREVIEW-RETRY` — Retry / proceed to full check · F-097 · **used by:** MF-A.U3
1. Offer retry. 2. Or proceed to the full check after sign-up. — ⤴ sign-up.

---

# DF — DOCTOR


## DF-A · Online visit & prescription

**Happy**
- `DF-A.H1` Open the assigned patient list · F-310
- `DF-A.H2` Open a patient profile + EMR · F-312 · may trigger: `DF-A.U1`
- `DF-A.H3` Conduct the phone visit · F-153
- `DF-A.H4` Write the SOAP note · F-328
- `DF-A.H5` Link the external prescription · F-327, F-329 · may trigger: `DF-A.U2`, `DF-A.U3`
- `DF-A.H6` Confirm prescription status · F-330
- `DF-A.H7` Auto-schedule the next visit · F-331
- `DF-A.H8` External e-prescription integration goes live · F-330

**Unhappy**
- `DF-A.U1` **Record incomplete at visit** (at H2) → `R-COMPLETE-RECORD` · F-312
- `DF-A.U2` **Rx rejected / invalid** (at H5) → `R-RXFIX` · F-330
- `DF-A.U3` **Rx provider unreachable** (at H5) → `R-RXFIX` · F-330

**Recoveries**

### `R-COMPLETE-RECORD` — Prompt to complete record · F-312 · **used by:** DF-A.U1
1. Prompt to complete the missing history/labs. 2. Or proceed with a note.


## DF-B · Doctor panel & finance

**Happy**
- `DF-B.H1` Doctor login · F-102 · may trigger: `DF-B.U1`
- `DF-B.H2` Doctor dashboard · F-301, F-305, F-302, F-303, F-304 · may trigger: `DF-B.U2`
- `DF-B.H3` Patient management · F-311, F-313, F-314, F-315, F-317, F-319, F-316 · may trigger: `DF-B.U3`
- `DF-B.H4` Doctor scheduling · F-340, F-341
- `DF-B.H5` Doctor visits list · F-301, F-313
- `DF-B.H6` Doctor prescriptions list · F-329, F-330
- `DF-B.H7` Doctor financial panel · F-350, F-356, F-358, F-351, F-352, F-353, F-354, F-355

**Unhappy**
- `DF-B.U1` **Provider deactivated** (at H1) → `R-REASSIGN` · F: —
- `DF-B.U2` **Nothing assigned yet** (at H2) → `R-EMPTY-STATE` · F: —
- `DF-B.U3` **Assignment lost mid-care** (at H3) → `R-REASSIGN` · F: —

**Recoveries**

### `R-REASSIGN` — Provider reassignment / in-flight care · F: — · **used by:** DF-B.U1, DF-B.U3
1. Ops reassigns the patient to a new provider (optionally keeping the old assignment until any in-flight visit finishes).
2. Both providers and the patient are notified.
- ⤴ care continues under the new provider.

---

# NF — NURSE


## NF-A · Home-visit safety sequence

**Happy**
- `NF-A.H1` Nurse logs in · F-401
- `NF-A.H2` Nurse dashboard & status · F-405, F-407
- `NF-A.H3` View route & schedule (today / tomorrow / week) · F-415, F-416, F-417, F-418, F-409
- `NF-A.H4` GPS check-in at the patient's location · F-430 · may trigger: `NF-A.U1`
- `NF-A.H5` View patient profile + today's dose · F-431, F-432 · may trigger: `NF-A.U2`
- `NF-A.H6` Record cold-chain temperature · F-433 · may trigger: `NF-A.U3`
- `NF-A.H7` Pre-injection assessment · F-434
- `NF-A.H8` Scan vial, then record injection site · F-437 · may trigger: `NF-A.U4`
- `NF-A.H9` Patient signs · F-441 · may trigger: `NF-A.U5`
- `NF-A.H10` Submit the visit · F-442

**Unhappy**
- `NF-A.U1` **GPS fails / location off** (at H4) → `R-GPS-OVERRIDE` · F-430
- `NF-A.U2` **Patient not home / refuses / unsafe** (at H5) → `R-RESCHEDULE` · F: —
- `NF-A.U3` **Cold-chain breach → do not administer** (at H6) → `R-REPLACE` · F-433
- `NF-A.U4` **Wrong / mismatched vial scanned** (at H8) → `R-VIAL-RESCAN` · F-437
- `NF-A.U5` **Signature capture fails / patient can't sign** (at H9) → `R-OTP-RECEIPT` · F-441
- `NF-A.U6` **Offline mid-visit** (any step) → `R-OFFLINE-CAPTURE` · F-430, F-442

**Recoveries**

### `R-GPS-OVERRIDE` — Manual GPS override · F-430 · **used by:** NF-A.U1
1. Request a supervisor-approved manual override. 2. Log it for audit. — ⤴ continue.

### `R-VIAL-RESCAN` — Wrong/mismatched vial · F-437 · **used by:** NF-A.U4
1. If a correct vial is in hand, rescan it and proceed.
2. If none is usable, report it and enter `R-REPLACE`.
- ⤓ proceed with correct vial, or abort + replacement.

### `R-OTP-RECEIPT` — OTP receipt fallback · F-441 · **used by:** NF-A.U5
1. Fall back to an OTP receipt to confirm delivery. — ⤴ continue.

### `R-OFFLINE-CAPTURE` — Offline capture + sync · F-430, F-442 · **used by:** NF-A.U6
1. Capture everything offline. 2. Sync when connectivity returns.


## NF-B · Safety & emergency

**Happy**
- `NF-B.H1` Escalate an emergency · F-443
- `NF-B.H2` Press the safety button · F-497 · may trigger: `NF-B.U1`
- `NF-B.H3` Emergency escalation in-software, routed to CS · F-443

**Unhappy**
- `NF-B.U1` **Safety button pressed** (at H2) → `R-SAFETY-ESC` · F-497

**Recoveries**

### `R-SAFETY-ESC` — Nurse safety escalation · F-497, F-443 · **used by:** NF-B.U1
1. Fire an immediate must-alert to ops/CS.
2. Ops/CS respond directly and record resolution.


## NF-C · Route, inventory & completion

**Happy**
- `NF-C.H1` Live tracking + ETA · F-707, F-708, F-172 · may trigger: `NF-C.U1`
- `NF-C.H2` Assigned-patient list · F-450, F-451, F-452, F-453, F-454
- `NF-C.H3` Nurse inventory · F-460, F-465, F-461, F-462, F-463, F-464 · may trigger: `NF-C.U2`
- `NF-C.H4` Reorder / reschedule · F-420, F-422
- `NF-C.H5` Record patient education · F-440
- `NF-C.H6` Week/month summary · F-408
- `NF-C.H7` Support inbox · F-410
- `NF-C.H8` Nurse financial panel · F-470, F-477, F-471, F-472, F-473, F-474, F-475, F-476
- `NF-C.H9` Performance + hotline · F-485, F-486, F-495 · may trigger: `NF-C.U3`

**Unhappy**
- `NF-C.U1` **Route unavailable / Optime down** (at H1) → `R-TRACK-FALLBACK` · F-703
- `NF-C.U2` **Shortage at pickup / count mismatch** (at H3) → `R-STOCKOUT` · F-465
- `NF-C.U3` **No visits / patients / inventory** (at H9) → `R-EMPTY-STATE` · F: —

---

# PH — PHARMACY

## PH-A · Inventory & dispense

**Happy**
- `PH-A.H1` Pharmacy login (OTP) · F-501
- `PH-A.H2` Inventory dashboard · F-502 · may trigger: `PH-A.U1`
- `PH-A.H3` Record supplier receipt · F-503
- `PH-A.H4` Monitor cold chain · F-505 · may trigger: `PH-A.U2`
- `PH-A.H5` Dispense to nurse · F-504 · may trigger: `PH-A.U3`, `PH-A.U4`
- `PH-A.H6` Report damage · F-506 · may trigger: `PH-A.U5`
- `PH-A.H7` View audit log · F-515
- `PH-A.H8` Pharmacy commission · F-507, F-508, F-512, F-513

**Unhappy**
- `PH-A.U1` **No inventory / no pending dispenses** (at H2) → `R-EMPTY-STATE` · F: —
- `PH-A.U2` **Hub cold-chain breach** (at H4) → `R-HUB-QUARANTINE` · F-505
- `PH-A.U3` **Barcode scan fails** (at H5) → `R-MANUAL-ENTRY` · F-504
- `PH-A.U4` **Stock-out / cannot fulfill** (at H5) → `R-STOCKOUT` · F-504
- `PH-A.U5` **Expired stock detected** (at H6) → `R-EXPIRE-QUARANTINE` · F-506

**Recoveries**

### `R-HUB-QUARANTINE` — Hub cold-chain quarantine · F-505, F-506 · **used by:** PH-A.U2
1. Quarantine the exposed stock (mark those vials damaged).
2. Raise an ops incident.
3. If quarantine leaves a scheduled visit without stock → `R-STOCKOUT`.
- ⤓ exposed stock quarantined.

### `R-MANUAL-ENTRY` — Manual barcode entry · F-504 · **used by:** PH-A.U3
1. Allow manual entry with verification. — ⤴ continue.

### `R-EXPIRE-QUARANTINE` — Quarantine + replace expired stock · F-506 · **used by:** PH-A.U5
1. Quarantine the expired vial. 2. Replace it.


## PH-B · Batch recall

**Happy**
- `PH-B.H1` Recall a batch — cascade to its vials · F-505, F-618 · may trigger: `PH-B.U1`
- `PH-B.H2` Compute affected patients · F-618

**Unhappy**
- `PH-B.U1` **Batch recalled** (at H1) → `R-RECALL` · F-618

**Recoveries**

### `R-RECALL` — Batch recall → reassign affected patients · F-618 · **used by:** PH-B.U1
1. Flag every vial in the batch as recalled.
2. Identify affected patients (scheduled vs already-injected).
3. Scheduled doses → `R-REPLACE`; already-injected → clinical follow-up + patient comms.

---

# OP — ADMIN / OPS


## OP-A · Patients, providers & roles

**Happy**
- `OP-A.H1` List patients · F-601
- `OP-A.H2` Patient detail + status · F-602, F-603
- `OP-A.H3` Edit Rx protocols · F-651
- `OP-A.H4` Manage roles & permissions · F-655 · may trigger: `OP-A.U1`
- `OP-A.H5` Manage API keys · F-656
- `OP-A.H6` Onboard a provider (doctor / nurse) · F-606, F-609
- `OP-A.H7` Verify credentials · F-403, F-404
- `OP-A.H8` Assign a patient to a provider · F-610 · may trigger: `OP-A.U2`
- `OP-A.H9` User management · F-604, F-605, F-607, F-608

**Unhappy**
- `OP-A.U1` **Role / permission misconfiguration** (at H4) → `R-ROLE-FIX` · F-655
- `OP-A.U2` **No nurse available / no capacity** (at H8) → `R-CAPACITY` · F: —

**Recoveries**

### `R-ROLE-FIX` — Role/permission correction · F-655 · **used by:** OP-A.U1
1. Guardrails + audit catch the misconfiguration. 2. Correct the assignment (API stays source of truth).

### `R-CAPACITY` — No nurse available / no capacity · F: — · **used by:** OP-A.U2
1. Assign from the board, or push an on-demand assignment.
2. If there's truly no capacity → `R-RESCHEDULE` and notify.
- ⤴ assigned when capacity frees.


## OP-B · Routing & fulfillment

**Happy**
- `OP-B.H1` Geocode the patient's address · F-701, F-702 · may trigger: `OP-B.U1`
- `OP-B.H2` Generate the optimized route · F-703, F-704, F-705
- `OP-B.H3` On-demand emergency assignment · F-706, F-406, F-421
- `OP-B.H4` Operations dashboard · F-615, F-616, F-617, F-618 · may trigger: `OP-B.U2`
- `OP-B.H5` Reschedule tool · F-619

**Unhappy**
- `OP-B.U1` **Geocode failed** (at H1) → `R-GEOCODE-MANUAL` · F-701
- `OP-B.U2` **Incident / cold-chain network alert** (at H4) → `R-INCIDENT-TRIAGE` · F-617, F-618

**Recoveries**

### `R-GEOCODE-MANUAL` — Manual geocode pin · F-701 · **used by:** OP-B.U1
1. Allow a manual map-pin. 2. Or retry. — ⤴ continue.

### `R-INCIDENT-TRIAGE` — Incident triage · F-617, F-618 · **used by:** OP-B.U2
1. Triage on the ops incident feed (the must-alert hub). 2. Drive the matching recovery.


## OP-C · Safety-critical recovery (control surface)

**Happy**
- `OP-C.H1` Monitor side-effect reports + assign a reviewer · F-620
- `OP-C.H2` Open a replacement order · F-433

*(No unhappy branches — this is the surface that executes `R-REPLACE`, `R-RECALL`, `R-CAPACITY`, and routes `R-CLINICAL-ESC`.)*

## OP-D · Finance, settlements & BI

**Happy**
- `OP-D.H1` Issue a refund · F-147 · may trigger: `OP-D.U1`
- `OP-D.H2` Run monthly settlements · F-627, F-628, F-629 · may trigger: `OP-D.U2`
- `OP-D.H3` Finance dashboard · F-625, F-626, F-631, F-632, F-633
- `OP-D.H4` Growth dashboard · F-640, F-641
- `OP-D.H5` Campaign manager · F-644 · may trigger: `OP-D.U3`
- `OP-D.H6` Settings · F-652, F-653, F-657
- `OP-D.H7` Reports + export · F-660, F-662, F-663

**Unhappy**
- `OP-D.U1` **Refund / chargeback edge** (at H1) → `R-CHARGEBACK` · F-633
- `OP-D.U2` **Settlement run fails / partial** (at H2) → `R-SETTLE-RETRY` · F-629
- `OP-D.U3` **Bulk action partial failure** (at H5) → `R-BULK-RETRY` · F-644

**Recoveries**

### `R-CHARGEBACK` — Chargeback reconciliation · F-633 · **used by:** OP-D.U1
1. A failed refund, external chargeback, or refund exceeding wallet balance surfaces on the finance dashboard.
2. Finance reconciles by hand.

### `R-SETTLE-RETRY` — Settlement-run failure · F-629 · **used by:** OP-D.U2
1. Show per-item results.
2. Retry only the failed items (idempotent rerun — no double-pay).
- ⤴ completes.

### `R-BULK-RETRY` — Per-row retry · F-644 · **used by:** OP-D.U3
1. Show per-row results. 2. Retry only the failed rows.

---

# CS — CUSTOMER SERVICE


## CS-A · CS console

**Happy**
- `CS-A.H1` Call console + routing · F-700, F-703
- `CS-A.H2` Six CS sub-panels, scoped per panel · F-710, F-761, F-711, F-720, F-721, F-730, F-731, F-740, F-741, F-750, F-751, F-760
- `CS-A.H3` CS messaging + KPI · F-770, F-771
- `CS-A.H4` Emergency escalation in-software · F-443

*(CS executes recoveries owned by other flows — `R-KYC-REVIEW`, `R-RESCHEDULE`, `R-REFUND` — and has no failure of its own.)*

---

# AF — AFFILIATE

## AF-A · Affiliate

**Happy**
- `AF-A.H1` Affiliate dashboard · F-800, F-801
- `AF-A.H2` Referral links + conversion · F-802, F-803 · may trigger: `AF-A.U1`
- `AF-A.H3` Affiliate onboarding + codes + audit · F-807, F-808, F-809 · may trigger: `AF-A.U2`
- `AF-A.H4` Commission + payment · F-804, F-805 · may trigger: `AF-A.U3`
- `AF-A.H5` A/B experiments · F: —

**Unhappy**
- `AF-A.U1` **Referral link invalid / expired** (at H2) → `R-NEW-LINK` · F-802
- `AF-A.U2` **Application rejected** (at H3) → `R-BLOCKED-STATE` · F-807
- `AF-A.U3` **Commission dispute / not yet payable** (at H4) → `R-CS-TICKET` · F-804, F-805

**Recoveries**

### `R-NEW-LINK` — Issue a fresh referral link · F-802 · **used by:** AF-A.U1
1. Show a factual state. 2. Issue a new link.

### `R-CS-TICKET` — Generic CS ticket · F-804, F-805 · **used by:** AF-A.U3
1. Open a standard CS ticket. 2. Apply clawback rules on early churn/refund (window pending).

---


# XF — CROSS-CUTTING (any flow)

**Unhappy** (can interrupt any happy step)
- `XF.U1` **Network offline** → `R-OFFLINE-BANNER` · F: —
- `XF.U2` **Session expired / token invalid** → `R-REAUTH` · F: —
- `XF.U3` **Permission denied (403)** → `R-NO-ACCESS` · F: —
- `XF.U4` **Not found (404) / deleted** → `R-NOT-FOUND` · F: —
- `XF.U5` **Server / dependent-service error (5xx)** → `R-RETRY-5XX` · F: —
- `XF.U6` **Rate limited (429)** → `R-RATE-LIMIT` · F: —
- `XF.U7` **Stale data / concurrent edit conflict** → `R-CONFLICT` · F: —

**Recoveries**

### `R-OFFLINE-BANNER` — Offline banner · F: — · **used by:** XF.U1
1. Show a banner. 2. Allow reads; queue writes where safe (nurse captures a full visit offline).

### `R-REAUTH` — Re-authenticate · F: — · **used by:** XF.U2
1. Silent token refresh, else prompt re-login. 2. Preserve in-progress form data. — ⤴ resume.

### `R-NO-ACCESS` — Permission-denied state · F: — · **used by:** XF.U3
1. Show a full-screen "no access" state (API is source of truth).

### `R-NOT-FOUND` — Not-found state · F: — · **used by:** XF.U4
1. Show "no longer available." 2. Send back to the list.

### `R-RETRY-5XX` — Server-error retry · F: — · **used by:** XF.U5
1. Silent-retry first. 2. Then a visible state; never lose input. — ⤴ resume.

### `R-RATE-LIMIT` — Rate-limit wait · F: — · **used by:** XF.U6
1. Inline/toast with wait guidance. — ⤴ retry.

### `R-CONFLICT` — Concurrent-edit conflict · F: — · **used by:** XF.U7
1. Show "this changed since you loaded it." 2. Offer reload/merge.

---

# RECOVERY LIBRARY (shared across flows)

Only recoveries reused by **more than one flow** live here. Recoveries unique to a single flow are shown inside that flow's section above.

## Shared recovery flows

### `R-RESCHEDULE` — Reschedule a missed/aborted visit · F-422, F-619 · **used by:** PF-D.U1, PF-E.U1, NF-A.U2, OP-A.U2 (overflow)
1. Mark the visit missed/aborted with a reason.
2. Patient self-reschedules, or ops reschedules; a new assigned visit is created.
3. Patient notified (a missed-visit fee may apply — policy pending).
- ⤴ new visit booked.

### `R-RXFIX` — External Rx re-issue + re-link · F-330 · **used by:** PF-D.U2, DF-A.U2, DF-A.U3
1. Show the rejection reason (or auto-retry if the provider is just unreachable).
2. Doctor re-issues the prescription in the external Rx software (keyed by national ID).
3. Re-link the new id; Medica re-fetches the status.
- ⤴ prescription active.

### `R-REPLACE` — Replacement-dose order · F-433, F-504, F-506 · **used by:** NF-A.U3, R-VIAL-RESCAN, R-RECALL
1. Record the trigger (breach/mismatch/recall) and block the injection.
2. Mark the affected vial damaged; abort the visit with reason.
3. Ops opens a replacement order (OP-C.H2).
4. Pharmacy dispenses a fresh vial; a replacement visit is scheduled. Patient not charged for the lost dose (default).
- ⤓ original visit aborted, replacement dispatched.

### `R-STOCKOUT` — Stock-out / shortage at pickup · F-465, F-504 · **used by:** NF-C.U2, PH-A.U4
1. Raise a restock/substitution request.
2. Pharmacy re-dispenses; if the hub is also out, ops sources stock or triggers `R-RESCHEDULE`.
- ⤴ re-dispense or reschedule.

## Shared UI handlings

### `R-BLOCKED-STATE` — Full-screen blocked/rejected state · F: — · **used by:** PF-A.U3, AF-A.U2
1. Show a factual full-screen state. 2. Route to support/ops. — ⤓ no entry.

### `R-INLINE-VALIDATE` — Inline validation + retry · F: — · **used by:** PF-B.U1, MF-A.U1
1. Validate inline (range/type/format) and explain the limit. — ⤴ retry.

### `R-EMPTY-STATE` — Designed empty state · F: — · **used by:** PF-G.U1, DF-B.U2, NF-C.U3, PH-A.U1
1. Render a friendly empty state with a next action (not an error).

### `R-TRACK-FALLBACK` — Tracking fallback · F-172, F-703 · **used by:** PF-E.U2, NF-C.U1
1. Fall back to ETA / last-known position. — ⤴ continue.

---

*Self-contained: the only external reference is the feature id (`F-…`). Recoveries unique to one flow sit in that flow; the eight shared above are referenced by several. Companion: `Medica_Flow_Feature_Map.md`.*
