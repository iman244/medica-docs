# Medica — Lifecycle State Machines

A standalone companion to the build specification and data model. Every entity that carries a `status` field has a defined set of states and a defined set of legal transitions. This document resolves each `enum (states defined in lifecycle spec)` placeholder in the data model, and gives each state its UI treatment so it can be designed directly.

---

## How to read this document

For each entity there are two tables.

**States** — `state · visibility · badge (tone) · actions available`
- **visibility** — `user` (shown to the relevant end user as a status badge) or `internal` (system/ops only; the end user sees a simplified label or nothing). Design badges only for `user` states.
- **badge / tone** — the label shown and its tone, mapping to design tokens: `neutral · info · success · warning · danger`. The label text is an **i18n message key** resolved per the user's locale (`Medica_Internationalization.md`); the tone is locale-independent.
- **actions available** — which actions/buttons are enabled in that state (the rest are hidden or disabled).

**Transitions** — `from → to · trigger · source · UI effect`
- **trigger type** — `action` (a STEP), `event` (a domain event from another service), `timer` (timeout/cron), or `admin` (manual override).
- **source** — the STEP or event that causes it.

**Markers:** `⊗` terminal state · `⚠` an unhappy-path transition (the state/transition is owned here; the screen + recovery flow is designed in the unhappy-paths spec) · `[clinical]` the rule governing this transition is defined in the clinical-safety spec, not here.

---

## subscription · `sub` service

States
| state | visibility | badge (tone) | actions available |
|---|---|---|---|
| pending_payment | internal | — (user sees "processing") | — |
| trial | user | Trial (info) | upgrade, cancel |
| active | user | Active (success) | manage renewal, cancel |
| past_due | user | Payment due (warning) | Pay now (primary), cancel |
| cancelled ⊗ | user | Cancelled (neutral) | resubscribe |
| expired ⊗ | user | Expired (neutral) | resubscribe |

Transitions
| from → to | trigger | source | UI effect |
|---|---|---|---|
| ∅ → pending_payment | action | STEP-3A-02 | checkout screen |
| pending_payment → active | event | payment `paid` | success screen, app unlocked |
| pending_payment → cancelled ⚠ | event/timer | payment `failed`/session expiry | failure + retry |
| trial → active | event | first paid renewal | badge flips to Active |
| active → past_due ⚠ | event/timer | renewal payment fails | amber banner + Pay-now CTA |
| past_due → active | action/event | STEP-3A-05 / payment `paid` | banner clears |
| past_due → expired ⚠ | timer | grace period elapsed | access limited |
| trial/active/past_due → cancelled | action | STEP-3A-04 | exit interview |
| cancelled/expired → active | action | STEP-3A-02 | re-onboard to active |

Timeout: `past_due` has a grace window (config, `gw.setting`) before `expired`.

---

## payment · `pay` service

States
| state | visibility | badge (tone) | actions available |
|---|---|---|---|
| pending | internal | — (user sees "processing") | — |
| paid | internal | — (user sees success) | — |
| failed ⊗ | internal | — (user sees failure) | retry |
| refunded ⊗ | internal | — (user sees "refunded") | — |

Transitions
| from → to | trigger | source | UI effect |
|---|---|---|---|
| ∅ → pending | action | STEP-3A-02 | redirect to gateway |
| pending → paid | event | gateway callback success | success screen |
| pending → failed ⚠ | event/timer | callback fail / session expiry | failure + retry screen |
| paid → refunded | admin | STEP-4C-06 | wallet shows refund |

Timeout: gateway session expiry → `failed`.

---

## visit (online) · `visit` service

States
| state | visibility | badge (tone) | actions available |
|---|---|---|---|
| scheduled | user | Scheduled (info) | join, reschedule, cancel |
| in_progress | internal | — (user sees "in call") | — |
| completed ⊗ | user | Completed (success) | view prescription |
| cancelled ⊗ | user | Cancelled (neutral) | rebook |
| no_show ⊗ | internal | — (user sees "missed") | rebook |

Transitions
| from → to | trigger | source | UI effect |
|---|---|---|---|
| ∅ → scheduled | action | STEP-3B-01 | appears in schedule |
| scheduled → in_progress | action | STEP-3B-06 | doctor console opens |
| in_progress → completed | action | STEP-3B-07/09 | prescription available |
| scheduled → cancelled | action | patient/admin | rebook offered |
| scheduled → no_show ⚠ | timer/action | absent at visit time | missed-visit prompt |

---

## prescription · `visit` service

The prescription is authored in **external** e-prescription software; Medica links it by `external_rx_id` and mirrors a fetched status (no in-app issuance/submission).

States
| state | visibility | badge (tone) | actions available |
|---|---|---|---|
| linked | internal | Linking (info) | — |
| active | user | Active (success) | download PDF |
| rejected ⊗ | internal | Rejected (danger) | doctor re-issues externally + re-links |
| expired ⊗ | user | Expired (neutral) | — |

Transitions
| from → to | trigger | source | UI effect |
|---|---|---|---|
| ∅ → linked | action | STEP-3B-07 (`external_rx_id` attached) | fetching |
| linked → active | event | `[int·RxProvider]` fetch = valid | PDF downloadable |
| linked → rejected ⚠ | event | `[int·RxProvider]` fetch = rejected/invalid | doctor re-issues externally (RFLOW-19) |
| active → expired | timer | validity elapsed | — |

Dose/titration content originates in the external Rx software; clinical rules remain governed by `[clinical]`.

---

## side_effect_report · `patient` service

States
| state | visibility | badge (tone) | actions available |
|---|---|---|---|
| reported | user | Reported (info) | view |
| acknowledged | internal | — (patient sees "received") | ops/clinician: open |
| in_review | internal | — | clinician: disposition |
| resolved ⊗ | user | Resolved (success) | view outcome |

Transitions
| from → to | trigger | source | UI effect |
|---|---|---|---|
| ∅ → reported | action | STEP-4B-08 | confirmation to patient |
| reported → acknowledged ⚠ | event | `urgent=true` must-alert routed `[clinical]` | doctor + ops alerted (RFLOW-05) |
| reported → acknowledged | action | ops/clinician pickup (non-urgent) | enters queue |
| acknowledged → in_review | action | clinician opens | — |
| in_review → resolved | action | clinician disposition (`outcome`, `reviewed_by`) | outcome shown to patient |
| reported → resolved | action | simple non-urgent close | — |

During `acknowledged`, ops assigns a reviewing doctor (`assigned_doctor_id`, `STEP-4C-09`); that doctor then moves the report `in_review → resolved`. The rule that sets `urgent` and the disposition wording are `[clinical]` (clinical-safety spec §9).

---

## nurse_visit · `field` service

States
| state | visibility | badge (tone) | actions available |
|---|---|---|---|
| assigned | user | patient: Scheduled (info) · nurse: Assigned (info) | patient: view schedule · nurse: navigate |
| en_route | user | patient: Nurse on the way (info) · nurse: En route (info) | patient: track · nurse: arrive |
| checked_in | internal | patient sees "Nurse arrived" | nurse: begin assessment |
| in_progress | internal | — | nurse: record injection |
| blocked | internal (nurse) | Cannot administer (danger) `[clinical]` | nurse: abort + report |
| completed ⊗ | user | Completed (success) | patient: confirm receipt, view history |
| aborted ⊗ | user | Could not complete (danger) | patient/ops: reschedule |
| rescheduled | user | Rescheduled (neutral) | (spawns a new `assigned` visit) |

Transitions
| from → to | trigger | source | UI effect |
|---|---|---|---|
| ∅ → assigned | event/action | STEP-5-08/09 | appears on route + patient schedule |
| assigned → en_route | action | STEP-4A-02/03 | patient sees "on the way", tracking opens |
| en_route → checked_in | action | STEP-4A-04 (GPS) | nurse visit screen unlocks |
| checked_in → in_progress | action | STEP-4A-07 | injection step enabled |
| checked_in/in_progress → blocked ⚠ | action | STEP-4A-06 cold-chain breach `[clinical]` | "do not administer" block |
| in_progress → completed | action | STEP-4A-10 | patient confirm-receipt prompt |
| assigned/en_route → rescheduled ⚠ | action | STEP-4B-04 / 5-13 | new visit scheduled |
| any active → aborted ⚠ | action | STEP-4A-11 (patient unavailable / safety) | reschedule offered |

---

## vial · `pharm` service

States
| state | visibility | badge (tone) | actions available |
|---|---|---|---|
| in_hub | internal | In stock (neutral) | dispense |
| dispensed | internal | With nurse (info) | use, return |
| used ⊗ | internal | Used (neutral) | — |
| returned | internal | Returned (neutral) | dispense again |
| damaged ⊗ | internal | Damaged (danger) | — |
| expired ⊗ | internal | Expired (danger) | — |

Transitions
| from → to | trigger | source | UI effect |
|---|---|---|---|
| ∅ → in_hub | action | STEP-5-02 | counted in inventory |
| in_hub → dispensed | action | STEP-5-04 | moves to nurse inventory |
| dispensed → used | event | STEP-4A-08 injection | leaves nurse inventory |
| dispensed → returned | action | STEP-5-12 | back in hub |
| any → damaged ⚠ | action | STEP-5-05 / nurse report | removed from usable stock |
| in_hub/dispensed → expired ⚠ | timer | expiry_date passed | removed from usable stock |

`batch.recall_status`: `none → flagged → recalled` (admin/supplier event). A `recalled` batch flags all its vials as unusable `[clinical/ops]` ⚠.

---

## provider_profile · `prov` service

States
| state | visibility | badge (tone) | actions available |
|---|---|---|---|
| pending | internal | Pending (neutral) | admin: review |
| verified | internal | Verified (info) | admin: activate |
| active | user (provider) | Active (success) | full work access |
| inactive ⊗ | user (provider) | Inactive (warning) | login only, no work |

Transitions
| from → to | trigger | source | UI effect |
|---|---|---|---|
| ∅ → pending | action | STEP-4C-07 | awaits verification |
| pending → verified | action | STEP-4C-08 | ready to activate |
| verified → active | admin | STEP-4C-07 / 8-01 | provider can work |
| active → inactive ⚠ | admin | STEP-8-01 | work screens locked |
| inactive → active | admin | STEP-8-01 | restored |

---

## ticket · `crm` service

States
| state | visibility | badge (tone) | actions available |
|---|---|---|---|
| open | user (simplified) / agent | Open (info) | agent: pick up |
| in_progress | agent | In progress (info) | agent: resolve, message |
| resolved | user / agent | Resolved (success) | reopen |
| closed ⊗ | user / agent | Closed (neutral) | — |

Transitions
| from → to | trigger | source | UI effect |
|---|---|---|---|
| ∅ → open | action | STEP-9-01 / 7-07 | enters queue |
| open → in_progress | action | STEP-9-01 | assigned to agent |
| in_progress → resolved | action | agent resolves | resolution shown |
| resolved → closed | timer/action | auto-close / patient confirm | archived |
| resolved → in_progress | action | reopened | back in queue |

---

## assignment · `prov` service

A two-state machine that powers the `:assigned` permission scope — only `active` rows grant access.

| state | visibility | meaning |
|---|---|---|
| active | internal | provider ↔ patient link in force |
| inactive ⊗ | internal | unassigned |

| from → to | trigger | source |
|---|---|---|
| ∅ → active | action | STEP-4C-10 |
| active → inactive | admin | STEP-4C-10 (unassign) |

---

## referral funnel · `crm` service

A monotonic funnel (stages only advance; no regression).

| stage | visibility | meaning |
|---|---|---|
| signup | internal (affiliate sees counts) | referred user registered |
| first_visit | internal | completed first visit |
| first_injection | internal | received first injection |
| retained ⊗ | internal | retained past threshold |

| from → to | trigger | source |
|---|---|---|
| signup → first_visit | event | visit `completed` |
| first_visit → first_injection | event | nurse_visit `completed` |
| first_injection → retained | event/timer | retention threshold met |

---

## Utility state machines (compact)

| entity | states | transitions / notes |
|---|---|---|
| identity · auth | pending → active → suspended ⇄ active → banned ⊗ | pending→active on OTP verify (STEP-1-02); suspend/ban by admin ⚠. `user`: active (no badge), suspended/banned show a block screen. |
| kyc_status · auth | unverified → pending → verified / failed ⚠ | set during STEP-1-07 `[int·Shahkar]`; `user` during onboarding. |
| otp_challenge · auth | issued → consumed / expired | internal; expiry is a timeout. |
| consent · patient | signed → withdrawn ⚠ | withdrawal is a user action (S-012); withdrawal handling deferred to unhappy-paths/compliance. |
| restock_request · field | requested → approved → fulfilled | nurse sees status badge (info → success). |
| notification · notif | queued → sent / failed | internal. |
| settlement_run · pay | pending → completed / failed ⚠ | finance/internal. |
| campaign · notif | draft → scheduled → sent | ops/internal. |
| replacement_order · field | requested → dispatched → fulfilled ⊗ / cancelled ⊗ | requested at STEP-R01-04; dispatched on replacement dispense (STEP-R01-05); fulfilled when the replacement visit records an injection. internal (ops sees status in ReplacementQueue). Recovery flow in `Medica_Recovery_Flows.md` (RFLOW-01). |
| recall · pharm | initiated → cascaded → closed ⊗ | initiated + cascaded at STEP-R02-01 (flags vials in hub **and** nurse inventory); closed when all affected scheduled doses are replaced and all administered doses have a logged follow-up. internal. Recovery flow in `Medica_Recovery_Flows.md` (RFLOW-02) `[clinical/ops]`. |

---

## Notes for the next specs

- Every `⚠` transition is an **unhappy path**; its state and trigger are fixed here, but the screen and copy are designed in the unhappy-paths spec (`Medica_Unhappy_Paths.md`) and the actor-by-actor recovery is designed in the recovery-flows spec (`Medica_Recovery_Flows.md`), so the documents don't overlap.
- Every `[clinical]` transition (cold-chain `blocked`, batch `recalled`, prescription titration) keeps its *state* here but defers its *rule* to the clinical-safety spec.
- The public landing eligibility check (`STEP-2-10`) is **stateless** — it produces a non-binding indicator with no persisted row, so it has no state machine here; only the authenticated `eligibility_assessment` does.
- `user`-visibility states are the complete list of status badges the design system needs to define; `internal` states need no badge (at most a simplified label noted in the table).
