# Medica — P0 Features (English)

> P0 = required for MVP launch. Without these the system cannot go live.
> This is the English translation of all P0-priority items from *Medica Feature List v2.1*.

---

## 4. Platform — Internationalization (cross-cutting)
*Added to make multilanguage a first-class concern; owned by `Medica_Internationalization.md`. Applies across every surface.*
- **F-090** — Multilanguage UI (message catalogs per surface; `fa-IR` default, `en` next, config-extensible)
- **F-091** — In-app language switcher; persists as `identity.preferred_locale`
- **F-092** — RTL/LTR layout by locale (no per-screen redesign)
- **F-093** — Locale-aware formatting: Jalali/Gregorian dates, Persian/Latin numerals, currency/number
- **F-094** — Notifications (SMS/push/email) rendered in the recipient's preferred locale

---

## 4b. Public site — Landing & marketing (cross-cutting)
*Added; the public, pre-auth entry point. Owned by `mfe-marketing`. BMI calculator and the public eligibility check are mandatory P0; other sections (pricing, FAQ, testimonials, about, content/blog) can follow.*
- **F-095** — Landing home: what Medica does, "how it works," CTAs into signup (config-gated sections)
- **F-096** — BMI calculator (public, client-side, no storage) — **mandatory**
- **F-097** — Public eligibility check: preliminary, non-binding indicator from a question subset, funnels to signup `[clinical]` — **mandatory**

---

## 5. Patient App

### 5.0 Home
- **F-100** — Patient home / dashboard (treatment status, next visit, next injection, quick actions, alerts) — the post-login hub

### 5.1 Authentication & Onboarding
- **F-101** — Sign up with mobile number + OTP (+ email)
- **F-102** — Login with password or OTP (+ email)
- **F-103** — Password recovery via SMS
- ~~**F-104** — Two-factor authentication (2FA)~~ — **removed** (per product decision; not used)
- **F-105** — Identity verification with national ID (KYC)

### 5.2 Profile & Health Data
- **F-110** — Complete basic profile (name, age, gender, address)
- **F-111** — Enter health info (height, weight, automatic BMI)
- **F-112** — Medical history (underlying conditions, allergies, current medication)
- **F-113** — Upload lab results (PDF, JPG, up to 10 images)
- **F-114** — Save multiple addresses (work, home, …)
- **F-115** — Select current address for medication delivery / nurse visit
- **F-116** — Weight history (weekly/monthly line chart)
- **F-117** — Manual weight entry + weekly reminder

### 5.3 Eligibility Assessment
- **F-121** — Medical questionnaire, 15–20 questions (history, BMI, medication)
- **F-122** — Automatic GLP-1 eligibility calculation (rule engine)
- **F-123** — Show result and recommendation (eligible / needs medical assessment / ineligible)
- **F-124** — Digital informed consent request
- **F-125** — Automatic referral to doctor for borderline cases
- **F-126** — Save answers in record — visible to doctor

### 5.4 Subscription
- **F-130** — Show subscription packages (Basic/Standard/Premium) — price, included services
- **F-131** — Purchase / auto-renew subscription via Venda payment facilitator
- **F-132** — First month 20% discount — medication and subscription paid separately (if medication issue)
- **F-133** — Auto-renew subscription — can be stopped by patient
- **F-139** — Cancel subscription with exit-interview process

### 5.5 Wallet
- **F-141** — Internal wallet — show current balance, top-up history
- **F-142** — Top up wallet via payment gateway (online)
- **F-143** — Full transaction history (wallet transactions log)
- **F-144** — Percentage payback — automatic cashback to wallet
- **F-146** — Pay from wallet for medication and services
- **F-147** — Refund to wallet (in case of medication or service issue)

### 5.6 Online Visit Booking & Scheduling
- **F-150** — Remove doctor selection — in initial phase, one doctor visits all patients
- **F-151** — Book online visit time (doctor's calendar)
- **F-152** — Visit reminder (SMS + Push, 24h and 1h before)
- **F-153** — Phone visit (phase 0 — whole system by phone, simplified)
- **F-154** — Text chat
- **F-155** — Show prescription and instructions after visit
- **F-156** — Survey as a form during home service (not online)
- **F-157** — Emergency visit request (in side-effect situations)
- **F-158** — Visit history — downloadable PDF
- **F-159** — Download digital prescription (JPG, PDF)

### 5.7 Nurse Visit & Medication Management
- **F-170** — View nurse visit schedule + see nurse profile + confirm medication receipt
- **F-171** — Confirm/change nurse visit time
- **F-172** — Live tracking of nurse location en route (Optime AI)
- **F-174** — Digital confirmation of service receipt (signature/OTP)
- **F-175** — My Injections — injection history (date, dose, vial)
- **F-176** — Next-injection reminder
- **F-177** — Side-effect report (drop-down + free text)
- **F-178** — Urgent flag for severe side effect → referral to doctor
- **F-179** — Weight-across-injections chart on My Injections (overlays the patient's own weight log on the injection timeline); replaces the **patient-app** dashboard "my prescriptions" tile

### 5.8 Adherence
- **F-185** — Weekly/monthly adherence chart
- **F-186** — Streak counter (consecutive successful weeks)
- **F-188** — Weight goal setting and progress
- **F-189** — Daily reminders (water, activity, sleep)

### 5.9 Educational Content
- **F-195** — Articles and educational videos about GLP-1
- **F-196** — Dynamic FAQ
- **F-197** — Injection guide (for self-injection)
- **F-199** — Side-effect management (comprehensive guide)

### 5.10 Notifications & Support
- **F-205** — Push notification + SMS fallback (Pushe + Kavenegar)
- **F-207** — Live chat with support (during working hours)
- **F-209** — FAQ self-service
- **F-210** — Emergency call to Medica emergency line (24/7)

---

## 6. Doctor Web Panel

### 6.1 Dashboard & Summary
- **F-301** — Daily dashboard (today's visits, notifications, key numbers)
- **F-302** — Active patient count + new patients this week
- **F-303** — Average patient satisfaction (NPS)
- **F-304** — Urgent notifications (urgent visit, severe side effect)
- **F-305** — Current-month income summary

### 6.2 My Patients
- **F-310** — Categorized patient list: new / in follow-up / no result (not a full list)
- **F-311** — Show each patient's status (active, paused, churned)
- **F-312** — Show full patient profile + EMR
- **F-313** — Patient visit history
- **F-314** — Injection history and nurse report
- **F-315** — Weight, BMI, measurement charts
- **F-316** — Patient side-effects report
- **F-317** — Doctor private notes
- **F-319** — Message patient only via app and Customer Service (not direct)

### 6.3 Conducting the Online Visit
- **F-327** — Prescription template — API connection to external prescription software
- **F-328** — Record SOAP notes in visit
- **F-329** — Digital prescription — via external prescription software API
- **F-330** — Send prescription to the e-prescription system
- **F-331** — Set next visit time (auto-schedule)

### 6.4 Calendar & Availability
- **F-340** — Doctor scheduling — managed only by the company side (not patient)
- **F-341** — Block time (leave, meeting)

### 6.5 Doctor Financial Panel
- **F-350** — Monthly income dashboard (amount, visit count)
- **F-351** — Daily report: visit count + amount
- **F-352** — Weekly + monthly report with charts
- **F-353** — Income breakdown (initial visit, follow-up, emergency)
- **F-354** — Commission report from patient referral
- **F-355** — History of payments from Medica to doctor
- **F-356** — Downloadable monthly invoice (PDF)
- **F-358** — Set bank info (IBAN / Sheba number)

---

## 7. Nurse Panel (Mobile + Web)

### 7.1 Authentication & Access
- **F-401** — Login with mobile number + OTP
- **F-403** — Upload and verify nursing credentials (license, national ID card)
- **F-404** — Background-check status (admin verification status)

### 7.2 Nurse Dashboard
- **F-405** — Nurse dashboard — confirm medication receipt + quality questionnaire + patient review
- **F-406** — Emergency visit notification (urgent assignment)
- **F-407** — Next status (ready, en route, in visit, break)
- **F-408** — Week/month summary (visit count, income)
- **F-409** — Schedule horizon — view tomorrow's and the week-ahead assigned visits on the route screen (today/tomorrow/week range selector; navigation/ETA active for today only)
- **F-410** — Notifications from Medica support

### 7.3 Daily Route (Optime AI)
- **F-415** — Show optimized daily route (map + ordered list)
- **F-416** — Estimated time per visit + arrival time
- **F-417** — Internal navigation (or redirect to Neshan/Balad)
- **F-418** — "En route" notification — location update for patient
- **F-420** — Change visit order (with Optime AI re-confirmation)
- **F-421** — Emergency visit (push assignment)
- **F-422** — Reschedule a visit (if patient unavailable)

### 7.4 Visit Workflow
- **F-430** — Check-in to visit (GPS verification)
- **F-431** — Show full patient profile
- **F-432** — Show doctor's prescription and today's injection dose
- **F-433** — Cold-chain check: record temperature at delivery (from datalogger)
- **F-434** — Pre-injection assessment (vital signs, recent side effects)
- **F-437** — Record injection site (rotation chart to prevent lipohypertrophy)
- **F-440** — Record patient education — covered in service-quality questionnaire
- **F-441** — Patient digital signature (service-receipt confirmation)
- **F-442** — Submit visit + automatic packaging into the financial system
- **F-443** — Report emergency issue — routed to Customer Service (callcenter), not doctor

### 7.5 Assigned Patients List
- **F-450** — Patient list — patients not assigned to a specific nurse (dynamic servicing)
- **F-451** — Search + filter (active, paused, area)
- **F-452** — History of visits to each patient by me
- **F-453** — Nurse notes about each patient
- **F-454** — Show next visit time for each patient

### 7.6 Nurse Personal Inventory
- **F-460** — Current nurse inventory (vial count, expiry date)
- **F-461** — Pickup from pharmacy hub (with barcode + signature)
- **F-462** — Record inventory after each delivery (not after each injection)
- **F-463** — Restock request — controlled by admin or logistics manager
- **F-464** — Cold-chain log (storage-bag temperature)
- **F-465** — Shortage/damage report — at both pickup and delivery

### 7.7 Nurse Financial Panel
- **F-470** — Today's income (completed visits × rate per visit)
- **F-471** — This week + this month income
- **F-472** — Income chart for the last 3 months
- **F-473** — Income breakdown (regular visit, emergency, full round)
- **F-474** — Bonuses (for quality, count, satisfaction)
- **F-475** — Payment history to account
- **F-476** — Downloadable monthly invoice
- **F-477** — Set bank info (IBAN / Sheba number)

### 7.8 Performance & Training
- **F-485** — Patient satisfaction score (NPS, 5-star)
- **F-486** — Successful visit count for the month

### 7.9 Emergency & Support
- **F-495** — Emergency button — hotline to Customer Service (callcenter)
- **F-497** — "My safety" button — for nurse and also patient

---

## 8. Pharmacy Hub Panel
- **F-501** — Login with OTP
- **F-502** — Medication inventory dashboard (count, expiry date, lot number)
- **F-503** — Record medication receipt from supplier
- **F-504** — Deliver medication to nurse (with barcode + signature)
- **F-505** — Cold-chain monitoring (fridge temperature — hourly log)
- **F-506** — Report damage or spoilage
- **F-507** — Weekly inventory report to admin
- **F-508** — Calculate monthly commission (5–8% of referred-patient ARPU)
- **F-512** — Monthly commission financial report
- **F-513** — Invoice for the pharmacy
- **F-515** — Audit log of all medication transactions

---

## 9. Admin Panel

### 9.1 User Management
- **F-601** — List all patients (filter, search, sort)
- **F-602** — Show full patient profile + history
- **F-603** — Change patient status (active, paused, banned)
- **F-604** — Manual support (override, discount, credit)
- **F-605** — List all doctors + status
- **F-606** — Onboarding new doctor and nurse — both need admin approval
- **F-607** — Activate/Deactivate doctor
- **F-608** — List all nurses + status + performance
- **F-609** — Onboarding new nurse
- **F-610** — Assign patients to doctor/nurse (manual or auto)

### 9.2 Operations & Logistics
- **F-615** — Daily operations dashboard (today's visits, en route, completed)
- **F-616** — Live map of nurses in the field (Optime AI)
- **F-617** — Network-wide cold-chain monitoring
- **F-618** — Today's incident report (with priority)
- **F-620** — Side-effect report monitor (ops oversight of incoming reports; **assigns a reviewing doctor** + routes urgent reports — pilot safety, M4)
- **F-619** — Manual reschedule of visit (admin intervention)

### 9.3 Finance & Subscription
- **F-625** — Daily financial dashboard (income, cost, EBITDA)
- **F-626** — MRR, ARR, cohort-retention report
- **F-627** — Monthly settlement with doctors (batch process)
- **F-628** — Monthly settlement with nurses
- **F-629** — Monthly settlement with pharmacies (commission)
- **F-631** — Audit trail of all financial transactions
- **F-632** — Issue official invoice for patient
- **F-633** — Manage refunds and chargebacks

### 9.4 Marketing & Growth
- **F-640** — Growth dashboard (CAC, LTV, conversion, retention)
- **F-641** — Acquisition-channel tracking (UTM, referral source)
- **F-644** — SMS campaign management (Kavenegar API)

### 9.5 System Settings
- **F-651** — Set medical protocols (prescription templates)
- **F-652** — Set doctor/nurse payment rates
- **F-653** — Manage Pharmacy Hubs (add, remove)
- **F-655** — Manage roles & permissions
- **F-656** — Set API keys for integrations
- **F-657** — Manage SMS and notification templates

### 9.6 Reports & BI
- **F-660** — Ready-made reports (daily, weekly, monthly to email)
- **F-662** — Export to Excel/CSV
- **F-663** — Full audit log (all actions by all users)

---

## 10. Optime AI Integration — Smart Logistics
*(Note: these IDs F-701–F-708 are reused from the original section numbering and are distinct from the Customer Service IDs below.)*
- **F-701** — Geocode patient address at signup
- **F-702** — Store coordinates (lat/lng) in patient profile
- **F-703** — Generate daily route for each nurse
- **F-704** — Optimize route by traffic, distance, visit time
- **F-705** — Send route to nurse app (push notification)
- **F-706** — On-demand assignment for emergency visit
- **F-707** — Real-time tracking of nurse in the field
- **F-708** — Show ETA to patient in the patient app

---

## 11. Other Integrations (P0)
- **Payment gateway** — ZarinPal or IDPay
- **SMS gateway** — Kavenegar or Melipayamak
- **Push notification** — Pushe (domestic) + FCM (Android)
- **Video conferencing** — Jitsi (self-hosted) — for online doctor visits
- **E-prescription** — TUMS / TaminICT
- **Cold-chain datalogger** — BLE protocol (Elitech)
- **Email service** — Mailtrap (domestic) or Postmark
- **Error tracking** — Sentry (self-hosted)
- **Crash reporting (mobile)** — Firebase Crashlytics
- **Object storage** — ArvanCloud Object Storage
- **Iranian calendar** — moment-jalaali (Jalali calendar)

---

## 14.1 Security Requirements (P0)
- **S-001** — HTTPS enforced on all communications (TLS 1.3+)
- **S-002** — Encryption at rest for PHI (AES-256)
- **S-003** — Encryption in transit (TLS) for all API calls
- **S-004** — Password hashing with bcrypt (cost factor 12+)
- **S-005** — JWT with short expiry (15 min) + refresh token (7 days)
- **S-006** — ~~Mandatory 2FA for doctor, nurse, admin~~ — **under review:** 2FA removed per product decision; staff accounts access PHI, so re-enabling 2FA for doctor/nurse/admin is recommended (security note).
- **S-007** — Rate limiting (per user, per IP, per endpoint)
- **S-008** — Input validation (XSS, SQL-injection prevention)
- **S-009** — CSRF tokens for web forms
- **S-010** — Security headers (CSP, HSTS, X-Frame-Options)
- **S-011** — Audit log of all sensitive actions
- **S-012** — GDPR-style consent management
- **S-016** — Automatic vulnerability scanning in CI/CD
- **S-017** — Dependency scanning (Snyk or similar)
- **S-018** — Secrets management (Vault, not in .env)

---

# v2.1 Additions

## Customer Service Panel (F-700–F-771, all P0)
- **F-700** — Customer-service hotline (callcenter) — 24/7
- **F-701** — Queue + call distribution to the right operator
- **F-702** — Call logging + call notes (CRM ticket)
- **F-703** — Routing to specialized sub-panel by issue type
- **F-710** — Logistics sub-panel — delivery management, courier issues, ETA
- **F-711** — Delivery-status tracking + arrival confirmation
- **F-720** — Marketing sub-panel — campaigns, discounts, onboarding
- **F-721** — New-lead management + follow-up
- **F-730** — Affiliate sub-panel — affiliate-marketer management, commission
- **F-731** — Referral tracking + commission payment
- **F-740** — Medical sub-panel — medical issues, side effects, doctor referral
- **F-741** — Contact on-call doctor + issue escalation
- **F-750** — New Patient sub-panel — onboarding new patients
- **F-751** — Initial screening + routing to the right path
- **F-760** — Payment-issue sub-panel
- **F-761** — Resolve payment problems + refunds + complaints
- **F-770** — Message patient via CS (only via app + CS system)
- **F-771** — CS dashboard — KPIs, resolution time, satisfaction

## Affiliate Panel (F-800–F-809, P0)
- **F-800** — Affiliate-marketer login with OTP
- **F-801** — Campaign dashboard — leads, conversions, income
- **F-802** — Generate unique referral link (with UTM)
- **F-803** — Conversion tracking (sign-up → first visit → first injection → retention)
- **F-804** — Calculate monthly commission (% of referred-patient ARPU)
- **F-805** — Monthly commission financial report + payment
- **F-807** — New-affiliate onboarding — admin approval
- **F-808** — Manage discount codes for affiliate campaigns
- **F-809** — Audit log of all referrals and transactions

## Patient App Additions (F-220–F-224, all P0)
- **F-220** — See assigned nurse profile (name, experience, photo)
- **F-221** — Online nurse identity verification — before home service
- **F-222** — Confirm medication receipt from pharmacy (patient confirmation)
- **F-223** — Foodnoise tracking — record food cravings (signal of GLP-1 effect)
- **F-224** — Weekly + monthly foodnoise chart for progress monitoring

## Compliance Additions (P0)
- **C-018** — AFTA license (Information Technology Org.) — obtain before launch
- **C-019** — Penetration testing — AFTA requirement — annual
- **C-020** — Security reporting to AFTA — cooperation on security incidents — ongoing

## QA Additions
- **Q-010** — Functional test — all panels — P0
- **Q-011** — A/B test for UI — main Patient App — P0
- **Q-012** — A/B test for onboarding flow — patient sign-up — P0

---

*Note: Features marked as removed in v2.1 (F-106, F-160, F-173, F-180, F-206, F-320, F-325, F-326, F-332, F-419, F-435, F-436, F-438, F-439, F-514) are excluded from this list.*