# Medica — Task Plan & Estimates

Re-keyed to the flow ids in `Medica_User_Flows.md`. **Every flow id is a task** — happy step (`.H`), unhappy branch (`.U`), recovery (`.R`) — organized by flow, in User-Flows order. **1 point ≈ 2 hours.**

**Rubric.** *Happy* carries the build: API subtask 1 pt (`GET`)/2 pt (mutation), +1 clinical/payment/routing, +1 third-party, cap 4; component 1 pt (display)/2 pt (form/wizard/queue/dashboard)/3 pt (live/video). *Unhappy* = 1 pt (shared error-state + branch wiring). *Recovery* = 1 pt (UI handling) or 2 pt (multi-actor flow), +1 `[clinical]`; new build a recovery needs is counted under its happy control step (`OP-C`, `PH-B`).

Dependencies: happy chains within a flow; unhappy depends on its anchor happy step; recovery depends on the branch(es) that use it.
**Totals: 530 points** — happy 400 · unhappy 59 · recovery 71.

| flow | pts |
|---|--:|
| PF-A | 34 |
| PF-B | 41 |
| PF-C | 31 |
| PF-D | 16 |
| PF-E | 25 |
| PF-F | 8 |
| PF-G | 30 |
| MF-A | 11 |
| DF-A | 27 |
| DF-B | 24 |
| NF-A | 44 |
| NF-B | 14 |
| NF-C | 32 |
| PH-A | 30 |
| PH-B | 12 |
| OP-A | 31 |
| OP-B | 18 |
| OP-C | 11 |
| OP-D | 34 |
| CS-A | 11 |
| AF-A | 19 |
| XF | 14 |
| (shared recoveries) | 13 |
| **total** | **530** |
---

## PF-A · Account & login — 34 pts

### `PF-A.H1` Enter phone + password, request an OTP — **5 pts**
*happy · depends on: — · STEP-1-01*
- API subtasks:
  - `API-AUTH-001` · POST /auth/register · auth — create account, trigger OTP **(3 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-PAT-001` · SignUpForm · mfe-patient — phone + password + request OTP **(2 pt)**

### `PF-A.H2` Verify the phone OTP — **5 pts**
*happy · depends on: PF-A.H1 · STEP-1-02*
- API subtasks:
  - `API-AUTH-002` · POST /auth/otp/verify · auth — verify OTP, issue tokens **(3 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-PAT-001` · SignUpForm · mfe-patient — OTP entry **(2 pt)**

### `PF-A.H3` Log in (phone + password, or phone + OTP) — **9 pts**
*happy · depends on: PF-A.H2 · STEP-1-03/STEP-1-04*
- API subtasks:
  - `API-AUTH-003` · POST /auth/login · auth — phone + password → tokens **(2 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-PAT-002` · LoginForm · mfe-patient — phone + password **(2 pt)**
- API subtasks:
  - `API-AUTH-003` · POST /auth/login · auth — phone + OTP → tokens **(3 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-PAT-002` · LoginForm · mfe-patient — OTP toggle **(2 pt)**

### `PF-A.H4` Recover password — **4 pts**
*happy · depends on: PF-A.H3 · STEP-1-05*
- API subtasks:
  - `API-AUTH-004/005` · POST /auth/password/reset/{request,confirm} · auth — SMS reset + set password **(3 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-PAT-003` · PasswordResetFlow · mfe-patient — request + confirm **(1 pt)**

### `PF-A.H5` Verify identity / KYC — **3 pts**
*happy · depends on: PF-A.H4 · STEP-1-07*
- API subtasks:
  - `API-AUTH-007` · POST /me/kyc/verify · auth — submit national ID, return status **(2 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-PAT-004` · KycVerification · mfe-patient — national-ID capture + result **(1 pt)**

### `PF-A.U1` OTP wrong or expired — **1 pt**
*unhappy · at PF-A.H2 · resolved by `R-RESEND-OTP`*

### `PF-A.U2` Locked out — **1 pt**
*unhappy · at PF-A.H3 · resolved by `R-COOLDOWN`*

### `PF-A.U3` Account suspended / banned — **1 pt**
*unhappy · at PF-A.H3 · resolved by `R-BLOCKED-STATE`*

### `PF-A.U4` KYC fails or mismatch — **1 pt**
*unhappy · at PF-A.H5 · resolved by `R-KYC-REVIEW`*

### `R-RESEND-OTP` Resend a fresh OTP — **1 pts**
*recovery (UI handling)*

### `R-COOLDOWN` Lockout cooldown — **1 pts**
*recovery (UI handling)*

### `R-KYC-REVIEW` KYC manual review — **2 pts**
*recovery (recovery flow)*

## PF-B · Onboarding (profile → eligibility → consent) — 41 pts

### `PF-B.H1` Complete basic profile — **4 pts**
*happy · depends on: — · STEP-2-01*
- API subtasks:
  - `API-PATIENT-001` · PUT /me/profile · patient — save profile **(2 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-PAT-010` · ProfileForm · mfe-patient — basic profile **(2 pt)**

### `PF-B.H2` Enter height/weight, see BMI — **4 pts**
*happy · depends on: PF-B.H1 · STEP-2-02*
- API subtasks:
  - `API-PATIENT-002` · PUT /me/health · patient — save measurement, compute BMI **(2 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-PAT-011` · HealthDataForm · mfe-patient — height/weight + BMI **(2 pt)**

### `PF-B.H3` Enter medical history — **4 pts**
*happy · depends on: PF-B.H2 · STEP-2-03*
- API subtasks:
  - `API-PATIENT-003` · PUT /me/history · patient — save history **(2 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-PAT-012` · MedicalHistoryForm · mfe-patient — history **(2 pt)**

### `PF-B.H4` Upload lab results — **4 pts**
*happy · depends on: PF-B.H3 · STEP-2-04*
- API subtasks:
  - `API-PATIENT-004` · POST /me/labs · patient — store lab file **(2 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-PAT-013` · LabUploader · mfe-patient — upload + thumbnails **(2 pt)**

### `PF-B.H5` Set delivery address — **4 pts**
*happy · depends on: PF-B.H4 · STEP-2-05*
- API subtasks:
  - `API-PATIENT-005` · PUT /me/address · patient — save delivery address **(2 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-PAT-014` · AddressForm · mfe-patient — address entry **(2 pt)**

### `PF-B.H6` Answer eligibility questionnaire — **3 pts**
*happy · depends on: PF-B.H5 · STEP-2-06*
- API subtasks:
  - `API-PATIENT-020` · GET /eligibility/questionnaire · patient — question set **(1 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-PAT-020` · EligibilityWizard · mfe-patient — questionnaire **(2 pt)**

### `PF-B.H7` Compute eligibility, show result — **3 pts**
*happy · depends on: PF-B.H6 · STEP-2-07*
- API subtasks:
  - `API-PATIENT-021` · POST /me/eligibility/answers · patient — rule engine + persist **(2 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-PAT-021` · EligibilityResult · mfe-patient — result + recommendation **(1 pt)**

### `PF-B.H8` Sign informed consent — **4 pts**
*happy · depends on: PF-B.H7 · STEP-2-08*
- API subtasks:
  - `API-PATIENT-022` · POST /me/consent · patient — record consent **(2 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-PAT-022` · ConsentForm · mfe-patient — consent capture **(2 pt)**

### `PF-B.H9` Receive confirmations — **3 pts**
*happy · depends on: PF-B.H8 · STEP-2-09*
- API subtasks:
  - `API-NOTIF-001/002` · POST /notify/{sms,push} · notif — send message **(2 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-SHL-005` · NotificationCenter · shell — in-app + push **(1 pt)**

### `PF-B.H10` Land on patient home, ready to subscribe — **2 pts**
*happy · depends on: PF-B.H9 · STEP-2-11*
- API subtasks:
  - `API-PATIENT-025` · GET /me/home · patient — home summary (next visit, treatment status, quick actions, alerts) **(1 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-PAT-000` · PatientHome · mfe-patient — post-login hub (the patient's main screen; precedes notifications and other patient sections) **(1 pt)**

### `PF-B.U1` Lab upload rejected — **1 pt**
*unhappy · at PF-B.H4 · resolved by `R-INLINE-VALIDATE`*

### `PF-B.U2` Borderline eligibility — **1 pt**
*unhappy · at PF-B.H7 · resolved by `R-ELIG-REVIEW`*

### `PF-B.U3` Ineligible — **1 pt**
*unhappy · at PF-B.H7 · resolved by `R-ELIG-REVIEW`*

### `R-ELIG-REVIEW` Paid doctor eligibility review — **3 pts**
*recovery (recovery flow, clinical)*

## PF-C · Subscription & payment — 31 pts

### `PF-C.H1` View packages — **3 pts**
*happy · depends on: — · STEP-3A-01*
- API subtasks:
  - `API-SUB-001` · GET /subscriptions/packages · sub — package catalog **(1 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-PAT-030` · PackagePicker · mfe-patient — package cards **(2 pt)**

### `PF-C.H2` Purchase & pay — **7 pts**
*happy · depends on: PF-C.H1 · STEP-3A-02*
- API subtasks:
  - `API-SUB-002` · POST /me/subscriptions · sub — create subscription **(2 pt)**
  - `API-PAY-001/002` · POST /payments/{checkout,callback} · pay — gateway session + verify **(3 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-PAT-031` · CheckoutFlow · mfe-patient — gateway redirect + result **(2 pt)**

### `PF-C.H3` Manage / stop renewal — **4 pts**
*happy · depends on: PF-C.H2 · STEP-3A-03*
- API subtasks:
  - `API-SUB-003` · PATCH /me/subscriptions/{subId}/renewal · sub — stop/resume **(2 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-PAT-032` · SubscriptionManager · mfe-patient — renewal controls **(2 pt)**

### `PF-C.H4` Cancel + exit interview — **3 pts**
*happy · depends on: PF-C.H3 · STEP-3A-04*
- API subtasks:
  - `API-SUB-004` · POST /me/subscriptions/{subId}/cancel · sub — cancel + interview **(2 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-PAT-034` · ExitInterview · mfe-patient — cancellation survey **(1 pt)**

### `PF-C.H5` Wallet — balance, top-up, pay — **1 pts**
*happy · depends on: PF-C.H4 · STEP-3A-05*
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-PAT-033` · WalletView · mfe-patient — balance + top-up + transactions **(1 pt)**

### `PF-C.U1` Payment failed / gateway unreachable — **1 pt**
*unhappy · at PF-C.H2 · resolved by `R-PAY-RETRY`*

### `PF-C.U2` Renewal failed → past due — **1 pt**
*unhappy · at PF-C.H3 · resolved by `R-DUNNING`*

### `PF-C.U3` Expired mid-treatment — **1 pt**
*unhappy · at PF-C.H3 · resolved by `R-REACTIVATE`*

### `PF-C.U4` Refund / cashback pending or failed — **1 pt**
*unhappy · at PF-C.H5 · resolved by `R-REFUND`*

### `R-PAY-RETRY` Payment-failure resolution — **2 pts**
*recovery (recovery flow)*

### `R-DUNNING` Past-due dunning — **2 pts**
*recovery (recovery flow)*

### `R-REACTIVATE` Expired-mid-treatment reactivation — **3 pts**
*recovery (recovery flow, clinical)*

### `R-REFUND` Refund / refund-pending — **2 pts**
*recovery (recovery flow)*

## PF-D · Online doctor visit (patient side) — 16 pts

### `PF-D.H1` Book a visit — **5 pts**
*happy · depends on: — · STEP-3B-01*
- API subtasks:
  - `API-VISIT-001` · GET /visits/availability · visit — open slots **(1 pt)**
  - `API-VISIT-002` · POST /me/visits · visit — book visit **(2 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-PAT-040` · VisitBooking · mfe-patient — slot picker **(2 pt)**

### `PF-D.H2` Get a reminder — **3 pts**
*happy · depends on: PF-D.H1 · STEP-3B-02*
- API subtasks:
  - `API-NOTIF-003` · POST /notify/schedule · notif — queue reminders **(2 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-PAT-041` · VisitReminderBanner · mfe-patient — banner **(1 pt)**

### `PF-D.H3` Join the phone visit — **1 pts**
*happy · depends on: PF-D.H2 · STEP-3B-03*
- (no API/component — config/integration step)

### `PF-D.H4` View & download the prescription — **2 pts**
*happy · depends on: PF-D.H3 · STEP-3B-10*
- API subtasks:
  - `API-VISIT-003` · GET /me/visits/{visitId} · visit — own visit detail **(1 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-PAT-042` · PrescriptionView · mfe-patient — show Rx + download PDF **(1 pt)**

### `PF-D.H5` Raise an emergency request — **3 pts**
*happy · depends on: PF-D.H4 · STEP-3B-11*
- API subtasks:
  - `API-VISIT-008` · POST /me/visits/emergency · visit — raise emergency **(2 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-PAT-043` · EmergencyRequestButton · mfe-patient — raise emergency **(1 pt)**

### `PF-D.U1` Patient no-show — **1 pt**
*unhappy · at PF-D.H3 · resolved by `R-RESCHEDULE`*

### `PF-D.U2` Prescription not ready — **1 pt**
*unhappy · at PF-D.H4 · resolved by `R-RXFIX`*

## PF-E · Home nurse visit & tracking (patient side) — 25 pts

### `PF-E.H1` See schedule + nurse profile — **2 pts**
*happy · depends on: — · STEP-4B-01*
- API subtasks:
  - `API-FIELD-020` · GET /me/nurse-visits · field — own schedule + nurse **(1 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-PAT-050` · NurseVisitSchedule · mfe-patient — schedule + nurse **(1 pt)**

### `PF-E.H2` Verify nurse identity on arrival — **2 pts**
*happy · depends on: PF-E.H1 · STEP-4B-02*
- API subtasks:
  - `API-PROV-010` · GET /me/nurse-visits/{id}/nurse-verify · prov — nurse code/photo **(1 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-PAT-051` · NurseIdentityVerify · mfe-patient — verify nurse **(1 pt)**

### `PF-E.H3` Confirm or change the time — **4 pts**
*happy · depends on: PF-E.H2 · STEP-4B-04*
- API subtasks:
  - `API-FIELD-021` · POST /me/nurse-visits/{id}/reschedule · field — confirm/change time **(2 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-PAT-053` · VisitTimeManager · mfe-patient — confirm/change **(2 pt)**

### `PF-E.H4` Confirm medication received — **3 pts**
*happy · depends on: PF-E.H3 · STEP-4B-03*
- API subtasks:
  - `API-FIELD-022` · POST /me/nurse-visits/{id}/confirm-receipt · field — medication receipt **(2 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-PAT-052` · MedReceiptConfirm · mfe-patient — confirm meds **(1 pt)**

### `PF-E.H5` Confirm service received — **3 pts**
*happy · depends on: PF-E.H4 · STEP-4B-05*
- API subtasks:
  - `API-FIELD-022` · POST /me/nurse-visits/{id}/confirm-receipt · field — service receipt **(2 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-PAT-054` · ServiceReceipt · mfe-patient — confirm service **(1 pt)**

### `PF-E.H6` Track the nurse live — **4 pts**
*happy · depends on: PF-E.H5 · STEP-4B-09*
- API subtasks:
  - `API-FIELD-034` · GET /me/nurse-visits/{id}/track · field — live position + ETA **(1 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-PAT-057` · NurseTracker · mfe-patient — live map **(3 pt)**

### `PF-E.H7` See injection history — **2 pts**
*happy · depends on: PF-E.H6 · STEP-4B-06*
- API subtasks:
  - `API-FIELD-023` · GET /me/injections · field — own injection history **(1 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-PAT-055` · InjectionHistory · mfe-patient — history **(1 pt)**

### `PF-E.H8` Next-injection reminder — **3 pts**
*happy · depends on: PF-E.H7 · STEP-4B-07*
- API subtasks:
  - `API-NOTIF-003` · POST /notify/schedule · notif — schedule reminder **(2 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-SHL-005` · NotificationCenter · shell — reminder **(1 pt)**

### `PF-E.U1` Patient not home / unreachable — **1 pt**
*unhappy · at PF-E.H3 · resolved by `R-RESCHEDULE`*

### `PF-E.U2` Live tracking unavailable — **1 pt**
*unhappy · at PF-E.H6 · resolved by `R-TRACK-FALLBACK`*

## PF-F · Side-effect report — 8 pts

### `PF-F.H1` Report a side effect — **4 pts**
*happy · depends on: — · STEP-4B-08*
- API subtasks:
  - `API-PATIENT-030` · POST /me/side-effects · patient — report; `urgent` set by clinical ruleset `[clinical]` **(3 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-PAT-056` · SideEffectReporter · mfe-patient — report **(1 pt)**

### `PF-F.U1` Urgent side effect — **1 pt**
*unhappy · at PF-F.H1 · resolved by `R-CLINICAL-ESC`*

### `R-CLINICAL-ESC` Urgent side-effect clinical escalation — **3 pts**
*recovery (recovery flow, clinical)*

## PF-G · Engagement & support — 30 pts

### `PF-G.H1` Adherence chart + streak — **2 pts**
*happy · depends on: — · STEP-7-01*
- API subtasks:
  - `API-PATIENT-050` · GET /me/adherence · patient — adherence + streak **(1 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-PAT-060` · AdherenceChart · mfe-patient — chart **(1 pt)**

### `PF-G.H2` Weight goal — **3 pts**
*happy · depends on: PF-G.H1 · STEP-7-02*
- API subtasks:
  - `API-PATIENT-051` · CRUD /me/goals · patient — weight goal + progress **(2 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-PAT-061` · WeightGoal · mfe-patient — goal **(1 pt)**

### `PF-G.H3` Daily reminders — **3 pts**
*happy · depends on: PF-G.H2 · STEP-7-03*
- API subtasks:
  - `API-PATIENT-052` · CRUD /me/reminders · patient — reminders **(2 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-PAT-062` · RemindersSettings · mfe-patient — reminders **(1 pt)**

### `PF-G.H4` Content library — **3 pts**
*happy · depends on: PF-G.H3 · STEP-7-04*
- API subtasks:
  - `API-PATIENT-053` · GET /content · patient — articles, videos, FAQ, guides **(2 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-PAT-063` · ContentLibrary · mfe-patient — content **(1 pt)**

### `PF-G.H5` Foodnoise tracking — **4 pts**
*happy · depends on: PF-G.H4 · STEP-7-05*
- API subtasks:
  - `API-PATIENT-054` · CRUD /me/foodnoise · patient — track + chart **(2 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-PAT-064` · FoodnoiseTracker · mfe-patient — log + chart **(2 pt)**

### `PF-G.H6` Multiple addresses + weight history — **3 pts**
*happy · depends on: PF-G.H5 · STEP-7-06*
- API subtasks:
  - `API-PATIENT-055` · CRUD /me/addresses · patient — addresses + weight history **(2 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-PAT-065` · AddressBook · mfe-patient — addresses + history **(1 pt)**

### `PF-G.H7` Support chat + FAQ + emergency line — **3 pts**
*happy · depends on: PF-G.H6 · STEP-7-07*
- API subtasks:
  - `API-CRM-001` · CRUD /me/support/chat · crm — live support **(2 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-PAT-066` · SupportChat · mfe-patient — support + FAQ **(1 pt)**

### `PF-G.H8` Visit chat + survey + history PDF — **3 pts**
*happy · depends on: PF-G.H7 · STEP-7-08*
- API subtasks:
  - `API-VISIT-025` · CRUD /me/visits/{id}/chat · visit — chat + survey + history PDF **(2 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-PAT-067` · VisitChat · mfe-patient — chat + survey + PDF **(1 pt)**

### `PF-G.H9` Pricing perks — **5 pts**
*happy · depends on: PF-G.H8 · STEP-7-09*
- API subtasks:
  - `API-SUB-010` · POST /me/subscriptions/discount · sub — first-month discount **(2 pt)**
  - `API-PAY-025` · POST /me/wallet/cashback · pay — cashback **(2 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-PAT-068` · PricingPerks · mfe-patient — discount + cashback **(1 pt)**

### `PF-G.U1` Nothing to show yet — **1 pt**
*unhappy · any step · resolved by `R-EMPTY-STATE`*

## MF-A · Landing & self-check — 11 pts

### `MF-A.H1` Landing home — **1 pts**
*happy · depends on: — · STEP-1-08*
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-MKT-001` · LandingHome · mfe-marketing — hero, "how it works," section composition, CTAs into signup; sections are config-gated (`marketing.enabled_sections`) **(1 pt)**

### `MF-A.H2` BMI calculator — **1 pts**
*happy · depends on: MF-A.H1 · STEP-1-09*
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-MKT-002` · BmiCalculator · mfe-marketing — height/weight → BMI instantly; no storage; CTA to the eligibility check / signup **(1 pt)**

### `MF-A.H3` Public eligibility check (preliminary) — **4 pts**
*happy · depends on: MF-A.H2 · STEP-2-10*
- API subtasks:
  - `API-PATIENT-023` · GET /public/eligibility/preview-questionnaire · patient — public question subset (localized) **(1 pt)**
  - `API-PATIENT-024` · POST /public/eligibility/preview · patient — anonymous preliminary indicator **(2 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-MKT-003` · EligibilityCheck · mfe-marketing — public questionnaire → preliminary indicator → CTA to sign up for the full assessment (`STEP-2-06/07`) **(1 pt)**

### `MF-A.U1` Invalid BMI input — **1 pt**
*unhappy · at MF-A.H2 · resolved by `R-INLINE-VALIDATE`*

### `MF-A.U2` Not a fit / borderline — **1 pt**
*unhappy · at MF-A.H3 · resolved by `R-GUIDANCE-CTA`*

### `MF-A.U3` Preview unavailable — **1 pt**
*unhappy · at MF-A.H3 · resolved by `R-PREVIEW-RETRY`*

### `R-GUIDANCE-CTA` Supportive guidance + sign-up CTA — **1 pts**
*recovery (UI handling)*

### `R-PREVIEW-RETRY` Retry / proceed to full check — **1 pts**
*recovery (UI handling)*

## DF-A · Online visit & prescription — 27 pts

### `DF-A.H1` Open the assigned patient list — **2 pts**
*happy · depends on: — · STEP-3B-04*
- API subtasks:
  - `API-PROV-001` · GET /me/patients · prov — patients assigned to caller **(1 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-DOC-001` · PatientList · mfe-doctor — assigned patients **(1 pt)**

### `DF-A.H2` Open a patient profile + EMR — **3 pts**
*happy · depends on: DF-A.H1 · STEP-3B-05*
- API subtasks:
  - `API-PATIENT-010` · GET /patients/{id} · patient — full profile + EMR **(1 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-DOC-002` · PatientProfilePanel · mfe-doctor — profile + EMR **(2 pt)**

### `DF-A.H3` Conduct the phone visit — **0 pts**
*happy · depends on: DF-A.H2 · STEP-3B-03*
- reuses build of STEP-3B-03

### `DF-A.H4` Write the SOAP note — **4 pts**
*happy · depends on: DF-A.H3 · STEP-3B-06*
- API subtasks:
  - `API-VISIT-004` · POST /visits/{id}/soap · visit — save SOAP **(2 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-DOC-003` · VisitConsole · mfe-doctor — SOAP editor **(2 pt)**

### `DF-A.H5` Link the external prescription — **4 pts**
*happy · depends on: DF-A.H4 · STEP-3B-07*
- API subtasks:
  - `API-VISIT-005` · POST /visits/{id}/prescription/link · visit — attach external Rx id + fetch details from the e-prescription provider **(3 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-DOC-004` · PrescriptionLink · mfe-doctor — enter external Rx id + view fetched prescription **(1 pt)**

### `DF-A.H6` Confirm prescription status — **3 pts**
*happy · depends on: DF-A.H5 · STEP-3B-08*
- API subtasks:
  - `API-VISIT-006` · GET /visits/{id}/prescription/status · visit — refresh status from the e-prescription provider **(2 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-DOC-004` · PrescriptionLink · mfe-doctor — status **(1 pt)**

### `DF-A.H7` Auto-schedule the next visit — **4 pts**
*happy · depends on: DF-A.H6 · STEP-3B-09*
- API subtasks:
  - `API-VISIT-007` · POST /visits/{id}/next · visit — schedule next **(2 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-DOC-005` · NextVisitScheduler · mfe-doctor — set next **(2 pt)**

### `DF-A.H8` External e-prescription integration goes live — **3 pts**
*happy · depends on: DF-A.H7 · STEP-5-20*
- API subtasks:
  - `API-VISIT-006` · GET /visits/{id}/prescription/status · visit — live fetch from the e-prescription provider **(2 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-DOC-004` · PrescriptionLink · mfe-doctor — status **(1 pt)**

### `DF-A.U1` Record incomplete at visit — **1 pt**
*unhappy · at DF-A.H2 · resolved by `R-COMPLETE-RECORD`*

### `DF-A.U2` Rx rejected / invalid — **1 pt**
*unhappy · at DF-A.H5 · resolved by `R-RXFIX`*

### `DF-A.U3` Rx provider unreachable — **1 pt**
*unhappy · at DF-A.H5 · resolved by `R-RXFIX`*

### `R-COMPLETE-RECORD` Prompt to complete record — **1 pts**
*recovery (UI handling)*

## DF-B · Doctor panel & finance — 24 pts

### `DF-B.H1` Doctor login — **1 pts**
*happy · depends on: — · STEP-6-00*
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-DOC-000` · DoctorLogin · mfe-doctor — login — **no self-signup**; providers are admin-provisioned (`identity_role.assigned_by = admin`); the screen states "no account? contact Medica admin" **(1 pt)**

### `DF-B.H2` Doctor dashboard — **3 pts**
*happy · depends on: DF-B.H1 · STEP-6-01*
- API subtasks:
  - `API-VISIT-020` · GET /me/dashboard · visit — own counts, NPS, alerts, income **(1 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-DOC-010` · DoctorDashboard · mfe-doctor — dashboard **(2 pt)**

### `DF-B.H3` Patient management — **4 pts**
*happy · depends on: DF-B.H2 · STEP-6-02*
- API subtasks:
  - `API-VISIT-021` · GET /patients/{id}/visit-history · visit — history + nurse report **(1 pt)**
  - `API-PATIENT-040` · GET /patients/{id}/charts · patient — weight/BMI series **(1 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-DOC-011` · PatientManagement · mfe-doctor — history, charts, notes, messaging **(2 pt)**

### `DF-B.H4` Doctor scheduling — **4 pts**
*happy · depends on: DF-B.H3 · STEP-6-03*
- API subtasks:
  - `API-PROV-030` · CRUD /me/schedule · prov — own slots + block time **(2 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-DOC-012` · DoctorScheduleManager · mfe-doctor — slots **(2 pt)**

### `DF-B.H5` Doctor visits list — **2 pts**
*happy · depends on: DF-B.H4 · STEP-6-09*
- API subtasks:
  - `API-VISIT-026` · GET /me/visits?range=&status= · visit — the doctor's own visits (today + history) **(1 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-DOC-014` · DoctorVisitsList · mfe-doctor — Visits tab **(1 pt)**

### `DF-B.H6` Doctor prescriptions list — **2 pts**
*happy · depends on: DF-B.H5 · STEP-6-10*
- API subtasks:
  - `API-VISIT-024` · GET /me/prescriptions · visit — the doctor's linked prescriptions (read) **(1 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-DOC-015` · DoctorPrescriptionsList · mfe-doctor — Prescriptions tab **(1 pt)**

### `DF-B.H7` Doctor financial panel — **3 pts**
*happy · depends on: DF-B.H6 · STEP-6-04*
- API subtasks:
  - `API-PAY-020` · GET /me/finance · pay — own income + invoices **(1 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-DOC-013` · DoctorFinancePanel · mfe-doctor — finance **(2 pt)**

### `DF-B.U1` Provider deactivated — **1 pt**
*unhappy · at DF-B.H1 · resolved by `R-REASSIGN`*

### `DF-B.U2` Nothing assigned yet — **1 pt**
*unhappy · at DF-B.H2 · resolved by `R-EMPTY-STATE`*

### `DF-B.U3` Assignment lost mid-care — **1 pt**
*unhappy · at DF-B.H3 · resolved by `R-REASSIGN`*

### `R-REASSIGN` Provider reassignment / in-flight care — **2 pts**
*recovery (recovery flow)*

## NF-A · Home-visit safety sequence — 44 pts

### `NF-A.H1` Nurse logs in — **4 pts**
*happy · depends on: — · STEP-4A-01*
- API subtasks:
  - `API-AUTH-003` · POST /auth/login · auth — phone + password/OTP **(3 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-NUR-000` · NurseLogin · mfe-nurse — login **(1 pt)**

### `NF-A.H2` Nurse dashboard & status — **4 pts**
*happy · depends on: NF-A.H1 · STEP-4A-02*
- API subtasks:
  - `API-FIELD-002` · POST /me/status · field — set own status **(2 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-NUR-001` · NurseDashboard · mfe-nurse — status + summary **(2 pt)**

### `NF-A.H3` View route & schedule (today / tomorrow / week) — **2 pts**
*happy · depends on: NF-A.H2 · STEP-4A-03*
- API subtasks:
  - `API-FIELD-001` · GET /me/route?range=today|tomorrow|week · field — own visit list for the selected horizon (default `today`; navigation/ETA active for `today` only — future ranges are read-only planning views) **(1 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-NUR-002` · DailyRouteList · mfe-nurse — list + ETA + nav link, with a today/tomorrow/week range selector **(1 pt)**

### `NF-A.H4` GPS check-in at the patient's location — **3 pts**
*happy · depends on: NF-A.H3 · STEP-4A-04*
- API subtasks:
  - `API-FIELD-003` · POST /nurse-visits/{id}/checkin · field — GPS check-in **(2 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-NUR-003` · VisitCheckin · mfe-nurse — check-in **(1 pt)**

### `NF-A.H5` View patient profile + today's dose — **2 pts**
*happy · depends on: NF-A.H4 · STEP-4A-05*
- API subtasks:
  - `API-FIELD-004` · GET /nurse-visits/{id} · field — profile + Rx + dose **(1 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-NUR-004` · PatientVisitCard · mfe-nurse — profile + dose **(1 pt)**

### `NF-A.H6` Record cold-chain temperature — **3 pts**
*happy · depends on: NF-A.H5 · STEP-4A-06*
- API subtasks:
  - `API-FIELD-005` · POST /nurse-visits/{id}/coldchain · field — record temperature **(2 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-NUR-005` · ColdChainEntry · mfe-nurse — temp input **(1 pt)**

### `NF-A.H7` Pre-injection assessment — **4 pts**
*happy · depends on: NF-A.H6 · STEP-4A-07*
- API subtasks:
  - `API-FIELD-006` · POST /nurse-visits/{id}/assessment · field — vitals **(2 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-NUR-006` · PreInjectionAssessment · mfe-nurse — vitals form **(2 pt)**

### `NF-A.H8` Scan vial, then record injection site — **4 pts**
*happy · depends on: NF-A.H7 · STEP-4A-08*
- API subtasks:
  - `API-FIELD-007` · POST /nurse-visits/{id}/injection · field — site, dose, vial **(3 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-NUR-007` · InjectionRecorder · mfe-nurse — site chart + vial **(1 pt)**

### `NF-A.H9` Patient signs — **3 pts**
*happy · depends on: NF-A.H8 · STEP-4A-09*
- API subtasks:
  - `API-FIELD-008` · POST /nurse-visits/{id}/signature · field — patient signature **(2 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-NUR-008` · SignaturePad · mfe-nurse — signature **(1 pt)**

### `NF-A.H10` Submit the visit — **3 pts**
*happy · depends on: NF-A.H9 · STEP-4A-10*
- API subtasks:
  - `API-FIELD-009` · POST /nurse-visits/{id}/submit · field — submit + package to finance **(2 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-NUR-009` · VisitSubmit · mfe-nurse — finalize **(1 pt)**

### `NF-A.U1` GPS fails / location off — **1 pt**
*unhappy · at NF-A.H4 · resolved by `R-GPS-OVERRIDE`*

### `NF-A.U2` Patient not home / refuses / unsafe — **1 pt**
*unhappy · at NF-A.H5 · resolved by `R-RESCHEDULE`*

### `NF-A.U3` Cold-chain breach → do not administer — **1 pt**
*unhappy · at NF-A.H6 · resolved by `R-REPLACE`*

### `NF-A.U4` Wrong / mismatched vial scanned — **1 pt**
*unhappy · at NF-A.H8 · resolved by `R-VIAL-RESCAN`*

### `NF-A.U5` Signature capture fails / patient can't sign — **1 pt**
*unhappy · at NF-A.H9 · resolved by `R-OTP-RECEIPT`*

### `NF-A.U6` Offline mid-visit — **1 pt**
*unhappy · any step · resolved by `R-OFFLINE-CAPTURE`*

### `R-GPS-OVERRIDE` Manual GPS override — **1 pts**
*recovery (UI handling)*

### `R-VIAL-RESCAN` Wrong/mismatched vial — **3 pts**
*recovery (recovery flow, clinical)*

### `R-OTP-RECEIPT` OTP receipt fallback — **1 pts**
*recovery (UI handling)*

### `R-OFFLINE-CAPTURE` Offline capture + sync — **1 pts**
*recovery (UI handling)*

## NF-B · Safety & emergency — 14 pts

### `NF-B.H1` Escalate an emergency — **4 pts**
*happy · depends on: — · STEP-4A-11*
- API subtasks:
  - `API-FIELD-010` · POST /nurse-visits/{id}/emergency · field — escalate **(3 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-NUR-010` · SafetyButton · mfe-nurse — escalation **(1 pt)**

### `NF-B.H2` Press the safety button — **3 pts**
*happy · depends on: NF-B.H1 · STEP-4A-12*
- API subtasks:
  - `API-FIELD-011` · POST /me/safety · field — personal safety alert **(2 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-NUR-010` · SafetyButton · mfe-nurse — safety alert **(1 pt)**

### `NF-B.H3` Emergency escalation in-software, routed to CS — **4 pts**
*happy · depends on: NF-B.H2 · STEP-9-09*
- API subtasks:
  - `API-FIELD-010` · POST /nurse-visits/{id}/emergency · field — escalate to CS **(3 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-NUR-010` · SafetyButton · mfe-nurse — escalation (now to CS) **(1 pt)**

### `NF-B.U1` Safety button pressed — **1 pt**
*unhappy · at NF-B.H2 · resolved by `R-SAFETY-ESC`*

### `R-SAFETY-ESC` Nurse safety escalation — **2 pts**
*recovery (recovery flow)*

## NF-C · Route, inventory & completion — 32 pts

### `NF-C.H1` Live tracking + ETA — **6 pts**
*happy · depends on: — · STEP-5-10*
- API subtasks:
  - `API-FIELD-034` · GET /logistics/track/{visitId} · field — live position + ETA **(1 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-NUR-020` · RouteMapOptimized · mfe-nurse — position **(2 pt)**
  - `CMP-PAT-057` · NurseTracker · mfe-patient — live map + ETA **(3 pt)**

### `NF-C.H2` Assigned-patient list — **2 pts**
*happy · depends on: NF-C.H1 · STEP-5-11*
- API subtasks:
  - `API-FIELD-040` · GET /me/patients · field — assigned patients + search **(1 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-NUR-021` · AssignedPatients · mfe-nurse — list + search + notes **(1 pt)**

### `NF-C.H3` Nurse inventory — **5 pts**
*happy · depends on: NF-C.H2 · STEP-5-12*
- API subtasks:
  - `API-FIELD-041` · CRUD /me/inventory · field — own vial count, restock, cold log **(3 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-NUR-022` · InventoryManager · mfe-nurse — count + restock + cold log **(2 pt)**

### `NF-C.H4` Reorder / reschedule — **1 pts**
*happy · depends on: NF-C.H3 · STEP-5-13*
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-NUR-024` · VisitReorder · mfe-nurse — reorder **(1 pt)**

### `NF-C.H5` Record patient education — **3 pts**
*happy · depends on: NF-C.H4 · STEP-5-16*
- API subtasks:
  - `API-FIELD-042` · POST /nurse-visits/{id}/education · field — log education **(2 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-NUR-027` · EducationRecorder · mfe-nurse — education **(1 pt)**

### `NF-C.H6` Week/month summary — **2 pts**
*happy · depends on: NF-C.H5 · STEP-5-14*
- API subtasks:
  - `API-PROV-023` · GET /me/summary · prov — own week/month **(1 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-NUR-025` · NurseSummary · mfe-nurse — summary **(1 pt)**

### `NF-C.H7` Support inbox — **3 pts**
*happy · depends on: NF-C.H6 · STEP-5-15*
- API subtasks:
  - `API-NOTIF-002` · POST /notify/push · notif — support notifications **(2 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-NUR-026` · SupportInbox · mfe-nurse — notifications **(1 pt)**

### `NF-C.H8` Nurse financial panel — **3 pts**
*happy · depends on: NF-C.H7 · STEP-6-05*
- API subtasks:
  - `API-PAY-021` · GET /me/finance · pay — own income, bonuses, invoices **(1 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-NUR-030` · NurseFinancePanel · mfe-nurse — finance **(2 pt)**

### `NF-C.H9` Performance + hotline — **4 pts**
*happy · depends on: NF-C.H8 · STEP-6-06*
- API subtasks:
  - `API-PAY-022` · GET /me/performance · pay — own NPS, counts **(1 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-NUR-031` · NursePerformance · mfe-nurse — performance **(2 pt)**
  - `CMP-NUR-032` · EmergencyHotline · mfe-nurse — hotline **(1 pt)**

### `NF-C.U1` Route unavailable / Optime down — **1 pt**
*unhappy · at NF-C.H1 · resolved by `R-TRACK-FALLBACK`*

### `NF-C.U2` Shortage at pickup / count mismatch — **1 pt**
*unhappy · at NF-C.H3 · resolved by `R-STOCKOUT`*

### `NF-C.U3` No visits / patients / inventory — **1 pt**
*unhappy · at NF-C.H9 · resolved by `R-EMPTY-STATE`*

## PH-A · Inventory & dispense — 30 pts

### `PH-A.H1` Pharmacy login (OTP) — **4 pts**
*happy · depends on: — · STEP-6-07*
- API subtasks:
  - `API-PAY-023` · GET /me/commission · pay — own commission + invoice **(2 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-PHM-010` · CommissionPanel · mfe-pharmacy — commission **(2 pt)**

### `PH-A.H2` Inventory dashboard — **3 pts**
*happy · depends on: PH-A.H1 · STEP-5-01*
- API subtasks:
  - `API-PHARM-002` · GET /me/pharmacy/inventory · pharm — own hub inventory **(1 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-PHM-001` · InventoryDashboard · mfe-pharmacy — inventory **(2 pt)**

### `PH-A.H3` Record supplier receipt — **3 pts**
*happy · depends on: PH-A.H2 · STEP-5-02*
- API subtasks:
  - `API-PHARM-003` · POST /me/pharmacy/receipts · pharm — supplier receipt **(2 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-PHM-002` · SupplierReceipt · mfe-pharmacy — log stock **(1 pt)**

### `PH-A.H4` Monitor cold chain — **3 pts**
*happy · depends on: PH-A.H3 · STEP-5-03*
- API subtasks:
  - `API-PHARM-005` · GET /me/pharmacy/coldchain · pharm — temp log **(1 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-PHM-004` · ColdChainMonitor · mfe-pharmacy — temp + alerts **(2 pt)**

### `PH-A.H5` Dispense to nurse — **3 pts**
*happy · depends on: PH-A.H4 · STEP-5-04*
- API subtasks:
  - `API-PHARM-004` · POST /me/pharmacy/dispense · pharm — dispense (barcode + signature) **(2 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-PHM-003` · DispenseToNurse · mfe-pharmacy — scan + signature **(1 pt)**

### `PH-A.H6` Report damage — **3 pts**
*happy · depends on: PH-A.H5 · STEP-5-05*
- API subtasks:
  - `API-PHARM-006` · POST /me/pharmacy/damage · pharm — damage report **(2 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-PHM-005` · DamageReport · mfe-pharmacy — report **(1 pt)**

### `PH-A.H7` View audit log — **2 pts**
*happy · depends on: PH-A.H6 · STEP-5-06*
- API subtasks:
  - `API-PHARM-007` · GET /me/pharmacy/audit · pharm — transaction log **(1 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-PHM-006` · AuditLogView · mfe-pharmacy — log **(1 pt)**

### `PH-A.H8` Pharmacy commission — **0 pts**
*happy · depends on: PH-A.H7 · STEP-6-07*
- reuses build of STEP-6-07

### `PH-A.U1` No inventory / no pending dispenses — **1 pt**
*unhappy · at PH-A.H2 · resolved by `R-EMPTY-STATE`*

### `PH-A.U2` Hub cold-chain breach — **1 pt**
*unhappy · at PH-A.H4 · resolved by `R-HUB-QUARANTINE`*

### `PH-A.U3` Barcode scan fails — **1 pt**
*unhappy · at PH-A.H5 · resolved by `R-MANUAL-ENTRY`*

### `PH-A.U4` Stock-out / cannot fulfill — **1 pt**
*unhappy · at PH-A.H5 · resolved by `R-STOCKOUT`*

### `PH-A.U5` Expired stock detected — **1 pt**
*unhappy · at PH-A.H6 · resolved by `R-EXPIRE-QUARANTINE`*

### `R-HUB-QUARANTINE` Hub cold-chain quarantine — **2 pts**
*recovery (recovery flow)*

### `R-MANUAL-ENTRY` Manual barcode entry — **1 pts**
*recovery (UI handling)*

### `R-EXPIRE-QUARANTINE` Quarantine + replace expired stock — **1 pts**
*recovery (UI handling)*

## PH-B · Batch recall — 12 pts

### `PH-B.H1` Recall a batch — cascade to its vials — **5 pts**
*happy · depends on: — · STEP-R02-01*
- API subtasks:
  - `API-PHARM-010` · POST /batches/{id}/recall · pharm — set recall + cascade to vials/inventory **(3 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-ADM-042` · RecallManager · mfe-admin — recall control + progress **(2 pt)**

### `PH-B.H2` Compute affected patients — **3 pts**
*happy · depends on: PH-B.H1 · STEP-R02-02*
- API subtasks:
  - `API-PHARM-011` · GET /recalls/{id}/affected · pharm — affected read model **(1 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-ADM-042` · RecallManager · mfe-admin — affected list (scheduled vs administered) **(2 pt)**

### `PH-B.U1` Batch recalled — **1 pt**
*unhappy · at PH-B.H1 · resolved by `R-RECALL`*

### `R-RECALL` Batch recall → reassign affected patients — **3 pts**
*recovery (recovery flow, clinical)*

## OP-A · Patients, providers & roles — 31 pts

### `OP-A.H1` List patients — **2 pts**
*happy · depends on: — · STEP-4C-01*
- API subtasks:
  - `API-GW-010` · GET /patients · gw — patient list **(1 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-ADM-001` · PatientTable · mfe-admin — list **(1 pt)**

### `OP-A.H2` Patient detail + status — **1 pts**
*happy · depends on: OP-A.H1 · STEP-4C-02*
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-ADM-002` · PatientDetail · mfe-admin — profile + status **(1 pt)**

### `OP-A.H3` Edit Rx protocols — **4 pts**
*happy · depends on: OP-A.H2 · STEP-4C-03*
- API subtasks:
  - `API-GW-013` · CRUD /admin/protocols · gw — Rx templates **(2 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-ADM-003` · ProtocolEditor · mfe-admin — templates **(2 pt)**

### `OP-A.H4` Manage roles & permissions — **2 pts**
*happy · depends on: OP-A.H3 · STEP-4C-04*
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-ADM-004` · RoleManager · mfe-admin — roles + assign (multi-role) **(2 pt)**

### `OP-A.H5` Manage API keys — **4 pts**
*happy · depends on: OP-A.H4 · STEP-4C-05*
- API subtasks:
  - `API-GW-015` · CRUD /admin/api-keys · gw — integration keys **(2 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-ADM-005` · ApiKeyManager · mfe-admin — keys **(2 pt)**

### `OP-A.H6` Onboard a provider (doctor / nurse) — **4 pts**
*happy · depends on: OP-A.H5 · STEP-4C-07*
- API subtasks:
  - `API-PROV-020` · POST /providers/onboard · prov — onboard + assign role **(2 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-ADM-010` · ProviderOnboarding · mfe-admin — onboard — **covers both doctors and nurses**; required in the pilot since both see/serve patients from M4 **(2 pt)**

### `OP-A.H7` Verify credentials — **3 pts**
*happy · depends on: OP-A.H6 · STEP-4C-08*
- API subtasks:
  - `API-PROV-021` · POST /providers/{id}/verify · prov — credential verification **(2 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-ADM-011` · CredentialVerify · mfe-admin — verify **(1 pt)**

### `OP-A.H8` Assign a patient to a provider — **4 pts**
*happy · depends on: OP-A.H7 · STEP-4C-10*
- API subtasks:
  - `API-PROV-022` · POST /assignments · prov — create assignment (drives `:assigned`) **(2 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-ADM-012` · AssignmentBoard · mfe-admin — assign — needed in the pilot so ops can assign/reassign doctors and nurses to patients from M4 **(2 pt)**

### `OP-A.H9` User management — **2 pts**
*happy · depends on: OP-A.H8 · STEP-8-01*
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-ADM-030` · UserManagement · mfe-admin — users + activate **(2 pt)**

### `OP-A.U1` Role / permission misconfiguration — **1 pt**
*unhappy · at OP-A.H4 · resolved by `R-ROLE-FIX`*

### `OP-A.U2` No nurse available / no capacity — **1 pt**
*unhappy · at OP-A.H8 · resolved by `R-CAPACITY`*

### `R-ROLE-FIX` Role/permission correction — **1 pts**
*recovery (UI handling)*

### `R-CAPACITY` No nurse available / no capacity — **2 pts**
*recovery (recovery flow)*

## OP-B · Routing & fulfillment — 18 pts

### `OP-B.H1` Geocode the patient's address — **2 pts**
*happy · depends on: — · STEP-5-07*
- API subtasks:
  - `API-FIELD-030` · POST /logistics/geocode · field — address → coords **(2 pt)**

### `OP-B.H2` Generate the optimized route — **2 pts**
*happy · depends on: OP-B.H1 · STEP-5-08*
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-NUR-020` · RouteMapOptimized · mfe-nurse — route + position **(2 pt)**

### `OP-B.H3` On-demand emergency assignment — **3 pts**
*happy · depends on: OP-B.H2 · STEP-5-09*
- API subtasks:
  - `API-FIELD-033` · POST /logistics/assign · field — on-demand assignment **(2 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-NUR-023` · EmergencyAssignment · mfe-nurse — push visit **(1 pt)**

### `OP-B.H4` Operations dashboard — **4 pts**
*happy · depends on: OP-B.H3 · STEP-8-02*
- API subtasks:
  - `API-GW-021` · GET /ops/dashboard · gw — visits, live map, cold-chain, incidents **(2 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-ADM-031` · OpsDashboard · mfe-admin — ops view **(2 pt)**

### `OP-B.H5` Reschedule tool — **3 pts**
*happy · depends on: OP-B.H4 · STEP-8-03*
- API subtasks:
  - `API-GW-022` · POST /ops/reschedule · gw — admin reschedule **(2 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-ADM-032` · RescheduleTool · mfe-admin — reschedule **(1 pt)**

### `OP-B.U1` Geocode failed — **1 pt**
*unhappy · at OP-B.H1 · resolved by `R-GEOCODE-MANUAL`*

### `OP-B.U2` Incident / cold-chain network alert — **1 pt**
*unhappy · at OP-B.H4 · resolved by `R-INCIDENT-TRIAGE`*

### `R-GEOCODE-MANUAL` Manual geocode pin — **1 pts**
*recovery (UI handling)*

### `R-INCIDENT-TRIAGE` Incident triage — **1 pts**
*recovery (UI handling)*

## OP-C · Safety-critical recovery (control surface) — 11 pts

### `OP-C.H1` Monitor side-effect reports + assign a reviewer — **6 pts**
*happy · depends on: — · STEP-4C-09*
- API subtasks:
  - `API-PATIENT-031` · GET /admin/side-effects · patient — list incoming reports (urgent first) **(1 pt)**
  - `API-PATIENT-032` · POST /admin/side-effects/{id}/assign · patient — assign a reviewing doctor (routing only) **(3 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-ADM-043` · SideEffectMonitor · mfe-admin — ops oversight of the urgent-escalation path (RFLOW-05) — **read + assign/route only, no clinical write**; the **assigned** doctor performs review & disposition **(2 pt)**

### `OP-C.H2` Open a replacement order — **5 pts**
*happy · depends on: OP-C.H1 · STEP-R01-04*
- API subtasks:
  - `API-FIELD-050` · POST /replacements · field — open a replacement order **(2 pt)**
  - `API-FIELD-051` · GET /replacements · field — open-replacement queue **(1 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-ADM-041` · ReplacementQueue · mfe-admin — queue + action open replacements **(2 pt)**

## OP-D · Finance, settlements & BI — 34 pts

### `OP-D.H1` Issue a refund — **4 pts**
*happy · depends on: — · STEP-4C-06*
- API subtasks:
  - `API-PAY-007` · POST /wallet/{patientId}/refund · pay — admin-issued refund **(3 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-ADM-006` · RefundAction · mfe-admin — issue refund **(1 pt)**

### `OP-D.H2` Run monthly settlements — **5 pts**
*happy · depends on: OP-D.H1 · STEP-6-08*
- API subtasks:
  - `API-PAY-024` · POST /settlements/run · pay — settle providers/pharmacies **(4 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-ADM-020` · SettlementRunner · mfe-admin — run settlements **(1 pt)**

### `OP-D.H3` Finance dashboard — **3 pts**
*happy · depends on: OP-D.H2 · STEP-8-04*
- API subtasks:
  - `API-PAY-030` · GET /finance/dashboard · pay — finance + BI **(1 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-ADM-033` · FinanceDashboard · mfe-admin — finance **(2 pt)**

### `OP-D.H4` Growth dashboard — **3 pts**
*happy · depends on: OP-D.H3 · STEP-8-05*
- API subtasks:
  - `API-RPT-001` · GET /growth · rpt — CAC/LTV, channels **(1 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-ADM-034` · GrowthDashboard · mfe-admin — growth **(2 pt)**

### `OP-D.H5` Campaign manager — **5 pts**
*happy · depends on: OP-D.H4 · STEP-8-06*
- API subtasks:
  - `API-NOTIF-010` · POST /campaigns/sms · notif — SMS campaign **(3 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-ADM-035` · CampaignManager · mfe-admin — campaigns **(2 pt)**

### `OP-D.H6` Settings — **4 pts**
*happy · depends on: OP-D.H5 · STEP-8-07*
- API subtasks:
  - `API-GW-023` · CRUD /admin/settings · gw — rates, hubs, templates, and other tunables **(2 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-ADM-036` · SettingsPanel · mfe-admin — settings **(2 pt)**

### `OP-D.H7` Reports + export — **2 pts**
*happy · depends on: OP-D.H6 · STEP-8-08*
- API subtasks:
  - `API-RPT-002` · GET /reports · rpt — scheduled reports, export, audit log **(1 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-ADM-037` · ReportsExport · mfe-admin — reports **(1 pt)**

### `OP-D.U1` Refund / chargeback edge — **1 pt**
*unhappy · at OP-D.H1 · resolved by `R-CHARGEBACK`*

### `OP-D.U2` Settlement run fails / partial — **1 pt**
*unhappy · at OP-D.H2 · resolved by `R-SETTLE-RETRY`*

### `OP-D.U3` Bulk action partial failure — **1 pt**
*unhappy · at OP-D.H5 · resolved by `R-BULK-RETRY`*

### `R-CHARGEBACK` Chargeback reconciliation — **2 pts**
*recovery (recovery flow)*

### `R-SETTLE-RETRY` Settlement-run failure — **2 pts**
*recovery (recovery flow)*

### `R-BULK-RETRY` Per-row retry — **1 pts**
*recovery (UI handling)*

## CS-A · CS console — 11 pts

### `CS-A.H1` Call console + routing — **5 pts**
*happy · depends on: — · STEP-9-01*
- API subtasks:
  - `API-CRM-010` · CRUD /cs/calls · crm — hotline, queue, routing, tickets **(3 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-CS-001` · CallConsole · mfe-cs — queue + ticket **(2 pt)**

### `CS-A.H2` Six CS sub-panels, scoped per panel — **4 pts**
*happy · depends on: CS-A.H1 · STEP-9-02 to 9-07*
- API subtasks:
  - `API-CRM-011` · CRUD /cs/panels/{logistics|marketing|affiliate|medical|new-patient|payment} · crm — per-domain queue + actions **(2 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-CS-002…007` · {Logistics,Marketing,Affiliate,Medical,NewPatient,Payment}Panel · mfe-cs — per-domain panel **(2 pt)**

### `CS-A.H3` CS messaging + KPI — **2 pts**
*happy · depends on: CS-A.H2 · STEP-9-08*
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-CS-008` · CSDashboard · mfe-cs — KPI + messaging **(2 pt)**

### `CS-A.H4` Emergency escalation in-software — **0 pts**
*happy · depends on: CS-A.H3 · STEP-9-09*
- reuses build of STEP-9-09

## AF-A · Affiliate — 19 pts

### `AF-A.H1` Affiliate dashboard — **3 pts**
*happy · depends on: — · STEP-9-10*
- API subtasks:
  - `API-CRM-020` · GET /me/affiliate/dashboard · crm — own performance **(1 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-AFF-001` · AffiliateDashboard · mfe-affiliate — dashboard **(2 pt)**

### `AF-A.H2` Referral links + conversion — **3 pts**
*happy · depends on: AF-A.H1 · STEP-9-11*
- API subtasks:
  - `API-CRM-021` · POST /me/affiliate/links · crm — links + funnel **(2 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-AFF-002` · ReferralLinks · mfe-affiliate — links + funnel **(1 pt)**

### `AF-A.H3` Affiliate onboarding + codes + audit — **3 pts**
*happy · depends on: AF-A.H2 · STEP-9-13*
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-AFF-004` · DiscountCodes · mfe-affiliate — codes **(1 pt)**
  - `CMP-ADM-040` · AffiliateOnboarding · mfe-admin — approve **(2 pt)**

### `AF-A.H4` Commission + payment — **4 pts**
*happy · depends on: AF-A.H3 · STEP-9-12*
- API subtasks:
  - `API-CRM-022` · GET /me/affiliate/commission · crm — own commission **(2 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-AFF-003` · CommissionPanel · mfe-affiliate — commission **(2 pt)**

### `AF-A.H5` A/B experiments — **1 pts**
*happy · depends on: AF-A.H4 · STEP-9-14*
- (no API/component — config/integration step)

### `AF-A.U1` Referral link invalid / expired — **1 pt**
*unhappy · at AF-A.H2 · resolved by `R-NEW-LINK`*

### `AF-A.U2` Application rejected — **1 pt**
*unhappy · at AF-A.H3 · resolved by `R-BLOCKED-STATE`*

### `AF-A.U3` Commission dispute / not yet payable — **1 pt**
*unhappy · at AF-A.H4 · resolved by `R-CS-TICKET`*

### `R-NEW-LINK` Issue a fresh referral link — **1 pts**
*recovery (UI handling)*

### `R-CS-TICKET` Generic CS ticket — **1 pts**
*recovery (UI handling)*

## XF · Cross-cutting failures — 14 pts

### `XF.U1` Network offline — **1 pt**
*unhappy · any step · resolved by `R-OFFLINE-BANNER`*

### `XF.U2` Session expired / token invalid — **1 pt**
*unhappy · any step · resolved by `R-REAUTH`*

### `XF.U3` Permission denied (403) — **1 pt**
*unhappy · any step · resolved by `R-NO-ACCESS`*

### `XF.U4` Not found (404) / deleted — **1 pt**
*unhappy · any step · resolved by `R-NOT-FOUND`*

### `XF.U5` Server / dependent-service error (5xx) — **1 pt**
*unhappy · any step · resolved by `R-RETRY-5XX`*

### `XF.U6` Rate limited (429) — **1 pt**
*unhappy · any step · resolved by `R-RATE-LIMIT`*

### `XF.U7` Stale data / concurrent edit conflict — **1 pt**
*unhappy · any step · resolved by `R-CONFLICT`*

### `R-OFFLINE-BANNER` Offline banner — **1 pts**
*recovery (UI handling)*

### `R-REAUTH` Re-authenticate — **1 pts**
*recovery (UI handling)*

### `R-NO-ACCESS` Permission-denied state — **1 pts**
*recovery (UI handling)*

### `R-NOT-FOUND` Not-found state — **1 pts**
*recovery (UI handling)*

### `R-RETRY-5XX` Server-error retry — **1 pts**
*recovery (UI handling)*

### `R-RATE-LIMIT` Rate-limit wait — **1 pts**
*recovery (UI handling)*

### `R-CONFLICT` Concurrent-edit conflict — **1 pts**
*recovery (UI handling)*

---

## Shared recoveries (used by several flows) — 13 pts

### `R-RESCHEDULE` Reschedule a missed/aborted visit — **2 pts**
*recovery (recovery flow) · shared*

### `R-RXFIX` External Rx re-issue + re-link — **2 pts**
*recovery (recovery flow) · shared*

### `R-REPLACE` Replacement-dose order — **3 pts**
*recovery (recovery flow, clinical) · shared*

### `R-STOCKOUT` Stock-out / shortage at pickup — **2 pts**
*recovery (recovery flow) · shared*

### `R-BLOCKED-STATE` Full-screen blocked/rejected state — **1 pts**
*recovery (UI handling) · shared*

### `R-INLINE-VALIDATE` Inline validation + retry — **1 pts**
*recovery (UI handling) · shared*

### `R-EMPTY-STATE` Designed empty state — **1 pts**
*recovery (UI handling) · shared*

### `R-TRACK-FALLBACK` Tracking fallback — **1 pts**
*recovery (UI handling) · shared*
