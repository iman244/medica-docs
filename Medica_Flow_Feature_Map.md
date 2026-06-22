# Medica — Flow ⇄ Feature Map

Complete traceability between `Medica_User_Flows.md` and the feature list (`F-` ids in `Medica_p0_features_en.md`). **Every feature in the list is accounted for**: Part 1 (flow id → feature), Part 2 (feature → flow ids), and Part 3 (features that intentionally have no flow step, with the reason and where they are handled). Types: **H** happy step · **U** unhappy branch · **R** shared recovery.

Coverage: **224 of 245** features are realized by a flow; the remaining **21** are listed in Part 3 (removed or cross-cutting).

Flow totals: 125 happy steps · 59 unhappy branches · 46 recoveries.

---

## Part 1 — Flow id → Feature id

| Flow id | Type | Title | Feature ids |
|---|---|---|---|
| `PF-A.H1` | H | Enter phone + password, request an OTP | F-101 |
| `PF-A.H2` | H | Verify the phone OTP | F-101 |
| `PF-A.H3` | H | Log in (phone + password, or phone + OTP) | F-102 |
| `PF-A.H4` | H | Recover password | F-103 |
| `PF-A.H5` | H | Verify identity / KYC | F-105 |
| `PF-A.U1` | U | OTP wrong or expired** (at H2) | F-101 |
| `PF-A.U2` | U | Locked out** (at H3) | F-102 |
| `PF-A.U3` | U | Account suspended / banned** (at H3) | F-102 |
| `PF-A.U4` | U | KYC fails or mismatch** (at H5) | F-105 |
| `PF-B.H1` | H | Complete basic profile | F-110 |
| `PF-B.H2` | H | Enter height/weight, see BMI | F-111 |
| `PF-B.H3` | H | Enter medical history | F-112 |
| `PF-B.H4` | H | Upload lab results | F-113 |
| `PF-B.H5` | H | Set delivery address | F-115 |
| `PF-B.H6` | H | Answer eligibility questionnaire | F-121 |
| `PF-B.H7` | H | Compute eligibility, show result | F-122, F-123, F-125, F-126 |
| `PF-B.H8` | H | Sign informed consent | F-124 |
| `PF-B.H9` | H | Receive confirmations | F-205 |
| `PF-B.H10` | H | Land on patient home, ready to subscribe | F-100 |
| `PF-B.U1` | U | Lab upload rejected** (at H4) | F-113 |
| `PF-B.U2` | U | Borderline eligibility** (at H7) | F-125 |
| `PF-B.U3` | U | Ineligible** (at H7) | F-123 |
| `PF-C.H1` | H | View packages | F-130 |
| `PF-C.H2` | H | Purchase & pay | F-131 |
| `PF-C.H3` | H | Manage / stop renewal | F-133 |
| `PF-C.H4` | H | Cancel + exit interview | F-139 |
| `PF-C.H5` | H | Wallet — balance, top-up, pay | F-141, F-142, F-143, F-146 |
| `PF-C.U1` | U | Payment failed / gateway unreachable** (at H2) | F-131 |
| `PF-C.U2` | U | Renewal failed | F-133 |
| `PF-C.U3` | U | Expired mid-treatment** (at H3) | F-133 |
| `PF-C.U4` | U | Refund / cashback pending or failed** (at H5) | F-147, F-144 |
| `PF-D.H1` | H | Book a visit | F-150, F-151 |
| `PF-D.H2` | H | Get a reminder | F-152 |
| `PF-D.H3` | H | Join the phone visit | F-153 |
| `PF-D.H4` | H | View & download the prescription | F-155, F-159 |
| `PF-D.H5` | H | Raise an emergency request | F-157 |
| `PF-D.U1` | U | Patient no-show** (at H3) | F-153 |
| `PF-D.U2` | U | Prescription not ready** (at H4) | F-330 |
| `PF-E.H1` | H | See schedule + nurse profile | F-170, F-220 |
| `PF-E.H2` | H | Verify nurse identity on arrival | F-221 |
| `PF-E.H3` | H | Confirm or change the time | F-171 |
| `PF-E.H4` | H | Confirm medication received | F-222 |
| `PF-E.H5` | H | Confirm service received | F-174 |
| `PF-E.H6` | H | Track the nurse live | F-172 |
| `PF-E.H7` | H | See injection history | F-175 |
| `PF-E.H8` | H | Next-injection reminder | F-176 |
| `PF-E.U1` | U | Patient not home / unreachable** (at H3) | F-171 |
| `PF-E.U2` | U | Live tracking unavailable** (at H6) | F-172 |
| `PF-F.H1` | H | Report a side effect | F-177, F-178 |
| `PF-F.U1` | U | Urgent side effect** (at H1) | F-178 |
| `PF-G.H1` | H | Adherence chart + streak | F-185, F-186 |
| `PF-G.H2` | H | Weight goal | F-188 |
| `PF-G.H3` | H | Daily reminders | F-189 |
| `PF-G.H4` | H | Content library | F-195, F-196, F-197, F-199 |
| `PF-G.H5` | H | Foodnoise tracking | F-223, F-224 |
| `PF-G.H6` | H | Multiple addresses + weight history | F-114, F-116, F-117 |
| `PF-G.H7` | H | Support chat + FAQ + emergency line | F-207, F-209, F-210 |
| `PF-G.H8` | H | Visit chat + survey + history PDF | F-154, F-156, F-158 |
| `PF-G.H9` | H | Pricing perks | F-132, F-144 |
| `PF-G.U1` | U | Nothing to show yet** (any step) | — |
| `MF-A.H1` | H | Landing home | F-095 |
| `MF-A.H2` | H | BMI calculator | F-096 |
| `MF-A.H3` | H | Public eligibility check (preliminary) | F-097 |
| `MF-A.U1` | U | Invalid BMI input** (at H2) | F-096 |
| `MF-A.U2` | U | Not a fit / borderline** (at H3) | F-097 |
| `MF-A.U3` | U | Preview unavailable** (at H3) | F-097 |
| `DF-A.H1` | H | Open the assigned patient list | F-310 |
| `DF-A.H2` | H | Open a patient profile + EMR | F-312 |
| `DF-A.H3` | H | Conduct the phone visit | F-153 |
| `DF-A.H4` | H | Write the SOAP note | F-328 |
| `DF-A.H5` | H | Link the external prescription | F-327, F-329 |
| `DF-A.H6` | H | Confirm prescription status | F-330 |
| `DF-A.H7` | H | Auto-schedule the next visit | F-331 |
| `DF-A.H8` | H | External e-prescription integration goes live | F-330 |
| `DF-A.U1` | U | Record incomplete at visit** (at H2) | F-312 |
| `DF-A.U2` | U | Rx rejected / invalid** (at H5) | F-330 |
| `DF-A.U3` | U | Rx provider unreachable** (at H5) | F-330 |
| `DF-B.H1` | H | Doctor login | F-102 |
| `DF-B.H2` | H | Doctor dashboard | F-301, F-305, F-302, F-303, F-304 |
| `DF-B.H3` | H | Patient management | F-311, F-313, F-314, F-315, F-317, F-319, F-316 |
| `DF-B.H4` | H | Doctor scheduling | F-340, F-341 |
| `DF-B.H5` | H | Doctor visits list | F-301, F-313 |
| `DF-B.H6` | H | Doctor prescriptions list | F-329, F-330 |
| `DF-B.H7` | H | Doctor financial panel | F-350, F-356, F-358, F-351, F-352, F-353, F-354, F-355 |
| `DF-B.U1` | U | Provider deactivated** (at H1) | — |
| `DF-B.U2` | U | Nothing assigned yet** (at H2) | — |
| `DF-B.U3` | U | Assignment lost mid-care** (at H3) | — |
| `NF-A.H1` | H | Nurse logs in | F-401 |
| `NF-A.H2` | H | Nurse dashboard & status | F-405, F-407 |
| `NF-A.H3` | H | View daily route & navigate | F-415, F-416, F-417, F-418 |
| `NF-A.H4` | H | GPS check-in at the patient's location | F-430 |
| `NF-A.H5` | H | View patient profile + today's dose | F-431, F-432 |
| `NF-A.H6` | H | Record cold-chain temperature | F-433 |
| `NF-A.H7` | H | Pre-injection assessment | F-434 |
| `NF-A.H8` | H | Scan vial, then record injection site | F-437 |
| `NF-A.H9` | H | Patient signs | F-441 |
| `NF-A.H10` | H | Submit the visit | F-442 |
| `NF-A.U1` | U | GPS fails / location off** (at H4) | F-430 |
| `NF-A.U2` | U | Patient not home / refuses / unsafe** (at H5) | — |
| `NF-A.U3` | U | Cold-chain breach | F-433 |
| `NF-A.U4` | U | Wrong / mismatched vial scanned** (at H8) | F-437 |
| `NF-A.U5` | U | Signature capture fails / patient can't sign** (at H9) | F-441 |
| `NF-A.U6` | U | Offline mid-visit** (any step) | F-430, F-442 |
| `NF-B.H1` | H | Escalate an emergency | F-443 |
| `NF-B.H2` | H | Press the safety button | F-497 |
| `NF-B.H3` | H | Emergency escalation in-software, routed to CS | F-443 |
| `NF-B.U1` | U | Safety button pressed** (at H2) | F-497 |
| `NF-C.H1` | H | Live tracking + ETA | F-707, F-708, F-172 |
| `NF-C.H2` | H | Assigned-patient list | F-450, F-451, F-452, F-453, F-454 |
| `NF-C.H3` | H | Nurse inventory | F-460, F-465, F-461, F-462, F-463, F-464 |
| `NF-C.H4` | H | Reorder / reschedule | F-420, F-422 |
| `NF-C.H5` | H | Record patient education | F-440 |
| `NF-C.H6` | H | Week/month summary | F-408 |
| `NF-C.H7` | H | Support inbox | F-410 |
| `NF-C.H8` | H | Nurse financial panel | F-470, F-477, F-471, F-472, F-473, F-474, F-475, F-476 |
| `NF-C.H9` | H | Performance + hotline | F-485, F-486, F-495 |
| `NF-C.U1` | U | Route unavailable / Optime down** (at H1) | F-703 |
| `NF-C.U2` | U | Shortage at pickup / count mismatch** (at H3) | F-465 |
| `NF-C.U3` | U | No visits / patients / inventory** (at H9) | — |
| `PH-A.H1` | H | Pharmacy login (OTP) | F-501 |
| `PH-A.H2` | H | Inventory dashboard | F-502 |
| `PH-A.H3` | H | Record supplier receipt | F-503 |
| `PH-A.H4` | H | Monitor cold chain | F-505 |
| `PH-A.H5` | H | Dispense to nurse | F-504 |
| `PH-A.H6` | H | Report damage | F-506 |
| `PH-A.H7` | H | View audit log | F-515 |
| `PH-A.H8` | H | Pharmacy commission | F-507, F-508, F-512, F-513 |
| `PH-A.U1` | U | No inventory / no pending dispenses** (at H2) | — |
| `PH-A.U2` | U | Hub cold-chain breach** (at H4) | F-505 |
| `PH-A.U3` | U | Barcode scan fails** (at H5) | F-504 |
| `PH-A.U4` | U | Stock-out / cannot fulfill** (at H5) | F-504 |
| `PH-A.U5` | U | Expired stock detected** (at H6) | F-506 |
| `PH-B.H1` | H | Recall a batch — cascade to its vials | F-505, F-618 |
| `PH-B.H2` | H | Compute affected patients | F-618 |
| `PH-B.U1` | U | Batch recalled** (at H1) | F-618 |
| `OP-A.H1` | H | List patients | F-601 |
| `OP-A.H2` | H | Patient detail + status | F-602, F-603 |
| `OP-A.H3` | H | Edit Rx protocols | F-651 |
| `OP-A.H4` | H | Manage roles & permissions | F-655 |
| `OP-A.H5` | H | Manage API keys | F-656 |
| `OP-A.H6` | H | Onboard a provider (doctor / nurse) | F-606, F-609 |
| `OP-A.H7` | H | Verify credentials | F-403, F-404 |
| `OP-A.H8` | H | Assign a patient to a provider | F-610 |
| `OP-A.H9` | H | User management | F-604, F-605, F-607, F-608 |
| `OP-A.U1` | U | Role / permission misconfiguration** (at H4) | F-655 |
| `OP-A.U2` | U | No nurse available / no capacity** (at H8) | — |
| `OP-B.H1` | H | Geocode the patient's address | F-701, F-702 |
| `OP-B.H2` | H | Generate the optimized route | F-703, F-704, F-705 |
| `OP-B.H3` | H | On-demand emergency assignment | F-706, F-406, F-421 |
| `OP-B.H4` | H | Operations dashboard | F-615, F-616, F-617, F-618 |
| `OP-B.H5` | H | Reschedule tool | F-619 |
| `OP-B.U1` | U | Geocode failed** (at H1) | F-701 |
| `OP-B.U2` | U | Incident / cold-chain network alert** (at H4) | F-617, F-618 |
| `OP-C.H1` | H | Monitor side-effect reports + assign a reviewer | F-620 |
| `OP-C.H2` | H | Open a replacement order | F-433 |
| `OP-D.H1` | H | Issue a refund | F-147 |
| `OP-D.H2` | H | Run monthly settlements | F-627, F-628, F-629 |
| `OP-D.H3` | H | Finance dashboard | F-625, F-626, F-631, F-632, F-633 |
| `OP-D.H4` | H | Growth dashboard | F-640, F-641 |
| `OP-D.H5` | H | Campaign manager | F-644 |
| `OP-D.H6` | H | Settings | F-652, F-653, F-657 |
| `OP-D.H7` | H | Reports + export | F-660, F-662, F-663 |
| `OP-D.U1` | U | Refund / chargeback edge** (at H1) | F-633 |
| `OP-D.U2` | U | Settlement run fails / partial** (at H2) | F-629 |
| `OP-D.U3` | U | Bulk action partial failure** (at H5) | F-644 |
| `CS-A.H1` | H | Call console + routing | F-700, F-703 |
| `CS-A.H2` | H | Six CS sub-panels, scoped per panel | F-710, F-761, F-711, F-720, F-721, F-730, F-731, F-740, F-741, F-750, F-751, F-760 |
| `CS-A.H3` | H | CS messaging + KPI | F-770, F-771 |
| `CS-A.H4` | H | Emergency escalation in-software | F-443 |
| `AF-A.H1` | H | Affiliate dashboard | F-800, F-801 |
| `AF-A.H2` | H | Referral links + conversion | F-802, F-803 |
| `AF-A.H3` | H | Affiliate onboarding + codes + audit | F-807, F-808, F-809 |
| `AF-A.H4` | H | Commission + payment | F-804, F-805 |
| `AF-A.H5` | H | A/B experiments | — |
| `AF-A.U1` | U | Referral link invalid / expired** (at H2) | F-802 |
| `AF-A.U2` | U | Application rejected** (at H3) | F-807 |
| `AF-A.U3` | U | Commission dispute / not yet payable** (at H4) | F-804, F-805 |
| `XF.U1` | U | Network offline | — |
| `XF.U2` | U | Session expired / token invalid | — |
| `XF.U3` | U | Permission denied (403) | — |
| `XF.U4` | U | Not found (404) / deleted | — |
| `XF.U5` | U | Server / dependent-service error (5xx) | — |
| `XF.U6` | U | Rate limited (429) | — |
| `XF.U7` | U | Stale data / concurrent edit conflict | — |
| `R-KYC-REVIEW` | R | KYC manual review | F-105 |
| `R-ELIG-REVIEW` | R | Paid doctor eligibility review | F-123, F-125 |
| `R-PAY-RETRY` | R | Payment-failure resolution | F-131 |
| `R-DUNNING` | R | Past-due dunning | F-133 |
| `R-REACTIVATE` | R | Expired-mid-treatment reactivation | F-133 |
| `R-REFUND` | R | Refund / refund-pending | F-147, F-144 |
| `R-CHARGEBACK` | R | Chargeback reconciliation | F-633 |
| `R-RESCHEDULE` | R | Reschedule a missed/aborted visit | F-422, F-619 |
| `R-RXFIX` | R | External Rx re-issue + re-link | F-330 |
| `R-CLINICAL-ESC` | R | Urgent side-effect clinical escalation | F-178 |
| `R-REPLACE` | R | Replacement-dose order | F-433, F-504, F-506 |
| `R-VIAL-RESCAN` | R | Wrong/mismatched vial | F-437 |
| `R-SAFETY-ESC` | R | Nurse safety escalation | F-497, F-443 |
| `R-STOCKOUT` | R | Stock-out / shortage at pickup | F-465, F-504 |
| `R-HUB-QUARANTINE` | R | Hub cold-chain quarantine | F-505, F-506 |
| `R-RECALL` | R | Batch recall → reassign affected patients | F-618 |
| `R-REASSIGN` | R | Provider reassignment / in-flight care | — |
| `R-CAPACITY` | R | No nurse available / no capacity | — |
| `R-SETTLE-RETRY` | R | Settlement-run failure | F-629 |
| `R-RESEND-OTP` | R | Resend a fresh OTP | F-101 |
| `R-COOLDOWN` | R | Lockout cooldown | F-102 |
| `R-BLOCKED-STATE` | R | Full-screen blocked/rejected state | — |
| `R-INLINE-VALIDATE` | R | Inline validation + retry | — |
| `R-EMPTY-STATE` | R | Designed empty state | — |
| `R-TRACK-FALLBACK` | R | Tracking fallback | F-172, F-703 |
| `R-GUIDANCE-CTA` | R | Supportive guidance + sign-up CTA | F-097 |
| `R-PREVIEW-RETRY` | R | Retry / proceed to full check | F-097 |
| `R-COMPLETE-RECORD` | R | Prompt to complete record | F-312 |
| `R-GPS-OVERRIDE` | R | Manual GPS override | F-430 |
| `R-OTP-RECEIPT` | R | OTP receipt fallback | F-441 |
| `R-OFFLINE-CAPTURE` | R | Offline capture + sync | F-430, F-442 |
| `R-MANUAL-ENTRY` | R | Manual barcode entry | F-504 |
| `R-EXPIRE-QUARANTINE` | R | Quarantine + replace expired stock | F-506 |
| `R-ROLE-FIX` | R | Role/permission correction | F-655 |
| `R-GEOCODE-MANUAL` | R | Manual geocode pin | F-701 |
| `R-INCIDENT-TRIAGE` | R | Incident triage | F-617, F-618 |
| `R-BULK-RETRY` | R | Per-row retry | F-644 |
| `R-NEW-LINK` | R | Issue a fresh referral link | F-802 |
| `R-CS-TICKET` | R | Generic CS ticket | F-804, F-805 |
| `R-OFFLINE-BANNER` | R | Offline banner | — |
| `R-REAUTH` | R | Re-authenticate | — |
| `R-NO-ACCESS` | R | Permission-denied state | — |
| `R-NOT-FOUND` | R | Not-found state | — |
| `R-RETRY-5XX` | R | Server-error retry | — |
| `R-RATE-LIMIT` | R | Rate-limit wait | — |
| `R-CONFLICT` | R | Concurrent-edit conflict | — |

## Part 2 — Feature id → Flow ids

| Feature id | Realized by (flow ids) |
|---|---|
| `F-095` | `MF-A.H1` |
| `F-096` | `MF-A.H2`, `MF-A.U1` |
| `F-097` | `MF-A.H3`, `MF-A.U2`, `MF-A.U3`, `R-GUIDANCE-CTA`, `R-PREVIEW-RETRY` |
| `F-100` | `PF-B.H10` |
| `F-101` | `PF-A.H1`, `PF-A.H2`, `PF-A.U1`, `R-RESEND-OTP` |
| `F-102` | `PF-A.H3`, `PF-A.U2`, `PF-A.U3`, `DF-B.H1`, `R-COOLDOWN` |
| `F-103` | `PF-A.H4` |
| `F-105` | `PF-A.H5`, `PF-A.U4`, `R-KYC-REVIEW` |
| `F-110` | `PF-B.H1` |
| `F-111` | `PF-B.H2` |
| `F-112` | `PF-B.H3` |
| `F-113` | `PF-B.H4`, `PF-B.U1` |
| `F-114` | `PF-G.H6` |
| `F-115` | `PF-B.H5` |
| `F-116` | `PF-G.H6` |
| `F-117` | `PF-G.H6` |
| `F-121` | `PF-B.H6` |
| `F-122` | `PF-B.H7` |
| `F-123` | `PF-B.H7`, `PF-B.U3`, `R-ELIG-REVIEW` |
| `F-124` | `PF-B.H8` |
| `F-125` | `PF-B.H7`, `PF-B.U2`, `R-ELIG-REVIEW` |
| `F-126` | `PF-B.H7` |
| `F-130` | `PF-C.H1` |
| `F-131` | `PF-C.H2`, `PF-C.U1`, `R-PAY-RETRY` |
| `F-132` | `PF-G.H9` |
| `F-133` | `PF-C.H3`, `PF-C.U2`, `PF-C.U3`, `R-DUNNING`, `R-REACTIVATE` |
| `F-139` | `PF-C.H4` |
| `F-141` | `PF-C.H5` |
| `F-142` | `PF-C.H5` |
| `F-143` | `PF-C.H5` |
| `F-144` | `PF-C.U4`, `PF-G.H9`, `R-REFUND` |
| `F-146` | `PF-C.H5` |
| `F-147` | `PF-C.U4`, `OP-D.H1`, `R-REFUND` |
| `F-150` | `PF-D.H1` |
| `F-151` | `PF-D.H1` |
| `F-152` | `PF-D.H2` |
| `F-153` | `PF-D.H3`, `PF-D.U1`, `DF-A.H3` |
| `F-154` | `PF-G.H8` |
| `F-155` | `PF-D.H4` |
| `F-156` | `PF-G.H8` |
| `F-157` | `PF-D.H5` |
| `F-158` | `PF-G.H8` |
| `F-159` | `PF-D.H4` |
| `F-170` | `PF-E.H1` |
| `F-171` | `PF-E.H3`, `PF-E.U1` |
| `F-172` | `PF-E.H6`, `PF-E.U2`, `NF-C.H1`, `R-TRACK-FALLBACK` |
| `F-174` | `PF-E.H5` |
| `F-175` | `PF-E.H7` |
| `F-176` | `PF-E.H8` |
| `F-177` | `PF-F.H1` |
| `F-178` | `PF-F.H1`, `PF-F.U1`, `R-CLINICAL-ESC` |
| `F-185` | `PF-G.H1` |
| `F-186` | `PF-G.H1` |
| `F-188` | `PF-G.H2` |
| `F-189` | `PF-G.H3` |
| `F-195` | `PF-G.H4` |
| `F-196` | `PF-G.H4` |
| `F-197` | `PF-G.H4` |
| `F-199` | `PF-G.H4` |
| `F-205` | `PF-B.H9` |
| `F-207` | `PF-G.H7` |
| `F-209` | `PF-G.H7` |
| `F-210` | `PF-G.H7` |
| `F-220` | `PF-E.H1` |
| `F-221` | `PF-E.H2` |
| `F-222` | `PF-E.H4` |
| `F-223` | `PF-G.H5` |
| `F-224` | `PF-G.H5` |
| `F-301` | `DF-B.H2`, `DF-B.H5` |
| `F-302` | `DF-B.H2` |
| `F-303` | `DF-B.H2` |
| `F-304` | `DF-B.H2` |
| `F-305` | `DF-B.H2` |
| `F-310` | `DF-A.H1` |
| `F-311` | `DF-B.H3` |
| `F-312` | `DF-A.H2`, `DF-A.U1`, `R-COMPLETE-RECORD` |
| `F-313` | `DF-B.H3`, `DF-B.H5` |
| `F-314` | `DF-B.H3` |
| `F-315` | `DF-B.H3` |
| `F-316` | `DF-B.H3` |
| `F-317` | `DF-B.H3` |
| `F-319` | `DF-B.H3` |
| `F-327` | `DF-A.H5` |
| `F-328` | `DF-A.H4` |
| `F-329` | `DF-A.H5`, `DF-B.H6` |
| `F-330` | `PF-D.U2`, `DF-A.H6`, `DF-A.H8`, `DF-A.U2`, `DF-A.U3`, `DF-B.H6`, `R-RXFIX` |
| `F-331` | `DF-A.H7` |
| `F-340` | `DF-B.H4` |
| `F-341` | `DF-B.H4` |
| `F-350` | `DF-B.H7` |
| `F-351` | `DF-B.H7` |
| `F-352` | `DF-B.H7` |
| `F-353` | `DF-B.H7` |
| `F-354` | `DF-B.H7` |
| `F-355` | `DF-B.H7` |
| `F-356` | `DF-B.H7` |
| `F-358` | `DF-B.H7` |
| `F-401` | `NF-A.H1` |
| `F-403` | `OP-A.H7` |
| `F-404` | `OP-A.H7` |
| `F-405` | `NF-A.H2` |
| `F-406` | `OP-B.H3` |
| `F-407` | `NF-A.H2` |
| `F-408` | `NF-C.H6` |
| `F-410` | `NF-C.H7` |
| `F-415` | `NF-A.H3` |
| `F-416` | `NF-A.H3` |
| `F-417` | `NF-A.H3` |
| `F-418` | `NF-A.H3` |
| `F-420` | `NF-C.H4` |
| `F-421` | `OP-B.H3` |
| `F-422` | `NF-C.H4`, `R-RESCHEDULE` |
| `F-430` | `NF-A.H4`, `NF-A.U1`, `NF-A.U6`, `R-GPS-OVERRIDE`, `R-OFFLINE-CAPTURE` |
| `F-431` | `NF-A.H5` |
| `F-432` | `NF-A.H5` |
| `F-433` | `NF-A.H6`, `NF-A.U3`, `OP-C.H2`, `R-REPLACE` |
| `F-434` | `NF-A.H7` |
| `F-437` | `NF-A.H8`, `NF-A.U4`, `R-VIAL-RESCAN` |
| `F-440` | `NF-C.H5` |
| `F-441` | `NF-A.H9`, `NF-A.U5`, `R-OTP-RECEIPT` |
| `F-442` | `NF-A.H10`, `NF-A.U6`, `R-OFFLINE-CAPTURE` |
| `F-443` | `NF-B.H1`, `NF-B.H3`, `CS-A.H4`, `R-SAFETY-ESC` |
| `F-450` | `NF-C.H2` |
| `F-451` | `NF-C.H2` |
| `F-452` | `NF-C.H2` |
| `F-453` | `NF-C.H2` |
| `F-454` | `NF-C.H2` |
| `F-460` | `NF-C.H3` |
| `F-461` | `NF-C.H3` |
| `F-462` | `NF-C.H3` |
| `F-463` | `NF-C.H3` |
| `F-464` | `NF-C.H3` |
| `F-465` | `NF-C.H3`, `NF-C.U2`, `R-STOCKOUT` |
| `F-470` | `NF-C.H8` |
| `F-471` | `NF-C.H8` |
| `F-472` | `NF-C.H8` |
| `F-473` | `NF-C.H8` |
| `F-474` | `NF-C.H8` |
| `F-475` | `NF-C.H8` |
| `F-476` | `NF-C.H8` |
| `F-477` | `NF-C.H8` |
| `F-485` | `NF-C.H9` |
| `F-486` | `NF-C.H9` |
| `F-495` | `NF-C.H9` |
| `F-497` | `NF-B.H2`, `NF-B.U1`, `R-SAFETY-ESC` |
| `F-501` | `PH-A.H1` |
| `F-502` | `PH-A.H2` |
| `F-503` | `PH-A.H3` |
| `F-504` | `PH-A.H5`, `PH-A.U3`, `PH-A.U4`, `R-REPLACE`, `R-STOCKOUT`, `R-MANUAL-ENTRY` |
| `F-505` | `PH-A.H4`, `PH-A.U2`, `PH-B.H1`, `R-HUB-QUARANTINE` |
| `F-506` | `PH-A.H6`, `PH-A.U5`, `R-REPLACE`, `R-HUB-QUARANTINE`, `R-EXPIRE-QUARANTINE` |
| `F-507` | `PH-A.H8` |
| `F-508` | `PH-A.H8` |
| `F-512` | `PH-A.H8` |
| `F-513` | `PH-A.H8` |
| `F-515` | `PH-A.H7` |
| `F-601` | `OP-A.H1` |
| `F-602` | `OP-A.H2` |
| `F-603` | `OP-A.H2` |
| `F-604` | `OP-A.H9` |
| `F-605` | `OP-A.H9` |
| `F-606` | `OP-A.H6` |
| `F-607` | `OP-A.H9` |
| `F-608` | `OP-A.H9` |
| `F-609` | `OP-A.H6` |
| `F-610` | `OP-A.H8` |
| `F-615` | `OP-B.H4` |
| `F-616` | `OP-B.H4` |
| `F-617` | `OP-B.H4`, `OP-B.U2`, `R-INCIDENT-TRIAGE` |
| `F-618` | `PH-B.H1`, `PH-B.H2`, `PH-B.U1`, `OP-B.H4`, `OP-B.U2`, `R-RECALL`, `R-INCIDENT-TRIAGE` |
| `F-619` | `OP-B.H5`, `R-RESCHEDULE` |
| `F-620` | `OP-C.H1` |
| `F-625` | `OP-D.H3` |
| `F-626` | `OP-D.H3` |
| `F-627` | `OP-D.H2` |
| `F-628` | `OP-D.H2` |
| `F-629` | `OP-D.H2`, `OP-D.U2`, `R-SETTLE-RETRY` |
| `F-631` | `OP-D.H3` |
| `F-632` | `OP-D.H3` |
| `F-633` | `OP-D.H3`, `OP-D.U1`, `R-CHARGEBACK` |
| `F-640` | `OP-D.H4` |
| `F-641` | `OP-D.H4` |
| `F-644` | `OP-D.H5`, `OP-D.U3`, `R-BULK-RETRY` |
| `F-651` | `OP-A.H3` |
| `F-652` | `OP-D.H6` |
| `F-653` | `OP-D.H6` |
| `F-655` | `OP-A.H4`, `OP-A.U1`, `R-ROLE-FIX` |
| `F-656` | `OP-A.H5` |
| `F-657` | `OP-D.H6` |
| `F-660` | `OP-D.H7` |
| `F-662` | `OP-D.H7` |
| `F-663` | `OP-D.H7` |
| `F-700` | `CS-A.H1` |
| `F-701` | `OP-B.H1`, `OP-B.U1`, `R-GEOCODE-MANUAL` |
| `F-702` | `OP-B.H1` |
| `F-703` | `NF-C.U1`, `OP-B.H2`, `CS-A.H1`, `R-TRACK-FALLBACK` |
| `F-704` | `OP-B.H2` |
| `F-705` | `OP-B.H2` |
| `F-706` | `OP-B.H3` |
| `F-707` | `NF-C.H1` |
| `F-708` | `NF-C.H1` |
| `F-710` | `CS-A.H2` |
| `F-711` | `CS-A.H2` |
| `F-720` | `CS-A.H2` |
| `F-721` | `CS-A.H2` |
| `F-730` | `CS-A.H2` |
| `F-731` | `CS-A.H2` |
| `F-740` | `CS-A.H2` |
| `F-741` | `CS-A.H2` |
| `F-750` | `CS-A.H2` |
| `F-751` | `CS-A.H2` |
| `F-760` | `CS-A.H2` |
| `F-761` | `CS-A.H2` |
| `F-770` | `CS-A.H3` |
| `F-771` | `CS-A.H3` |
| `F-800` | `AF-A.H1` |
| `F-801` | `AF-A.H1` |
| `F-802` | `AF-A.H2`, `AF-A.U1`, `R-NEW-LINK` |
| `F-803` | `AF-A.H2` |
| `F-804` | `AF-A.H4`, `AF-A.U3`, `R-CS-TICKET` |
| `F-805` | `AF-A.H4`, `AF-A.U3`, `R-CS-TICKET` |
| `F-807` | `AF-A.H3`, `AF-A.U2` |
| `F-808` | `AF-A.H3` |
| `F-809` | `AF-A.H3` |

## Part 3 — Features with no flow step (accounted for)

These features are intentionally absent from the flows above. Each is listed with its reason and where it is handled.

### Removed features (not implemented)

Marked removed in the feature list (v2.1 / product decision). No flow, by design.

| Feature id | Status |
|---|---|
| `F-104` | Removed — 2FA, per product decision. |
| `F-106` | Removed in v2.1 — excluded from scope. |
| `F-160` | Removed in v2.1 — excluded from scope. |
| `F-173` | Removed in v2.1 — excluded from scope. |
| `F-180` | Removed in v2.1 — excluded from scope. |
| `F-206` | Removed in v2.1 — excluded from scope. |
| `F-320` | Removed in v2.1 — excluded from scope. |
| `F-325` | Removed in v2.1 — excluded from scope. |
| `F-326` | Removed in v2.1 — excluded from scope. |
| `F-332` | Removed in v2.1 — excluded from scope. |
| `F-419` | Removed in v2.1 — excluded from scope. |
| `F-435` | Removed in v2.1 — excluded from scope. |
| `F-436` | Removed in v2.1 — excluded from scope. |
| `F-438` | Removed in v2.1 — excluded from scope. |
| `F-439` | Removed in v2.1 — excluded from scope. |
| `F-514` | Removed in v2.1 — excluded from scope. |

### Cross-cutting internationalization (no single step)

Owned by the i18n layer (`pkg-i18n`) and applied to **every** flow's UI and notifications, so they map to no single happy step. Where done: the shared i18n layer across all surfaces.

| Feature id | Where it is handled |
|---|---|
| `F-090` | Multilanguage UI — message catalogs per surface (`fa-IR` default, `en` next). |
| `F-091` | In-app language switcher — surfaced in app/settings; persists `identity.preferred_locale`. |
| `F-092` | RTL/LTR layout by locale — no per-screen redesign. |
| `F-093` | Locale-aware formatting — Jalali/Gregorian dates, Persian/Latin numerals, currency. |
| `F-094` | Localized notifications — SMS/push/email rendered in the recipient locale. |

---

*Accounting: 224 mapped + 16 removed + 5 cross-cutting = 245 of 245 features. **All features accounted for.** Generated from `Medica_User_Flows.md`.*