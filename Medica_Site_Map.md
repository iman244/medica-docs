# Medica — Site Map & Routing

A standalone companion to the screen inventory and build specification. It maps every screen (`SCR-`) to its **URL path** in the Next.js frontend, defines the routing architecture across the microfrontends, and specifies the health endpoint each deployable exposes.

Paths are the only new artifact here; every path resolves to an existing `SCR-` screen in `Medica_Screen_Inventory.md`.

---

## 1. Routing architecture

- **Host shell + federated remotes.** A single Next.js **host shell** (`CMP-SHL-001 AppShell`) owns top-level routing and composes the surface remotes (`mfe-patient`, `mfe-doctor`, …) via Module Federation. Each surface is mounted under a **path prefix**; the patient app is the root.
- **Path-prefix per surface (primary scheme):** `/` patient · `/doctor` · `/nurse` · `/pharmacy` · `/admin` · `/cs` · `/affiliate`. One origin, one shell, clean deep links. (Alternative: per-surface subdomains — see §6; the path map below holds either way, only the origin changes.)
- **Public marketing at root.** The `mfe-marketing` remote owns the public landing at `/`, plus `/bmi` and `/eligibility-check`. `/` serves the landing to anonymous visitors; an authenticated visitor at `/` is redirected into the app. Marketing pages are SSG/SSR for SEO; the authenticated app can be CSR.
- **App Router.** Each surface remote is a Next.js App-Router tree. Route **groups** separate public from protected: `(public)` for pre-auth (`/login`, `/signup`, `/reset-password`), `(app)` for everything behind a session. Each surface has its own `layout.tsx` (nav, role chrome, RTL direction).
- **Dynamic segments mirror the API access model.** Self screens use flat paths (the id comes from the session, like the API `/me/*`). Cross-subject screens use `[id]` segments — `/doctor/patients/[id]`, `/admin/patients/[id]`, `/nurse/visits/[id]` — mirroring the API `/{type}/{id}` convention and its `:assigned`/`:any` checks.
- **Auth + role gating in middleware.** `middleware.ts` validates the session (via `pkg-auth`) and the surface's required role before a protected route renders; a missing role yields the 403 screen (UP-X-03), an expired session a re-login (UP-X-02). The **public allowlist** — `/`, `/bmi`, `/eligibility-check`, `/login`, `/doctor/login`, `/signup`, `/reset-password`, public `/content`, and `/api/health*` — bypasses the session check; everything else requires one. The gateway remains the source of truth — the middleware only prevents rendering a screen the API would reject.
- **Locale + direction.** Locale is resolved in middleware (cookie → `identity.preferred_locale` → `Accept-Language` → `i18n.default_locale`) and sets document direction (RTL for `fa`, LTR for `en`); no mandatory `/[locale]` path prefix, so paths stay stable across languages (`Medica_Internationalization.md`). Public content pages may additionally accept `?lang=` for shareable links.

---

## 2. Health & ops endpoints (Next.js route handlers)

Every deployable Next.js app (the shell and each remote that ships independently) exposes a simple, **unauthenticated** health route via an App-Router route handler (`app/api/health/route.ts`), for Kubernetes liveness/readiness probes:

| path | purpose | returns |
|---|---|---|
| `GET /api/health` | **liveness** — the process is up | `200 { "status": "ok" }` |
| `GET /api/health/ready` | **readiness** — can serve traffic (optionally pings the gateway) | `200 { "status": "ready" }` / `503` when a dependency is down |

Notes: these are **frontend** health checks (the Next.js process); each FastAPI backend service exposes its own health route from the shared service template, separate from these. The health routes are excluded from `middleware.ts` auth and from locale handling, carry no PHI, and are the probe targets referenced by the DevOps/k8s setup. Under the path-prefix scheme the shell serves `/api/health`; if remotes deploy separately, each answers `/{prefix}/api/health` (e.g. `/nurse/api/health`).

---

## 3. Route map by surface

`kind`: **index** (surface landing) · **static** (fixed path) · **dynamic** (`[id]`/`[slug]` segment) · **public** (pre-auth) · **overlay** (modal/drawer/push — rendered over a route, no addressable page; in App Router typically a parallel/intercepting route).

### Marketing / landing — public (`/`)

| screen id | path | kind | screen (component) |
|---|---|---|---|
| `SCR-MKT-01` | `/` | index (public) | LandingHome (`CMP-MKT-001`) |
| `SCR-MKT-02` | `/bmi` | static (public) | BmiCalculator (`CMP-MKT-002`) |
| `SCR-MKT-03` | `/eligibility-check` | static (public) | EligibilityCheck (`CMP-MKT-003`) |

### Patient app — authenticated, root paths

| screen id | path | kind | screen (component) |
|---|---|---|---|
| `SCR-PAT-39` | `/home` | index | PatientHome (`CMP-PAT-000`) |
| `SCR-PAT-01` | `/signup` | public | SignUpForm (`CMP-PAT-001`) |
| `SCR-PAT-02` | `/login` | public | LoginForm (`CMP-PAT-002`) |
| `SCR-PAT-03` | `/reset-password` | public | PasswordResetFlow (`CMP-PAT-003`) |
| `SCR-PAT-04` | `/onboarding/kyc` | static | KycVerification (`CMP-PAT-004`) |
| `SCR-PAT-05` | `/onboarding/profile` | static | ProfileForm (`CMP-PAT-010`) |
| `SCR-PAT-06` | `/onboarding/health` | static | HealthDataForm (`CMP-PAT-011`) |
| `SCR-PAT-07` | `/onboarding/history` | static | MedicalHistoryForm (`CMP-PAT-012`) |
| `SCR-PAT-08` | `/onboarding/labs` | static | LabUploader (`CMP-PAT-013`) |
| `SCR-PAT-09` | `/onboarding/address` | static | AddressForm (`CMP-PAT-014`) |
| `SCR-PAT-10` | `/eligibility` | static | EligibilityWizard (`CMP-PAT-020`) |
| `SCR-PAT-11` | `/eligibility/result` | static | EligibilityResult (`CMP-PAT-021`) |
| `SCR-PAT-12` | `/consent` | static | ConsentForm (`CMP-PAT-022`) |
| `SCR-PAT-13` | `/plans` | static | PackagePicker (`CMP-PAT-030`) |
| `SCR-PAT-14` | `/checkout` | static | CheckoutFlow (`CMP-PAT-031`) |
| `SCR-PAT-15` | `/subscription` | static | SubscriptionManager (`CMP-PAT-032`) |
| `SCR-PAT-16` | `/subscription/cancel` | static | ExitInterview (`CMP-PAT-034`) |
| `SCR-PAT-17` | `/wallet` | static | WalletView (`CMP-PAT-033`) |
| `SCR-PAT-18` | `/visits/book` | static | VisitBooking (`CMP-PAT-040`) |
| `SCR-PAT-19` | `— (banner overlay)` | overlay | VisitReminderBanner (`CMP-PAT-041`) |
| `SCR-PAT-20` | `/visits/[visitId]` | dynamic | PrescriptionView (`CMP-PAT-042`) |
| `SCR-PAT-21` | `/visits/emergency` | static | EmergencyRequestButton (`CMP-PAT-043`) |
| `SCR-PAT-22` | `/nurse-visits` | index | NurseVisitSchedule (`CMP-PAT-050`) |
| `SCR-PAT-23` | `/nurse-visits/[id]/verify` | dynamic | NurseIdentityVerify (`CMP-PAT-051`) |
| `SCR-PAT-24` | `/nurse-visits/[id]/confirm-receipt` | dynamic | MedReceiptConfirm (`CMP-PAT-052`) |
| `SCR-PAT-25` | `/nurse-visits/[id]/reschedule` | dynamic | VisitTimeManager (`CMP-PAT-053`) |
| `SCR-PAT-40` | `/nurse-visits/schedule` | static | WeeklyScheduleManager (`CMP-PAT-069`) |
| `SCR-PAT-26` | `/nurse-visits/[id]/receipt` | dynamic | ServiceReceipt (`CMP-PAT-054`) |
| `SCR-PAT-27` | `/injections` | index | MyInjections (`CMP-PAT-055`) — list + weight chart |
| `SCR-PAT-28` | `/side-effects/report` | static | SideEffectReporter (`CMP-PAT-056`) |
| `SCR-PAT-29` | `/nurse-visits/[id]/track` | dynamic | NurseTracker (`CMP-PAT-057`) |
| `SCR-PAT-30` | `/progress` | static | AdherenceChart (`CMP-PAT-060`) |
| `SCR-PAT-31` | `/progress/weight-goal` | static | WeightGoal (`CMP-PAT-061`) |
| `SCR-PAT-32` | `/settings/reminders` | static | RemindersSettings (`CMP-PAT-062`) |
| `SCR-PAT-33` | `/content  ·  /content/[slug]` | index+dynamic | ContentLibrary (`CMP-PAT-063`) |
| `SCR-PAT-34` | `/foodnoise` | static | FoodnoiseTracker (`CMP-PAT-064`) |
| `SCR-PAT-35` | `/settings/addresses` | static | AddressBook (`CMP-PAT-065`) |
| `SCR-PAT-36` | `/support` | static | SupportChat (`CMP-PAT-066`) |
| `SCR-PAT-37` | `/visits/[visitId]/chat` | dynamic | VisitChat (`CMP-PAT-067`) |
| `SCR-PAT-38` | `/perks` | static | PricingPerks (`CMP-PAT-068`) |

### Doctor panel (`/doctor`)

| screen id | path | kind | screen (component) |
|---|---|---|---|
| `SCR-DOC-01` | `/doctor/patients` | index | PatientList (`CMP-DOC-001`) |
| `SCR-DOC-02` | `/doctor/patients/[id]` | dynamic | PatientProfilePanel (`CMP-DOC-002`) |
| `SCR-DOC-03` | `/doctor/visits/[id]/console` | dynamic | VisitConsole (`CMP-DOC-003`) |
| `SCR-DOC-04` | `/doctor/visits/[id]/prescription` | dynamic | PrescriptionLink (`CMP-DOC-004`) |
| `SCR-DOC-05` | `/doctor/visits/[id]/next` | dynamic | NextVisitScheduler (`CMP-DOC-005`) |
| `SCR-DOC-06` | `/doctor` | index | DoctorDashboard (`CMP-DOC-010`) |
| `SCR-DOC-07` | `/doctor/patients/[id]/manage` | dynamic | PatientManagement (`CMP-DOC-011`) |
| `SCR-DOC-08` | `/doctor/schedule` | static | DoctorScheduleManager (`CMP-DOC-012`) |
| `SCR-DOC-09` | `/doctor/finance` | static | DoctorFinancePanel (`CMP-DOC-013`) |
| `SCR-DOC-00` | `/doctor/login` | public | DoctorLogin (`CMP-DOC-000`) |
| `SCR-DOC-10` | `/doctor/visits` | index | DoctorVisitsList (`CMP-DOC-014`) |
| `SCR-DOC-11` | `/doctor/prescriptions` | index | DoctorPrescriptionsList (`CMP-DOC-015`) |

### Nurse app (`/nurse`)

| screen id | path | kind | screen (component) |
|---|---|---|---|
| `SCR-NUR-01` | `/nurse/login` | public | NurseLogin (`CMP-NUR-000`) |
| `SCR-NUR-02` | `/nurse` | index | NurseDashboard (`CMP-NUR-001`) |
| `SCR-NUR-03` | `/nurse/route` (range: today / tomorrow / week) | static | DailyRouteList (`CMP-NUR-002`) |
| `SCR-NUR-04` | `/nurse/visits/[id]/checkin` | dynamic | VisitCheckin (`CMP-NUR-003`) |
| `SCR-NUR-05` | `/nurse/visits/[id]` | dynamic | PatientVisitCard (`CMP-NUR-004`) |
| `SCR-NUR-06` | `/nurse/visits/[id]/cold-chain` | dynamic | ColdChainEntry (`CMP-NUR-005`) |
| `SCR-NUR-07` | `/nurse/visits/[id]/assessment` | dynamic | PreInjectionAssessment (`CMP-NUR-006`) |
| `SCR-NUR-08` | `/nurse/visits/[id]/injection` | dynamic | InjectionRecorder (`CMP-NUR-007`) |
| `SCR-NUR-09` | `/nurse/visits/[id]/signature` | dynamic | SignaturePad (`CMP-NUR-008`) |
| `SCR-NUR-10` | `/nurse/visits/[id]/submit` | dynamic | VisitSubmit (`CMP-NUR-009`) |
| `SCR-NUR-11` | `/nurse/safety` | overlay | SafetyButton (`CMP-NUR-010`) |
| `SCR-NUR-12` | `/nurse/route/map` | static | RouteMapOptimized (`CMP-NUR-020`) |
| `SCR-NUR-13` | `— (push notification)` | overlay | EmergencyAssignment (`CMP-NUR-023`) |
| `SCR-NUR-14` | `/nurse/patients` | index | AssignedPatients (`CMP-NUR-021`) |
| `SCR-NUR-15` | `/nurse/inventory` | static | InventoryManager (`CMP-NUR-022`) |
| `SCR-NUR-16` | `/nurse/route/reorder` | static | VisitReorder (`CMP-NUR-024`) |
| `SCR-NUR-17` | `/nurse/summary` | static | NurseSummary (`CMP-NUR-025`) |
| `SCR-NUR-18` | `/nurse/inbox` | static | SupportInbox (`CMP-NUR-026`) |
| `SCR-NUR-19` | `/nurse/visits/[id]/education` | dynamic | EducationRecorder (`CMP-NUR-027`) |
| `SCR-NUR-20` | `/nurse/finance` | static | NurseFinancePanel (`CMP-NUR-030`) |
| `SCR-NUR-21` | `/nurse/performance` | static | NursePerformance (`CMP-NUR-031`) |
| `SCR-NUR-22` | `/nurse/hotline` | overlay | EmergencyHotline (`CMP-NUR-032`) |

### Pharmacy-hub panel (`/pharmacy`)

| screen id | path | kind | screen (component) |
|---|---|---|---|
| `SCR-PHM-01` | `/pharmacy` | index | InventoryDashboard (`CMP-PHM-001`) |
| `SCR-PHM-02` | `/pharmacy/receipts` | static | SupplierReceipt (`CMP-PHM-002`) |
| `SCR-PHM-03` | `/pharmacy/cold-chain` | static | ColdChainMonitor (`CMP-PHM-004`) |
| `SCR-PHM-04` | `/pharmacy/dispense` | static | DispenseToNurse (`CMP-PHM-003`) |
| `SCR-PHM-05` | `/pharmacy/damage` | static | DamageReport (`CMP-PHM-005`) |
| `SCR-PHM-06` | `/pharmacy/audit` | static | AuditLogView (`CMP-PHM-006`) |
| `SCR-PHM-07` | `/pharmacy/commission` | static | CommissionPanel (`CMP-PHM-010`) |

### Admin / ops panel (`/admin`)

| screen id | path | kind | screen (component) |
|---|---|---|---|
| `SCR-ADM-01` | `/admin/patients` | index | PatientTable (`CMP-ADM-001`) |
| `SCR-ADM-02` | `/admin/patients/[id]` | dynamic | PatientDetail (`CMP-ADM-002`) |
| `SCR-ADM-03` | `/admin/protocols` | static | ProtocolEditor (`CMP-ADM-003`) |
| `SCR-ADM-04` | `/admin/roles` | static | RoleManager (`CMP-ADM-004`) |
| `SCR-ADM-05` | `/admin/api-keys` | static | ApiKeyManager (`CMP-ADM-005`) |
| `SCR-ADM-06` | `/admin/refunds` | static | RefundAction (`CMP-ADM-006`) |
| `SCR-ADM-07` | `/admin/providers/onboard` | static | ProviderOnboarding (`CMP-ADM-010`) |
| `SCR-ADM-08` | `/admin/providers/[id]/verify` | dynamic | CredentialVerify (`CMP-ADM-011`) |
| `SCR-ADM-09` | `/admin/assignments` | static | AssignmentBoard (`CMP-ADM-012`) |
| `SCR-ADM-10` | `/admin/replacements` | static | ReplacementQueue (`CMP-ADM-041`) |
| `SCR-ADM-11` | `/admin/recalls` | static | RecallManager (`CMP-ADM-042`) |
| `SCR-ADM-12` | `/admin/settlements` | static | SettlementRunner (`CMP-ADM-020`) |
| `SCR-ADM-13` | `/admin/users` | static | UserManagement (`CMP-ADM-030`) |
| `SCR-ADM-14` | `/admin/ops` | index | OpsDashboard (`CMP-ADM-031`) |
| `SCR-ADM-15` | `/admin/ops/reschedule` | static | RescheduleTool (`CMP-ADM-032`) |
| `SCR-ADM-16` | `/admin/finance` | static | FinanceDashboard (`CMP-ADM-033`) |
| `SCR-ADM-17` | `/admin/growth` | static | GrowthDashboard (`CMP-ADM-034`) |
| `SCR-ADM-18` | `/admin/campaigns` | static | CampaignManager (`CMP-ADM-035`) |
| `SCR-ADM-19` | `/admin/settings` | static | SettingsPanel (`CMP-ADM-036`) |
| `SCR-ADM-20` | `/admin/reports` | static | ReportsExport (`CMP-ADM-037`) |
| `SCR-ADM-21` | `/admin/affiliates` | static | AffiliateOnboarding (`CMP-ADM-040`) |

### Customer-service panel (`/cs`)

| screen id | path | kind | screen (component) |
|---|---|---|---|
| `SCR-CS-01` | `/cs` | index | CallConsole (`CMP-CS-001`) |
| `SCR-CS-02` | `/cs/logistics` | static | LogisticsPanel (`CMP-CS-002`) |
| `SCR-CS-03` | `/cs/marketing` | static | MarketingPanel (`CMP-CS-003`) |
| `SCR-CS-04` | `/cs/affiliate` | static | AffiliatePanel (`CMP-CS-004`) |
| `SCR-CS-05` | `/cs/medical` | static | MedicalPanel (`CMP-CS-005`) |
| `SCR-CS-06` | `/cs/new-patient` | static | NewPatientPanel (`CMP-CS-006`) |
| `SCR-CS-07` | `/cs/payment` | static | PaymentPanel (`CMP-CS-007`) |
| `SCR-CS-08` | `/cs/dashboard` | static | CSDashboard (`CMP-CS-008`) |

### Affiliate panel (`/affiliate`)

| screen id | path | kind | screen (component) |
|---|---|---|---|
| `SCR-AFF-01` | `/affiliate` | index | AffiliateDashboard (`CMP-AFF-001`) |
| `SCR-AFF-02` | `/affiliate/links` | static | ReferralLinks (`CMP-AFF-002`) |
| `SCR-AFF-03` | `/affiliate/commission` | static | CommissionPanel (`CMP-AFF-003`) |
| `SCR-AFF-04` | `/affiliate/codes` | static | DiscountCodes (`CMP-AFF-004`) |

### Shared (shell overlays)

| screen id | path | kind | screen (component) |
|---|---|---|---|
| `SCR-SHL-02` | `— (drawer overlay)` | overlay | NotificationCenter (`CMP-SHL-005`) |

---

## 4. Routing conventions

- **Self vs cross-subject.** A patient acting on their own data gets a flat path (`/wallet`, `/subscription`); a provider/admin acting on another subject gets an `[id]` segment (`/doctor/patients/[id]`). This keeps the URL aligned with the API path and its scope check.
- **Nested visit flows** are child routes of the visit, so progress and deep links are natural: the nurse home-visit sequence is `/nurse/visits/[id]/{checkin → cold-chain → assessment → injection → signature → submit}`, reflecting the enforced lifecycle order.
- **Overlays are not pages.** The notification drawer (`SCR-SHL-02`), the nurse safety action (`SCR-NUR-11`), reminder banners (`SCR-PAT-19`), and push-driven emergency assignment (`SCR-NUR-13`) render over the current route (parallel/intercepting routes), so they have no standalone path.
- **Notification deep links.** Every actionable notification links to the relevant path — dunning → `/subscription`, reschedule → `/nurse-visits/[id]/reschedule`, recall → `/nurse-visits` — rendered in the recipient's locale.
- **Trailing slashes off; lowercase, hyphenated segments.** Dynamic segment names (`[id]`, `[visitId]`, `[slug]`) match the data the screen loads.

---

## 5. Coverage

All 118 screens in `Medica_Screen_Inventory.md` have a path (112 addressable routes + 6 overlays), including the public marketing surface (`SCR-MKT-01..03`), `SCR-PAT-39 PatientHome`, and doctor `SCR-DOC-00/10/11`. `SCR-SHL-01 TwoFAModal` was retired (2FA removed). The four cross-cutting shell components that are pure infrastructure — `AppShell`, `SessionProvider`, `DesignSystem`, `LocaleProvider` — are not routes; `LocaleSwitcher` is a control in the shell chrome, not a page.

---

## 6. Open decisions (deployment & routing)

1. **Path-prefix vs subdomains.** One origin with `/doctor`, `/nurse`, … (assumed here) vs `doctor.medica.*`, `nurse.medica.*`. Path-prefix is simplest for one shell; subdomains isolate cookies/CSP per surface. The `SCR-`→path map is unaffected — only the origin and the health-route location change.
2. **Patient at root vs `/app`.** *Resolved:* the public landing (`mfe-marketing`) owns `/`; authenticated patient screens keep their root-level paths (`/wallet`, `/visits`, …) behind the middleware allowlist, and authed visitors at `/` are redirected into the app. (If a larger marketing site later needs the namespace, the app can still move under `/app` without changing any `SCR-`→path leaf.)
3. **Rendering strategy per surface.** Content/marketing (`/content`) benefits from SSR/SSG + SEO; authenticated panels can be CSR. Confirm per surface.
4. **Nurse login path.** `SCR-NUR-01` is shown at `/nurse/login`; if all roles share one `/login`, drop it and route post-auth by role.

---

## 7. Changes folded back

- **`Medica_Build_and_Technical_Specification.md`** — the DevOps note records that each Next.js deployable exposes `/api/health` (liveness) and `/api/health/ready` (readiness) as k8s probe targets, alongside the existing backend observability.
- No entities, APIs, or components change; this document adds `path` over the existing `SCR-` inventory and is read-only with respect to the other specs.
