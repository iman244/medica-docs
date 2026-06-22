# Medica — Month × Service / MFE Allocation

Tasks from `Medica_Task_Plan.md` packed at **88 points/month**. Rows = months; columns = **services** then **MFEs (micro-frontends)** then **devops**; last row/column = totals. A happy task splits: API points → service column, component points → MFE column; unhappy/recovery → the flow's surface MFE; DevOps → `devops`. **Cell detail** (which tasks make up each cell) follows the matrix.

| Month | auth | patient | prov | visit | field | pay | pharm | sub | notif | crm | rpt | gw | fe-mkt | fe-pat | fe-doc | fe-nur | fe-phm | fe-adm | fe-cs | fe-aff | fe-shl | devops | (config) | **Total** |
|---|--:|--:|--:|--:|--:|--:|--:|--:|--:|--:|--:|--:|--:|--:|--:|--:|--:|--:|--:|--:|--:|--:|--:|--:|
| **M1** | 16 | 3 | · | · | · | · | · | · | · | · | · | · | 8 | 10 | · | · | · | · | · | · | · | 51 | · | **88** |
| **M2** | · | 17 | 1 | 11 | · | 3 | · | 3 | 2 | · | · | · | · | 34 | 14 | · | · | · | · | · | 1 | · | · | **86** |
| **M3** | 3 | · | · | 6 | 20 | · | · | 4 | 2 | · | · | · | · | 26 | · | 25 | · | · | · | · | · | · | 1 | **87** |
| **M4** | · | 4 | 6 | · | 9 | 8 | · | · | 3 | · | 2 | 7 | · | 1 | · | 5 | · | 43 | · | · | · | · | · | **88** |
| **M5** | · | 3 | 2 | · | 19 | 4 | 1 | · | 4 | · | · | 4 | · | 19 | · | 20 | 4 | 7 | · | · | 1 | · | · | **88** |
| **M6** | · | 12 | 2 | 6 | · | 3 | 12 | 2 | · | 2 | · | · | · | 10 | 16 | · | 19 | 4 | · | · | · | · | · | **88** |
| **M7** | · | · | · | · | · | · | · | · | · | 10 | · | · | · | 1 | · | · | · | 2 | 6 | 11 | 27 | · | 1 | **58** |
| **Total** | **19** | **39** | **11** | **23** | **48** | **18** | **13** | **9** | **11** | **12** | **2** | **11** | **8** | **101** | **30** | **50** | **23** | **56** | **6** | **11** | **29** | **51** | **2** | **583** |

*7 months · 583 points.*

---

## Cell detail — tasks behind each cell

### M1 — 88 pts

- **auth** (16): `T-001` PF-A.H1 (3) · `T-002` PF-A.H2 (3) · `T-003` PF-A.H3 (5) · `T-004` PF-A.H4 (3) · `T-005` PF-A.H5 (2)
- **patient** (3): `T-074` MF-A.H3 (3)
- **fe-mkt** (8): `T-072` MF-A.H1 (1) · `T-073` MF-A.H2 (1) · `T-074` MF-A.H3 (1) · `T-075` MF-A.U1 (1) · `T-076` MF-A.U2 (1) · `T-077` MF-A.U3 (1) · `T-078` R-GUIDANCE-CTA (1) · `T-079` R-PREVIEW-RETRY (1)
- **fe-pat** (10): `T-001` PF-A.H1 (2) · `T-002` PF-A.H2 (2) · `T-003` PF-A.H3 (4) · `T-004` PF-A.H4 (1) · `T-005` PF-A.H5 (1)
- **devops** (51): `T-233` DVO-K8S-DEV (4) · `T-234` DVO-K8S-PROD (6) · `T-235` DVO-CHARTS (6) · `T-236` DVO-INGRESS (3) · `T-237` DVO-SECRETS (3) · `T-238` DVO-CI (4) · `T-239` DVO-CD (3) · `T-240` DVO-ARGOCD (5) · `T-241` DVO-METRICS (4) · `T-242` DVO-GRAFANA (4) · `T-243` DVO-OTEL (3) · `T-244` DVO-ALERTS (3) · `T-245` DVO-LOKI (3)

### M2 — 86 pts

- **patient** (17): `T-013` PF-B.H1 (2) · `T-014` PF-B.H2 (2) · `T-015` PF-B.H3 (2) · `T-016` PF-B.H4 (2) · `T-017` PF-B.H5 (2) · `T-018` PF-B.H6 (1) · `T-019` PF-B.H7 (2) · `T-020` PF-B.H8 (2) · `T-022` PF-B.H10 (1) · `T-081` DF-A.H2 (1)
- **prov** (1): `T-080` DF-A.H1 (1)
- **visit** (11): `T-083` DF-A.H4 (2) · `T-084` DF-A.H5 (3) · `T-085` DF-A.H6 (2) · `T-086` DF-A.H7 (2) · `T-087` DF-A.H8 (2)
- **pay** (3): `T-028` PF-C.H2 (3)
- **sub** (3): `T-027` PF-C.H1 (1) · `T-028` PF-C.H2 (2)
- **notif** (2): `T-021` PF-B.H9 (2)
- **fe-pat** (34): `T-006` PF-A.U1 (1) · `T-007` PF-A.U2 (1) · `T-008` PF-A.U3 (1) · `T-009` PF-A.U4 (1) · `T-010` R-RESEND-OTP (1) · `T-011` R-COOLDOWN (1) · `T-012` R-KYC-REVIEW (2) · `T-013` PF-B.H1 (2) · `T-014` PF-B.H2 (2) · `T-015` PF-B.H3 (2) · `T-016` PF-B.H4 (2) · `T-017` PF-B.H5 (2) · `T-018` PF-B.H6 (2) · `T-019` PF-B.H7 (1) · `T-020` PF-B.H8 (2) · `T-022` PF-B.H10 (1) · `T-023` PF-B.U1 (1) · `T-024` PF-B.U2 (1) · `T-025` PF-B.U3 (1) · `T-026` R-ELIG-REVIEW (3) · `T-027` PF-C.H1 (2) · `T-028` PF-C.H2 (2)
- **fe-doc** (14): `T-080` DF-A.H1 (1) · `T-081` DF-A.H2 (2) · `T-083` DF-A.H4 (2) · `T-084` DF-A.H5 (1) · `T-085` DF-A.H6 (1) · `T-086` DF-A.H7 (2) · `T-087` DF-A.H8 (1) · `T-088` DF-A.U1 (1) · `T-089` DF-A.U2 (1) · `T-090` DF-A.U3 (1) · `T-091` R-COMPLETE-RECORD (1)
- **fe-shl** (1): `T-021` PF-B.H9 (1)

### M3 — 87 pts

- **auth** (3): `T-103` NF-A.H1 (3)
- **visit** (6): `T-040` PF-D.H1 (3) · `T-043` PF-D.H4 (1) · `T-044` PF-D.H5 (2)
- **field** (20): `T-104` NF-A.H2 (2) · `T-105` NF-A.H3 (1) · `T-106` NF-A.H4 (2) · `T-107` NF-A.H5 (1) · `T-108` NF-A.H6 (2) · `T-109` NF-A.H7 (2) · `T-110` NF-A.H8 (3) · `T-111` NF-A.H9 (2) · `T-112` NF-A.H10 (2) · `T-123` NF-B.H1 (3)
- **sub** (4): `T-029` PF-C.H3 (2) · `T-030` PF-C.H4 (2)
- **notif** (2): `T-041` PF-D.H2 (2)
- **fe-pat** (26): `T-029` PF-C.H3 (2) · `T-030` PF-C.H4 (1) · `T-031` PF-C.H5 (1) · `T-032` PF-C.U1 (1) · `T-033` PF-C.U2 (1) · `T-034` PF-C.U3 (1) · `T-035` PF-C.U4 (1) · `T-036` R-PAY-RETRY (2) · `T-037` R-DUNNING (2) · `T-038` R-REACTIVATE (3) · `T-039` R-REFUND (2) · `T-040` PF-D.H1 (2) · `T-041` PF-D.H2 (1) · `T-043` PF-D.H4 (1) · `T-044` PF-D.H5 (1) · `T-045` PF-D.U1 (1) · `T-046` PF-D.U2 (1) · `T-047` PF-D.U3 (1) · `T-048` R-SUBSCRIBE-GATE (1)
- **fe-nur** (25): `T-103` NF-A.H1 (1) · `T-104` NF-A.H2 (2) · `T-105` NF-A.H3 (1) · `T-106` NF-A.H4 (1) · `T-107` NF-A.H5 (1) · `T-108` NF-A.H6 (1) · `T-109` NF-A.H7 (2) · `T-110` NF-A.H8 (1) · `T-111` NF-A.H9 (1) · `T-112` NF-A.H10 (1) · `T-113` NF-A.U1 (1) · `T-114` NF-A.U2 (1) · `T-115` NF-A.U3 (1) · `T-116` NF-A.U4 (1) · `T-117` NF-A.U5 (1) · `T-118` NF-A.U6 (1) · `T-119` R-GPS-OVERRIDE (1) · `T-120` R-VIAL-RESCAN (3) · `T-121` R-OTP-RECEIPT (1) · `T-122` R-OFFLINE-CAPTURE (1) · `T-123` NF-B.H1 (1)
- **(config)** (1): `T-042` PF-D.H3 (1)

### M4 — 88 pts

- **patient** (4): `T-182` OP-C.H1 (4)
- **prov** (6): `T-165` OP-A.H6 (2) · `T-166` OP-A.H7 (2) · `T-167` OP-A.H8 (2)
- **field** (9): `T-124` NF-B.H2 (2) · `T-125` NF-B.H3 (3) · `T-183` OP-C.H2 (3) · `T-049` PF-E.H1 (1)
- **pay** (8): `T-184` OP-D.H1 (3) · `T-185` OP-D.H2 (4) · `T-186` OP-D.H3 (1)
- **notif** (3): `T-188` OP-D.H5 (3)
- **rpt** (2): `T-187` OP-D.H4 (1) · `T-190` OP-D.H7 (1)
- **gw** (7): `T-160` OP-A.H1 (1) · `T-162` OP-A.H3 (2) · `T-164` OP-A.H5 (2) · `T-189` OP-D.H6 (2)
- **fe-pat** (1): `T-049` PF-E.H1 (1)
- **fe-nur** (5): `T-124` NF-B.H2 (1) · `T-125` NF-B.H3 (1) · `T-126` NF-B.U1 (1) · `T-127` R-SAFETY-ESC (2)
- **fe-adm** (43): `T-160` OP-A.H1 (1) · `T-161` OP-A.H2 (1) · `T-162` OP-A.H3 (2) · `T-163` OP-A.H4 (2) · `T-164` OP-A.H5 (2) · `T-165` OP-A.H6 (2) · `T-166` OP-A.H7 (1) · `T-167` OP-A.H8 (2) · `T-168` OP-A.H9 (2) · `T-169` OP-A.U1 (1) · `T-170` OP-A.U2 (1) · `T-171` R-ROLE-FIX (1) · `T-172` R-CAPACITY (2) · `T-182` OP-C.H1 (2) · `T-183` OP-C.H2 (2) · `T-184` OP-D.H1 (1) · `T-185` OP-D.H2 (1) · `T-186` OP-D.H3 (2) · `T-187` OP-D.H4 (2) · `T-188` OP-D.H5 (2) · `T-189` OP-D.H6 (2) · `T-190` OP-D.H7 (1) · `T-191` OP-D.U1 (1) · `T-192` OP-D.U2 (1) · `T-193` OP-D.U3 (1) · `T-194` R-CHARGEBACK (2) · `T-195` R-SETTLE-RETRY (2) · `T-196` R-BULK-RETRY (1)

### M5 — 88 pts

- **patient** (3): `T-059` PF-F.H1 (3)
- **prov** (2): `T-050` PF-E.H2 (1) · `T-133` NF-C.H6 (1)
- **field** (19): `T-051` PF-E.H3 (2) · `T-052` PF-E.H4 (2) · `T-053` PF-E.H5 (2) · `T-054` PF-E.H6 (1) · `T-055` PF-E.H7 (1) · `T-128` NF-C.H1 (1) · `T-129` NF-C.H2 (1) · `T-130` NF-C.H3 (3) · `T-132` NF-C.H5 (2) · `T-173` OP-B.H1 (2) · `T-175` OP-B.H3 (2)
- **pay** (4): `T-135` NF-C.H8 (1) · `T-136` NF-C.H9 (1) · `T-140` PH-A.H1 (2)
- **pharm** (1): `T-141` PH-A.H2 (1)
- **notif** (4): `T-056` PF-E.H8 (2) · `T-134` NF-C.H7 (2)
- **gw** (4): `T-176` OP-B.H4 (2) · `T-177` OP-B.H5 (2)
- **fe-pat** (19): `T-050` PF-E.H2 (1) · `T-051` PF-E.H3 (2) · `T-052` PF-E.H4 (1) · `T-053` PF-E.H5 (1) · `T-054` PF-E.H6 (3) · `T-055` PF-E.H7 (1) · `T-057` PF-E.U1 (1) · `T-058` PF-E.U2 (1) · `T-059` PF-F.H1 (1) · `T-060` PF-F.U1 (1) · `T-061` R-CLINICAL-ESC (3) · `T-128` NF-C.H1 (3)
- **fe-nur** (20): `T-128` NF-C.H1 (2) · `T-129` NF-C.H2 (1) · `T-130` NF-C.H3 (2) · `T-131` NF-C.H4 (1) · `T-132` NF-C.H5 (1) · `T-133` NF-C.H6 (1) · `T-134` NF-C.H7 (1) · `T-135` NF-C.H8 (2) · `T-136` NF-C.H9 (3) · `T-137` NF-C.U1 (1) · `T-138` NF-C.U2 (1) · `T-139` NF-C.U3 (1) · `T-174` OP-B.H2 (2) · `T-175` OP-B.H3 (1)
- **fe-phm** (4): `T-140` PH-A.H1 (2) · `T-141` PH-A.H2 (2)
- **fe-adm** (7): `T-176` OP-B.H4 (2) · `T-177` OP-B.H5 (1) · `T-178` OP-B.U1 (1) · `T-179` OP-B.U2 (1) · `T-180` R-GEOCODE-MANUAL (1) · `T-181` R-INCIDENT-TRIAGE (1)
- **fe-shl** (1): `T-056` PF-E.H8 (1)

### M6 — 88 pts

- **patient** (12): `T-094` DF-B.H3 (1) · `T-062` PF-G.H1 (1) · `T-063` PF-G.H2 (2) · `T-064` PF-G.H3 (2) · `T-065` PF-G.H4 (2) · `T-066` PF-G.H5 (2) · `T-067` PF-G.H6 (2)
- **prov** (2): `T-095` DF-B.H4 (2)
- **visit** (6): `T-093` DF-B.H2 (1) · `T-094` DF-B.H3 (1) · `T-096` DF-B.H5 (1) · `T-097` DF-B.H6 (1) · `T-069` PF-G.H8 (2)
- **pay** (3): `T-098` DF-B.H7 (1) · `T-070` PF-G.H9 (2)
- **pharm** (12): `T-142` PH-A.H3 (2) · `T-143` PH-A.H4 (1) · `T-144` PH-A.H5 (2) · `T-145` PH-A.H6 (2) · `T-146` PH-A.H7 (1) · `T-156` PH-B.H1 (3) · `T-157` PH-B.H2 (1)
- **sub** (2): `T-070` PF-G.H9 (2)
- **crm** (2): `T-068` PF-G.H7 (2)
- **fe-pat** (10): `T-062` PF-G.H1 (1) · `T-063` PF-G.H2 (1) · `T-064` PF-G.H3 (1) · `T-065` PF-G.H4 (1) · `T-066` PF-G.H5 (2) · `T-067` PF-G.H6 (1) · `T-068` PF-G.H7 (1) · `T-069` PF-G.H8 (1) · `T-070` PF-G.H9 (1)
- **fe-doc** (16): `T-092` DF-B.H1 (1) · `T-093` DF-B.H2 (2) · `T-094` DF-B.H3 (2) · `T-095` DF-B.H4 (2) · `T-096` DF-B.H5 (1) · `T-097` DF-B.H6 (1) · `T-098` DF-B.H7 (2) · `T-099` DF-B.U1 (1) · `T-100` DF-B.U2 (1) · `T-101` DF-B.U3 (1) · `T-102` R-REASSIGN (2)
- **fe-phm** (19): `T-142` PH-A.H3 (1) · `T-143` PH-A.H4 (2) · `T-144` PH-A.H5 (1) · `T-145` PH-A.H6 (1) · `T-146` PH-A.H7 (1) · `T-148` PH-A.U1 (1) · `T-149` PH-A.U2 (1) · `T-150` PH-A.U3 (1) · `T-151` PH-A.U4 (1) · `T-152` PH-A.U5 (1) · `T-153` R-HUB-QUARANTINE (2) · `T-154` R-MANUAL-ENTRY (1) · `T-155` R-EXPIRE-QUARANTINE (1) · `T-158` PH-B.U1 (1) · `T-159` R-RECALL (3)
- **fe-adm** (4): `T-156` PH-B.H1 (2) · `T-157` PH-B.H2 (2)

### M7 — 58 pts

- **crm** (10): `T-201` AF-A.H1 (1) · `T-202` AF-A.H2 (2) · `T-204` AF-A.H4 (2) · `T-197` CS-A.H1 (3) · `T-198` CS-A.H2 (2)
- **fe-pat** (1): `T-071` PF-G.U1 (1)
- **fe-adm** (2): `T-203` AF-A.H3 (2)
- **fe-cs** (6): `T-197` CS-A.H1 (2) · `T-198` CS-A.H2 (2) · `T-199` CS-A.H3 (2)
- **fe-aff** (11): `T-201` AF-A.H1 (2) · `T-202` AF-A.H2 (1) · `T-203` AF-A.H3 (1) · `T-204` AF-A.H4 (2) · `T-206` AF-A.U1 (1) · `T-207` AF-A.U2 (1) · `T-208` AF-A.U3 (1) · `T-209` R-NEW-LINK (1) · `T-210` R-CS-TICKET (1)
- **fe-shl** (27): `T-211` XF.U1 (1) · `T-212` XF.U2 (1) · `T-213` XF.U3 (1) · `T-214` XF.U4 (1) · `T-215` XF.U5 (1) · `T-216` XF.U6 (1) · `T-217` XF.U7 (1) · `T-218` R-OFFLINE-BANNER (1) · `T-219` R-REAUTH (1) · `T-220` R-NO-ACCESS (1) · `T-221` R-NOT-FOUND (1) · `T-222` R-RETRY-5XX (1) · `T-223` R-RATE-LIMIT (1) · `T-224` R-CONFLICT (1) · `T-225` R-RESCHEDULE (2) · `T-226` R-RXFIX (2) · `T-227` R-REPLACE (3) · `T-228` R-STOCKOUT (2) · `T-229` R-BLOCKED-STATE (1) · `T-230` R-INLINE-VALIDATE (1) · `T-231` R-EMPTY-STATE (1) · `T-232` R-TRACK-FALLBACK (1)
- **(config)** (1): `T-205` AF-A.H5 (1)
