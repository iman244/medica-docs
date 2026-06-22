# Medica — Task Plan & Estimates

Every **build step (`STEP-`) is a task** with its own id, title, description, dependencies, and a **point** estimate. Each task's **APIs and components are subtasks**; the component subtasks depend on the task's API subtasks (you can't wire the UI before the endpoint exists). Points are estimated **per subtask from the work the spec describes**, then summed into the task, the cell, and the month. As a rough guide **1 point ≈ 2 hours**, but the point is the unit of work — convert to time at your team's velocity.

**Estimation rubric (transparent, so you can adjust):** API subtask = 1 pt (`GET`/read) or 2 pt (mutation), +1 pt if it touches clinical/payment/routing logic, +1 pt if it integrates a third party (video/OTP/SMS/gateway), cap 4 pt. Component subtask = 1 pt (display), 2 pt for forms/wizards/queues/dashboards/maps, 3 pt for live/video, cap 3 pt. Tasks run **in the listed order** within a flow (each depends on the previous); each flow also depends on the prerequisite flow noted at the cell header.

**Total: 400 points** across 123 tasks.

Source: `Medica_Build_and_Technical_Specification.md`. Re-generate if it changes.

## Column legend (user-flows)

| code | user-flow |
|---|---|
| **A** | Onboarding & Identity |
| **B** | Eligibility & Profile |
| **C** | Subscription & Payment |
| **D** | Online Visit & Doctor |
| **E** | Nurse Home Visit |
| **F** | Pharmacy & Logistics |
| **G** | Patient Self-Service |
| **H** | Admin / Ops & Finance/BI |
| **I** | Safety Recovery |
| **J** | Customer Service & Affiliate |

## Matrix — estimated points per cell

| Month | A | B | C | D | E | F | G | H | I | J | **Total** |
|---|--:|--:|--:|--:|--:|--:|--:|--:|--:|--:|--:|
| **M1** Foundation + identity | 28 | · | · | · | · | · | · | · | · | · | **28** |
| **M2** Profile + eligibility | · | 39 | · | · | · | · | · | · | · | · | **39** |
| **M3** Subscription + visit | · | · | 18 | 34 | · | · | · | · | · | · | **52** |
| **M4** Nurse + tracking + ops | · | · | · | · | 39 | · | 27 | 34 | · | · | **100** |
| **M5** Pharmacy + cold-chain | · | · | · | · | · | 49 | · | · | 13 | · | **62** |
| **M6** Doctor + finance | · | · | · | 35 | · | · | · | · | · | · | **35** |
| **M7** Patient engagement | · | · | · | · | · | · | 29 | · | · | · | **29** |
| **M8** Admin + finance/BI | · | · | · | · | · | · | · | 26 | · | · | **26** |
| **M9** CS + affiliate | · | · | · | · | · | · | · | · | · | 29 | **29** |
| **Total** | **28** | **39** | **18** | **69** | **39** | **49** | **56** | **60** | **13** | **29** | **400** |

---

# Task plan by cell (in execution order)

## M1 · A — Onboarding & Identity — 28 pts (8 tasks)

### T-001 · STEP-1-01 — Enter phone + password, request OTP — **5 pts**
*depends on: — · serves F-101 · actor: anonymous (pre-auth)*
- API subtasks:
  - `API-AUTH-001` · POST /auth/register · auth — create account, trigger OTP **(3 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-PAT-001` · SignUpForm · mfe-patient — phone + password + request OTP **(2 pt)**

### T-002 · STEP-1-02 — Verify phone OTP — **5 pts**
*depends on: T-001 · serves F-101 · actor: anonymous (pre-auth)*
- API subtasks:
  - `API-AUTH-002` · POST /auth/otp/verify · auth — verify OTP, issue tokens **(3 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-PAT-001` · SignUpForm · mfe-patient — OTP entry **(2 pt)**

### T-003 · STEP-1-03 — Log in (phone + password) — **4 pts**
*depends on: T-002 · serves F-102 · actor: anonymous (pre-auth)*
- API subtasks:
  - `API-AUTH-003` · POST /auth/login · auth — phone + password → tokens **(2 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-PAT-002` · LoginForm · mfe-patient — phone + password **(2 pt)**

### T-004 · STEP-1-04 — Log in (phone + OTP) — **5 pts**
*depends on: T-003 · serves F-102 · actor: anonymous (pre-auth)*
- API subtasks:
  - `API-AUTH-003` · POST /auth/login · auth — phone + OTP → tokens **(3 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-PAT-002` · LoginForm · mfe-patient — OTP toggle **(2 pt)**

### T-005 · STEP-1-05 — Recover password — **4 pts**
*depends on: T-004 · serves F-103 · actor: anonymous (pre-auth)*
- API subtasks:
  - `API-AUTH-004/005` · POST /auth/password/reset/{request,confirm} · auth — SMS reset + set password **(3 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-PAT-003` · PasswordResetFlow · mfe-patient — request + confirm **(1 pt)**

### T-006 · STEP-1-07 — Verify identity (KYC) — **3 pts**
*depends on: T-005 · serves F-105 · [int·Shahkar] · actor: patient (self)*
- API subtasks:
  - `API-AUTH-007` · POST /me/kyc/verify · auth — submit national ID, return status **(2 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-PAT-004` · KycVerification · mfe-patient — national-ID capture + result **(1 pt)**

### T-007 · STEP-1-08 — Landing home — **1 pts**
*depends on: T-006 · serves F-095 · [public] · actor: anonymous (public)*
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-MKT-001` · LandingHome · mfe-marketing — hero, "how it works," section composition, CTAs into signup; sections are config-gated (`marketing.enabled_sections`) **(1 pt)**

### T-008 · STEP-1-09 — BMI calculator — **1 pts**
*depends on: T-007 · serves F-096 · [public] · actor: anonymous (public)*
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-MKT-002` · BmiCalculator · mfe-marketing — height/weight → BMI instantly; no storage; CTA to the eligibility check / signup **(1 pt)**

## M2 · B — Eligibility & Profile — 39 pts (11 tasks) — depends on: M1 · Onboarding & Identity (after T-008)

### T-009 · STEP-2-01 — Complete basic profile — **4 pts**
*depends on: T-008 · serves F-110 · actor: patient (self)*
- API subtasks:
  - `API-PATIENT-001` · PUT /me/profile · patient — save profile **(2 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-PAT-010` · ProfileForm · mfe-patient — basic profile **(2 pt)**

### T-010 · STEP-2-02 — Enter health data, see BMI — **4 pts**
*depends on: T-009 · serves F-111 · actor: patient (self)*
- API subtasks:
  - `API-PATIENT-002` · PUT /me/health · patient — save measurement, compute BMI **(2 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-PAT-011` · HealthDataForm · mfe-patient — height/weight + BMI **(2 pt)**

### T-011 · STEP-2-03 — Enter medical history — **4 pts**
*depends on: T-010 · serves F-112 · actor: patient (self)*
- API subtasks:
  - `API-PATIENT-003` · PUT /me/history · patient — save history **(2 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-PAT-012` · MedicalHistoryForm · mfe-patient — history **(2 pt)**

### T-012 · STEP-2-04 — Upload lab results — **4 pts**
*depends on: T-011 · serves F-113 · [int·ArvanCloud] · actor: patient (self)*
- API subtasks:
  - `API-PATIENT-004` · POST /me/labs · patient — store lab file **(2 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-PAT-013` · LabUploader · mfe-patient — upload + thumbnails **(2 pt)**

### T-013 · STEP-2-05 — Set delivery address — **4 pts**
*depends on: T-012 · serves F-115 · actor: patient (self)*
- API subtasks:
  - `API-PATIENT-005` · PUT /me/address · patient — save delivery address **(2 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-PAT-014` · AddressForm · mfe-patient — address entry **(2 pt)**

### T-014 · STEP-2-06 — Answer eligibility questionnaire — **3 pts**
*depends on: T-013 · serves F-121 · actor: patient (self)*
- API subtasks:
  - `API-PATIENT-020` · GET /eligibility/questionnaire · patient — question set **(1 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-PAT-020` · EligibilityWizard · mfe-patient — questionnaire **(2 pt)**

### T-015 · STEP-2-07 — Compute eligibility, show result — **3 pts**
*depends on: T-014 · serves F-122, F-123, F-125, F-126 · actor: patient (self)*
- API subtasks:
  - `API-PATIENT-021` · POST /me/eligibility/answers · patient — rule engine + persist **(2 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-PAT-021` · EligibilityResult · mfe-patient — result + recommendation **(1 pt)**

### T-016 · STEP-2-08 — Sign informed consent — **4 pts**
*depends on: T-015 · serves F-124 · actor: patient (self)*
- API subtasks:
  - `API-PATIENT-022` · POST /me/consent · patient — record consent **(2 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-PAT-022` · ConsentForm · mfe-patient — consent capture **(2 pt)**

### T-017 · STEP-2-09 — Receive confirmations — **3 pts**
*depends on: T-016 · serves F-205 · [int·Kavenegar/Pushe] · actor: system → patient*
- API subtasks:
  - `API-NOTIF-001/002` · POST /notify/{sms,push} · notif — send message **(2 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-SHL-005` · NotificationCenter · shell — in-app + push **(1 pt)**

### T-018 · STEP-2-10 — Public eligibility check (preliminary) — **4 pts**
*depends on: T-017 · serves F-097 · [clinical] [public] · actor: anonymous (public)*
- API subtasks:
  - `API-PATIENT-023` · GET /public/eligibility/preview-questionnaire · patient — public question subset (localized) **(1 pt)**
  - `API-PATIENT-024` · POST /public/eligibility/preview · patient — anonymous preliminary indicator **(2 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-MKT-003` · EligibilityCheck · mfe-marketing — public questionnaire → preliminary indicator → CTA to sign up for the full assessment (`STEP-2-06/07`) **(1 pt)**

### T-019 · STEP-2-11 — Patient home — **2 pts**
*depends on: T-018 · serves F-100 · actor: patient (self)*
- API subtasks:
  - `API-PATIENT-025` · GET /me/home · patient — home summary (next visit, treatment status, quick actions, alerts) **(1 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-PAT-000` · PatientHome · mfe-patient — post-login hub (the patient's main screen; precedes notifications and other patient sections) **(1 pt)**

## M3 · C — Subscription & Payment — 18 pts (5 tasks) — depends on: M2 · Eligibility & Profile (after T-019)

### T-020 · STEP-3A-01 — View packages — **3 pts**
*depends on: T-019 · serves F-130 · actor: patient (self)*
- API subtasks:
  - `API-SUB-001` · GET /subscriptions/packages · sub — package catalog **(1 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-PAT-030` · PackagePicker · mfe-patient — package cards **(2 pt)**

### T-021 · STEP-3A-02 — Purchase & pay — **7 pts**
*depends on: T-020 · serves F-131 · [int·ZarinPal/Venda] · actor: patient (self)*
- API subtasks:
  - `API-SUB-002` · POST /me/subscriptions · sub — create subscription **(2 pt)**
  - `API-PAY-001/002` · POST /payments/{checkout,callback} · pay — gateway session + verify **(3 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-PAT-031` · CheckoutFlow · mfe-patient — gateway redirect + result **(2 pt)**

### T-022 · STEP-3A-03 — Manage / stop renewal — **4 pts**
*depends on: T-021 · serves F-133 · actor: patient (self)*
- API subtasks:
  - `API-SUB-003` · PATCH /me/subscriptions/{subId}/renewal · sub — stop/resume **(2 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-PAT-032` · SubscriptionManager · mfe-patient — renewal controls **(2 pt)**

### T-023 · STEP-3A-04 — Cancel + exit interview — **3 pts**
*depends on: T-022 · serves F-139 · actor: patient (self)*
- API subtasks:
  - `API-SUB-004` · POST /me/subscriptions/{subId}/cancel · sub — cancel + interview **(2 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-PAT-034` · ExitInterview · mfe-patient — cancellation survey **(1 pt)**

### T-024 · STEP-3A-05 — Wallet (balance, top-up, pay) — **1 pts**
*depends on: T-023 · serves F-141, F-142, F-143, F-146 · actor: patient (self)*
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-PAT-033` · WalletView · mfe-patient — balance + top-up + transactions **(1 pt)**

## M3 · D — Online Visit & Doctor — 34 pts (11 tasks) — depends on: M3 · Subscription & Payment (after T-024)

### T-025 · STEP-3B-01 — Patient books visit — **5 pts**
*depends on: T-024 · serves F-150, F-151 · actor: patient (self)*
- API subtasks:
  - `API-VISIT-001` · GET /visits/availability · visit — open slots **(1 pt)**
  - `API-VISIT-002` · POST /me/visits · visit — book visit **(2 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-PAT-040` · VisitBooking · mfe-patient — slot picker **(2 pt)**

### T-026 · STEP-3B-02 — Patient gets reminder — **3 pts**
*depends on: T-025 · serves F-152 · [int·Kavenegar/Pushe] · actor: system → patient*
- API subtasks:
  - `API-NOTIF-003` · POST /notify/schedule · notif — queue reminders **(2 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-PAT-041` · VisitReminderBanner · mfe-patient — banner **(1 pt)**

### T-027 · STEP-3B-03 — Phone visit — **1 pts**
*depends on: T-026 · serves F-153 · [manual] · actor: doctor ↔ patient (operational)*
- (no API/component — integration/config step)

### T-028 · STEP-3B-04 — Doctor opens their patient list — **2 pts**
*depends on: T-027 · serves F-310 · actor: doctor (assigned)*
- API subtasks:
  - `API-PROV-001` · GET /me/patients · prov — patients assigned to caller **(1 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-DOC-001` · PatientList · mfe-doctor — assigned patients **(1 pt)**

### T-029 · STEP-3B-05 — Doctor opens a patient profile + EMR — **3 pts**
*depends on: T-028 · serves F-312 · actor: doctor (assigned)*
- API subtasks:
  - `API-PATIENT-010` · GET /patients/{id} · patient — full profile + EMR **(1 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-DOC-002` · PatientProfilePanel · mfe-doctor — profile + EMR **(2 pt)**

### T-030 · STEP-3B-06 — Doctor writes SOAP — **4 pts**
*depends on: T-029 · serves F-328 · actor: doctor (assigned)*
- API subtasks:
  - `API-VISIT-004` · POST /visits/{id}/soap · visit — save SOAP **(2 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-DOC-003` · VisitConsole · mfe-doctor — SOAP editor **(2 pt)**

### T-031 · STEP-3B-07 — Doctor links external prescription — **4 pts**
*depends on: T-030 · serves F-327, F-329 · [int·RxProvider→M5] · actor: doctor (assigned)*
- API subtasks:
  - `API-VISIT-005` · POST /visits/{id}/prescription/link · visit — attach external Rx id + fetch details from the e-prescription provider **(3 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-DOC-004` · PrescriptionLink · mfe-doctor — enter external Rx id + view fetched prescription **(1 pt)**

### T-032 · STEP-3B-08 — Confirm prescription status — **3 pts**
*depends on: T-031 · serves F-330 · [int·RxProvider→M5] · actor: doctor (assigned)*
- API subtasks:
  - `API-VISIT-006` · GET /visits/{id}/prescription/status · visit — refresh status from the e-prescription provider **(2 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-DOC-004` · PrescriptionLink · mfe-doctor — status **(1 pt)**

### T-033 · STEP-3B-09 — Auto-schedule next visit — **4 pts**
*depends on: T-032 · serves F-331 · actor: doctor (assigned)*
- API subtasks:
  - `API-VISIT-007` · POST /visits/{id}/next · visit — schedule next **(2 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-DOC-005` · NextVisitScheduler · mfe-doctor — set next **(2 pt)**

### T-034 · STEP-3B-10 — Patient views & downloads prescription — **2 pts**
*depends on: T-033 · serves F-155, F-159 · actor: patient (self)*
- API subtasks:
  - `API-VISIT-003` · GET /me/visits/{visitId} · visit — own visit detail **(1 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-PAT-042` · PrescriptionView · mfe-patient — show Rx + download PDF **(1 pt)**

### T-035 · STEP-3B-11 — Patient raises emergency request — **3 pts**
*depends on: T-034 · serves F-157 · actor: patient (self)*
- API subtasks:
  - `API-VISIT-008` · POST /me/visits/emergency · visit — raise emergency **(2 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-PAT-043` · EmergencyRequestButton · mfe-patient — raise emergency **(1 pt)**

## M4 · E — Nurse Home Visit — 39 pts (12 tasks) — depends on: M3 · Online Visit & Doctor (after T-035)

### T-036 · STEP-4A-01 — Nurse logs in — **4 pts**
*depends on: T-035 · serves F-401 · actor: nurse (self)*
- API subtasks:
  - `API-AUTH-003` · POST /auth/login · auth — phone + password/OTP **(3 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-NUR-000` · NurseLogin · mfe-nurse — login **(1 pt)**

### T-037 · STEP-4A-02 — Nurse dashboard & status — **4 pts**
*depends on: T-036 · serves F-405, F-407 · actor: nurse (self)*
- API subtasks:
  - `API-FIELD-002` · POST /me/status · field — set own status **(2 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-NUR-001` · NurseDashboard · mfe-nurse — status + summary **(2 pt)**

### T-038 · STEP-4A-03 — View daily route & navigate — **2 pts**
*depends on: T-037 · serves F-415, F-416, F-417, F-418 · [order manual→M5] · actor: nurse (self)*
- API subtasks:
  - `API-FIELD-001` · GET /me/route · field — own day's visit list **(1 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-NUR-002` · DailyRouteList · mfe-nurse — list + ETA + nav link **(1 pt)**

### T-039 · STEP-4A-04 — GPS check-in — **3 pts**
*depends on: T-038 · serves F-430 · actor: nurse (assigned)*
- API subtasks:
  - `API-FIELD-003` · POST /nurse-visits/{id}/checkin · field — GPS check-in **(2 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-NUR-003` · VisitCheckin · mfe-nurse — check-in **(1 pt)**

### T-040 · STEP-4A-05 — View patient profile + dose — **2 pts**
*depends on: T-039 · serves F-431, F-432 · actor: nurse (assigned)*
- API subtasks:
  - `API-FIELD-004` · GET /nurse-visits/{id} · field — profile + Rx + dose **(1 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-NUR-004` · PatientVisitCard · mfe-nurse — profile + dose **(1 pt)**

### T-041 · STEP-4A-06 — Record cold-chain temp — **3 pts**
*depends on: T-040 · serves F-433 · [manual entry; auto int·Elitech→M5] · actor: nurse (assigned)*
- API subtasks:
  - `API-FIELD-005` · POST /nurse-visits/{id}/coldchain · field — record temperature **(2 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-NUR-005` · ColdChainEntry · mfe-nurse — temp input **(1 pt)**

### T-042 · STEP-4A-07 — Pre-injection assessment — **4 pts**
*depends on: T-041 · serves F-434 · actor: nurse (assigned)*
- API subtasks:
  - `API-FIELD-006` · POST /nurse-visits/{id}/assessment · field — vitals **(2 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-NUR-006` · PreInjectionAssessment · mfe-nurse — vitals form **(2 pt)**

### T-043 · STEP-4A-08 — Record injection site — **4 pts**
*depends on: T-042 · serves F-437 · actor: nurse (assigned)*
- API subtasks:
  - `API-FIELD-007` · POST /nurse-visits/{id}/injection · field — site, dose, vial **(3 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-NUR-007` · InjectionRecorder · mfe-nurse — site chart + vial **(1 pt)**

### T-044 · STEP-4A-09 — Patient signs — **3 pts**
*depends on: T-043 · serves F-441 · actor: nurse (assigned, captures patient signature)*
- API subtasks:
  - `API-FIELD-008` · POST /nurse-visits/{id}/signature · field — patient signature **(2 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-NUR-008` · SignaturePad · mfe-nurse — signature **(1 pt)**

### T-045 · STEP-4A-10 — Submit visit — **3 pts**
*depends on: T-044 · serves F-442 · actor: nurse (assigned)*
- API subtasks:
  - `API-FIELD-009` · POST /nurse-visits/{id}/submit · field — submit + package to finance **(2 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-NUR-009` · VisitSubmit · mfe-nurse — finalize **(1 pt)**

### T-046 · STEP-4A-11 — Escalate emergency — **4 pts**
*depends on: T-045 · serves F-443 · [manual→M9] · actor: nurse (self)*
- API subtasks:
  - `API-FIELD-010` · POST /nurse-visits/{id}/emergency · field — escalate **(3 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-NUR-010` · SafetyButton · mfe-nurse — escalation **(1 pt)**

### T-047 · STEP-4A-12 — Safety button — **3 pts**
*depends on: T-046 · serves F-497 · actor: nurse (self)*
- API subtasks:
  - `API-FIELD-011` · POST /me/safety · field — personal safety alert **(2 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-NUR-010` · SafetyButton · mfe-nurse — safety alert **(1 pt)**

## M4 · G — Patient Self-Service — 27 pts (9 tasks) — depends on: M4 · Nurse Home Visit (after T-047)

### T-048 · STEP-4B-01 — See schedule + nurse profile — **2 pts**
*depends on: T-047 · serves F-170, F-220 · actor: patient (self)*
- API subtasks:
  - `API-FIELD-020` · GET /me/nurse-visits · field — own schedule + nurse **(1 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-PAT-050` · NurseVisitSchedule · mfe-patient — schedule + nurse **(1 pt)**

### T-049 · STEP-4B-02 — Verify nurse identity — **2 pts**
*depends on: T-048 · serves F-221 · actor: patient (self)*
- API subtasks:
  - `API-PROV-010` · GET /me/nurse-visits/{id}/nurse-verify · prov — nurse code/photo **(1 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-PAT-051` · NurseIdentityVerify · mfe-patient — verify nurse **(1 pt)**

### T-050 · STEP-4B-03 — Confirm medication received — **3 pts**
*depends on: T-049 · serves F-222 · actor: patient (self)*
- API subtasks:
  - `API-FIELD-022` · POST /me/nurse-visits/{id}/confirm-receipt · field — medication receipt **(2 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-PAT-052` · MedReceiptConfirm · mfe-patient — confirm meds **(1 pt)**

### T-051 · STEP-4B-04 — Confirm / change time — **4 pts**
*depends on: T-050 · serves F-171 · actor: patient (self)*
- API subtasks:
  - `API-FIELD-021` · POST /me/nurse-visits/{id}/reschedule · field — confirm/change time **(2 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-PAT-053` · VisitTimeManager · mfe-patient — confirm/change **(2 pt)**

### T-052 · STEP-4B-05 — Confirm service received — **3 pts**
*depends on: T-051 · serves F-174 · actor: patient (self)*
- API subtasks:
  - `API-FIELD-022` · POST /me/nurse-visits/{id}/confirm-receipt · field — service receipt **(2 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-PAT-054` · ServiceReceipt · mfe-patient — confirm service **(1 pt)**

### T-053 · STEP-4B-06 — See injection history — **2 pts**
*depends on: T-052 · serves F-175 · actor: patient (self)*
- API subtasks:
  - `API-FIELD-023` · GET /me/injections · field — own injection history **(1 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-PAT-055` · InjectionHistory · mfe-patient — history **(1 pt)**

### T-054 · STEP-4B-07 — Next-injection reminder — **3 pts**
*depends on: T-053 · serves F-176 · [int·Kavenegar/Pushe] · actor: system → patient*
- API subtasks:
  - `API-NOTIF-003` · POST /notify/schedule · notif — schedule reminder **(2 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-SHL-005` · NotificationCenter · shell — reminder **(1 pt)**

### T-055 · STEP-4B-08 — Report side effect (urgent → doctor) — **4 pts**
*depends on: T-054 · serves F-177, F-178 · actor: patient (self)*
- API subtasks:
  - `API-PATIENT-030` · POST /me/side-effects · patient — report; `urgent` set by clinical ruleset `[clinical]` **(3 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-PAT-056` · SideEffectReporter · mfe-patient — report **(1 pt)**

### T-056 · STEP-4B-09 — Track nurse live — **4 pts**
*depends on: T-055 · serves F-172 · [int·Optime→M5] · actor: patient (self)*
- API subtasks:
  - `API-FIELD-034` · GET /me/nurse-visits/{id}/track · field — live position + ETA **(1 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-PAT-057` · NurseTracker · mfe-patient — live map **(3 pt)**

## M4 · H — Admin / Ops & Finance/BI — 34 pts (10 tasks) — depends on: M2 · Eligibility & Profile (after T-019)

### T-057 · STEP-4C-01 — List patients — **2 pts**
*depends on: T-019 · serves F-601 · actor: admin (any)*
- API subtasks:
  - `API-GW-010` · GET /patients · gw — patient list **(1 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-ADM-001` · PatientTable · mfe-admin — list **(1 pt)**

### T-058 · STEP-4C-02 — Patient detail + status — **1 pts**
*depends on: T-057 · serves F-602, F-603 · actor: admin (any)*
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-ADM-002` · PatientDetail · mfe-admin — profile + status **(1 pt)**

### T-059 · STEP-4C-03 — Edit Rx protocols — **4 pts**
*depends on: T-058 · serves F-651 · actor: admin (any)*
- API subtasks:
  - `API-GW-013` · CRUD /admin/protocols · gw — Rx templates **(2 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-ADM-003` · ProtocolEditor · mfe-admin — templates **(2 pt)**

### T-060 · STEP-4C-04 — Manage roles & permissions — **2 pts**
*depends on: T-059 · serves F-655 · actor: admin (any)*
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-ADM-004` · RoleManager · mfe-admin — roles + assign (multi-role) **(2 pt)**

### T-061 · STEP-4C-05 — Manage API keys — **4 pts**
*depends on: T-060 · serves F-656 · actor: admin (any)*
- API subtasks:
  - `API-GW-015` · CRUD /admin/api-keys · gw — integration keys **(2 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-ADM-005` · ApiKeyManager · mfe-admin — keys **(2 pt)**

### T-062 · STEP-4C-06 — Issue refund — **4 pts**
*depends on: T-061 · serves F-147 · actor: admin / finance (any)*
- API subtasks:
  - `API-PAY-007` · POST /wallet/{patientId}/refund · pay — admin-issued refund **(3 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-ADM-006` · RefundAction · mfe-admin — issue refund **(1 pt)**

### T-063 · STEP-4C-07 — Onboard provider (doctor / nurse) — **4 pts**
*depends on: T-062 · serves F-606, F-609 · actor: ops_manager / admin (any)*
- API subtasks:
  - `API-PROV-020` · POST /providers/onboard · prov — onboard + assign role **(2 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-ADM-010` · ProviderOnboarding · mfe-admin — onboard — **covers both doctors and nurses**; required in the pilot since both see/serve patients from M4 **(2 pt)**

### T-064 · STEP-4C-08 — Verify credentials — **3 pts**
*depends on: T-063 · serves F-403, F-404 · actor: ops_manager / admin (any)*
- API subtasks:
  - `API-PROV-021` · POST /providers/{id}/verify · prov — credential verification **(2 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-ADM-011` · CredentialVerify · mfe-admin — verify **(1 pt)**

### T-065 · STEP-4C-09 — Monitor side-effect reports + assign reviewer — **6 pts**
*depends on: T-064 · serves F-620 · actor: ops_manager / admin (any)*
- API subtasks:
  - `API-PATIENT-031` · GET /admin/side-effects · patient — list incoming reports (urgent first) **(1 pt)**
  - `API-PATIENT-032` · POST /admin/side-effects/{id}/assign · patient — assign a reviewing doctor (routing only) **(3 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-ADM-043` · SideEffectMonitor · mfe-admin — ops oversight of the urgent-escalation path (RFLOW-05) — **read + assign/route only, no clinical write**; the **assigned** doctor performs review & disposition **(2 pt)**

### T-066 · STEP-4C-10 — Assign patient to provider — **4 pts**
*depends on: T-065 · serves F-610 · actor: ops_manager / admin (any)*
- API subtasks:
  - `API-PROV-022` · POST /assignments · prov — create assignment (drives `:assigned`) **(2 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-ADM-012` · AssignmentBoard · mfe-admin — assign — needed in the pilot so ops can assign/reassign doctors and nurses to patients from M4 **(2 pt)**

## M5 · F — Pharmacy & Logistics — 49 pts (17 tasks) — depends on: M4 · Nurse Home Visit (after T-047)

### T-067 · STEP-5-01 — Pharmacy inventory dashboard — **3 pts**
*depends on: T-047 · serves F-502 · actor: pharmacy (self)*
- API subtasks:
  - `API-PHARM-002` · GET /me/pharmacy/inventory · pharm — own hub inventory **(1 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-PHM-001` · InventoryDashboard · mfe-pharmacy — inventory **(2 pt)**

### T-068 · STEP-5-02 — Record supplier receipt — **3 pts**
*depends on: T-067 · serves F-503 · actor: pharmacy (self)*
- API subtasks:
  - `API-PHARM-003` · POST /me/pharmacy/receipts · pharm — supplier receipt **(2 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-PHM-002` · SupplierReceipt · mfe-pharmacy — log stock **(1 pt)**

### T-069 · STEP-5-03 — Monitor cold chain — **3 pts**
*depends on: T-068 · serves F-505 · [int·Elitech] · actor: pharmacy (self)*
- API subtasks:
  - `API-PHARM-005` · GET /me/pharmacy/coldchain · pharm — temp log **(1 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-PHM-004` · ColdChainMonitor · mfe-pharmacy — temp + alerts **(2 pt)**

### T-070 · STEP-5-04 — Dispense to nurse — **3 pts**
*depends on: T-069 · serves F-504 · actor: pharmacy (self)*
- API subtasks:
  - `API-PHARM-004` · POST /me/pharmacy/dispense · pharm — dispense (barcode + signature) **(2 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-PHM-003` · DispenseToNurse · mfe-pharmacy — scan + signature **(1 pt)**

### T-071 · STEP-5-05 — Report damage — **3 pts**
*depends on: T-070 · serves F-506 · actor: pharmacy (self)*
- API subtasks:
  - `API-PHARM-006` · POST /me/pharmacy/damage · pharm — damage report **(2 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-PHM-005` · DamageReport · mfe-pharmacy — report **(1 pt)**

### T-072 · STEP-5-06 — View audit log — **2 pts**
*depends on: T-071 · serves F-515 · actor: pharmacy (self), admin (any)*
- API subtasks:
  - `API-PHARM-007` · GET /me/pharmacy/audit · pharm — transaction log **(1 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-PHM-006` · AuditLogView · mfe-pharmacy — log **(1 pt)**

### T-073 · STEP-5-07 — Geocode patient address — **2 pts**
*depends on: T-072 · serves F-701, F-702 · [int·Optime] · actor: system/ops*
- API subtasks:
  - `API-FIELD-030` · POST /logistics/geocode · field — address → coords **(2 pt)**

### T-074 · STEP-5-08 — Generate optimized route — **2 pts**
*depends on: T-073 · serves F-703, F-704, F-705 · [int·Optime] · actor: system → nurse*
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-NUR-020` · RouteMapOptimized · mfe-nurse — route + position **(2 pt)**

### T-075 · STEP-5-09 — On-demand emergency assignment — **3 pts**
*depends on: T-074 · serves F-706 · [int·Optime] · actor: ops_manager (any) → nurse*
- API subtasks:
  - `API-FIELD-033` · POST /logistics/assign · field — on-demand assignment **(2 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-NUR-023` · EmergencyAssignment · mfe-nurse — push visit **(1 pt)**

### T-076 · STEP-5-10 — Live tracking + ETA — **6 pts**
*depends on: T-075 · serves F-707, F-708, F-172 · [int·Optime] · actor: nurse (self), patient (self)*
- API subtasks:
  - `API-FIELD-034` · GET /logistics/track/{visitId} · field — live position + ETA **(1 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-NUR-020` · RouteMapOptimized · mfe-nurse — position **(2 pt)**
  - `CMP-PAT-057` · NurseTracker · mfe-patient — live map + ETA **(3 pt)**

### T-077 · STEP-5-11 — Nurse assigned-patient list — **2 pts**
*depends on: T-076 · serves F-450, F-451, F-452, F-453, F-454 · actor: nurse (assigned)*
- API subtasks:
  - `API-FIELD-040` · GET /me/patients · field — assigned patients + search **(1 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-NUR-021` · AssignedPatients · mfe-nurse — list + search + notes **(1 pt)**

### T-078 · STEP-5-12 — Nurse inventory — **5 pts**
*depends on: T-077 · serves F-460, F-465 · [int·Elitech] · actor: nurse (self)*
- API subtasks:
  - `API-FIELD-041` · CRUD /me/inventory · field — own vial count, restock, cold log **(3 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-NUR-022` · InventoryManager · mfe-nurse — count + restock + cold log **(2 pt)**

### T-079 · STEP-5-13 — Reorder / reschedule — **1 pts**
*depends on: T-078 · serves F-420, F-422 · actor: nurse (self/assigned)*
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-NUR-024` · VisitReorder · mfe-nurse — reorder **(1 pt)**

### T-080 · STEP-5-14 — Nurse week/month summary — **2 pts**
*depends on: T-079 · serves F-408 · actor: nurse (self)*
- API subtasks:
  - `API-PROV-023` · GET /me/summary · prov — own week/month **(1 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-NUR-025` · NurseSummary · mfe-nurse — summary **(1 pt)**

### T-081 · STEP-5-15 — Support inbox — **3 pts**
*depends on: T-080 · serves F-410 · actor: nurse (self)*
- API subtasks:
  - `API-NOTIF-002` · POST /notify/push · notif — support notifications **(2 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-NUR-026` · SupportInbox · mfe-nurse — notifications **(1 pt)**

### T-082 · STEP-5-16 — Record patient education — **3 pts**
*depends on: T-081 · serves F-440 · actor: nurse (assigned)*
- API subtasks:
  - `API-FIELD-042` · POST /nurse-visits/{id}/education · field — log education **(2 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-NUR-027` · EducationRecorder · mfe-nurse — education **(1 pt)**

### T-083 · STEP-5-20 — External e-prescription integration live — **3 pts**
*depends on: T-082 · serves F-330 · [int·RxProvider] · actor: doctor (assigned)*
- API subtasks:
  - `API-VISIT-006` · GET /visits/{id}/prescription/status · visit — live fetch from the e-prescription provider **(2 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-DOC-004` · PrescriptionLink · mfe-doctor — status **(1 pt)**

## M5 · I — Safety Recovery — 13 pts (3 tasks) — depends on: M5 · Pharmacy & Logistics (after T-083); M4 · Nurse Home Visit (after T-047)

### T-084 · STEP-R01-04 — Open a replacement order — **5 pts**
*depends on: T-083 / T-047 · serves F-433 · [clinical] · actor: ops_manager (any), or system on a cold-chain/mismatch/recall abort*
- API subtasks:
  - `API-FIELD-050` · POST /replacements · field — open a replacement order **(2 pt)**
  - `API-FIELD-051` · GET /replacements · field — open-replacement queue **(1 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-ADM-041` · ReplacementQueue · mfe-admin — queue + action open replacements **(2 pt)**

### T-085 · STEP-R02-01 — Recall a batch (cascade) — **5 pts**
*depends on: T-084 · serves F-505, F-618 · [clinical/ops] · actor: admin / ops_manager (any)*
- API subtasks:
  - `API-PHARM-010` · POST /batches/{id}/recall · pharm — set recall + cascade to vials/inventory **(3 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-ADM-042` · RecallManager · mfe-admin — recall control + progress **(2 pt)**

### T-086 · STEP-R02-02 — Compute affected patients — **3 pts**
*depends on: T-085 · serves F-618 · actor: ops_manager (any)*
- API subtasks:
  - `API-PHARM-011` · GET /recalls/{id}/affected · pharm — affected read model **(1 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-ADM-042` · RecallManager · mfe-admin — affected list (scheduled vs administered) **(2 pt)**

## M6 · D — Online Visit & Doctor — 35 pts (11 tasks) — depends on: M3 · Online Visit & Doctor (after T-035)

### T-087 · STEP-6-01 — Doctor dashboard — **3 pts**
*depends on: T-035 · serves F-301, F-305 · actor: doctor (self)*
- API subtasks:
  - `API-VISIT-020` · GET /me/dashboard · visit — own counts, NPS, alerts, income **(1 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-DOC-010` · DoctorDashboard · mfe-doctor — dashboard **(2 pt)**

### T-088 · STEP-6-02 — Patient management — **4 pts**
*depends on: T-087 · serves F-311, F-313, F-314, F-315, F-317, F-319 · actor: doctor (assigned)*
- API subtasks:
  - `API-VISIT-021` · GET /patients/{id}/visit-history · visit — history + nurse report **(1 pt)**
  - `API-PATIENT-040` · GET /patients/{id}/charts · patient — weight/BMI series **(1 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-DOC-011` · PatientManagement · mfe-doctor — history, charts, notes, messaging **(2 pt)**

### T-089 · STEP-6-03 — Doctor scheduling — **4 pts**
*depends on: T-088 · serves F-340 · actor: doctor (self)*
- API subtasks:
  - `API-PROV-030` · CRUD /me/schedule · prov — own slots + block time **(2 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-DOC-012` · DoctorScheduleManager · mfe-doctor — slots **(2 pt)**

### T-090 · STEP-6-00 — Doctor login — **1 pts**
*depends on: T-089 · serves F-102 · actor: anonymous (pre-app)*
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-DOC-000` · DoctorLogin · mfe-doctor — login — **no self-signup**; providers are admin-provisioned (`identity_role.assigned_by = admin`); the screen states "no account? contact Medica admin" **(1 pt)**

### T-091 · STEP-6-09 — Doctor visits list — **2 pts**
*depends on: T-090 · serves F-301, F-313 · actor: doctor (assigned)*
- API subtasks:
  - `API-VISIT-026` · GET /me/visits?range=&status= · visit — the doctor's own visits (today + history) **(1 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-DOC-014` · DoctorVisitsList · mfe-doctor — Visits tab **(1 pt)**

### T-092 · STEP-6-10 — Doctor prescriptions list — **2 pts**
*depends on: T-091 · serves F-329, F-330 · actor: doctor (assigned)*
- API subtasks:
  - `API-VISIT-024` · GET /me/prescriptions · visit — the doctor's linked prescriptions (read) **(1 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-DOC-015` · DoctorPrescriptionsList · mfe-doctor — Prescriptions tab **(1 pt)**

### T-093 · STEP-6-04 — Doctor financial panel — **3 pts**
*depends on: T-092 · serves F-350, F-356, F-358 · actor: doctor (self)*
- API subtasks:
  - `API-PAY-020` · GET /me/finance · pay — own income + invoices **(1 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-DOC-013` · DoctorFinancePanel · mfe-doctor — finance **(2 pt)**

### T-094 · STEP-6-05 — Nurse financial panel — **3 pts**
*depends on: T-093 · serves F-470, F-477 · actor: nurse (self)*
- API subtasks:
  - `API-PAY-021` · GET /me/finance · pay — own income, bonuses, invoices **(1 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-NUR-030` · NurseFinancePanel · mfe-nurse — finance **(2 pt)**

### T-095 · STEP-6-06 — Nurse performance + hotline — **4 pts**
*depends on: T-094 · serves F-485, F-486, F-495 · actor: nurse (self)*
- API subtasks:
  - `API-PAY-022` · GET /me/performance · pay — own NPS, counts **(1 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-NUR-031` · NursePerformance · mfe-nurse — performance **(2 pt)**
  - `CMP-NUR-032` · EmergencyHotline · mfe-nurse — hotline **(1 pt)**

### T-096 · STEP-6-07 — Pharmacy commission — **4 pts**
*depends on: T-095 · serves F-507, F-508, F-512, F-513 · actor: pharmacy (self)*
- API subtasks:
  - `API-PAY-023` · GET /me/commission · pay — own commission + invoice **(2 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-PHM-010` · CommissionPanel · mfe-pharmacy — commission **(2 pt)**

### T-097 · STEP-6-08 — Run monthly settlements — **5 pts**
*depends on: T-096 · serves F-627, F-628, F-629 · actor: finance / admin (any)*
- API subtasks:
  - `API-PAY-024` · POST /settlements/run · pay — settle providers/pharmacies **(4 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-ADM-020` · SettlementRunner · mfe-admin — run settlements **(1 pt)**

## M7 · G — Patient Self-Service — 29 pts (9 tasks) — depends on: M4 · Patient Self-Service (after T-056)

### T-098 · STEP-7-01 — Adherence chart + streak — **2 pts**
*depends on: T-056 · serves F-185, F-186 · actor: patient (self)*
- API subtasks:
  - `API-PATIENT-050` · GET /me/adherence · patient — adherence + streak **(1 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-PAT-060` · AdherenceChart · mfe-patient — chart **(1 pt)**

### T-099 · STEP-7-02 — Weight goal — **3 pts**
*depends on: T-098 · serves F-188 · actor: patient (self)*
- API subtasks:
  - `API-PATIENT-051` · CRUD /me/goals · patient — weight goal + progress **(2 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-PAT-061` · WeightGoal · mfe-patient — goal **(1 pt)**

### T-100 · STEP-7-03 — Daily reminders — **3 pts**
*depends on: T-099 · serves F-189 · actor: patient (self)*
- API subtasks:
  - `API-PATIENT-052` · CRUD /me/reminders · patient — reminders **(2 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-PAT-062` · RemindersSettings · mfe-patient — reminders **(1 pt)**

### T-101 · STEP-7-04 — Content library — **3 pts**
*depends on: T-100 · serves F-195, F-196, F-197, F-199 · actor: patient (self)*
- API subtasks:
  - `API-PATIENT-053` · GET /content · patient — articles, videos, FAQ, guides **(2 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-PAT-063` · ContentLibrary · mfe-patient — content **(1 pt)**

### T-102 · STEP-7-05 — Foodnoise tracking — **4 pts**
*depends on: T-101 · serves F-223, F-224 · actor: patient (self)*
- API subtasks:
  - `API-PATIENT-054` · CRUD /me/foodnoise · patient — track + chart **(2 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-PAT-064` · FoodnoiseTracker · mfe-patient — log + chart **(2 pt)**

### T-103 · STEP-7-06 — Multiple addresses + weight history — **3 pts**
*depends on: T-102 · serves F-114, F-116, F-117 · actor: patient (self)*
- API subtasks:
  - `API-PATIENT-055` · CRUD /me/addresses · patient — addresses + weight history **(2 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-PAT-065` · AddressBook · mfe-patient — addresses + history **(1 pt)**

### T-104 · STEP-7-07 — Support chat + FAQ + emergency line — **3 pts**
*depends on: T-103 · serves F-207, F-209, F-210 · actor: patient (self) ↔ cs_agent*
- API subtasks:
  - `API-CRM-001` · CRUD /me/support/chat · crm — live support **(2 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-PAT-066` · SupportChat · mfe-patient — support + FAQ **(1 pt)**

### T-105 · STEP-7-08 — Visit chat + survey + history PDF — **3 pts**
*depends on: T-104 · serves F-154, F-156, F-158 · actor: patient (self)*
- API subtasks:
  - `API-VISIT-025` · CRUD /me/visits/{id}/chat · visit — chat + survey + history PDF **(2 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-PAT-067` · VisitChat · mfe-patient — chat + survey + PDF **(1 pt)**

### T-106 · STEP-7-09 — Pricing perks — **5 pts**
*depends on: T-105 · serves F-132, F-144 · actor: patient (self)*
- API subtasks:
  - `API-SUB-010` · POST /me/subscriptions/discount · sub — first-month discount **(2 pt)**
  - `API-PAY-025` · POST /me/wallet/cashback · pay — cashback **(2 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-PAT-068` · PricingPerks · mfe-patient — discount + cashback **(1 pt)**

## M8 · H — Admin / Ops & Finance/BI — 26 pts (8 tasks) — depends on: M4 · Admin / Ops & Finance/BI (after T-066)

### T-107 · STEP-8-01 — User management — **2 pts**
*depends on: T-066 · serves F-604, F-605, F-607, F-608 · actor: admin (any)*
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-ADM-030` · UserManagement · mfe-admin — users + activate **(2 pt)**

### T-108 · STEP-8-02 — Operations dashboard — **4 pts**
*depends on: T-107 · serves F-615, F-616, F-617, F-618 · actor: ops_manager (any)*
- API subtasks:
  - `API-GW-021` · GET /ops/dashboard · gw — visits, live map, cold-chain, incidents **(2 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-ADM-031` · OpsDashboard · mfe-admin — ops view **(2 pt)**

### T-109 · STEP-8-03 — Reschedule tool — **3 pts**
*depends on: T-108 · serves F-619 · actor: ops_manager (any)*
- API subtasks:
  - `API-GW-022` · POST /ops/reschedule · gw — admin reschedule **(2 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-ADM-032` · RescheduleTool · mfe-admin — reschedule **(1 pt)**

### T-110 · STEP-8-04 — Finance dashboard — **3 pts**
*depends on: T-109 · serves F-625, F-626, F-631, F-632, F-633 · actor: finance / admin (any)*
- API subtasks:
  - `API-PAY-030` · GET /finance/dashboard · pay — finance + BI **(1 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-ADM-033` · FinanceDashboard · mfe-admin — finance **(2 pt)**

### T-111 · STEP-8-05 — Growth dashboard — **3 pts**
*depends on: T-110 · serves F-640, F-641 · actor: ops_manager / admin (any)*
- API subtasks:
  - `API-RPT-001` · GET /growth · rpt — CAC/LTV, channels **(1 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-ADM-034` · GrowthDashboard · mfe-admin — growth **(2 pt)**

### T-112 · STEP-8-06 — Campaign manager — **5 pts**
*depends on: T-111 · serves F-644 · [int·Kavenegar] · actor: ops_manager / admin (any)*
- API subtasks:
  - `API-NOTIF-010` · POST /campaigns/sms · notif — SMS campaign **(3 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-ADM-035` · CampaignManager · mfe-admin — campaigns **(2 pt)**

### T-113 · STEP-8-07 — Settings — **4 pts**
*depends on: T-112 · serves F-652, F-653, F-657 · actor: admin (any)*
- API subtasks:
  - `API-GW-023` · CRUD /admin/settings · gw — rates, hubs, templates, and other tunables **(2 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-ADM-036` · SettingsPanel · mfe-admin — settings **(2 pt)**

### T-114 · STEP-8-08 — Reports + export — **2 pts**
*depends on: T-113 · serves F-660, F-662, F-663 · actor: admin / finance (any)*
- API subtasks:
  - `API-RPT-002` · GET /reports · rpt — scheduled reports, export, audit log **(1 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-ADM-037` · ReportsExport · mfe-admin — reports **(1 pt)**

## M9 · J — Customer Service & Affiliate — 29 pts (9 tasks) — depends on: M4 · Admin / Ops & Finance/BI (after T-066)

### T-115 · STEP-9-01 — CS call console + routing — **5 pts**
*depends on: T-066 · serves F-700, F-703 · actor: cs_agent (any)*
- API subtasks:
  - `API-CRM-010` · CRUD /cs/calls · crm — hotline, queue, routing, tickets **(3 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-CS-001` · CallConsole · mfe-cs — queue + ticket **(2 pt)**

### T-116 · STEP-9-02 to 9-07 — Six CS sub-panels — **4 pts**
*depends on: T-115 · serves F-710, F-761 · actor: cs_agent (any), scoped per panel*
- API subtasks:
  - `API-CRM-011` · CRUD /cs/panels/{logistics|marketing|affiliate|medical|new-patient|payment} · crm — per-domain queue + actions **(2 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-CS-002…007` · {Logistics,Marketing,Affiliate,Medical,NewPatient,Payment}Panel · mfe-cs — per-domain panel **(2 pt)**

### T-117 · STEP-9-08 — CS messaging + KPI — **2 pts**
*depends on: T-116 · serves F-770, F-771 · actor: cs_agent (any)*
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-CS-008` · CSDashboard · mfe-cs — KPI + messaging **(2 pt)**

### T-118 · STEP-9-09 — Emergency escalation in-software — **4 pts**
*depends on: T-117 · serves F-443 · actor: nurse (self) → cs_agent*
- API subtasks:
  - `API-FIELD-010` · POST /nurse-visits/{id}/emergency · field — escalate to CS **(3 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-NUR-010` · SafetyButton · mfe-nurse — escalation (now to CS) **(1 pt)**

### T-119 · STEP-9-10 — Affiliate dashboard — **3 pts**
*depends on: T-118 · serves F-800, F-801 · actor: affiliate (self)*
- API subtasks:
  - `API-CRM-020` · GET /me/affiliate/dashboard · crm — own performance **(1 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-AFF-001` · AffiliateDashboard · mfe-affiliate — dashboard **(2 pt)**

### T-120 · STEP-9-11 — Referral links + conversion — **3 pts**
*depends on: T-119 · serves F-802, F-803 · actor: affiliate (self)*
- API subtasks:
  - `API-CRM-021` · POST /me/affiliate/links · crm — links + funnel **(2 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-AFF-002` · ReferralLinks · mfe-affiliate — links + funnel **(1 pt)**

### T-121 · STEP-9-12 — Commission + payment — **4 pts**
*depends on: T-120 · serves F-804, F-805 · actor: affiliate (self, read), finance (any, pay)*
- API subtasks:
  - `API-CRM-022` · GET /me/affiliate/commission · crm — own commission **(2 pt)**
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-AFF-003` · CommissionPanel · mfe-affiliate — commission **(2 pt)**

### T-122 · STEP-9-13 — Affiliate onboarding + codes + audit — **3 pts**
*depends on: T-121 · serves F-807, F-808, F-809 · actor: admin (onboard, any), affiliate (codes, self)*
- Component subtasks *(depend on this task's APIs)*:
  - `CMP-AFF-004` · DiscountCodes · mfe-affiliate — codes **(1 pt)**
  - `CMP-ADM-040` · AffiliateOnboarding · mfe-admin — approve **(2 pt)**

### T-123 · STEP-9-14 — A/B experiments — **1 pts**
*depends on: T-122 · serves — · actor: ops_manager (any) configures, patient experiences*
- (no API/component — integration/config step)
