# Medica — Service & Microfrontend Map

Part A services → endpoints → flow steps. Part B MFEs → routes → flow steps.
---
## Part A — Backend microservices

### `auth` · identity & sessions
| API id | method · path | serves (flow ids) |
|---|---|---|
| `API-AUTH-001` | `POST /auth/register` | `PF-A.H1`, `PF-A.H2`, `PF-A.U1`, `R-RESEND-OTP` |
| `API-AUTH-002` | `POST /auth/otp/verify` | `PF-A.H1`, `PF-A.H2`, `PF-A.U1`, `R-RESEND-OTP` |
| `API-AUTH-003` | `POST /auth/login` | `DF-B.H1`, `NF-A.H1`, `PF-A.H3`, `PF-A.U2`, `PF-A.U3`, `R-COOLDOWN` |
| `API-AUTH-004/005` | `POST /auth/password/reset/{request,confirm}` | `PF-A.H4` |
| `API-AUTH-007` | `POST /me/kyc/verify` | `PF-A.H5`, `PF-A.U4`, `R-KYC-REVIEW` |

### `patient` · profiles, health, eligibility
| API id | method · path | serves (flow ids) |
|---|---|---|
| `API-PATIENT-001` | `PUT /me/profile` | `PF-B.H1` |
| `API-PATIENT-002` | `PUT /me/health` | `PF-B.H2` |
| `API-PATIENT-003` | `PUT /me/history` | `PF-B.H3` |
| `API-PATIENT-004` | `POST /me/labs` | `PF-B.H4`, `PF-B.U1` |
| `API-PATIENT-005` | `PUT /me/address` | `PF-B.H5` |
| `API-PATIENT-010` | `GET /patients/{id}` | `DF-A.H2`, `DF-A.U1`, `R-COMPLETE-RECORD` |
| `API-PATIENT-020` | `GET /eligibility/questionnaire` | `PF-B.H6` |
| `API-PATIENT-021` | `POST /me/eligibility/answers` | `PF-B.H7`, `PF-B.U2`, `PF-B.U3`, `R-ELIG-REVIEW` |
| `API-PATIENT-022` | `POST /me/consent` | `PF-B.H8` |
| `API-PATIENT-023` | `GET /public/eligibility/preview-questionnaire` | `MF-A.H3`, `MF-A.U2`, `MF-A.U3`, `R-GUIDANCE-CTA`, `R-PREVIEW-RETRY` |
| `API-PATIENT-024` | `POST /public/eligibility/preview` | `MF-A.H3`, `MF-A.U2`, `MF-A.U3`, `R-GUIDANCE-CTA`, `R-PREVIEW-RETRY` |
| `API-PATIENT-025` | `GET /me/home` | `PF-B.H10` |
| `API-PATIENT-030` | `POST /me/side-effects` | `PF-F.H1`, `PF-F.U1`, `R-CLINICAL-ESC` |
| `API-PATIENT-031` | `GET /admin/side-effects` | `OP-C.H1` |
| `API-PATIENT-032` | `POST /admin/side-effects/{id}/assign` | `OP-C.H1` |
| `API-PATIENT-040` | `GET /patients/{id}/charts` | `DF-B.H3`, `DF-B.H5` |
| `API-PATIENT-050` | `GET /me/adherence` | `PF-G.H1` |
| `API-PATIENT-051` | `CRUD /me/goals` | `PF-G.H2` |
| `API-PATIENT-052` | `CRUD /me/reminders` | `PF-G.H3` |
| `API-PATIENT-053` | `GET /content` | `PF-G.H4` |
| `API-PATIENT-054` | `CRUD /me/foodnoise` | `PF-G.H5` |
| `API-PATIENT-055` | `CRUD /me/addresses` | `PF-G.H6` |

### `prov` · providers & assignments
| API id | method · path | serves (flow ids) |
|---|---|---|
| `API-PROV-001` | `GET /me/patients` | `DF-A.H1` |
| `API-PROV-010` | `GET /me/nurse-visits/{id}/nurse-verify` | `PF-E.H2` |
| `API-PROV-020` | `POST /providers/onboard` | `OP-A.H6` |
| `API-PROV-021` | `POST /providers/{id}/verify` | `OP-A.H7` |
| `API-PROV-022` | `POST /assignments` | `OP-A.H8` |
| `API-PROV-023` | `GET /me/summary` | `NF-C.H6` |
| `API-PROV-030` | `CRUD /me/schedule` | `DF-B.H4` |

### `visit` · online visits & prescriptions
| API id | method · path | serves (flow ids) |
|---|---|---|
| `API-VISIT-001` | `GET /visits/availability` | `PF-D.H1` |
| `API-VISIT-002` | `POST /me/visits` | `PF-D.H1` |
| `API-VISIT-003` | `GET /me/visits/{visitId}` | `PF-D.H4` |
| `API-VISIT-004` | `POST /visits/{id}/soap` | `DF-A.H4` |
| `API-VISIT-005` | `POST /visits/{id}/prescription/link` | `DF-A.H5`, `DF-B.H6` |
| `API-VISIT-006` | `GET /visits/{id}/prescription/status` | `DF-A.H6`, `DF-A.H8`, `DF-B.H6`, `DF-A.U2`, `DF-A.U3`, `PF-D.U2`, `R-RXFIX` |
| `API-VISIT-007` | `POST /visits/{id}/next` | `DF-A.H7` |
| `API-VISIT-008` | `POST /me/visits/emergency` | `PF-D.H5` |
| `API-VISIT-020` | `GET /me/dashboard` | `DF-B.H2`, `DF-B.H5` |
| `API-VISIT-021` | `GET /patients/{id}/visit-history` | `DF-B.H3`, `DF-B.H5` |
| `API-VISIT-024` | `GET /me/prescriptions` | `DF-A.H5`, `DF-A.H6`, `DF-A.H8`, `DF-B.H6`, `DF-A.U2`, `DF-A.U3`, `PF-D.U2`, `R-RXFIX` |
| `API-VISIT-025` | `CRUD /me/visits/{id}/chat` | `PF-G.H8` |
| `API-VISIT-026` | `GET /me/visits?range=&status=` | `DF-B.H2`, `DF-B.H3`, `DF-B.H5` |

### `field` · logistics, nurse visits, routing
| API id | method · path | serves (flow ids) |
|---|---|---|
| `API-FIELD-001` | `GET /me/route?range=today|tomorrow|week` | `NF-A.H3` |
| `API-FIELD-002` | `POST /me/status` | `NF-A.H2` |
| `API-FIELD-003` | `POST /nurse-visits/{id}/checkin` | `NF-A.H4`, `NF-A.U1`, `NF-A.U6`, `R-GPS-OVERRIDE`, `R-OFFLINE-CAPTURE` |
| `API-FIELD-004` | `GET /nurse-visits/{id}` | `NF-A.H5` |
| `API-FIELD-005` | `POST /nurse-visits/{id}/coldchain` | `NF-A.H6`, `OP-C.H2`, `NF-A.U3`, `R-REPLACE` |
| `API-FIELD-006` | `POST /nurse-visits/{id}/assessment` | `NF-A.H7` |
| `API-FIELD-007` | `POST /nurse-visits/{id}/injection` | `NF-A.H8`, `NF-A.U4`, `R-VIAL-RESCAN` |
| `API-FIELD-008` | `POST /nurse-visits/{id}/confirm` | `NF-A.H9`, `NF-A.U5`, `R-OTP-RECEIPT` |
| `API-FIELD-009` | `POST /nurse-visits/{id}/submit` | `NF-A.H10`, `NF-A.U6`, `R-OFFLINE-CAPTURE` |
| `API-FIELD-010` | `POST /nurse-visits/{id}/emergency` | `CS-A.H4`, `NF-B.H1`, `NF-B.H3`, `R-SAFETY-ESC` |
| `API-FIELD-011` | `POST /me/safety` | `NF-B.H2`, `NF-B.U1`, `R-SAFETY-ESC` |
| `API-FIELD-020` | `GET /me/nurse-visits` | `PF-E.H1` |
| `API-FIELD-021` | `POST /me/nurse-visits/{id}/reschedule` | `PF-E.H3`, `PF-E.U1` |
| `API-FIELD-022` | `POST /me/nurse-visits/{id}/confirm-receipt` | `PF-E.H4`, `PF-E.H5` |
| `API-FIELD-023` | `GET /me/injections` | `PF-E.H7` |
| `API-FIELD-024` | `GET /me/nurse-visits/availability` | `PF-E.H9` |
| `API-FIELD-025` | `PUT /me/injection-schedule` | `PF-E.H9` |
| `API-FIELD-030` | `POST /logistics/geocode` | `OP-B.H1`, `OP-B.U1`, `R-GEOCODE-MANUAL` |
| `API-FIELD-033` | `POST /logistics/assign` | `OP-B.H3` |
| `API-FIELD-034` | `GET /me/nurse-visits/{id}/track` | `NF-C.H1`, `PF-E.H6`, `PF-E.U2`, `R-TRACK-FALLBACK` |
| `API-FIELD-040` | `GET /me/patients` | `NF-C.H2` |
| `API-FIELD-041` | `CRUD /me/inventory` | `NF-C.H3`, `NF-C.U2`, `R-STOCKOUT` |
| `API-FIELD-042` | `POST /nurse-visits/{id}/education` | `NF-C.H5` |
| `API-FIELD-050` | `POST /replacements` | `NF-A.H6`, `OP-C.H2`, `NF-A.U3`, `R-REPLACE` |
| `API-FIELD-051` | `GET /replacements` | `NF-A.H6`, `OP-C.H2`, `NF-A.U3`, `R-REPLACE` |

### `pay` · payments, wallet, settlements
| API id | method · path | serves (flow ids) |
|---|---|---|
| `API-PAY-001/002` | `POST /payments/{checkout,callback}` | `PF-C.H2`, `PF-C.U1`, `PF-D.U3`, `R-PAY-RETRY`, `R-SUBSCRIBE-GATE` |
| `API-PAY-007` | `POST /wallet/{patientId}/refund` | `OP-D.H1`, `PF-C.U4`, `R-REFUND` |
| `API-PAY-020` | `GET /me/finance` | `DF-B.H7` |
| `API-PAY-021` | `GET /me/finance` | `NF-C.H8` |
| `API-PAY-022` | `GET /me/performance` | `NF-C.H9` |
| `API-PAY-023` | `GET /me/commission` | `PH-A.H8` |
| `API-PAY-024` | `POST /settlements/run` | `OP-D.H2`, `OP-D.U2`, `R-SETTLE-RETRY` |
| `API-PAY-025` | `POST /me/wallet/cashback` | `PF-G.H9`, `PF-C.U4`, `R-REFUND` |
| `API-PAY-030` | `GET /finance/dashboard` | `OP-D.H3`, `OP-D.U1`, `R-CHARGEBACK` |

### `pharm` · pharmacy inventory & cold-chain
| API id | method · path | serves (flow ids) |
|---|---|---|
| `API-PHARM-002` | `GET /me/pharmacy/inventory` | `PH-A.H2` |
| `API-PHARM-003` | `POST /me/pharmacy/receipts` | `PH-A.H3` |
| `API-PHARM-004` | `POST /me/pharmacy/dispense` | `PH-A.H5`, `PH-A.U3`, `PH-A.U4`, `R-MANUAL-ENTRY`, `R-REPLACE`, `R-STOCKOUT` |
| `API-PHARM-005` | `GET /me/pharmacy/coldchain` | `PH-A.H4`, `PH-B.H1`, `PH-A.U2`, `R-HUB-QUARANTINE` |
| `API-PHARM-006` | `POST /me/pharmacy/damage` | `PH-A.H6`, `PH-A.U5`, `R-EXPIRE-QUARANTINE`, `R-HUB-QUARANTINE`, `R-REPLACE` |
| `API-PHARM-007` | `GET /me/pharmacy/audit` | `PH-A.H7` |
| `API-PHARM-010` | `POST /batches/{id}/recall` | `OP-B.H4`, `PH-A.H4`, `PH-B.H1`, `PH-B.H2`, `OP-B.U2`, `PH-A.U2`, `PH-B.U1`, `R-HUB-QUARANTINE`, `R-INCIDENT-TRIAGE`, `R-RECALL` |
| `API-PHARM-011` | `GET /recalls/{id}/affected` | `OP-B.H4`, `PH-B.H1`, `PH-B.H2`, `OP-B.U2`, `PH-B.U1`, `R-INCIDENT-TRIAGE`, `R-RECALL` |

### `sub` · subscriptions
| API id | method · path | serves (flow ids) |
|---|---|---|
| `API-SUB-001` | `GET /subscriptions/packages` | `PF-C.H1`, `PF-D.U3`, `R-SUBSCRIBE-GATE` |
| `API-SUB-002` | `POST /me/subscriptions` | `PF-C.H2`, `PF-C.U1`, `PF-D.U3`, `R-PAY-RETRY`, `R-SUBSCRIBE-GATE` |
| `API-SUB-003` | `PATCH /me/subscriptions/{subId}/renewal` | `PF-C.H3`, `PF-C.U2`, `PF-C.U3`, `R-DUNNING`, `R-REACTIVATE` |
| `API-SUB-004` | `POST /me/subscriptions/{subId}/cancel` | `PF-C.H4` |
| `API-SUB-010` | `POST /me/subscriptions/discount` | `PF-G.H9`, `PF-C.U4`, `R-REFUND` |

### `notif` · notifications
| API id | method · path | serves (flow ids) |
|---|---|---|
| `API-NOTIF-001/002` | `POST /notify/{sms,push}` | `PF-B.H9` |
| `API-NOTIF-002` | `POST /notify/push` | `NF-C.H7` |
| `API-NOTIF-003` | `POST /notify/schedule` | `PF-D.H2`, `PF-E.H8` |
| `API-NOTIF-004` | `POST /nurse-visits/{id}/confirm/send-sms` | `NF-A.H9`, `NF-A.U5`, `R-OTP-RECEIPT` |
| `API-NOTIF-010` | `POST /campaigns/sms` | `OP-D.H5`, `OP-D.U3`, `R-BULK-RETRY` |

### `crm` · CS & affiliate
| API id | method · path | serves (flow ids) |
|---|---|---|
| `API-CRM-001` | `CRUD /me/support/chat` | `PF-G.H7` |
| `API-CRM-010` | `CRUD /cs/calls` | `CS-A.H1`, `OP-B.H2`, `NF-C.U1`, `R-TRACK-FALLBACK` |
| `API-CRM-011` | `CRUD /cs/panels/{logistics|marketing|affiliate|medical|new-patient|payment}` | `CS-A.H2` |
| `API-CRM-020` | `GET /me/affiliate/dashboard` | `AF-A.H1` |
| `API-CRM-021` | `POST /me/affiliate/links` | `AF-A.H2`, `AF-A.U1`, `R-NEW-LINK` |
| `API-CRM-022` | `GET /me/affiliate/commission` | `AF-A.H4`, `AF-A.U3`, `R-CS-TICKET` |

### `rpt` · reporting/BI
| API id | method · path | serves (flow ids) |
|---|---|---|
| `API-RPT-001` | `GET /growth` | `OP-D.H4` |
| `API-RPT-002` | `GET /reports` | `OP-D.H7` |

### `gw` · gateway / admin
| API id | method · path | serves (flow ids) |
|---|---|---|
| `API-GW-010` | `GET /patients` | `OP-A.H1` |
| `API-GW-013` | `CRUD /admin/protocols` | `OP-A.H3` |
| `API-GW-015` | `CRUD /admin/api-keys` | `OP-A.H5` |
| `API-GW-021` | `GET /ops/dashboard` | `OP-B.H4`, `PH-B.H1`, `PH-B.H2`, `OP-B.U2`, `PH-B.U1`, `R-INCIDENT-TRIAGE`, `R-RECALL` |
| `API-GW-022` | `POST /ops/reschedule` | `OP-B.H5`, `R-RESCHEDULE` |
| `API-GW-023` | `CRUD /admin/settings` | `OP-D.H6` |

## Part B — Microfrontends

### Marketing / landing — public (`/`)
| path | screen | serves (flow ids) |
|---|---|---|
| `/` | CMP-MKT-001 | `MF-A.H1` |
| `/bmi` | CMP-MKT-002 | `MF-A.H2`, `MF-A.U1` |
| `/eligibility-check` | CMP-MKT-003 | `MF-A.H3`, `MF-A.U2`, `MF-A.U3`, `R-GUIDANCE-CTA`, `R-PREVIEW-RETRY` |

### Patient app — authenticated, root paths
| path | screen | serves (flow ids) |
|---|---|---|
| `/home` | CMP-PAT-000 | `PF-B.H10` |
| `/signup` | CMP-PAT-001 | `PF-A.H1`, `PF-A.H2`, `PF-A.U1`, `R-RESEND-OTP` |
| `/login` | CMP-PAT-002 | `DF-B.H1`, `PF-A.H3`, `PF-A.U2`, `PF-A.U3`, `R-COOLDOWN` |
| `/reset-password` | CMP-PAT-003 | `PF-A.H4` |
| `/onboarding/kyc` | CMP-PAT-004 | `PF-A.H5`, `PF-A.U4`, `R-KYC-REVIEW` |
| `/onboarding/profile` | CMP-PAT-010 | `PF-B.H1` |
| `/onboarding/health` | CMP-PAT-011 | `PF-B.H2` |
| `/onboarding/history` | CMP-PAT-012 | `PF-B.H3` |
| `/onboarding/labs` | CMP-PAT-013 | `PF-B.H4`, `PF-B.U1` |
| `/onboarding/address` | CMP-PAT-014 | `PF-B.H5` |
| `/eligibility` | CMP-PAT-020 | `PF-B.H6` |
| `/eligibility/result` | CMP-PAT-021 | `PF-B.H7`, `PF-B.U2`, `PF-B.U3`, `R-ELIG-REVIEW` |
| `/consent` | CMP-PAT-022 | `PF-B.H8` |
| `/plans` | CMP-PAT-030 | `PF-C.H1`, `PF-D.U3`, `R-SUBSCRIBE-GATE` |
| `/checkout` | CMP-PAT-031 | `PF-C.H2`, `PF-C.U1`, `PF-D.U3`, `R-PAY-RETRY`, `R-SUBSCRIBE-GATE` |
| `/subscription` | CMP-PAT-032 | `PF-C.H3`, `PF-C.U2`, `PF-C.U3`, `R-DUNNING`, `R-REACTIVATE` |
| `/subscription/cancel` | CMP-PAT-034 | `PF-C.H4` |
| `/wallet` | CMP-PAT-033 | `PF-C.H5` |
| `/visits/book` | CMP-PAT-040 | `PF-D.H1` |
| `— (banner overlay)` | CMP-PAT-041 | `PF-D.H2` |
| `/visits/[visitId]` | CMP-PAT-042 | `PF-D.H4` |
| `/visits/emergency` | CMP-PAT-043 | `PF-D.H5` |
| `/nurse-visits` | CMP-PAT-050 | `PF-E.H1` |
| `/nurse-visits/[id]/verify` | CMP-PAT-051 | `PF-E.H2` |
| `/nurse-visits/[id]/confirm-receipt` | CMP-PAT-052 | `PF-E.H4` |
| `/nurse-visits/[id]/reschedule` | CMP-PAT-053 | `PF-E.H3`, `PF-E.U1` |
| `/nurse-visits/schedule` | CMP-PAT-069 | `PF-E.H9` |
| `/nurse-visits/[id]/receipt` | CMP-PAT-054 | `PF-E.H5` |
| `/injections` | CMP-PAT-055 | `PF-E.H7` |
| `/side-effects/report` | CMP-PAT-056 | `PF-F.H1`, `PF-F.U1`, `R-CLINICAL-ESC` |
| `/nurse-visits/[id]/track` | CMP-PAT-057 | `NF-C.H1`, `PF-E.H6`, `PF-E.U2`, `R-TRACK-FALLBACK` |
| `/progress` | CMP-PAT-060 | `PF-G.H1` |
| `/progress/weight-goal` | CMP-PAT-061 | `PF-G.H2` |
| `/settings/reminders` | CMP-PAT-062 | `PF-G.H3` |
| `/content  ·  /content/[slug]` | CMP-PAT-063 | `PF-G.H4` |
| `/foodnoise` | CMP-PAT-064 | `PF-G.H5` |
| `/settings/addresses` | CMP-PAT-065 | `PF-G.H6` |
| `/support` | CMP-PAT-066 | `PF-G.H7` |
| `/visits/[visitId]/chat` | CMP-PAT-067 | `PF-G.H8` |
| `/perks` | CMP-PAT-068 | `PF-G.H9`, `PF-C.U4`, `R-REFUND` |

### Doctor panel (`/doctor`)
| path | screen | serves (flow ids) |
|---|---|---|
| `/doctor/patients` | CMP-DOC-001 | `DF-A.H1` |
| `/doctor/patients/[id]` | CMP-DOC-002 | `DF-A.H2`, `DF-A.U1`, `R-COMPLETE-RECORD` |
| `/doctor/visits/[id]/console` | CMP-DOC-003 | `DF-A.H4` |
| `/doctor/visits/[id]/prescription` | CMP-DOC-004 | `DF-A.H5`, `DF-A.H6`, `DF-A.H8`, `DF-B.H6`, `DF-A.U2`, `DF-A.U3`, `PF-D.U2`, `R-RXFIX` |
| `/doctor/visits/[id]/next` | CMP-DOC-005 | `DF-A.H7` |
| `/doctor` | CMP-DOC-010 | `DF-B.H2`, `DF-B.H5` |
| `/doctor/patients/[id]/manage` | CMP-DOC-011 | `DF-B.H3`, `DF-B.H5` |
| `/doctor/schedule` | CMP-DOC-012 | `DF-B.H4` |
| `/doctor/finance` | CMP-DOC-013 | `DF-B.H7` |
| `/doctor/login` | CMP-DOC-000 | `DF-B.H1`, `PF-A.H3`, `PF-A.U2`, `PF-A.U3`, `R-COOLDOWN` |
| `/doctor/visits` | CMP-DOC-014 | `DF-B.H2`, `DF-B.H3`, `DF-B.H5` |
| `/doctor/prescriptions` | CMP-DOC-015 | `DF-A.H5`, `DF-A.H6`, `DF-A.H8`, `DF-B.H6`, `DF-A.U2`, `DF-A.U3`, `PF-D.U2`, `R-RXFIX` |

### Nurse app (`/nurse`)
| path | screen | serves (flow ids) |
|---|---|---|
| `/nurse/login` | CMP-NUR-000 | `NF-A.H1` |
| `/nurse` | CMP-NUR-001 | `NF-A.H2` |
| `/nurse/route` (range: today / tomorrow / week) | CMP-NUR-002 | `NF-A.H3` |
| `/nurse/visits/[id]/checkin` | CMP-NUR-003 | `NF-A.H4`, `NF-A.U1`, `NF-A.U6`, `R-GPS-OVERRIDE`, `R-OFFLINE-CAPTURE` |
| `/nurse/visits/[id]` | CMP-NUR-004 | `NF-A.H5` |
| `/nurse/visits/[id]/cold-chain` | CMP-NUR-005 | `NF-A.H6`, `OP-C.H2`, `NF-A.U3`, `R-REPLACE` |
| `/nurse/visits/[id]/assessment` | CMP-NUR-006 | `NF-A.H7` |
| `/nurse/visits/[id]/injection` | CMP-NUR-007 | `NF-A.H8`, `NF-A.U4`, `R-VIAL-RESCAN` |
| `/nurse/visits/[id]/confirm` | CMP-NUR-008 | `NF-A.H9`, `NF-A.U5`, `R-OTP-RECEIPT` |
| `/nurse/visits/[id]/submit` | CMP-NUR-009 | `NF-A.H10`, `NF-A.U6`, `R-OFFLINE-CAPTURE` |
| `/nurse/safety` | CMP-NUR-010 | `CS-A.H4`, `NF-B.H1`, `NF-B.H2`, `NF-B.H3`, `NF-B.U1`, `R-SAFETY-ESC` |
| `/nurse/route/map` | CMP-NUR-020 | `CS-A.H1`, `NF-C.H1`, `OP-B.H2`, `PF-E.H6`, `NF-C.U1`, `PF-E.U2`, `R-TRACK-FALLBACK` |
| `— (push notification)` | CMP-NUR-023 | `OP-B.H3` |
| `/nurse/patients` | CMP-NUR-021 | `NF-C.H2` |
| `/nurse/inventory` | CMP-NUR-022 | `NF-C.H3`, `NF-C.U2`, `R-STOCKOUT` |
| `/nurse/route/reorder` | CMP-NUR-024 | `NF-C.H4`, `R-RESCHEDULE` |
| `/nurse/summary` | CMP-NUR-025 | `NF-C.H6` |
| `/nurse/inbox` | CMP-NUR-026 | `NF-C.H7` |
| `/nurse/visits/[id]/education` | CMP-NUR-027 | `NF-C.H5` |
| `/nurse/finance` | CMP-NUR-030 | `NF-C.H8` |
| `/nurse/performance` | CMP-NUR-031 | `NF-C.H9` |
| `/nurse/hotline` | CMP-NUR-032 | `NF-C.H9` |

### Pharmacy-hub panel (`/pharmacy`)
| path | screen | serves (flow ids) |
|---|---|---|
| `/pharmacy` | CMP-PHM-001 | `PH-A.H2` |
| `/pharmacy/receipts` | CMP-PHM-002 | `PH-A.H3` |
| `/pharmacy/cold-chain` | CMP-PHM-004 | `PH-A.H4`, `PH-B.H1`, `PH-A.U2`, `R-HUB-QUARANTINE` |
| `/pharmacy/dispense` | CMP-PHM-003 | `PH-A.H5`, `PH-A.U3`, `PH-A.U4`, `R-MANUAL-ENTRY`, `R-REPLACE`, `R-STOCKOUT` |
| `/pharmacy/damage` | CMP-PHM-005 | `PH-A.H6`, `PH-A.U5`, `R-EXPIRE-QUARANTINE`, `R-HUB-QUARANTINE`, `R-REPLACE` |
| `/pharmacy/audit` | CMP-PHM-006 | `PH-A.H7` |
| `/pharmacy/commission` | CMP-PHM-010 | `PH-A.H8` |

### Admin / ops panel (`/admin`)
| path | screen | serves (flow ids) |
|---|---|---|
| `/admin/patients` | CMP-ADM-001 | `OP-A.H1` |
| `/admin/patients/[id]` | CMP-ADM-002 | `OP-A.H2` |
| `/admin/protocols` | CMP-ADM-003 | `OP-A.H3` |
| `/admin/roles` | CMP-ADM-004 | `OP-A.H4`, `OP-A.U1`, `R-ROLE-FIX` |
| `/admin/api-keys` | CMP-ADM-005 | `OP-A.H5` |
| `/admin/refunds` | CMP-ADM-006 | `OP-D.H1`, `PF-C.U4`, `R-REFUND` |
| `/admin/providers/onboard` | CMP-ADM-010 | `OP-A.H6` |
| `/admin/providers/[id]/verify` | CMP-ADM-011 | `OP-A.H7` |
| `/admin/assignments` | CMP-ADM-012 | `OP-A.H8` |
| `/admin/replacements` | CMP-ADM-041 | `NF-A.H6`, `OP-C.H2`, `NF-A.U3`, `R-REPLACE` |
| `/admin/recalls` | CMP-ADM-042 | `OP-B.H4`, `PH-A.H4`, `PH-B.H1`, `PH-B.H2`, `OP-B.U2`, `PH-A.U2`, `PH-B.U1`, `R-HUB-QUARANTINE`, `R-INCIDENT-TRIAGE`, `R-RECALL` |
| `/admin/settlements` | CMP-ADM-020 | `OP-D.H2`, `OP-D.U2`, `R-SETTLE-RETRY` |
| `/admin/users` | CMP-ADM-030 | `OP-A.H9` |
| `/admin/ops` | CMP-ADM-031 | `OP-B.H4`, `PH-B.H1`, `PH-B.H2`, `OP-B.U2`, `PH-B.U1`, `R-INCIDENT-TRIAGE`, `R-RECALL` |
| `/admin/ops/reschedule` | CMP-ADM-032 | `OP-B.H5`, `R-RESCHEDULE` |
| `/admin/finance` | CMP-ADM-033 | `OP-D.H3`, `OP-D.U1`, `R-CHARGEBACK` |
| `/admin/growth` | CMP-ADM-034 | `OP-D.H4` |
| `/admin/campaigns` | CMP-ADM-035 | `OP-D.H5`, `OP-D.U3`, `R-BULK-RETRY` |
| `/admin/settings` | CMP-ADM-036 | `OP-D.H6` |
| `/admin/reports` | CMP-ADM-037 | `OP-D.H7` |
| `/admin/affiliates` | CMP-ADM-040 | `AF-A.H3`, `AF-A.U2` |

### Customer-service panel (`/cs`)
| path | screen | serves (flow ids) |
|---|---|---|
| `/cs` | CMP-CS-001 | `CS-A.H1`, `OP-B.H2`, `NF-C.U1`, `R-TRACK-FALLBACK` |
| `/cs/logistics` | CMP-CS-002 | `CS-A.H2` |
| `/cs/marketing` | CMP-CS-003 | `CS-A.H2` |
| `/cs/affiliate` | CMP-CS-004 | `CS-A.H2` |
| `/cs/medical` | CMP-CS-005 | `CS-A.H2` |
| `/cs/new-patient` | CMP-CS-006 | `CS-A.H2` |
| `/cs/payment` | CMP-CS-007 | `CS-A.H2` |
| `/cs/dashboard` | CMP-CS-008 | `CS-A.H3` |

### Affiliate panel (`/affiliate`)
| path | screen | serves (flow ids) |
|---|---|---|
| `/affiliate` | CMP-AFF-001 | `AF-A.H1` |
| `/affiliate/links` | CMP-AFF-002 | `AF-A.H2`, `AF-A.U1`, `R-NEW-LINK` |
| `/affiliate/commission` | CMP-AFF-003 | `AF-A.H4`, `AF-A.U3`, `R-CS-TICKET` |
| `/affiliate/codes` | CMP-AFF-004 | `AF-A.H3`, `AF-A.U2` |

### Shared (shell overlays)
| path | screen | serves (flow ids) |
|---|---|---|
| `— (drawer overlay)` | CMP-SHL-005 | `PF-B.H9`, `PF-E.H8` |

---
*Generated from build spec, site map, feature map.*