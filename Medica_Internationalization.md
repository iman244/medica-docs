# Medica — Internationalization (i18n) & Localization

A standalone companion to the build specification, data model, lifecycle, unhappy-paths, recovery-flows, PHI-access, and configuration-registry specs. It owns everything about language and locale across Medica: which languages are supported, how a locale is resolved and propagated, what is translated and what is not, how strings and content are stored and rendered, and how i18n threads through every microservice, microfrontend, and the timeline.

Every other doc references this one and carries only its `locale` / `preferred_locale` touchpoints; the rules live here so the documents stay in sync.

---

## 1. Scope & principles

- **Locale = BCP-47** (`language[-REGION]`). `fa-IR` (Persian, RTL) is the **default and source locale**; `en` (LTR) is the first additional locale. The supported set is **config** (`i18n.supported_locales`) and extensible (e.g. `ar`, `az`) by adding a catalog — no code change.
- **No user-visible literals.** Every label, button, badge, error, email/SMS/push body, and content item is a **message key** resolved at render time, never a hard-coded string. The lifecycle badge labels and the unhappy-paths `copy:` lines are *intents* that map to keys.
- **Locale-independent vs locale-dependent.** State machines, permissions, IDs, enum *values*, money (stored minor units), and UTC timestamps are locale-independent. Their **presentation** — text, calendar system, numerals, number/currency format, text direction — is locale-dependent.
- **Direction.** RTL for `fa`, LTR for `en`. Layout mirrors by document direction in `pkg-i18n`/`pkg-design`, not per screen; components use logical (start/end) CSS, never hard left/right.
- **Dates & numerals.** Jalali calendar + Persian numerals for `fa-IR`; Gregorian + Latin numerals for `en`. Storage stays UTC/ISO; conversion happens at presentation only (consistent with the data model).
- **Fallback.** A missing translation falls back to `i18n.fallback_locale` (default `fa-IR`); a raw key is never shown to a user.

---

## 2. What is and isn't translated

- **Static UI (translated):** all interface chrome — owned by frontend message catalogs in `pkg-i18n`: one catalog per MFE remote plus a shared catalog, lazy-loaded per locale. This includes the **public marketing surface** (`mfe-marketing`: landing, BMI calculator, public eligibility check), which is fully localized and — being the only public, SEO-relevant surface — is the candidate for an explicit `/[locale]` path prefix (e.g. `/fa`, `/en`) for shareable, indexable URLs, unlike the cookie-resolved authenticated app.
- **Dynamic content (translated, stored per-locale in the owning service):** `patient.content` (`locale` already present), `notif.template` (`locale` already present), the eligibility questionnaire questions (`STEP-2-06`), `consent` text (versioned per locale), package `included_services` display labels, and `protocol_template` patient-facing strings.
- **Clinical wording (translated *and* clinician-approved per locale):** every `[clinical]` patient-facing string — ineligible reasons, side-effect/emergency guidance, recall communications — is owned by the clinical-safety spec, translated per locale, and signed off per locale. Never machine-translated.
- **Never translated:** user- or clinician-entered free-text PHI (medical history notes, SOAP, side-effect `free_text`, nurse notes, chat). It is stored and displayed verbatim in the language it was entered; machine translation of clinical free-text is **prohibited** for safety. Also never translated: identifiers, codes, barcodes, IBANs.

---

## 3. Backend model (per service)

- **`auth`** owns `identity.preferred_locale` (BCP-47, default = `i18n.default_locale`): set at onboarding, editable in settings, returned by `API-AUTH-008 GET /auth/me`, and carried as a non-authoritative `locale` hint in the JWT for convenience.
- **Locale resolution order** at the gateway (`API-GW-001`): explicit request override → `identity.preferred_locale` → `Accept-Language` header → `i18n.default_locale`. The resolved locale is forwarded to any service that renders content or notifications.
- **`notif`** renders each message in the **recipient's** `preferred_locale` by selecting the matching `template.locale` row (falling back per §1). This is what every reminder, dunning, recovery, and recall notification uses.
- **`patient`** returns the `content` and questionnaire rows for the resolved locale, and stores free-text as entered.
- **Localized labels/enums are not stored as text.** Services return the locale-neutral enum value (e.g. `status = past_due`); the frontend resolves the visible label and tone from its catalog + the lifecycle spec. Services stay locale-agnostic.
- No cross-service translation calls: a service either holds per-locale rows or returns locale-neutral values.

---

## 4. Frontend model (microfrontends)

- **`pkg-i18n`** (shared) provides: the locale context/provider, catalog loading (per-remote + shared, lazy by locale), the `t(key)` lookup, RTL/LTR direction switching, and formatters (Jalali/Gregorian dates, Persian/Latin numerals, currency/number per locale).
- **`CMP-SHL-006 LocaleProvider`** wraps the shell and sets document direction; **`CMP-SHL-007 LocaleSwitcher`** lets a user change locale (writes `identity.preferred_locale` when authenticated; session-only before auth).
- **Each MFE remote** ships its own message catalog keyed by the `CMP-` ids it owns, versioned and deployed with the remote.
- **RTL/LTR** is a single document-direction switch in the shell; direction-agnostic components are a `pkg-design` constraint, so no screen is designed twice.

---

## 5. Timeline placement

- **Month 1 (foundation):** the `pkg-i18n` framework, `CMP-SHL-006/007`, gateway locale resolution, `identity.preferred_locale`, and the `fa-IR` source catalog + `en` scaffold are built into the shell and service template. Every later screen is authored against `t(key)` from day one — authoring keys as you build is cheap; retrofitting literals later is not, so the framework is non-negotiable in M1.
- **Each surface's month:** that surface's MFE ships its message catalog alongside its components — patient (M1–M4), doctor (M3/M6), nurse (M4/M5), pharmacy (M5), admin (M4/M8), CS + affiliate (M9). No new months; i18n authoring rides each surface's existing build.
- **`en` content parity** (translating the dynamic content, templates, questionnaire, consent) is a milestone that may trail the `fa-IR` launch. The pilot can launch **`fa-IR`-only with the framework in place**, then add `en` without rework.
- **`[clinical]` per-locale wording** is gated on the clinical-safety spec plus translators, the same as other clinical copy.

---

## 6. Configuration

Owned by the configuration registry under a new `i18n` category: `i18n.default_locale`, `i18n.supported_locales`, `i18n.fallback_locale`, `i18n.allow_user_locale_override`. The `setting.category` enum gains `i18n`.

---

## 7. Changes folded back into the other docs

- **Build spec** — `pkg-i18n` redefined as multilingual; `CMP-SHL-006 LocaleProvider` + `CMP-SHL-007 LocaleSwitcher` and gateway locale resolution added to the Month-1 cross-cutting set; `preferred_locale` added to `API-AUTH-008`; §7 notes the i18n read shapes; Month-1 foundation notes the framework.
- **Data model** — `identity.preferred_locale` added; an i18n convention bullet added next to the Jalali rule; existing `locale` columns (`content`, `template`) noted as the localized-content pattern; an i18n open question added.
- **Lifecycle** — note that badge labels are i18n keys (tones are locale-independent).
- **Unhappy paths** — note that every `copy:` intent resolves to a localized key and patterns mirror in RTL/LTR.
- **Recovery flows** — note that notifications/comms render in the recipient's `preferred_locale`.
- **PHI matrix** — `preferred_locale` classified as non-sensitive profile data; free-text PHI never machine-translated.
- **Configuration registry** — `i18n.*` keys added and mapped.
- **P0 feature list** — a cross-cutting platform block (F-090–F-094) for locale support.
