# Medica — Feature → Flow Map

For each feature (`F-` id from `Medica_p0_features_en.md`), the flow ids in `Medica_User_Flows.md` that realize it. Flow id types: **H** happy step · **U** unhappy branch · **R** recovery.

Coverage: **225 of 246** features realized by a flow; the rest have no flow step (listed at end).
---
## Feature id → Flow ids
| Feature id | Realized by (flow ids) |
|---|---|
| `F-095` | `MF-A.H1` |
| `F-096` | `MF-A.H2`, `MF-A.U1` |
| `F-097` | `MF-A.H3`, `MF-A.U2`, `MF-A.U3`, `R-GUIDANCE-CTA`, `R-PREVIEW-RETRY` |
| `F-100` | `PF-B.H10` |
| `F-101` | `PF-A.H1`, `PF-A.H2`, `PF-A.U1`, `R-RESEND-OTP` |
| `F-102` | `PF-A.H3`, `PF-A.U2`, `PF-A.U3`, `R-COOLDOWN`, `DF-B.H1` |
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
| `F-130` | `PF-C.H1`, `PF-D.U3`, `R-SUBSCRIBE-GATE` |
| `F-131` | `PF-C.H2`, `PF-C.U1`, `R-PAY-RETRY`, `PF-D.U3`, `R-SUBSCRIBE-GATE` |
| `F-132` | `PF-G.H9` |
| `F-133` | `PF-C.H3`, `PF-C.U2`, `PF-C.U3`, `R-DUNNING`, `R-REACTIVATE` |
| `F-139` | `PF-C.H4` |
| `F-141` | `PF-C.H5` |
| `F-142` | `PF-C.H5` |
| `F-143` | `PF-C.H5` |
| `F-144` | `PF-C.U4`, `R-REFUND`, `PF-G.H9` |
| `F-146` | `PF-C.H5` |
| `F-147` | `PF-C.U4`, `R-REFUND`, `OP-D.H1` |
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
| `F-409` | `NF-A.H3` |
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
| `F-443` | `NF-B.H1`, `NF-B.H3`, `R-SAFETY-ESC`, `CS-A.H4` |
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
| `F-504` | `PH-A.H5`, `PH-A.U3`, `PH-A.U4`, `R-MANUAL-ENTRY`, `R-REPLACE`, `R-STOCKOUT` |
| `F-505` | `PH-A.H4`, `PH-A.U2`, `R-HUB-QUARANTINE`, `PH-B.H1` |
| `F-506` | `PH-A.H6`, `PH-A.U5`, `R-HUB-QUARANTINE`, `R-EXPIRE-QUARANTINE`, `R-REPLACE` |
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
| `F-618` | `PH-B.H1`, `PH-B.H2`, `PH-B.U1`, `R-RECALL`, `OP-B.H4`, `OP-B.U2`, `R-INCIDENT-TRIAGE` |
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

## Features with no flow step (accounted for)
### Removed features (not implemented)
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
Owned by the i18n layer (`pkg-i18n`).
| Feature id | Where it is handled |
|---|---|
| `F-090` | Multilanguage UI — message catalogs per surface (`fa-IR` default, `en` next). |
| `F-091` | In-app language switcher — surfaced in app/settings; persists `identity.preferred_locale`. |
| `F-092` | RTL/LTR layout by locale — no per-screen redesign. |
| `F-093` | Locale-aware formatting — Jalali/Gregorian dates, Persian/Latin numerals, currency. |
| `F-094` | Localized notifications — SMS/push/email rendered in the recipient locale. |

---
*Accounting: 225 mapped + 16 removed + 5 cross-cutting = 246 of 246. **All accounted for.***