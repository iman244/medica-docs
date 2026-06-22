# Medica — Screen Inventory

A standalone companion to the build specification, data model, lifecycle, unhappy-paths, recovery-flows, PHI-access, configuration-registry, and internationalization specs. It is the **design-ready handoff**: the build spec lists work in *build order* (by month), while a designer needs the inverse cut — every screen per surface, what it shows, the states it can be in, and how screens connect.

Nothing here is invented: every screen maps to an existing component (`CMP-`), the steps it realizes (`STEP-`), the features it serves (`F-`), and the APIs it calls (`API-`). The only new ids are the **screen ids** (`SCR-`). The navigation/journey map (§3) is a *proposed* first cut — some information architecture is a real design decision, flagged in §8.

---

## 1. How to read this document

- **Screen id** = `SCR-<surface>-<nn>` where surface ∈ `PAT, DOC, NUR, PHM, ADM, CS, AFF, SHL`, mirroring the microfrontend codes. One screen = one `CMP-` component; a component reused across several steps is one screen covering all of them.
- Each row links the screen to its **component**, the **feature(s)** it serves, the **step(s)** it realizes, the **API(s)** it calls, and the **month** it's implemented. The month is derived from the step id's leading number (`STEP-<month>…`); a screen built across several months shows each (e.g. `M3, M5`).
- **States, errors, and masking** that matter for design are called out per surface after each table (§4–§... notes), linking to the lifecycle badges, the unhappy-paths `UP-`/error patterns, the recovery `RFLOW-`s, and the PHI matrix.
- **Empty states** are designed per the unhappy-paths empty-state entries (UP-PAT-16, UP-NUR-10, UP-DOC-06, UP-PHM-06) and are not separate screens.

---

## 2. Surfaces at a glance

| surface | screens | primary device | role(s) |
|---|---|---|---|
| Marketing / landing (`mfe-marketing`) | 3 | responsive (public) | anonymous (pre-auth) |
| Patient app (`mfe-patient`) | 39 | mobile-first | patient |
| Doctor panel (`mfe-doctor`) | 12 | desktop-first | doctor |
| Nurse app (`mfe-nurse`) | 22 | mobile-first | nurse |
| Pharmacy-hub panel (`mfe-pharmacy`) | 7 | desktop-first | pharmacy |
| Admin / ops panel (`mfe-admin`) | 22 | desktop-first | ops_manager, finance, admin |
| Customer-service panel (`mfe-cs`) | 8 | desktop-first | cs_agent |
| Affiliate panel (`mfe-affiliate`) | 4 | desktop-first | affiliate |
| Shared (shell) | 1 | all | any |

---

## 3. Primary journey map (proposed)

The end-to-end patient path, with the surfaces that pick it up:

0. **Discover (public)** — `SCR-MKT-01 → 03` (landing home, BMI calculator, public eligibility check) → CTA into signup.
1. **Onboard** — `SCR-PAT-01 → 04` (sign up, verify phone, KYC); post-login hub is `SCR-PAT-39 PatientHome`.
2. **Profile & eligibility** — `SCR-PAT-05 → 12` (profile, health, history, labs, address, questionnaire, result, consent). Branches at `SCR-PAT-11`: ineligible/borderline → doctor review (RFLOW-18).
3. **Subscribe & pay** — `SCR-PAT-13 → 17` (package, checkout, manage, wallet). Branches on payment failure → RFLOW-12.
4. **Online visit** — patient `SCR-PAT-18 → 21`; doctor `SCR-DOC-01 → 05` (list, EMR, SOAP, link external prescription, next visit). External Rx rejection → RFLOW-19.
5. **Home injection** — patient `SCR-PAT-22 → 29` (schedule, verify nurse, track, confirm, report); nurse `SCR-NUR-04 → 10` (check-in, patient card, cold-chain, assessment, inject, sign, submit); pharmacy `SCR-PHM-04` dispenses. Safety branches: cold-chain → RFLOW-01, mismatch → RFLOW-03, abort → RFLOW-10, recall → RFLOW-02.
6. **Ongoing** — engagement `SCR-PAT-30 → 38`; doctor follow-up `SCR-DOC-06 → 09`; finance/ops/CS/affiliate panels.

Cross-cutting screens (`SCR-SHL-*`) — notifications, locale switch — overlay every step.

---

## 4. Screen tables

### Marketing / landing — `mfe-marketing`

| screen id | screen | component | serves | step(s) | api(s) | month |
|---|---|---|---|---|---|---|
| `SCR-MKT-01` | LandingHome | `CMP-MKT-001` | F-095 | `STEP-1-08` | — (static/SSG) | M1 |
| `SCR-MKT-02` | BmiCalculator | `CMP-MKT-002` | F-096 | `STEP-1-09` | — (client-side) | M1 |
| `SCR-MKT-03` | EligibilityCheck | `CMP-MKT-003` | F-097 | `STEP-2-10` | `API-PATIENT-023`, `API-PATIENT-024` | M2 |

### Patient app — `mfe-patient`

| screen id | screen | component | serves | step(s) | api(s) | month |
|---|---|---|---|---|---|---|
| `SCR-PAT-01` | SignUpForm | `CMP-PAT-001` | F-101 | `STEP-1-01`, `STEP-1-02` | `API-AUTH-001`, `API-AUTH-002` | M1 |
| `SCR-PAT-02` | LoginForm | `CMP-PAT-002` | F-102 | `STEP-1-03`, `STEP-1-04` | `API-AUTH-003` | M1 |
| `SCR-PAT-03` | PasswordResetFlow | `CMP-PAT-003` | F-103 | `STEP-1-05` | `API-AUTH-004/005` | M1 |
| `SCR-PAT-04` | KycVerification | `CMP-PAT-004` | F-105 | `STEP-1-07` | `API-AUTH-007` | M1 |
| `SCR-PAT-05` | ProfileForm | `CMP-PAT-010` | F-110 | `STEP-2-01` | `API-PATIENT-001` | M2 |
| `SCR-PAT-06` | HealthDataForm | `CMP-PAT-011` | F-111 | `STEP-2-02` | `API-PATIENT-002` | M2 |
| `SCR-PAT-07` | MedicalHistoryForm | `CMP-PAT-012` | F-112 | `STEP-2-03` | `API-PATIENT-003` | M2 |
| `SCR-PAT-08` | LabUploader | `CMP-PAT-013` | F-113 | `STEP-2-04` | `API-PATIENT-004` | M2 |
| `SCR-PAT-09` | AddressForm | `CMP-PAT-014` | F-115 | `STEP-2-05` | `API-PATIENT-005` | M2 |
| `SCR-PAT-10` | EligibilityWizard | `CMP-PAT-020` | F-121 | `STEP-2-06` | `API-PATIENT-020` | M2 |
| `SCR-PAT-11` | EligibilityResult | `CMP-PAT-021` | F-122, F-123, F-125, F-126 | `STEP-2-07` | `API-PATIENT-021` | M2 |
| `SCR-PAT-12` | ConsentForm | `CMP-PAT-022` | F-124 | `STEP-2-08` | `API-PATIENT-022` | M2 |
| `SCR-PAT-13` | PackagePicker | `CMP-PAT-030` | F-130 | `STEP-3A-01` | `API-SUB-001` | M3 |
| `SCR-PAT-14` | CheckoutFlow | `CMP-PAT-031` | F-131 | `STEP-3A-02` | `API-SUB-002`, `API-PAY-001/002` | M3 |
| `SCR-PAT-15` | SubscriptionManager | `CMP-PAT-032` | F-133 | `STEP-3A-03` | `API-SUB-003` | M3 |
| `SCR-PAT-16` | ExitInterview | `CMP-PAT-034` | F-139 | `STEP-3A-04` | `API-SUB-004` | M3 |
| `SCR-PAT-17` | WalletView | `CMP-PAT-033` | F-141, F-142, F-143, F-146 | `STEP-3A-05` | `API-PAY-003/004/005/006` | M3 |
| `SCR-PAT-18` | VisitBooking | `CMP-PAT-040` | F-150, F-151 | `STEP-3B-01` | `API-VISIT-001`, `API-VISIT-002` | M3 |
| `SCR-PAT-19` | VisitReminderBanner | `CMP-PAT-041` | F-152 | `STEP-3B-02` | `API-NOTIF-003` | M3 |
| `SCR-PAT-20` | PrescriptionView | `CMP-PAT-042` | F-155, F-159 | `STEP-3B-10` | `API-VISIT-003` | M3 |
| `SCR-PAT-21` | EmergencyRequestButton | `CMP-PAT-043` | F-157 | `STEP-3B-11` | `API-VISIT-008` | M3 |
| `SCR-PAT-22` | NurseVisitSchedule | `CMP-PAT-050` | F-170, F-220 | `STEP-4B-01` | `API-FIELD-020` | M4 |
| `SCR-PAT-23` | NurseIdentityVerify | `CMP-PAT-051` | F-221 | `STEP-4B-02` | `API-PROV-010` | M4 |
| `SCR-PAT-24` | MedReceiptConfirm | `CMP-PAT-052` | F-222 | `STEP-4B-03` | `API-FIELD-022` | M4 |
| `SCR-PAT-25` | VisitTimeManager | `CMP-PAT-053` | F-171 | `STEP-4B-04` | `API-FIELD-021` | M4 |
| `SCR-PAT-26` | ServiceReceipt | `CMP-PAT-054` | F-174 | `STEP-4B-05` | `API-FIELD-022` | M4 |
| `SCR-PAT-27` | MyInjections (list + weight chart) | `CMP-PAT-055` | F-175, F-179 | `STEP-4B-06` | `API-FIELD-023` | M4 |
| `SCR-PAT-28` | SideEffectReporter | `CMP-PAT-056` | F-177, F-178 | `STEP-4B-08` | `API-PATIENT-030` | M4 |
| `SCR-PAT-29` | NurseTracker | `CMP-PAT-057` | F-172, F-707, F-708 | `STEP-4B-09`, `STEP-5-10` | `API-FIELD-034` | M4, M5 |
| `SCR-PAT-30` | AdherenceChart | `CMP-PAT-060` | F-185, F-186 | `STEP-7-01` | `API-PATIENT-050` | M7 |
| `SCR-PAT-31` | WeightGoal | `CMP-PAT-061` | F-188 | `STEP-7-02` | `API-PATIENT-051` | M7 |
| `SCR-PAT-32` | RemindersSettings | `CMP-PAT-062` | F-189 | `STEP-7-03` | `API-PATIENT-052` | M7 |
| `SCR-PAT-33` | ContentLibrary | `CMP-PAT-063` | F-195, F-196, F-197, F-199 | `STEP-7-04` | `API-PATIENT-053` | M7 |
| `SCR-PAT-34` | FoodnoiseTracker | `CMP-PAT-064` | F-223, F-224 | `STEP-7-05` | `API-PATIENT-054` | M7 |
| `SCR-PAT-35` | AddressBook | `CMP-PAT-065` | F-114, F-116, F-117 | `STEP-7-06` | `API-PATIENT-055` | M7 |
| `SCR-PAT-36` | SupportChat | `CMP-PAT-066` | F-207, F-209, F-210 | `STEP-7-07` | `API-CRM-001` | M7 |
| `SCR-PAT-37` | VisitChat | `CMP-PAT-067` | F-154, F-156, F-158 | `STEP-7-08` | `API-VISIT-025` | M7 |
| `SCR-PAT-38` | PricingPerks | `CMP-PAT-068` | F-132, F-144 | `STEP-7-09` | `API-SUB-010`, `API-PAY-025` | M7 |
| `SCR-PAT-39` | PatientHome | `CMP-PAT-000` | F-100 | `STEP-2-11` | `API-PATIENT-025` | M2 |

### Doctor panel — `mfe-doctor`

| screen id | screen | component | serves | step(s) | api(s) | month |
|---|---|---|---|---|---|---|
| `SCR-DOC-01` | PatientList | `CMP-DOC-001` | F-310 | `STEP-3B-04` | `API-PROV-001` | M3 |
| `SCR-DOC-02` | PatientProfilePanel | `CMP-DOC-002` | F-312 | `STEP-3B-05` | `API-PATIENT-010` | M3 |
| `SCR-DOC-03` | VisitConsole | `CMP-DOC-003` | F-328 | `STEP-3B-06` | `API-VISIT-004` | M3 |
| `SCR-DOC-04` | PrescriptionLink | `CMP-DOC-004` | F-327, F-329, F-330 | `STEP-3B-07`, `STEP-3B-08`, `STEP-5-20` | `API-VISIT-005`, `API-VISIT-006` | M3, M5 |
| `SCR-DOC-05` | NextVisitScheduler | `CMP-DOC-005` | F-331 | `STEP-3B-09` | `API-VISIT-007` | M3 |
| `SCR-DOC-06` | DoctorDashboard | `CMP-DOC-010` | F-301, F-305 | `STEP-6-01` | `API-VISIT-020` | M6 |
| `SCR-DOC-07` | PatientManagement | `CMP-DOC-011` | F-311, F-313, F-314, F-315, F-317, F-319 | `STEP-6-02` | `API-VISIT-021`, `API-PATIENT-040`, `API-VISIT-022/023` | M6 |
| `SCR-DOC-08` | DoctorScheduleManager | `CMP-DOC-012` | F-340 | `STEP-6-03` | `API-PROV-030` | M6 |
| `SCR-DOC-09` | DoctorFinancePanel | `CMP-DOC-013` | F-350, F-356, F-358 | `STEP-6-04` | `API-PAY-020` | M6 |
| `SCR-DOC-00` | DoctorLogin | `CMP-DOC-000` | F-102 | `STEP-6-00` | `API-AUTH-002`, `API-AUTH-003` | M6 |
| `SCR-DOC-10` | DoctorVisitsList | `CMP-DOC-014` | F-301, F-313 | `STEP-6-09` | `API-VISIT-026` | M6 |
| `SCR-DOC-11` | DoctorPrescriptionsList | `CMP-DOC-015` | F-329, F-330 | `STEP-6-10` | `API-VISIT-024` | M6 |

### Nurse app — `mfe-nurse`

| screen id | screen | component | serves | step(s) | api(s) | month |
|---|---|---|---|---|---|---|
| `SCR-NUR-01` | NurseLogin | `CMP-NUR-000` | F-401 | `STEP-4A-01` | `API-AUTH-003` | M4 |
| `SCR-NUR-02` | NurseDashboard | `CMP-NUR-001` | F-405, F-407 | `STEP-4A-02` | `API-FIELD-002` | M4 |
| `SCR-NUR-03` | DailyRouteList (today/tomorrow/week) | `CMP-NUR-002` | F-415, F-416, F-417, F-418, F-409 | `STEP-4A-03` | `API-FIELD-001` | M4 |
| `SCR-NUR-04` | VisitCheckin | `CMP-NUR-003` | F-430 | `STEP-4A-04` | `API-FIELD-003` | M4 |
| `SCR-NUR-05` | PatientVisitCard | `CMP-NUR-004` | F-431, F-432 | `STEP-4A-05` | `API-FIELD-004` | M4 |
| `SCR-NUR-06` | ColdChainEntry | `CMP-NUR-005` | F-433 | `STEP-4A-06` | `API-FIELD-005` | M4 |
| `SCR-NUR-07` | PreInjectionAssessment | `CMP-NUR-006` | F-434 | `STEP-4A-07` | `API-FIELD-006` | M4 |
| `SCR-NUR-08` | InjectionRecorder | `CMP-NUR-007` | F-437 | `STEP-4A-08` | `API-FIELD-007` | M4 |
| `SCR-NUR-09` | SignaturePad | `CMP-NUR-008` | F-441 | `STEP-4A-09` | `API-FIELD-008` | M4 |
| `SCR-NUR-10` | VisitSubmit | `CMP-NUR-009` | F-442 | `STEP-4A-10` | `API-FIELD-009` | M4 |
| `SCR-NUR-11` | SafetyButton | `CMP-NUR-010` | F-443, F-497 | `STEP-4A-11`, `STEP-4A-12`, `STEP-9-09` | `API-FIELD-010`, `API-FIELD-011` | M4, M9 |
| `SCR-NUR-12` | RouteMapOptimized | `CMP-NUR-020` | F-703, F-704, F-705, F-707, F-708, F-172 | `STEP-5-08`, `STEP-5-10` | `API-FIELD-031/032`, `API-FIELD-034` | M5 |
| `SCR-NUR-13` | EmergencyAssignment | `CMP-NUR-023` | F-706 | `STEP-5-09` | `API-FIELD-033` | M5 |
| `SCR-NUR-14` | AssignedPatients | `CMP-NUR-021` | F-450, F-451, F-452, F-453, F-454 | `STEP-5-11` | `API-FIELD-040` | M5 |
| `SCR-NUR-15` | InventoryManager | `CMP-NUR-022` | F-460, F-465 | `STEP-5-12` | `API-FIELD-041` | M5 |
| `SCR-NUR-16` | VisitReorder | `CMP-NUR-024` | F-420, F-422 | `STEP-5-13` | `API-FIELD-001` | M5 |
| `SCR-NUR-17` | NurseSummary | `CMP-NUR-025` | F-408 | `STEP-5-14` | `API-PROV-023` | M5 |
| `SCR-NUR-18` | SupportInbox | `CMP-NUR-026` | F-410 | `STEP-5-15` | `API-NOTIF-002` | M5 |
| `SCR-NUR-19` | EducationRecorder | `CMP-NUR-027` | F-440 | `STEP-5-16` | `API-FIELD-042` | M5 |
| `SCR-NUR-20` | NurseFinancePanel | `CMP-NUR-030` | F-470, F-477 | `STEP-6-05` | `API-PAY-021` | M6 |
| `SCR-NUR-21` | NursePerformance | `CMP-NUR-031` | F-485, F-486, F-495 | `STEP-6-06` | `API-PAY-022` | M6 |
| `SCR-NUR-22` | EmergencyHotline | `CMP-NUR-032` | F-485, F-486, F-495 | `STEP-6-06` | `API-PAY-022` | M6 |

### Pharmacy-hub panel — `mfe-pharmacy`

| screen id | screen | component | serves | step(s) | api(s) | month |
|---|---|---|---|---|---|---|
| `SCR-PHM-01` | InventoryDashboard | `CMP-PHM-001` | F-502 | `STEP-5-01` | `API-PHARM-002` | M5 |
| `SCR-PHM-02` | SupplierReceipt | `CMP-PHM-002` | F-503 | `STEP-5-02` | `API-PHARM-003` | M5 |
| `SCR-PHM-03` | ColdChainMonitor | `CMP-PHM-004` | F-505 | `STEP-5-03` | `API-PHARM-005` | M5 |
| `SCR-PHM-04` | DispenseToNurse | `CMP-PHM-003` | F-504 | `STEP-5-04` | `API-PHARM-004` | M5 |
| `SCR-PHM-05` | DamageReport | `CMP-PHM-005` | F-506 | `STEP-5-05` | `API-PHARM-006` | M5 |
| `SCR-PHM-06` | AuditLogView | `CMP-PHM-006` | F-515 | `STEP-5-06` | `API-PHARM-007` | M5 |
| `SCR-PHM-07` | CommissionPanel | `CMP-PHM-010` | F-507, F-508, F-512, F-513 | `STEP-6-07` | `API-PAY-023` | M6 |

### Admin / ops panel — `mfe-admin`

| screen id | screen | component | serves | step(s) | api(s) | month |
|---|---|---|---|---|---|---|
| `SCR-ADM-01` | PatientTable | `CMP-ADM-001` | F-601 | `STEP-4C-01` | `API-GW-010` | M4 |
| `SCR-ADM-02` | PatientDetail | `CMP-ADM-002` | F-602, F-603 | `STEP-4C-02` | `API-GW-011/012` | M4 |
| `SCR-ADM-03` | ProtocolEditor | `CMP-ADM-003` | F-651 | `STEP-4C-03` | `API-GW-013` | M4 |
| `SCR-ADM-04` | RoleManager | `CMP-ADM-004` | F-655 | `STEP-4C-04` | `API-GW-014` | M4 |
| `SCR-ADM-05` | ApiKeyManager | `CMP-ADM-005` | F-656 | `STEP-4C-05` | `API-GW-015` | M4 |
| `SCR-ADM-06` | RefundAction | `CMP-ADM-006` | F-147 | `STEP-4C-06` | `API-PAY-007` | M4 |
| `SCR-ADM-07` | ProviderOnboarding | `CMP-ADM-010` | F-606, F-609 | `STEP-4C-07` | `API-PROV-020` | M4 |
| `SCR-ADM-08` | CredentialVerify | `CMP-ADM-011` | F-403, F-404 | `STEP-4C-08` | `API-PROV-021` | M4 |
| `SCR-ADM-09` | AssignmentBoard | `CMP-ADM-012` | F-610 | `STEP-4C-10` | `API-PROV-022` | M4 |
| `SCR-ADM-10` | ReplacementQueue | `CMP-ADM-041` | F-433 | `STEP-R01-04` | `API-FIELD-050`, `API-FIELD-051`, `API-PHARM-004`, `API-FIELD-033`, `API-FIELD-021` | — |
| `SCR-ADM-11` | RecallManager | `CMP-ADM-042` | F-505, F-618 | `STEP-R02-01`, `STEP-R02-02` | `API-PHARM-010`, `API-PHARM-011` | — |
| `SCR-ADM-12` | SettlementRunner | `CMP-ADM-020` | F-627, F-628, F-629 | `STEP-6-08` | `API-PAY-024` | M6 |
| `SCR-ADM-13` | UserManagement | `CMP-ADM-030` | F-604, F-605, F-607, F-608 | `STEP-8-01` | `API-GW-020` | M8 |
| `SCR-ADM-14` | OpsDashboard | `CMP-ADM-031` | F-615, F-616, F-617, F-618 | `STEP-8-02` | `API-GW-021` | M8 |
| `SCR-ADM-15` | RescheduleTool | `CMP-ADM-032` | F-619 | `STEP-8-03` | `API-GW-022` | M8 |
| `SCR-ADM-16` | FinanceDashboard | `CMP-ADM-033` | F-625, F-626, F-631, F-632, F-633 | `STEP-8-04` | `API-PAY-030` | M8 |
| `SCR-ADM-17` | GrowthDashboard | `CMP-ADM-034` | F-640, F-641 | `STEP-8-05` | `API-RPT-001` | M8 |
| `SCR-ADM-18` | CampaignManager | `CMP-ADM-035` | F-644 | `STEP-8-06` | `API-NOTIF-010` | M8 |
| `SCR-ADM-19` | SettingsPanel | `CMP-ADM-036` | F-652, F-653, F-657 | `STEP-8-07` | `API-GW-023` | M8 |
| `SCR-ADM-20` | ReportsExport | `CMP-ADM-037` | F-660, F-662, F-663 | `STEP-8-08` | `API-RPT-002` | M8 |
| `SCR-ADM-21` | AffiliateOnboarding | `CMP-ADM-040` | F-807, F-808, F-809 | `STEP-9-13` | `API-CRM-023` | M9 |
| `SCR-ADM-22` | SideEffectMonitor | `CMP-ADM-043` | F-620 | `STEP-4C-09` | `API-PATIENT-031`, `API-PATIENT-032` | M4 |

### Customer-service panel — `mfe-cs`

| screen id | screen | component | serves | step(s) | api(s) | month |
|---|---|---|---|---|---|---|
| `SCR-CS-01` | CallConsole | `CMP-CS-001` | F-700, F-703 | `STEP-9-01` | `API-CRM-010` | M9 |
| `SCR-CS-02` | LogisticsPanel | `CMP-CS-002` | F-710–F-761 | `STEP-9-02..07` | `API-CRM-011` | M9 |
| `SCR-CS-03` | MarketingPanel | `CMP-CS-003` | F-710–F-761 | `STEP-9-02..07` | `API-CRM-011` | M9 |
| `SCR-CS-04` | AffiliatePanel | `CMP-CS-004` | F-710–F-761 | `STEP-9-02..07` | `API-CRM-011` | M9 |
| `SCR-CS-05` | MedicalPanel | `CMP-CS-005` | F-710–F-761 | `STEP-9-02..07` | `API-CRM-011` | M9 |
| `SCR-CS-06` | NewPatientPanel | `CMP-CS-006` | F-710–F-761 | `STEP-9-02..07` | `API-CRM-011` | M9 |
| `SCR-CS-07` | PaymentPanel | `CMP-CS-007` | F-710–F-761 | `STEP-9-02..07` | `API-CRM-011` | M9 |
| `SCR-CS-08` | CSDashboard | `CMP-CS-008` | F-770, F-771 | `STEP-9-08` | `API-CRM-012` | M9 |

### Affiliate panel — `mfe-affiliate`

| screen id | screen | component | serves | step(s) | api(s) | month |
|---|---|---|---|---|---|---|
| `SCR-AFF-01` | AffiliateDashboard | `CMP-AFF-001` | F-800, F-801 | `STEP-9-10` | `API-CRM-020` | M9 |
| `SCR-AFF-02` | ReferralLinks | `CMP-AFF-002` | F-802, F-803 | `STEP-9-11` | `API-CRM-021` | M9 |
| `SCR-AFF-03` | CommissionPanel | `CMP-AFF-003` | F-804, F-805 | `STEP-9-12` | `API-CRM-022` | M9 |
| `SCR-AFF-04` | DiscountCodes | `CMP-AFF-004` | F-807, F-808, F-809 | `STEP-9-13` | `API-CRM-023` | M9 |

### Shared (shell) — `shell`

| screen id | screen | component | serves | step(s) | api(s) | month |
|---|---|---|---|---|---|---|
| `SCR-SHL-02` | NotificationCenter | `CMP-SHL-005` | F-205, F-176 | `STEP-2-09`, `STEP-4B-07` | `API-NOTIF-001/002`, `API-NOTIF-003` | M2, M4 |


---

## 5. States, errors & masking notes

Only screens with notable lifecycle states, recovery handoffs, or PHI masking are listed; the rest are standard forms/lists with the usual loading/empty/5xx patterns (UP-X-05) and empty states.

### Marketing / landing
- `SCR-MKT-02` BmiCalculator — client-side only; **anonymous, nothing persisted, no PHI** (PHI matrix §7); invalid input UP-MKT-01.
- `SCR-MKT-03` EligibilityCheck — **non-binding preliminary indicator, no stored PHI**; not-a-fit/borderline framing UP-MKT-02 (`[clinical]`); preview unavailable degrades but still allows signup UP-MKT-03; the real ineligible→review path is RFLOW-18 post-signup.

### Patient app
- `SCR-PAT-01/02` — OTP wrong/expired (UP-PAT-01), login lockout (UP-PAT-02); `identity` states.
- `SCR-PAT-04` KycVerification — `kyc_status` unverified→pending→verified/failed (UP-PAT-03); manual review → RFLOW-20.
- `SCR-PAT-11` EligibilityResult — `result` eligible/borderline/ineligible (a **categorical** determination, not a score); UP-PAT-05/06; copy is `[clinical]`; ineligible/borderline → RFLOW-18. The product's most emotionally loaded screen.
- `SCR-PAT-14` CheckoutFlow — `payment` pending/paid/failed and `subscription` pending_payment→active; failure → UP-PAT-07 → RFLOW-12; **idempotent**.
- `SCR-PAT-15` SubscriptionManager — `subscription` trial/active/past_due/cancelled/expired; past_due banner (UP-PAT-08 → RFLOW-13), expired mid-treatment (UP-PAT-09 → RFLOW-14, `[clinical]`).
- `SCR-PAT-17` WalletView — `wallet_transaction` pending/failed; refund handling UP-PAT-13 → RFLOW-15.
- `SCR-PAT-18` VisitBooking — `visit` scheduled/no_show; missed visit UP-PAT-10.
- `SCR-PAT-20` PrescriptionView — `prescription` issued (user) / Sent / rejected (internal).
- `SCR-PAT-22` NurseVisitSchedule — `nurse_visit` Scheduled/Nurse-on-the-way/Completed/aborted/rescheduled; not-home UP-PAT-11 → RFLOW-08.
- `SCR-PAT-28` SideEffectReporter — urgent flag → UP-PAT-14 → RFLOW-05; emergency guidance `[clinical]`.
- `SCR-PAT-29` NurseTracker — tracking unavailable degrades gracefully (UP-PAT-12, `[int·Optime]`).

### Doctor panel
- `SCR-DOC-02` PatientProfilePanel — assigned doctor sees the **full EMR including the eligibility result + answers** (PHI matrix §4.B); incomplete record UP-DOC-03.
- `SCR-DOC-04` PrescriptionLink — Rx is authored in **external** software; Medica links by `external_rx_id` and mirrors status `linked→active/rejected`; external rejection UP-DOC-01 → RFLOW-19; provider unreachable UP-DOC-02 (retry-fetch).
- `SCR-DOC-00` DoctorLogin — password/OTP; **no self-signup** (admin-provisioned); session expiry re-auth UP-X-02.
- `SCR-DOC-10/11` DoctorVisitsList / DoctorPrescriptionsList — read lists of the doctor's own visits / linked prescriptions.
- `SCR-DOC-01/06/07` — assignment lost mid-care UP-DOC-04 → RFLOW-11; provider deactivated UP-DOC-05.

### Nurse app
- `SCR-NUR-04` VisitCheckin — GPS fail/off-location UP-NUR-03 (manual override config-gated); `nurse_visit` → checked_in.
- `SCR-NUR-05` PatientVisitCard — **minimum-necessary PHI: allergies + current dose only** (PHI matrix §4.B baseline).
- `SCR-NUR-06` ColdChainEntry — breach → `nurse_visit` blocked (UP-NUR-01, `[clinical]`) → RFLOW-01.
- `SCR-NUR-08` InjectionRecorder — vial mismatch UP-NUR-02 (`[clinical]`) → RFLOW-03; records `injection_record`.
- `SCR-NUR-09` SignaturePad — capture fails → OTP fallback UP-NUR-08.
- `SCR-NUR-10` VisitSubmit — abort → `nurse_visit` aborted (UP-NUR-04) → RFLOW-10.
- `SCR-NUR-11` SafetyButton — safety alert / emergency escalation UP-NUR-05 → RFLOW-06.
- `SCR-NUR-12` RouteMapOptimized — Optime down → manual list (UP-NUR-07).
- `SCR-NUR-15` InventoryManager — `nurse_inventory` in_stock/used/returned/damaged; shortage UP-NUR-09 → RFLOW-09; **surfaces recalled-stock flag (RFLOW-02)**.

### Pharmacy-hub panel
- `SCR-PHM-01` InventoryDashboard — `vial` in_hub/dispensed/used/damaged/expired; expired stock UP-PHM-05.
- `SCR-PHM-03` ColdChainMonitor — hub breach UP-PHM-01 (`[clinical]`) → RFLOW-04.
- `SCR-PHM-04` DispenseToNurse — barcode scan fail UP-PHM-03; stock-out UP-PHM-04 → RFLOW-09.

### Admin / ops panel
- `SCR-ADM-02` PatientDetail — **`national_id` is never shown; PHI rendered per the access matrix** (masked/withheld by relationship).
- `SCR-ADM-04` RoleManager — role/permission misconfig validation UP-ADM-06.
- `SCR-ADM-06` RefundAction — refund/refund-pending RFLOW-15.
- `SCR-ADM-08` CredentialVerify — `provider_profile` pending/verified/active.
- `SCR-ADM-09` AssignmentBoard — no nurse available RFLOW-07; provider reassignment RFLOW-11.
- `SCR-ADM-10` ReplacementQueue — `replacement_order` requested/dispatched/fulfilled (RFLOW-01).
- `SCR-ADM-11` RecallManager — `recall` initiated/cascaded/closed and `batch.recall_status` (RFLOW-02, `[clinical/ops]`).
- `SCR-ADM-12` SettlementRunner — `settlement_run` pending/completed/failed; failure UP-ADM-01 → RFLOW-17.
- `SCR-ADM-14` OpsDashboard — **the must-alert triage hub** (`incident`): no-capacity UP-ADM-04, cold-chain/incident UP-ADM-05.
- `SCR-ADM-15` RescheduleTool — admin reschedule RFLOW-08.
- `SCR-ADM-16` FinanceDashboard — chargebacks UP-ADM-02 → RFLOW-16.

### Customer-service panel
- `SCR-CS-01..07` — `ticket` open/in_progress/resolved/closed; the **generic-CS-ticket resolver** for edge unhappy paths (PHI: `SCR-CS-05 MedicalPanel` shows a `Dv` problem-summary, **not** the full EMR — PHI open decision #1).

### Affiliate panel
- `SCR-AFF-01/02` — referral funnel signup→first_visit→first_injection→retained; **never exposes `referred_identity_id` (PHI matrix §5.C)**.
- `SCR-AFF-03` CommissionPanel — commission held/dispute UP-AFF-02 (clawback config).
- `SCR-AFF-04`/onboarding — application rejected UP-AFF-03.

### Shared (shell)
- `SCR-SHL-02` NotificationCenter — in-app/push, rendered in the recipient's locale. (LocaleSwitcher + LocaleProvider are shell infrastructure per `Medica_Internationalization.md`.) *(2FA removed — `SCR-SHL-01 TwoFAModal` retired; session expiry re-auth uses the normal login.)*

---

## 6. Presentation conventions

- **Direction.** RTL for `fa`, LTR for `en`; one document-direction switch in the shell, logical (start/end) CSS — no screen designed twice (`Medica_Internationalization.md`).
- **Dates & numerals.** Jalali calendar + Persian numerals for `fa-IR`; Gregorian + Latin for `en`. Storage is UTC; convert at presentation only.
- **Money.** Stored as integer minor units (Rial); formatted per locale at presentation; never a float.
- **Status badges.** The complete user-visible badge set and tones (`neutral · info · success · warning · danger`) is defined in the lifecycle spec — that is the badge inventory the design system builds.
- **Error/empty patterns.** The unhappy-paths taxonomy (`toast · inline · banner · full-screen state · blocking modal · badge`) is the error-display inventory.
- **Masking.** Field-level visibility per screen follows the PHI matrix: `national_id` never shown, `bank_iban` last-4, phone partially masked to non-self roles, free-text PHI shown verbatim and never machine-translated.
- **Device targets.** Patient and nurse apps are mobile-first; doctor, pharmacy, admin, CS, and affiliate are desktop-first panels.

---

## 7. Coverage

Every `CMP-` component in the build spec maps to exactly one `SCR-`; components reused across steps (e.g. `CMP-DOC-004` across `STEP-3B-07/08/5-20`) are one screen with multiple steps. **118 screens total** (incl. marketing `SCR-MKT-01..03`, `SCR-PAT-39 PatientHome`, and doctor `SCR-DOC-00/10/11`). Components that are pure infrastructure — `CMP-SHL-001 AppShell`, `CMP-SHL-002 SessionProvider`, `CMP-SHL-003 DesignSystem`, `CMP-SHL-006 LocaleProvider`, and `CMP-DOC-020 DoctorShell` (panel nav chrome) — are not screens. `SCR-SHL-01 TwoFAModal` was retired (2FA removed). `STEP-9-14` (A/B experiments) has no screen of its own.

---

## 8. Open information-architecture decisions (proposed, for design to refine)

These are genuine design choices the specs imply but do not fix:

1. **Patient navigation grouping** — e.g. Home / Treatment / Wallet / Content / Support as bottom-nav; the 38 patient screens need an IA, not a flat list.
2. **Onboarding as a linear wizard** — `SCR-PAT-01 → 12` as a guided flow vs. a dashboard with tasks.
3. **Nurse visit as a step flow** — `SCR-NUR-04 → 10` as an enforced sequence (check-in gates cold-chain gates injection), reflecting the lifecycle order.
4. **Admin panel grouping** — patients / providers / logistics / finance / config as sections across the 21 admin screens.
5. **CS sub-panel routing** — how `SCR-CS-02 → 07` surface from the queue (`SCR-CS-01`).

---

## 9. Changes folded back

No new entities, APIs, or components — this document only adds `SCR-` ids over the existing inventory and is read-only with respect to the other specs. It will need a row added whenever a future `CMP-` is introduced (e.g. when the deferred offline or clinical-safety work adds screens).
