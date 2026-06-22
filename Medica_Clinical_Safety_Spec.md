# Medica — Clinical-Safety Specification

> **Status: TEMPLATE — awaiting completion and sign-off by a licensed clinician.**
>
> **This document deliberately contains no medical values.** Every threshold, schedule, criterion, and patient-facing medical message below is a **blank to be filled by a qualified, licensed clinician** responsible for Medica's care model. Nothing here is medical advice, and no value should be inferred from the engineering specs. The product must **not** go live on any clinical path until that path's values are filled and signed off; the **pilot-gating** items (§13) must be signed off before the M4 pilot treats real patients.

This is the single authority for every rule marked `[clinical]` across the other specs. The engineering documents define the *mechanism* (when a rule fires, what the system does, which config key or template holds the value); this document defines the *clinical content* (the value, the safe bounds, the wording). Values feed the `clinical.*` keys in `Medica_Configuration_Registry.md`, the `protocol_template` rows (titration), and the per-locale `notif.template` rows (patient messaging).

---

## How to read / complete this document

Each section governs one clinical rule and is laid out as:

```
What it governs · references (UP- / STEP- / RFLOW-) · binds to (config key / template)
Parameters to set  — table: parameter · unit · value (clinician-set) · safe min/max (clinician-set)
Decision questions — what the clinician must decide
Patient-facing wording — per-locale slots (fa-IR, en), where applicable
Pilot-gating — yes/no
Sign-off — clinician · date · version
```

Conventions: `___` is a blank for the clinician. **Bounds matter:** for each numeric parameter the clinician sets both the **value** and the **safe min/max** that the admin SettingsPanel will enforce, so no operator can later set an unsafe value (`Medica_Configuration_Registry.md` §6). All patient-facing wording must be provided **and clinically approved per locale** (`fa-IR`, `en`) — never machine-translated (`Medica_Internationalization.md`).

---

## 1. Cold-chain breach — "do not administer" (field) · **pilot-gating**

- governs: the temperature/duration that marks a delivered dose unusable and hard-blocks injection.
- references: `STEP-4A-06`, `UP-NUR-01` (nurse_visit → blocked), `RFLOW-01`. binds to: `clinical.coldchain.*`.

Parameters to set:

| parameter | unit | value | safe min / max |
|---|---|---|---|
| `clinical.coldchain.min_temp_c` | °C | ___ | ___ / ___ |
| `clinical.coldchain.max_temp_c` | °C | ___ | ___ / ___ |
| `clinical.coldchain.max_excursion_minutes` | minutes | ___ | ___ / ___ |

Decision questions:
- What temperature range keeps the medication safe to administer, and over what duration does an excursion render it unusable?
- Is a single instantaneous reading sufficient, or is cumulative time-out-of-range the rule?
- Does the rule differ by product/presentation?

Pilot-gating: **yes.** · Sign-off: clinician ___ · date ___ · version ___

---

## 2. Hub cold-chain breach — exposed stock (pharmacy)

- governs: the storage breach rule for the hub fridge, and what happens to exposed stock.
- references: `STEP-5-03`, `UP-PHM-01`, `RFLOW-04`. binds to: `clinical.coldchain.exposed_stock_action` (+ hub thresholds, if different from §1).

Parameters to set:

| parameter | unit | value | safe min / max |
|---|---|---|---|
| hub `min/max_temp_c`, `max_excursion_minutes` | °C / min | ___ | ___ / ___ |
| `clinical.coldchain.exposed_stock_action` | enum {quarantine, dispose} | ___ | — |

Decision questions:
- Are hub thresholds the same as field delivery (§1) or stricter?
- On a confirmed hub breach, what defines "exposed stock," and is it quarantined for assessment or disposed?

Pilot-gating: recommended before stock is held at scale (M5). · Sign-off: clinician ___ · date ___ · version ___

---

## 3. GLP-1 eligibility ruleset (authenticated assessment) · **pilot-gating**

- governs: the criteria that classify a patient `eligible / borderline / ineligible`.
- references: `STEP-2-07`, `UP-PAT-05/06`, `RFLOW-18`. binds to: `clinical.eligibility.ruleset_version`, `clinical.eligibility.show_reasons`, `clinical.eligibility.offer_paid_review`.

To define (structure only — clinician fills):
- **Inclusion criteria:** ___
- **Exclusion criteria / contraindications:** ___
- **Borderline triggers** (route to doctor review rather than auto-decide): ___
- **Result mapping:** how the questionnaire answers + measurements map to `eligible / borderline / ineligible`: ___
- **Questionnaire content** (the 15–20 questions, F-121): ___ (also localized, §11)

Decision questions:
- Which conditions/medications are absolute vs. relative contraindications?
- What is reviewed by a human (borderline) vs. auto-decided?
- Are reasons shown to the patient on an ineligible result (`show_reasons`)? Is a paid doctor-override review offered, and at what framing/price (`offer_paid_review`)?

Pilot-gating: **yes.** · Sign-off: clinician ___ · date ___ · version ___

---

## 4. Public eligibility check — subset & framing (landing)

- governs: which questions appear in the **public, anonymous** pre-signup check, and its non-binding result framing.
- references: `STEP-2-10`, `UP-MKT-02`, `F-097`. binds to: `marketing.eligibility_preview.ruleset_version`.

To define:
- **Public question subset:** which of the §3 questions are safe/appropriate to ask anonymously: ___
- **Result framing:** the preview must **never** return a definitive verdict — define the non-binding indicator wording and the mandatory disclaimer: ___ (wording in §11)

Decision questions:
- What can be asked pre-signup without implying a diagnosis?
- How is a "may be a candidate" vs. "may not be a fit" message framed so it is encouraging and non-final, and clearly directs to a proper assessment?

Pilot-gating: required before the public check is enabled. · Sign-off: clinician ___ · date ___ · version ___

---

## 5. Ineligible & borderline result — patient copy

- governs: the wording of the most emotionally sensitive screen and the borderline message.
- references: `UP-PAT-05` (ineligible), `UP-PAT-06` (borderline), `SCR-PAT-11`. binds to: per-locale strings.

Patient-facing wording (provide per locale, §11):
- **Ineligible** — kind, clear, non-final, offers alternatives (no numeric score — eligibility is categorical): › fa-IR: ___ › en: ___
- **Borderline → review** — positive, "a doctor needs to review this first," not a rejection: › fa-IR: ___ › en: ___

Sign-off: clinician ___ · date ___ · version ___

---

## 6. Titration / dose schedule

- governs: the GLP-1 dose-escalation schedule carried in a prescription.
- references: `prescription.items.titration_step` (data model §14), `STEP-3B-07`. binds to: `protocol_template` (gw), referenced by `clinical.titration.protocol_template_id`.
- **Note (external Rx):** prescriptions are now authored in external e-prescription software; the titration schedule is therefore set there and **mirrored** read-only into Medica. This section's value is the clinically-approved schedule the external software should follow / Medica should validate against, not an in-app generator.

Schedule to define (blank — clinician fills; one row per step):

| step | dose | unit | duration before advancing | criteria to advance | criteria to hold / step-down |
|---|---|---|---|---|---|
| 1 | ___ | ___ | ___ | ___ | ___ |
| … | ___ | ___ | ___ | ___ | ___ |

Decision questions:
- Standard titration schedule and the rules for advancing, holding, or stepping down based on tolerance/response?
- Maximum dose; handling of missed doses; restart-after-gap rule?

Pilot-gating: required before prescriptions are issued (M3). · Sign-off: clinician ___ · date ___ · version ___

---

## 7. Vial-match rules (injection safety)

- governs: what counts as a valid match between the scanned vial and the patient's prescription at injection.
- references: `STEP-4A-08`, `UP-NUR-02`, `RFLOW-03`. binds to: `clinical.vial_match.rules_version`.

To define:
- Match criteria (drug, dose/strength, patient, expiry, recall status): ___
- Which mismatches **hard-block** vs. warn: ___

Pilot-gating: required before injections (M4). · Sign-off: clinician ___ · date ___ · version ___

---

## 8. Pre-injection assessment criteria

- governs: the vitals/checks a nurse records before administering, and the criteria to proceed vs. abort.
- references: `STEP-4A-07` (`assessment_record`).

To define:
- Required vitals/checks: ___
- Proceed criteria; abort/escalate criteria (e.g. out-of-range vitals, recent reactions): ___

Pilot-gating: required before injections (M4). · Sign-off: clinician ___ · date ___ · version ___

---

## 9. Urgent side-effect — classification & emergency guidance

- governs: what makes a reported side-effect "urgent," and the patient-facing when-to-seek-emergency-care guidance.
- references: `STEP-4B-08`, `UP-PAT-14`, `RFLOW-05`. binds to: urgent-classification rule + `clinical.side_effect.emergency_guidance_template_id`.

To define:
- **Urgent classification:** which symptoms/severities set `side_effect_report.urgent = true` and trigger must-alert: ___
- **Emergency guidance copy** (should cover: red-flag symptoms, when to call the emergency line vs. local emergency services, what to do while waiting — clinician writes the content): › fa-IR: ___ › en: ___

Pilot-gating: **yes** (safety-critical patient guidance). · Sign-off: clinician ___ · date ___ · version ___

---

## 10. Expired mid-treatment — clinical messaging & grace

- governs: the health implications of stopping GLP-1 mid-course, the grace window, and whether an in-flight visit is honored.
- references: `UP-PAT-09`, `RFLOW-14`. binds to: `billing.grace_window_days`, `billing.expired.honor_inflight_visit` (+ messaging).

| parameter | unit | value | safe min / max |
|---|---|---|---|
| `billing.grace_window_days` | days | ___ | ___ / ___ |
| `billing.expired.honor_inflight_visit` | bool | ___ | — |

To define:
- Clinical messaging when treatment lapses (encourage continuity / talk to a doctor, not just "pay to unlock"): › fa-IR: ___ › en: ___

Sign-off: clinician ___ · date ___ · version ___

---

## 11. Batch recall — clinical follow-up & patient communication

- governs: the follow-up for patients already injected from a recalled batch, and the recall message wording.
- references: `UP-PHM-02`, `RFLOW-02` (`STEP-R02-04`). binds to: `clinical.recall.patient_comms_template_id` + follow-up protocol.

To define:
- **Already-administered follow-up protocol:** what clinical action/outreach for patients who received a dose from a recalled batch: ___
- **Recall message copy** (scheduled vs. already-dosed patients): › fa-IR: ___ › en: ___

Pilot-gating: recommended before stock is held at scale (M5). · Sign-off: clinician ___ · date ___ · version ___

---

## 12. Replacement-dose SLA (clinical / ops)

- governs: how quickly a replacement dose must reach a patient after a cold-chain/mismatch/recall loss.
- references: `RFLOW-01`. binds to: `clinical.replacement.sla_hours`.

| parameter | unit | value | safe min / max |
|---|---|---|---|
| `clinical.replacement.sla_hours` | hours | ___ | ___ / ___ |

Decision question: is there a clinically meaningful deadline for a missed dose given the titration schedule (§6)? Sign-off: clinician ___ · date ___ · version ___

---

## 13. Informed consent content (clinical / legal)

- governs: the informed-consent text the patient signs and its versioning.
- references: `STEP-2-08` (`consent`).

To define (with legal counsel as needed):
- Consent body, version, and what it covers (treatment risks, data use as relevant): › fa-IR: ___ › en: ___

Sign-off: clinician/legal ___ · date ___ · version ___

---

## 14. Emergency escalation protocol (on-call)

- governs: which events auto-escalate and the on-call clinical coverage that receives them.
- references: `RFLOW-05`, `RFLOW-06`, `F-741`, `STEP-9-09`.

To define:
- Which reasons auto-escalate to a clinician vs. CS/ops: ___
- On-call coverage model and response expectations during the pilot: ___

Sign-off: clinician ___ · date ___ · version ___

---

## 15. Consolidated clinical register

Every clinical value the system reads, where it lives, and whether it gates the pilot:

| § | rule | binds to | pilot-gating | signed off? |
|---|---|---|---|---|
| 1 | Cold-chain breach (field) | `clinical.coldchain.min/max_temp_c`, `max_excursion_minutes` | **yes** | ☐ |
| 2 | Hub cold-chain / exposed stock | hub thresholds, `clinical.coldchain.exposed_stock_action` | M5 | ☐ |
| 3 | Eligibility ruleset | `clinical.eligibility.ruleset_version`, `show_reasons`, `offer_paid_review` | **yes** | ☐ |
| 4 | Public eligibility subset/framing | `marketing.eligibility_preview.ruleset_version` | at enable | ☐ |
| 5 | Ineligible/borderline copy | per-locale strings | with §3 | ☐ |
| 6 | Titration schedule | `protocol_template`, `clinical.titration.protocol_template_id` | M3 | ☐ |
| 7 | Vial-match rules | `clinical.vial_match.rules_version` | M4 | ☐ |
| 8 | Pre-injection assessment | assessment criteria | M4 | ☐ |
| 9 | Urgent side-effect + guidance | classification + `clinical.side_effect.emergency_guidance_template_id` | **yes** | ☐ |
| 10 | Expired mid-treatment | `billing.grace_window_days`, `billing.expired.honor_inflight_visit` | M3 | ☐ |
| 11 | Recall follow-up + comms | `clinical.recall.patient_comms_template_id` | M5 | ☐ |
| 12 | Replacement SLA | `clinical.replacement.sla_hours` | M5 | ☐ |
| 13 | Informed consent | `consent` content/version | M2 | ☐ |
| 14 | Emergency escalation | escalation + on-call protocol | M4 | ☐ |

---

## 16. Pilot-gating minimum set (before M4)

The smallest set that must be signed off before the supervised pilot treats real patients — a one-page approved settings sheet is sufficient; the rest of this document can follow:

1. **Cold-chain breach** (§1) — without it, no dose can be safely administered.
2. **Eligibility ruleset** (§3) — without it, no one can be assessed.
3. **Titration schedule** (§6) — without it, no prescription can be issued.
4. **Vial-match rules** (§7) and **pre-injection assessment** (§8) — injection safety.
5. **Urgent side-effect classification + emergency guidance** (§9) — post-injection safety.
6. **Informed consent** (§13) — required to enroll.

Until each is signed off, its path runs under direct human supervision or stays disabled.

---

## 17. Governance

- **Versioning.** Each rule carries a version; bumping a `*_version` config key or a `protocol_template` is an audited change (`gw.audit_log`, `S-011`).
- **Authority.** Values and their safe bounds are owned and approved here by the clinician; the admin SettingsPanel can only move a value **within** the bounds set in this document.
- **Change control.** A change to any signed-off value requires re-sign-off and a version bump; the change is logged with who/when.
- **Localization.** Every patient-facing string is provided and clinically approved per locale before that locale is enabled (`Medica_Internationalization.md`).

---

## 18. Coverage / fold-back

This document resolves every `[clinical]` marker in the set: cold-chain (`UP-NUR-01`, `UP-PHM-01`, `RFLOW-01/04`), recall (`UP-PHM-02`, `RFLOW-02`), vial match (`UP-NUR-02`, `RFLOW-03`), eligibility (`UP-PAT-05/06`, `STEP-2-07`, `RFLOW-18`), public eligibility (`UP-MKT-02`, `STEP-2-10`), titration (`prescription.items.titration_step`), urgent side-effect (`UP-PAT-14`, `RFLOW-05`), expired mid-treatment (`UP-PAT-09`, `RFLOW-14`), and consent (`STEP-2-08`). It introduces no new entities, APIs, or components — only the values behind existing `clinical.*` config keys, `protocol_template`, and `notif.template` rows.
