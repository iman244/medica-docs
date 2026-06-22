# Medica — Unhappy Paths

A standalone companion to the build specification, data model, and lifecycle state machines. It designs what happens when things go wrong: every `⚠` transition from the lifecycle spec, plus the flow failures that never reach a happy state. Each entry is written so the screen and recovery can be designed directly.

---

## How to read this document

Entries are grouped by surface (patient, nurse, doctor, pharmacy, admin/ops, affiliate, cross-cutting). Each entry is:

```
UP-ID — name
  trigger:  what causes it (+ STEP / state transition it maps to)
  who:      actor who sees it
  pattern:  the UI pattern (from the taxonomy below)
  shows:    what appears on screen
  copy:     the intent/tone of the message (not final wording)
  recovery: the way out (every dead-end has at least one exit)
  handling: silent-retry · user-visible · must-alert-human
```

`[clinical]` marks entries whose threshold or medical wording is owned by the clinical-safety spec. `→#6` marks entries handed to the nurse-offline spec. **Decision needed** callouts mark business/clinical policy that must be chosen, not assumed.

**Process diagrams (BPMN):** the decision/abort branches where these entries fire are drawn as swimlanes — `UP-PAT-05` in `P1_onboarding_eligibility_subscription.bpmn`, `UP-DOC-02` in `P2_visit_external_prescription.bpmn`, `UP-NUR-03` in `P4_nurse_safety_sequence.bpmn`, `UP-PAT-14` in `P7_urgent_side_effect.bpmn`, and the recall path in `P6_batch_recall.bpmn`.

### Error-pattern taxonomy (the finite set the design system builds)

| pattern | when to use | dismissible |
|---|---|---|
| `toast` | transient, non-blocking confirmation of a failure that auto-retries or is minor | yes (auto) |
| `inline` | field- or section-level error inside a form/list (validation, one failed item) | n/a |
| `banner` | persistent state affecting the whole screen but not blocking it (past_due, offline) | no, until resolved |
| `full-screen state` | the screen's entire purpose failed or is unavailable (ineligible, no slots, empty) | via recovery action |
| `blocking modal` | must be acknowledged before continuing; safety- or money-critical | only via explicit choice |
| `badge` | status change shown in a list/detail (cancelled, aborted, rejected) | n/a |

---

## A. Patient surface

**UP-PAT-01 — OTP wrong or expired** · serves F-101/102
- trigger: bad/expired code at STEP-1-02/04 · who: anonymous · pattern: inline
- shows: error under the OTP field, resend timer · copy: neutral, "code didn't match or expired," not alarming
- recovery: resend OTP (rate-limited), or change phone number · handling: user-visible
- Decision needed: max OTP attempts before temporary lockout, and lockout duration.
- resolver: self-service (resend / change number) — no RFLOW

**UP-PAT-02 — Login locked out** · serves F-102
- trigger: too many failed logins · who: anonymous · pattern: full-screen state
- shows: temporary lockout with countdown · copy: reassuring, "for your security…" · recovery: wait, or password reset (STEP-1-05) · handling: user-visible
- resolver: self-service (wait / reset) — no RFLOW

**UP-PAT-03 — KYC failed / mismatch** · serves F-105 · `[int·Shahkar]`
- trigger: Shahkar mismatch at STEP-1-07 (kyc_status→failed) · who: patient · pattern: full-screen state
- shows: verification couldn't be completed · copy: non-accusatory, names likely causes (typo, name mismatch) · recovery: retry, edit details, or contact support · handling: user-visible
- Decision needed: how many KYC retries before manual review; can onboarding continue while KYC is pending?
- resolver: ops / CS · RFLOW-20 (KYC manual review)

**UP-PAT-04 — Lab upload too large / wrong type / failed** · serves F-113
- trigger: file rejected or upload error at STEP-2-04 · who: patient · pattern: inline
- shows: per-file error in the uploader, size/type hint · copy: practical, states the limit · recovery: retry, pick another file, skip (uploads optional) · handling: user-visible
- resolver: self-service (retry / skip) — no RFLOW

**UP-PAT-05 — Ineligible result** · serves F-123 · `[clinical]`
- trigger: rule engine returns `ineligible` at STEP-2-07 · who: patient · pattern: full-screen state
- shows: a respectful explanation, no medical jargon · copy: **the most emotionally loaded screen in the product** — kind, clear, non-final-sounding, offers alternatives · recovery: talk to a doctor (paid review / referral), read why, contact support; never a hard dead-end · handling: user-visible
- `[clinical]`: the reasons shown and whether a paid override-review is offered are clinical/business calls.
- resolver: doctor (via ops/CS) · RFLOW-18 (ineligible → paid review)

**UP-PAT-06 — Borderline → needs doctor review** · serves F-125 · `[clinical]`
- trigger: `borderline` at STEP-2-07 · who: patient · pattern: full-screen state
- shows: "a doctor needs to review this first" · copy: positive, not a rejection · recovery: book/await review · handling: user-visible
- resolver: doctor (via ops/CS) · RFLOW-18 (borderline → review)

**UP-PAT-07 — Payment failed / gateway unreachable** · serves F-131 (payment→failed ⚠)
- trigger: gateway callback fail or session expiry at STEP-3A-02 · who: patient · pattern: full-screen state
- shows: payment didn't go through; nothing was charged · copy: reassuring about no double-charge · recovery: retry, switch method, pay from wallet, contact support · handling: user-visible; **idempotency on checkout is mandatory** so retries can't double-charge
- Decision needed: number/cadence of auto-retries before giving up; is the subscription held in `pending_payment` or released?
- resolver: self → CS / finance · RFLOW-12 (payment-failure resolution)

**UP-PAT-08 — Subscription past_due** · serves F-133 (active→past_due ⚠)
- trigger: renewal payment failed · who: patient · pattern: banner (persistent) + badge
- shows: amber "Payment due" across the app, Pay-now CTA · copy: urgent but not punitive; states what lapses and when · recovery: pay now (STEP-3A-05), update method · handling: user-visible + must-alert (dunning notifications)
- Decision needed: the dunning schedule (when/how many reminders) and what features degrade while past_due.
- resolver: self → ops · RFLOW-13 (past_due dunning)

**UP-PAT-09 — Subscription expired mid-treatment** · serves (past_due→expired ⚠) · `[clinical]`
- trigger: grace window elapsed · who: patient · pattern: full-screen state on gated areas
- shows: access limited, treatment paused · copy: **clinically sensitive** — stopping GLP-1 mid-course has health implications; tone must encourage re-subscription and flag talking to a doctor, not just "pay to unlock" · recovery: resubscribe, contact support · handling: user-visible + must-alert (ops)
- Decision needed: grace-window length; whether an in-progress nurse_visit is honored after expiry; clinical messaging `[clinical]`.
- resolver: ops · RFLOW-14 (expired-mid-treatment reactivation)

**UP-PAT-10 — Visit no-show (patient)** · serves (visit→no_show ⚠)
- trigger: patient absent at visit time · who: patient · pattern: badge + prompt
- shows: "missed visit" with rebook · copy: neutral · recovery: rebook (STEP-3B-01) · handling: user-visible
- Decision needed: is a missed visit charged or free to rebook? how many before intervention?
- resolver: self-service (rebook, `API-VISIT-002`) — no RFLOW

**UP-PAT-11 — Nurse on the way but patient not home / unreachable** · serves (nurse_visit→aborted ⚠)
- trigger: nurse can't reach patient (mirrors UP-NUR-04) · who: patient · pattern: badge + prompt
- shows: visit couldn't be completed · copy: neutral, no blame · recovery: reschedule (STEP-4B-04) · handling: user-visible
- Decision needed: missed-home-visit fee policy and cap.
- resolver: ops / nurse · RFLOW-08 (missed-home-visit reschedule)

**UP-PAT-12 — Live tracking unavailable** · serves F-172 · `[int·Optime]`
- trigger: Optime down / no fix · who: patient · pattern: inline (within tracker)
- shows: ETA without live map, or "tracking unavailable" · copy: low-key · recovery: fall back to scheduled time + nurse contact · handling: user-visible (graceful degradation)
- resolver: self-service (graceful degradation) — no RFLOW

**UP-PAT-13 — Wallet refund / cashback pending or failed** · serves F-147, F-144
- trigger: refund initiated but not yet settled, or fails · who: patient · pattern: inline in wallet
- shows: pending/failed transaction row · copy: states expected timing · recovery: contact support · handling: user-visible
- resolver: finance / CS · RFLOW-15 (refund / refund-pending)

**UP-PAT-14 — Side-effect report submitted (urgent)** · serves F-178 · `[clinical]`
- trigger: urgent side-effect at STEP-4B-08 · who: patient · pattern: full-screen confirmation
- shows: "we've flagged this to your doctor," what to do now, emergency-line CTA · copy: **safety-critical** — calm, actionable, when-to-seek-emergency-care guidance `[clinical]` · recovery: emergency line, await doctor · handling: must-alert-human (doctor + ops)
- resolver: doctor + ops · RFLOW-05 (urgent side-effect escalation)

**UP-PAT-15 — Account suspended / banned** · serves (identity→suspended/banned ⚠)
- trigger: admin action · who: patient · pattern: full-screen state (login blocked)
- shows: account unavailable · copy: factual, support contact · recovery: contact support · handling: user-visible
- resolver: CS → admin · generic CS ticket (account reinstatement)

**UP-PAT-16 — Empty states** (not failures, but designed here for consistency)
- no visits yet, no injections yet, no addresses, empty wallet history, no content · pattern: full-screen/empty state · copy: encouraging, points to the first action · recovery: the primary CTA for that screen.
- resolver: n/a (empty states)

---

## B. Nurse surface

**UP-NUR-01 — Cold-chain breach → do not administer** · serves F-433 (nurse_visit→blocked ⚠) · `[clinical]`
- trigger: temperature out of range at STEP-4A-06 · who: nurse · pattern: blocking modal
- shows: hard block — injection cannot proceed · copy: **safety-critical, unambiguous** — "do not administer," next steps · recovery: report damaged vial (STEP-5-05), abort visit, contact ops; **cannot** bypass to injection · handling: must-alert-human (ops + pharmacy)
- `[clinical]`: the temperature thresholds and the exact protocol are owned by the clinical spec.
- resolver: pharmacy + ops · RFLOW-01 (cold-chain → replacement dose)

**UP-NUR-02 — Wrong / mismatched vial scanned** · serves F-437 · `[clinical]`
- trigger: scanned vial doesn't match prescription/patient, or is expired/recalled at STEP-4A-08 · who: nurse · pattern: blocking modal
- shows: mismatch warning, blocks recording · copy: safety-critical, states the mismatch · recovery: rescan correct vial, report issue, abort · handling: must-alert-human
- `[clinical]`: matching rules (dose, drug, recall) owned by clinical spec.
- resolver: nurse → ops · RFLOW-03 (wrong / mismatched vial)

**UP-NUR-03 — GPS check-in fails / location off** · serves F-430
- trigger: no GPS fix or location far from address at STEP-4A-04 · who: nurse · pattern: blocking modal (soft)
- shows: can't confirm location · copy: practical · recovery: retry, enable location, manual-confirm with reason (logged), contact ops · handling: user-visible
- Decision needed: is a manual override allowed, and is it logged for audit?
- resolver: nurse (logged override) → ops · generic CS/ops ticket

**UP-NUR-04 — Patient not home / refuses / unsafe** · serves (nurse_visit→aborted ⚠)
- trigger: nurse on site, can't complete · who: nurse · pattern: full-screen state
- shows: abort flow with reason picker · copy: neutral, structured reasons · recovery: mark aborted (STEP-4A-11), trigger reschedule, safety button if unsafe · handling: user-visible + must-alert (ops); safety reason → must-alert-human
- Decision needed: which abort reasons are billable; what reasons auto-escalate.
- resolver: nurse → ops · RFLOW-10 (abort; safety reason → RFLOW-06)

**UP-NUR-05 — Safety button pressed** · serves F-497
- trigger: nurse triggers safety alert at STEP-4A-12 · who: nurse + ops · pattern: blocking modal (nurse) + must-alert (ops)
- shows: confirmation + emergency contacts · copy: safety-critical, minimal · recovery: call, share location · handling: must-alert-human (immediate)
- resolver: ops / CS · RFLOW-06 (nurse safety escalation)

**UP-NUR-06 — Offline mid-visit** · serves F-430–F-442 · →#6
- trigger: no connectivity during the home visit · who: nurse · pattern: banner (offline) + queued state
- shows: "saved locally, will sync" indicators on each action · copy: reassuring, data is safe · recovery: continue offline, auto-sync on reconnect · handling: user-visible — **full treatment in the nurse-offline spec (#6)**
- resolver: deferred (offline spec dropped) — no RFLOW

**UP-NUR-07 — Route unavailable / Optime down** · serves F-703 · `[int·Optime]`
- trigger: no optimized route at STEP-4A-03/5-08 · who: nurse · pattern: banner
- shows: fall back to unordered assigned list + manual nav · copy: low-key · recovery: work the list manually, contact ops · handling: user-visible (graceful degradation)
- resolver: self-service (work the list manually) — no RFLOW

**UP-NUR-08 — Signature capture fails / patient can't sign** · serves F-441
- trigger: device/signature issue at STEP-4A-09 · who: nurse · pattern: inline
- shows: alternative confirmation (OTP to patient phone) · copy: practical · recovery: OTP fallback, retry · handling: user-visible
- resolver: self-service (OTP fallback) — no RFLOW

**UP-NUR-09 — Inventory shortage at pickup / count mismatch** · serves F-461/465
- trigger: missing or damaged vials at pickup (STEP-5-12) · who: nurse · pattern: inline + report
- shows: discrepancy entry · copy: factual · recovery: report shortage/damage, restock request (STEP-5-12) · handling: user-visible + must-alert (pharmacy/ops)
- resolver: pharmacy + ops · RFLOW-09 (stock-out / shortage)

**UP-NUR-10 — Empty states**: no visits assigned today, no patients assigned, empty inventory · pattern: empty state · recovery: contact ops / await assignment.
- resolver: n/a (empty states)

---

## C. Doctor surface

**UP-DOC-01 — External prescription rejected / invalid** · serves F-330 (prescription→rejected ⚠) · `[int·RxProvider]`
- trigger: the fetched status from the external e-prescription provider is rejected/invalid at STEP-3B-07/08 · who: doctor · pattern: badge + inline
- shows: rejected status + reason from the provider · copy: states the rejection reason · recovery: re-issue in the **external** Rx software and re-link the new id (STEP-3B-07) · handling: user-visible + must-alert (doctor)
- resolver: doctor · RFLOW-19 (external prescription rejected → re-issue + re-link)

**UP-DOC-02 — External Rx provider unreachable** · serves F-330 · `[int·RxProvider]`
- trigger: the e-prescription provider is unreachable on link/fetch · who: doctor · pattern: toast + retry-queue
- shows: "will fetch when available" · copy: reassuring · recovery: auto-retry the fetch, manual refresh · handling: silent-retry then user-visible
- resolver: self-service (auto-retry / refresh) — no RFLOW

**UP-DOC-03 — Patient record incomplete at visit** · serves F-312
- trigger: missing eligibility/history when opening EMR at STEP-3B-05 · who: doctor · pattern: inline within EMR
- shows: gaps highlighted · copy: factual · recovery: request patient complete it / proceed with note · handling: user-visible
- resolver: doctor self / patient — no RFLOW

**UP-DOC-04 — Assignment lost mid-care** · serves :assigned scope
- trigger: doctor unassigned from a patient they were treating (assignment→inactive) · who: doctor · pattern: full-screen state on that patient
- shows: no longer assigned · copy: factual, contact ops · recovery: contact ops · handling: user-visible
- Decision needed: are in-flight visits grandfathered when an assignment is removed?
- resolver: ops · RFLOW-11 (provider reassignment / in-flight care)

**UP-DOC-05 — Provider deactivated** · serves (provider_profile→inactive ⚠)
- trigger: admin deactivates · who: doctor · pattern: full-screen state
- shows: work access suspended · copy: factual, contact ops · recovery: contact ops · handling: user-visible
- resolver: ops / admin · generic CS ticket

**UP-DOC-06 — Empty states**: no patients assigned, no visits today, no income yet.
- resolver: n/a (empty states)

---

## D. Pharmacy surface

**UP-PHM-01 — Hub cold-chain breach** · serves F-505 (coldchain_log.breach) · `[clinical]`
- trigger: fridge out of range · who: pharmacy + ops · pattern: blocking modal + must-alert
- shows: affected stock flagged · copy: safety-critical · recovery: quarantine stock, report, contact ops · handling: must-alert-human
- `[clinical]`: thresholds and what happens to exposed stock owned by clinical spec.
- resolver: pharmacy + ops · RFLOW-04 (hub cold-chain quarantine)

**UP-PHM-02 — Batch recall** · serves (batch.recall_status→recalled ⚠) · `[clinical/ops]`
- trigger: supplier/admin recall · who: pharmacy + ops + nurses holding stock · pattern: blocking modal + cascade
- shows: recalled vials flagged unusable everywhere they sit · copy: safety-critical · recovery: quarantine, return, reassign affected patients · handling: must-alert-human (cascades to nurse inventory)
- Decision needed: patient-facing communication when a recall affects already-scheduled doses.
- resolver: ops + pharmacy · RFLOW-02 (batch recall → reassign)

**UP-PHM-03 — Barcode scan fails at dispense** · serves F-504
- trigger: unreadable/unknown barcode at STEP-5-04 · who: pharmacy · pattern: inline
- shows: scan error · copy: practical · recovery: rescan, manual entry (logged), report · handling: user-visible
- resolver: self-service (manual entry, logged) — no RFLOW

**UP-PHM-04 — Stock-out / cannot fulfill dispense** · serves F-502/504
- trigger: no usable vial for a scheduled visit · who: pharmacy + ops · pattern: inline + alert
- shows: shortage flag against the visit · copy: factual · recovery: restock, reassign, ops escalation · handling: must-alert (ops) — risks a missed dose
- Decision needed: substitution policy and how far ahead stock-outs are predicted/alerted.
- resolver: pharmacy + ops · RFLOW-09 (stock-out / cannot fulfill)

**UP-PHM-05 — Expired stock detected** · serves (vial→expired ⚠)
- trigger: expiry passes · who: pharmacy · pattern: badge + inline · recovery: quarantine/dispose, record · handling: user-visible.
- resolver: pharmacy self (quarantine / record) — no RFLOW

**UP-PHM-06 — Empty states**: no inventory, no pending dispenses.
- resolver: n/a (empty states)

---

## E. Admin / ops surface

**UP-ADM-01 — Settlement run fails / partial** · serves (settlement_run→failed ⚠)
- trigger: settlement batch error at STEP-6-08 · who: finance/admin · pattern: full-screen state + per-item inline
- shows: which items failed, which succeeded · copy: factual, actionable · recovery: retry failed items, manual adjust · handling: user-visible; **idempotent settlement** so reruns don't double-pay
- Decision needed: are partial settlements committed or all-or-nothing per run?
- resolver: finance · RFLOW-17 (settlement-run failure)

**UP-ADM-02 — Refund / chargeback edge** · serves F-147/633
- trigger: refund exceeds balance, or external chargeback arrives · who: admin/finance · pattern: inline + alert
- shows: the discrepancy · copy: factual · recovery: manual reconciliation · handling: must-alert (finance)
- Decision needed: refund policy when wallet has been spent down; chargeback handling workflow.
- resolver: finance · RFLOW-15 / RFLOW-16 (refund / chargeback reconciliation)

**UP-ADM-03 — Geocode failed for an address** · serves F-701 · `[int·Optime]`
- trigger: address won't geocode at STEP-5-07 · who: ops · pattern: inline on the patient/address
- shows: address needs manual pin · copy: practical · recovery: manual map pin, contact patient · handling: user-visible (blocks routing until fixed)
- resolver: ops (manual map pin) · generic — no RFLOW

**UP-ADM-04 — No nurse available / no capacity** · serves assignment/routing
- trigger: no nurse can cover a visit's area/time · who: ops · pattern: full-screen state / inline on the visit
- shows: unassignable visit flagged · copy: factual · recovery: manual assign, reschedule, expand coverage · handling: must-alert (ops) — risks a missed dose
- Decision needed: how far ahead capacity gaps are surfaced; patient-facing message when a visit can't be staffed.
- resolver: ops · RFLOW-07 (no nurse available)

**UP-ADM-05 — Incident / cold-chain network alert** · serves F-617/618
- trigger: any must-alert event lands in ops · who: ops · pattern: must-alert feed + badge
- shows: prioritized incident list · copy: severity-coded · recovery: triage workflow · handling: must-alert-human
- resolver: ops (triage hub) — destination for must-alert RFLOWs, not its own RFLOW

**UP-ADM-06 — Role/permission misconfiguration** · serves F-655
- trigger: a role left without needed permissions, or a user with conflicting roles · who: admin · pattern: inline warning in RoleManager
- shows: validation warning before save · copy: explains the gap · recovery: fix before save · handling: user-visible
- resolver: admin self — no RFLOW

**UP-ADM-07 — Bulk action partial failure** (imports, campaigns): per-row results, retry failed rows.
- resolver: admin self — no RFLOW

---

## F. Affiliate surface

**UP-AFF-01 — Referral link invalid / expired** · serves F-802
- trigger: bad or expired referral code · who: prospective patient · pattern: inline at signup
- shows: code not applied · copy: neutral · recovery: proceed without code, contact affiliate · handling: user-visible
- resolver: self-service (proceed without code) — no RFLOW

**UP-AFF-02 — Commission dispute / not yet payable** · serves F-804/805
- trigger: conversion not yet at payable stage, or disputed · who: affiliate · pattern: inline in commission panel
- shows: pending/held status with reason · copy: factual · recovery: await stage, contact ops · handling: user-visible
- Decision needed: commission clawback rules if a referred patient refunds/churns early.
- resolver: ops / finance · generic CS ticket (commission dispute / clawback)

**UP-AFF-03 — Affiliate application rejected** · serves F-807 · pattern: full-screen state · recovery: contact ops.
- resolver: ops · generic CS ticket

---

## F2. Public / landing surface (pre-auth)

**UP-MKT-01 — BMI calculator invalid input** · serves F-096
- trigger: out-of-range/empty height or weight at STEP-1-09 · who: anonymous · pattern: inline
- shows: per-field validation with sensible bounds · copy: practical · recovery: correct input (client-side) · handling: user-visible
- resolver: self-service — no RFLOW

**UP-MKT-02 — Public eligibility result: not a fit / borderline** · serves F-097 · `[clinical]`
- trigger: preview indicator negative/borderline at STEP-2-10 · who: anonymous · pattern: full-screen state (within the widget)
- shows: a **non-binding**, respectful message — never a definitive medical verdict · copy: **`[clinical]`** — kind, non-final, encourages a proper assessment; not a rejection · recovery: sign up for the full assessment (→ authenticated STEP-2-07; the ineligible→review path is RFLOW-18 post-signup), read more, contact support · handling: user-visible
- resolver: self-service → signup; post-signup clinical path RFLOW-18

**UP-MKT-03 — Public eligibility preview unavailable** · serves F-097
- trigger: preview service down / rule unavailable · who: anonymous · pattern: inline/toast
- shows: "couldn't check right now" · copy: low-key · recovery: **still allow signup** (never block the funnel), retry · handling: graceful degradation
- resolver: self-service — no RFLOW

---

## G. Cross-cutting failures (all surfaces)

**UP-X-01 — Network offline** · pattern: banner; reads/queues writes where safe; full offline capture for nurse →#6. Decision needed: which write actions are safe to queue per surface vs. must block.
- resolver: self-service / platform — no RFLOW

**UP-X-02 — Session expired / token invalid** · pattern: blocking modal → re-auth (refresh via `API-AUTH-010`, else re-login); preserve in-progress form data where possible.
- resolver: self-service / platform — no RFLOW

**UP-X-03 — Permission denied (403)** · pattern: full-screen state; should be rare if UI hides unavailable actions, but the API is the source of truth (a multi-role user hitting a scope they lack). Copy: factual, "you don't have access," no detail leakage.
- resolver: self-service / platform — no RFLOW

**UP-X-04 — Not found (404) / resource deleted** · pattern: full-screen state; "no longer available," back to list.
- resolver: self-service / platform — no RFLOW

**UP-X-05 — Server / dependent-service error (5xx)** · pattern: toast for transient (with auto-retry) escalating to full-screen state if persistent; never lose user input. Handling: silent-retry → user-visible.
- resolver: self-service / platform — no RFLOW

**UP-X-06 — Rate limited (429)** · pattern: inline/toast with wait guidance.
- resolver: self-service / platform — no RFLOW

**UP-X-07 — Stale data / concurrent edit conflict** · pattern: inline; "this changed since you loaded it," reload/merge. Relevant to admin and provider edits.
- resolver: self-service / platform — no RFLOW

---

## H. Patterns that recur (build once, reuse)

- **No double side-effects on retry.** Payments (UP-PAT-07), injection submit (UP-NUR-06 sync), and settlements (UP-ADM-01) must be idempotent so a retry can't charge twice, record an injection twice, or double-pay. This is an API-convention concern flagged for that spec.
- **Every dead-end has an exit.** No unhappy path leaves the user with no action — minimum is "contact support."
- **Safety-critical → human, always.** Cold-chain breach, vial mismatch, safety button, urgent side-effect, recall: never auto-resolved silently; always alert a human.
- **Graceful degradation for integrations.** Optime (tracking/route), TUMS (submit), datalogger, gateway: when the third party is down, fall back to a manual/scheduled path rather than blocking the whole flow.

---

## I. Open decisions surfaced by this document

These are business/clinical policy calls this document deliberately does not assume:

1. OTP/login attempt limits and lockout durations (UP-PAT-01/02).
2. KYC retry count and whether onboarding proceeds while pending (UP-PAT-03).
3. Ineligible: reasons shown + whether a paid doctor-override review exists `[clinical]` (UP-PAT-05).
4. Payment auto-retry cadence; hold vs. release subscription on failure (UP-PAT-07).
5. Dunning schedule + what degrades while `past_due` (UP-PAT-08).
6. `past_due → expired` grace length; honoring in-flight visits; clinical messaging `[clinical]` (UP-PAT-09).
7. Missed online-visit and missed-home-visit fee policy (UP-PAT-10/11).
8. Manual GPS-override allowance + audit (UP-NUR-03).
9. Billable abort reasons + auto-escalation reasons (UP-NUR-04).
10. In-flight care when an assignment is removed (UP-DOC-04).
11. Recall patient-communication when scheduled doses are affected (UP-PHM-02).
12. Substitution + stock-out prediction policy (UP-PHM-04); no-capacity surfacing (UP-ADM-04).
13. Partial vs all-or-nothing settlements (UP-ADM-01); refund-after-spend + chargeback workflow (UP-ADM-02).
14. Commission clawback on early churn/refund (UP-AFF-02).
15. Which write actions are safe to queue offline per surface (UP-X-01).

**Operationalized by recovery flows.** Several of these decisions are now carried by a flow in `Medica_Recovery_Flows.md` — the flow provides the *mechanism* while the *value* stays open here: #3 (RFLOW-18), #5 (RFLOW-13), #6 (RFLOW-14), #7 (RFLOW-08), #9 (RFLOW-10), #10 (RFLOW-11), #11 (RFLOW-02), #12 (RFLOW-07 / RFLOW-09), #13 (RFLOW-16 / RFLOW-17), #14 (generic CS ticket, §4.C). The decisions themselves remain open and are marked `Decision needed`/`[clinical]` inside those flows.

---

## Notes for the next specs

- Offline behavior (UP-NUR-06, UP-X-01) is stubbed here and fully designed in the nurse-offline spec (#6).
- The **resolution** of every entry that hands off to another actor (the `resolver:` line) is designed step-by-step in the recovery-flows spec (`Medica_Recovery_Flows.md`); this document owns the screen and copy, that one owns the actor-by-actor flow.
- Every `copy:` line describes an **intent**, not final wording: each resolves to a localized message key per the user's locale, and every pattern mirrors correctly in RTL/LTR (`Medica_Internationalization.md`). Clinical copy is additionally owned/approved per locale by the clinical-safety spec.
- All `[clinical]` thresholds and wording (cold-chain, vial match, ineligible reasons, urgent side-effect guidance, recall) are owned by the clinical-safety spec.
- Idempotency requirements raised in §H belong to the API-conventions spec.
