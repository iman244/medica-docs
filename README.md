# Medica — Documentation README

Start here. This is the index and reading guide for the Medica specification set: what each document owns, how they connect, the id conventions, and how to take the set into design (Figma / Claude Design) and build.

The documents are **standalone and cross-linked**: each is authoritative for one concern and references the others by filename. New entities/ids introduced anywhere are folded back into the data model and build spec, so the set stays in sync.

---

## 1. The document set

| # | document | owns (authority for) | status |
|---|---|---|---|
| 1 | `Medica_Build_and_Technical_Specification.md` | The spine. Every flow as steps: actor · role/permission · API · component · data. | complete |
| 2 | `Medica_Data_Model.md` | Persistent entities per microservice (DB-per-service, no cross-service FKs). | complete |
| 3 | `Medica_Lifecycle_State_Machines.md` | Entity states, legal transitions, and user-visible **badges + tones**. | complete |
| 4 | `Medica_Unhappy_Paths.md` | Error/edge screens, the **error-pattern taxonomy**, and resolver handoffs. | complete |
| 5 | `Medica_Recovery_Flows.md` | Actor-by-actor resolutions (`RFLOW-`) for the handoffs. | complete |
| 6 | `Medica_PHI_Access_Matrix.md` | Which accessor may read/write/mask **each sensitive field**. | complete |
| 7 | `Medica_Configuration_Registry.md` | Every tunable **config value** (key, default, guardrail, owner). | complete |
| 8 | `Medica_Internationalization.md` | Locale/i18n model: languages, resolution, RTL, what's translated. | complete |
| 9 | `Medica_Screen_Inventory.md` | Every **screen** (`SCR-`) per surface → component, feature, step, API. | complete |
| 10 | `Medica_Site_Map.md` | Next.js **routes** per screen + the health endpoint + routing model. | complete |
| 11 | `Medica_Design_System.md` | Design **tokens**, shadcn/ui mapping, badge/error catalogs, Figma org. | complete |
| — | `Medica_p0_features_en.md` | The source **feature list** (`F-`). Upstream reference. | source |
| 12 | `Medica_Clinical_Safety_Spec.md` | All `[clinical]` rules/thresholds/wording. **Clinician-owned.** | **template — awaiting clinician** |

### 1.1 Process diagrams (BPMN)

Eight BPMN diagrams visualize the cross-document flows as end-to-end swimlanes. They are **not** a separate source of truth — every task in them is labelled with the `STEP-`/`API-`/`RFLOW-`/`UP-` id it depicts, so each diagram is a visual index back into the specs below. Open them in any BPMN viewer (e.g. bpmn.io / Camunda Modeler).

| diagram | depicts (flow) | key ids | primary docs |
|---|---|---|---|
| `P1_onboarding_eligibility_subscription.bpmn` | Sign-up → eligibility → consent → subscribe & pay | `STEP-1A-*`, `STEP-2-*`, `STEP-3A-*`, `RFLOW-18`, `UP-PAT-05` | Build spec (M1–M3), Recovery Flows, Unhappy Paths |
| `P2_visit_external_prescription.bpmn` | Online visit → SOAP → external Rx link/fetch | `STEP-3B-*`, `API-VISIT-005/006`, `RFLOW-19`, `UP-DOC-02` | Build spec (M3 Flow B), Recovery Flows, Unhappy Paths |
| `P3_home_visit_fulfillment.bpmn` | Dose order → pick/pack/cold-chain → nurse → settlement | `STEP-5-*`, `STEP-4A/4B-*`, `STEP-6-*` | Build spec (M4–M6) |
| `P4_nurse_safety_sequence.bpmn` | GPS check-in → cold-chain → pre-injection → vial scan → administer | `STEP-4A-04..10`, `UP-NUR-03`, `RFLOW-01`, `RFLOW-03` | Build spec (M4 Flow A), Unhappy Paths, Recovery Flows |
| `P5_coldchain_replacement.bpmn` | Cold-chain breach → replacement order → reassign (re-enters P4) | `STEP-5-03`, `RFLOW-01` | Recovery Flows, Build spec (M5 Flow R) |
| `P6_batch_recall.bpmn` | Recall → flag batch → quarantine/replace → clinical follow-up | `RFLOW-02` `[clinical/ops]` | Recovery Flows, Build spec (M5 Flow R) |
| `P7_urgent_side_effect.bpmn` | Report side-effect → urgency triage → escalate/resolve | `STEP-4B-08`, `RFLOW-05`, `UP-PAT-14` | Build spec (M4 Flow B), Recovery Flows, Unhappy Paths |
| `P8_payment_subscription.bpmn` | Checkout → charge/renewal → activate, with retry | `STEP-3A-02`, `RFLOW-12` | Build spec (M3 Flow A), Recovery Flows |

---

## 2. Reading order, by what you're doing

- **Designing (Figma / Claude Design):** `Screen Inventory` → `Site Map` → `Design System` → `Unhappy Paths` (error/empty states) → `Lifecycle` (badges) → `Internationalization` (RTL). The Build spec is the backup reference for exact data per screen.
- **Building the frontend:** `Site Map` + `Screen Inventory` + `Design System`, then `Build spec` per step, with `i18n`, `Unhappy Paths`, and `PHI` for behavior.
- **Building the backend:** `Data Model` → `Build spec` (the `API-` + data lines) → `Lifecycle` → `Configuration Registry` → `PHI` → `Recovery Flows`.
- **Ops / clinical:** `Recovery Flows` → `Configuration Registry` → `Clinical-Safety Spec` (when written).

---

## 3. How the documents connect (traceability)

One chain ties everything together:

```
F- (feature)  →  STEP- (build step)  →  API- (backend) + CMP- (frontend)  →  SCR- (screen)  →  route (path)
                                   │
   status badges ── Lifecycle      │      errors ── Unhappy Paths ── resolver ── RFLOW-
   field access ── PHI Matrix      │      tunable values ── Configuration Registry
   visible strings ── i18n         │      visual tokens/components ── Design System
```

Use `STEP-` as the unit of work for backend tickets/tests, and `SCR-` as the unit of work for design/frontend. The **BPMN diagrams (§1.1)** render these chains as swimlanes — each diagram task carries the `STEP-`/`API-`/`RFLOW-`/`UP-` id it covers, so they trace back into the specs without duplicating them.

---

## 4. ID glossary

| prefix | meaning | defined in |
|---|---|---|
| `F-` | feature | P0 feature list |
| `STEP-` | build step (`STEP-<month><flow?>-<nn>`) | Build spec |
| `API-` | backend endpoint (`API-<service>-<nnn>`) | Build spec |
| `CMP-` | frontend component (`CMP-<mfe>-<nnn>`) | Build spec |
| `SCR-` | screen (`SCR-<surface>-<nn>`) | Screen Inventory |
| `UP-` | unhappy-path entry | Unhappy Paths |
| `RFLOW-` | recovery flow | Recovery Flows |

**Surfaces / MFE codes:** `MKT` marketing (public) · `PAT` patient · `DOC` doctor · `NUR` nurse · `PHM` pharmacy · `ADM` admin/ops · `CS` customer-service · `AFF` affiliate · `SHL` shared shell.

**Services:** `auth, patient, prov, visit, field, pay, pharm, sub, notif, crm, rpt, gw`.

**Permissions:** `resource:action:scope`, scope ∈ `self | assigned | any`.

**Status tags:** `[impl]` our code · `[manual→M_]` by hand until that month · `[reuse]` existing ids · `[int·party→M_]` third-party integration · `[deferred→M_]` · `[public]` pre-auth · `[clinical]` rule owned by the clinical-safety spec.

---

## 5. Conventions in force

- **Standalone + cross-linked**, no "revised/continuation" language; reference others by filename.
- **DB-per-service**, no cross-service FKs; reference by `identity_id` or resource UUID.
- **Config not code:** values that change over time/product/decision live in the Configuration Registry, not hard-coded.
- **`[clinical]` is deferred:** specs define the *mechanism*; the clinical-safety spec owns the *value/wording*.
- **PHI minimum-necessary:** field visibility follows the PHI matrix, by the accessor's *relationship to the record*, not their role.
- **i18n:** no user-visible literals; everything is a message key; `fa-IR` default + RTL, `en` next.

---

## 6. Status & what's still open

- **Complete:** the 11 specs + this README + the Design System.
- **Template awaiting clinician:** the **Clinical-Safety Spec** — owns every `[clinical]` rule, scaffolded with parameters, decision questions, and per-locale wording slots, but **no medical values** (those are the clinician's to set and sign off). Not a design blocker. Its *pilot-gating* values (§16 of that doc: cold-chain, eligibility, titration, vial-match, pre-injection, urgent side-effect, consent) must be signed off before the M4 pilot treats real patients; everything else can follow.
- **Open decisions are tracked, not lost:** Unhappy Paths §I (business/clinical policy), Configuration Registry §5 (the config keys that resolve them), Site Map §6 (deployment/routing), PHI Matrix §8 (access policy), Screen Inventory §8 + Design System (information architecture & visual choices).

---

## 7. Design-handoff checklist

Before handing to Figma / Claude Design, confirm you have:

- [x] Screen list with ids, data, and states — `Screen Inventory`
- [x] Routes per screen — `Site Map`
- [x] Badge inventory + tones — `Lifecycle` (consolidated in `Design System` §4)
- [x] Error/empty-state patterns — `Unhappy Paths` (consolidated in `Design System` §4)
- [x] Field masking rules — `PHI Matrix`
- [x] RTL / locale / date / numeral rules — `Internationalization` + `Design System` §5
- [x] Device targets per surface — `Screen Inventory` §2 / `Design System` §6
- [x] Design tokens structure + shadcn mapping + Figma organization — `Design System`
- [ ] Final palette, type scale, and IA groupings — **the designer's call** (Design System gives a starting direction + the open list)
- [ ] `[clinical]` copy (ineligible, side-effect, recall, public eligibility) — from the **Clinical-Safety Spec** when ready; use placeholders until then

The unchecked items are deliberately design/clinical decisions, not missing specs.

---

## 8. A note on the architecture (one paragraph)

Backend: ~10 FastAPI microservices, database-per-service, behind a gateway that validates JWT + permission; observability (Loki, VictoriaMetrics/Grafana, OpenTelemetry) and a health probe baked into the template. Frontend: Next.js microfrontends via Module Federation composed by a host shell — one remote per surface (`mfe-patient`, …, `mfe-marketing`), shared `pkg-design / pkg-auth / pkg-api / pkg-i18n`. Launch is a supervised pilot of <50 patients, then ~9 months to completion; many operations run by hand at pilot scale and are automated in the month noted on each step.
