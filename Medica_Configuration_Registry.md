# Medica — Configuration Registry

A standalone companion to the build specification, data model, lifecycle, unhappy-paths, recovery-flows, and PHI-access specs. It is the catalog of every **tunable value** in Medica: the things that change over time, by product, or by business/clinical decision — and therefore must live in configuration, never hard-coded in a spec or in code.

It exists because the other docs deliberately reference a *rule* (a grace window, a breach threshold, a retry count) without fixing its *value*. This document fixes where each value lives, its type, its default, the safe bounds it may move within, who may change it, and which rule it parameterizes.

### Two homes for configuration

1. **Central key/value store** — `gw.setting` (`key`, `value` jsonb, `category`), edited via `STEP-8-07 SettingsPanel` (`API-GW-023`). For cross-cutting scalar/array tunables. **This is what most of this document catalogs.**
2. **Structured domain-table config** — config that is naturally relational lives as rows in its owning service and is edited through that service's own panel (§4): subscription `package` prices, `protocol_template` Rx protocols, `notif.template` message bodies, `sub.discount` / `crm.discount_code`, `crm.affiliate.commission_rate`.

---

## 1. How to read this document

Each `gw.setting` key is listed as:

```
key (dot.notation) · category · type · default · guardrail · editor · parameterizes
```

- **default** — a proposed starting value. `clinician-set` means there is no safe default to propose; the value comes from the clinical-safety spec.
- **guardrail** — the min/max or allowed set the value may move within; a write outside the guardrail is rejected by `API-GW-023`. For `clinical` keys the guardrail itself is clinician-set.
- **editor** — who may change it: `admin` (`config:manage:any`), `finance`, `ops_manager`, or `clinician sign-off` (a `clinical` key — admin may type it, but the value + bounds are owned and approved by the clinical-safety spec).

### Markers

- `[clinical]` — value and safe bounds are owned by the clinical-safety spec (`#4`). **Pilot-gating** ones must have a clinician-approved value before the M4 pilot puts real patients on that path.
- `[decision]` — an open business policy from the unhappy-paths open-decisions list (§I) or a recovery flow; the default shown is illustrative, not assumed. §5 maps each to its decision number.

---

## 2. Central `setting` keys

### 2.A `auth` — identity, OTP, login, KYC

| key | type | default | guardrail | editor | parameterizes |
|---|---|---|---|---|---|
| `auth.otp.max_attempts` | int | 5 | 3–10 | admin | OTP lockout (UP-PAT-01) `[decision]` |
| `auth.otp.lockout_minutes` | int | 15 | 5–60 | admin | OTP lockout duration (UP-PAT-01) `[decision]` |
| `auth.otp.expiry_seconds` | int | 120 | 60–600 | admin | `otp_challenge.expires_at` |
| `auth.login.max_attempts` | int | 5 | 3–10 | admin | login lockout (UP-PAT-02) `[decision]` |
| `auth.login.lockout_minutes` | int | 15 | 5–60 | admin | login lockout duration (UP-PAT-02) `[decision]` |
| `auth.kyc.max_retries` | int | 3 | 1–5 | admin | KYC retries before manual review (UP-PAT-03, RFLOW-20) `[decision]` |
| `auth.kyc.allow_onboarding_while_pending` | bool | true | — | admin | onboarding while KYC pending (UP-PAT-03) `[decision]` |

### 2.B `billing` — payments, dunning, grace, fees, settlement, refunds

| key | type | default | guardrail | editor | parameterizes |
|---|---|---|---|---|---|
| `billing.payment.max_auto_retries` | int | 3 | 0–6 | finance | payment auto-retry (UP-PAT-07, RFLOW-12) `[decision]` |
| `billing.payment.retry_cadence_minutes` | array | `[5,30,120]` | each 1–1440 | finance | retry spacing (RFLOW-12) `[decision]` |
| `billing.payment.hold_subscription_on_failure` | bool | true | — | finance | hold vs release `pending_payment` (UP-PAT-07) `[decision]` |
| `billing.dunning.schedule_days` | array | `[1,3,7]` | each 0–30 | finance | dunning reminders (UP-PAT-08, RFLOW-13) `[decision]` |
| `billing.dunning.degraded_features` | array | `["booking"]` | subset of feature flags | ops_manager | what lapses while `past_due` (UP-PAT-08) `[decision]` |
| `billing.grace_window_days` | int | clinician-set | clinician-set | clinician sign-off | `past_due → expired` window (UP-PAT-09) `[clinical]` `[decision]` |
| `billing.expired.honor_inflight_visit` | bool | clinician-set | — | clinician sign-off | honor a scheduled visit after expiry (UP-PAT-09) `[clinical]` |
| `billing.missed_online_visit.fee_minor` | int | 0 | 0–package price | finance | missed online-visit fee (UP-PAT-10) `[decision]` |
| `billing.missed_home_visit.fee_minor` | int | 0 | 0–package price | finance | missed home-visit fee (UP-PAT-11, RFLOW-08) `[decision]` |
| `billing.missed_visit.fee_cap_per_month` | int | 0 | 0–package price | finance | fee cap (UP-PAT-11) `[decision]` |
| `billing.abort.billable_reasons` | array | `[]` | subset of abort-reason codes | ops_manager | which abort reasons bill (UP-NUR-04, RFLOW-10) `[decision]` |
| `billing.settlement.partial_commit` | bool | true | — | finance | partial vs all-or-nothing settlement (UP-ADM-01, RFLOW-17) `[decision]` |
| `billing.refund.allow_after_spend` | bool | false | — | finance | refund when wallet spent down (UP-ADM-02, RFLOW-16) `[decision]` |

### 2.C `clinical` — owned by the clinical-safety spec (all `[clinical]`)

These keys hold the value but the clinical-safety spec (`#4`) defines the value, the safe min/max bounds, and the patient-facing wording. **Cold-chain and eligibility are pilot-gating.**

| key | type | default | guardrail | editor | parameterizes |
|---|---|---|---|---|---|
| `clinical.coldchain.min_temp_c` | number | clinician-set | clinician-set | clinician sign-off | breach floor (UP-NUR-01/PHM-01, `coldchain_reading.breach`) — **pilot-gating** |
| `clinical.coldchain.max_temp_c` | number | clinician-set | clinician-set | clinician sign-off | breach ceiling — **pilot-gating** |
| `clinical.coldchain.max_excursion_minutes` | int | clinician-set | clinician-set | clinician sign-off | how long out-of-range before "do not administer" — **pilot-gating** |
| `clinical.coldchain.exposed_stock_action` | enum | clinician-set | {quarantine, dispose} | clinician sign-off | what happens to exposed stock (UP-PHM-01, RFLOW-04) |
| `clinical.eligibility.ruleset_version` | string | clinician-set | published versions | clinician sign-off | which eligibility ruleset the engine runs (`STEP-2-07`) — **pilot-gating** |
| `clinical.eligibility.show_reasons` | bool | clinician-set | — | clinician sign-off | whether/which reasons the ineligible screen shows (UP-PAT-05) |
| `clinical.eligibility.offer_paid_review` | bool | clinician-set | — | clinician sign-off | whether a paid override-review exists (UP-PAT-05, RFLOW-18) |
| `clinical.vial_match.rules_version` | string | clinician-set | published versions | clinician sign-off | drug/dose/recall match (UP-NUR-02, RFLOW-03) |
| `clinical.titration.protocol_template_id` | ref | clinician-set | a `protocol_template` row | clinician sign-off | titration schedule (`prescription.items.titration_step`) — defers to `protocol_template`, value never inlined here |
| `clinical.replacement.sla_hours` | int | clinician-set | clinician-set | clinician sign-off | replacement-dose deadline (RFLOW-01) |
| `clinical.recall.patient_comms_template_id` | ref | clinician-set | a `notif.template` row | clinician sign-off | recall wording to patients (UP-PHM-02, RFLOW-02) |
| `clinical.side_effect.emergency_guidance_template_id` | ref | clinician-set | a `notif.template` row | clinician sign-off | when-to-seek-care guidance (UP-PAT-14, RFLOW-05) |

### 2.D `logistics` — capacity, substitution, routing

| key | type | default | guardrail | editor | parameterizes |
|---|---|---|---|---|---|
| `logistics.capacity.lookahead_days` | int | 3 | 1–14 | ops_manager | how far ahead capacity gaps surface (UP-ADM-04, RFLOW-07) `[decision]` |
| `logistics.substitution.policy` | enum | `none` | {none, same_drug, ops_approval} | ops_manager | stock-out substitution (UP-PHM-04, RFLOW-09) `[decision]` |
| `logistics.stockout.predict_days` | int | 5 | 1–30 | ops_manager | stock-out prediction horizon (UP-PHM-04) `[decision]` |
| `logistics.gps.allow_manual_override` | bool | true | — | ops_manager | GPS check-in override + audit (UP-NUR-03) `[decision]` |
| `logistics.route.optime_enabled` | bool | true | — | ops_manager | use Optime vs manual list (UP-NUR-07) |

### 2.E `rates` · `retention` · `hubs`

| key | type | default | guardrail | editor | parameterizes |
|---|---|---|---|---|---|
| `rates.cashback_percent` | number | 0 | 0–50 | finance | wallet cashback (F-144, `STEP-7-09`) |
| `rates.first_month_discount_percent` | number | 20 | 0–100 | finance | first-month discount (F-132, `STEP-7-09`) |
| `crm.commission.clawback_window_days` | int | 30 | 0–180 | finance | commission clawback window (UP-AFF-02) `[decision]` |
| `crm.commission.clawback_on` | array | `["refund"]` | subset of {refund, early_churn} | finance | clawback triggers (UP-AFF-02) `[decision]` |
| `retention.referral.retained_threshold_days` | int | 90 | 30–365 | ops_manager | referral funnel `retained` threshold |
| `hubs.default_hub_id` | ref | — | a `pharmacy_hub` row | admin | default dispensing hub |

> Provider/pharmacy payout *rates* (per-visit, per-injection, commission %) are structured config and live in §4, not as flat keys.

### 2.F `i18n` — language & locale

| key | type | default | guardrail | editor | parameterizes |
|---|---|---|---|---|---|
| `i18n.default_locale` | string | `fa-IR` | a member of `supported_locales` | admin | default/source locale (`Medica_Internationalization.md`) |
| `i18n.supported_locales` | array | `["fa-IR","en"]` | valid BCP-47 tags | admin | which locales the app offers + loads catalogs for |
| `i18n.fallback_locale` | string | `fa-IR` | a member of `supported_locales` | admin | shown when a translation is missing |
| `i18n.allow_user_locale_override` | bool | true | — | admin | whether `CMP-SHL-007 LocaleSwitcher` may set `identity.preferred_locale` |

### 2.G `marketing` — public landing site

| key | type | default | guardrail | editor | parameterizes |
|---|---|---|---|---|---|
| `marketing.enabled_sections` | array | `["hero","how_it_works","bmi","eligibility_check"]` | subset of known section keys | ops_manager | which landing sections render (lets other sections ship later) (F-095) |
| `marketing.eligibility_preview.enabled` | bool | true | — | admin | toggle the public eligibility check (F-097) |
| `marketing.eligibility_preview.ruleset_version` | string | clinician-set | published versions | clinician sign-off | the public question subset + non-binding result framing (`STEP-2-10`) `[clinical]` |

---

## 3. Pilot-gating subset (must be set before M4)

Everything else can take its proposed default into the pilot and be tuned later. These cannot, because a wrong value can harm a patient and they go live in M4–M5:

- `clinical.coldchain.min_temp_c` / `max_temp_c` / `max_excursion_minutes`
- `clinical.eligibility.ruleset_version`

Each needs a clinician-approved value (a one-page signed settings sheet is enough — the full clinical-safety spec can follow). Until then, the pilot runs these paths under direct human supervision, consistent with the supervised-pilot model.

---

## 4. Structured domain-table config (not in `setting`)

Relational config edited through its owning service's panel, not the central settings panel.

| config | entity · service | edited at | notes |
|---|---|---|---|
| Subscription packages + prices | `package` · sub | `STEP-3A-01` catalog / admin | tier, `price_minor`, `included_services`, `discount_rules` |
| Rx / titration protocols | `protocol_template` · gw | `STEP-4C-03 ProtocolEditor` | the titration schedule itself; `clinical.titration.protocol_template_id` points here `[clinical]` |
| Message templates | `template` · notif | `STEP-8-07` templates | SMS/push/email bodies; recall + emergency-guidance templates are `[clinical]` |
| Promo / first-month discounts | `discount` · sub | `STEP-7-09` | `code`, `percent`, validity |
| Affiliate codes + rate | `affiliate.commission_rate`, `discount_code` · crm | `STEP-9-13` | per-affiliate commission and codes |
| Hub definitions | `pharmacy_hub` · pharm | `STEP-5-01` / admin | name, location, status |
| API integration keys | `api_key` · gw | `STEP-4C-05 ApiKeyManager` | secret; `key_hash` never returned |

---

## 5. Mapping to the unhappy-paths open decisions

Each `[decision]` key is the mechanism that resolves an open decision in `Medica_Unhappy_Paths.md` §I — the key exists now; the *value* is the policy call.

| UP open decision | resolved by key(s) |
|---|---|
| #1 OTP/login limits & lockout | `auth.otp.*`, `auth.login.*` |
| #2 KYC retries / onboarding while pending | `auth.kyc.max_retries`, `auth.kyc.allow_onboarding_while_pending` |
| #3 Ineligible reasons / paid review `[clinical]` | `clinical.eligibility.show_reasons`, `clinical.eligibility.offer_paid_review` |
| #4 Payment auto-retry; hold vs release | `billing.payment.max_auto_retries`, `billing.payment.retry_cadence_minutes`, `billing.payment.hold_subscription_on_failure` |
| #5 Dunning schedule + degraded features | `billing.dunning.schedule_days`, `billing.dunning.degraded_features` |
| #6 Grace length; honor in-flight visit `[clinical]` | `billing.grace_window_days`, `billing.expired.honor_inflight_visit` |
| #7 Missed-visit fees | `billing.missed_online_visit.fee_minor`, `billing.missed_home_visit.fee_minor`, `billing.missed_visit.fee_cap_per_month` |
| #8 GPS manual override + audit | `logistics.gps.allow_manual_override` |
| #9 Billable / escalating abort reasons | `billing.abort.billable_reasons` |
| #11 Recall patient communication `[clinical]` | `clinical.recall.patient_comms_template_id` |
| #12 Substitution / stock-out prediction; no-capacity | `logistics.substitution.policy`, `logistics.stockout.predict_days`, `logistics.capacity.lookahead_days` |
| #13 Partial settlement; refund-after-spend | `billing.settlement.partial_commit`, `billing.refund.allow_after_spend` |
| #14 Commission clawback | `crm.commission.clawback_window_days`, `crm.commission.clawback_on` |

(#10 in-flight care on unassignment and #15 offline-queue policy are flow/architecture decisions, not single config values — they stay in the recovery-flows and deferred offline work.)

---

## 6. Editing & enforcement rules

- **Guardrails are enforced server-side.** `API-GW-023` rejects any write outside a key's guardrail; the SettingsPanel shows the bound. This is what lets a manager tune a value safely without a code change.
- **`clinical` keys are write-guarded.** They may only be set to values within clinician-approved bounds; the bounds themselves change only through the clinical-safety spec, not the settings panel.
- **Every change is audited.** `gw.audit_log` records the key, old/new value, and editor (`S-011`).
- **Defaults are seeded at deploy.** Each key ships with the default in this registry; a fresh environment is fully operable except for the pilot-gating `clinical` keys (§3), which start unset and block their path until a clinician sets them.
- **Adding a key** means adding it here first (with type, default, guardrail, editor, owning rule), then to the seed set — the registry is the source of truth, the `setting` table is the store.

---

## 7. Changes folded back into the other docs

- **`Medica_Data_Model.md`** — `setting.category` enum expanded to `rates/hubs/templates/auth/billing/clinical/logistics/retention/i18n/marketing`, with a pointer to this registry and a note that `clinical` keys defer their value/bounds to the clinical-safety spec.
- **`Medica_Build_and_Technical_Specification.md`** — `STEP-8-07` notes that the editable key catalog is this registry and that `clinical`-category keys are write-guarded to clinician-set bounds.
- No new APIs, components, or entities — the registry is delivered entirely through the existing `gw.setting` store and `STEP-8-07 SettingsPanel`.
