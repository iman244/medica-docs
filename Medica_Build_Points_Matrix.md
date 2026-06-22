# Medica — Build Effort Matrix

Build effort laid out as **months (rows) × user-flows (columns)**. **1 point = 1 build step (`STEP-`) = 2 hours.** The step is the spec's own unit of work ("Use the STEP_ID as the unit of work in tickets and tests", Build spec §6). Each cell shows **hours** (points × 2); the last row/column are aggregates. Below the matrix, every populated cell is broken down into the exact `F-` (feature), `STEP-` (step), `API-` (endpoint), and `CMP-` (component) ids it consists of.

Total scope: **123 points = 246 hours** across 14 cells (≈ 31 working days at 8h, or ~6 weeks for one engineer). Source: `Medica_Build_and_Technical_Specification.md`.

## Column legend (user-flows)

| code | user-flow |
|---|---|
| **A** | Onboarding & Identity (auth, KYC, public landing) |
| **B** | Eligibility & Profile |
| **C** | Subscription & Payment |
| **D** | Online Visit & Doctor |
| **E** | Nurse Home Visit |
| **F** | Pharmacy & Logistics (routing, cold-chain) |
| **G** | Patient Self-Service (tracking, engagement, content) |
| **H** | Admin / Ops & Finance / BI |
| **I** | Safety Recovery (cold-chain replacement, batch recall) |
| **J** | Customer Service & Affiliate |

## The matrix (hours per cell)

Cells are **hours** = build steps × 2. Point counts are in the per-cell detail below.

| Month | A | B | C | D | E | F | G | H | I | J | **Total (h)** |
|---|--:|--:|--:|--:|--:|--:|--:|--:|--:|--:|--:|
| **M1** Foundation + identity | 16 | · | · | · | · | · | · | · | · | · | **16** |
| **M2** Profile + eligibility | · | 22 | · | · | · | · | · | · | · | · | **22** |
| **M3** Subscription + visit | · | · | 10 | 22 | · | · | · | · | · | · | **32** |
| **M4** Nurse + tracking + ops | · | · | · | · | 24 | · | 18 | 20 | · | · | **62** |
| **M5** Pharmacy + cold-chain | · | · | · | · | · | 34 | · | · | 6 | · | **40** |
| **M6** Doctor + finance | · | · | · | 22 | · | · | · | · | · | · | **22** |
| **M7** Patient engagement | · | · | · | · | · | · | 18 | · | · | · | **18** |
| **M8** Admin + finance/BI | · | · | · | · | · | · | · | 16 | · | · | **16** |
| **M9** CS + affiliate | · | · | · | · | · | · | · | · | · | 18 | **18** |
| **Total (h)** | **16** | **22** | **10** | **44** | **24** | **34** | **36** | **36** | **6** | **18** | **246** |

Reading it: M4 is the heaviest month (**62 h** — the nurse home-visit flow plus patient tracking and admin ops all land before the pilot). The **Online Visit & Doctor** lane (D, **44 h**) is the largest user-flow, split across M3 (visit) and M6 (doctor completion + settlements). **Safety Recovery** (I, **6 h**) is small in code because every other recovery is reused or run by hand for the pilot.

---

# Cell detail — what each point total consists of

Each section is one cell of the matrix: its `F-` features, `STEP-` steps (= the points), `API-` endpoints, and `CMP-` components.

## M1 · A — Onboarding & Identity — 8 pts · 16 h

Foundation + identity (6) and public landing (2).

- **Steps (8):** STEP-1-01, 1-02, 1-03, 1-04, 1-05, 1-07 · STEP-1-08, 1-09 (landing)
- **Features:** F-101, F-102, F-103, F-105 · F-095, F-096
- **APIs (5):** API-AUTH-001, 002, 003, 004, 007
- **Components (6):** CMP-PAT-001, 002, 003, 004 · CMP-MKT-001, 002

## M2 · B — Eligibility & Profile — 11 pts · 22 h

- **Steps (11):** STEP-2-01 → 2-11
- **Features (14):** F-110, 111, 112, 113, 115, 121, 122, 123, 124, 125, 126, 205, 097, 100
- **APIs (12):** API-PATIENT-001, 002, 003, 004, 005, 020, 021, 022, 023, 024, 025 · API-NOTIF-001
- **Components (11):** CMP-PAT-000, 010, 011, 012, 013, 014, 020, 021, 022 · CMP-SHL-005 · CMP-MKT-003

## M3 · C — Subscription & Payment — 5 pts · 10 h

- **Steps (5):** STEP-3A-01 → 3A-05
- **Features (8):** F-130, 131, 133, 139, 141, 142, 143, 146
- **APIs (6):** API-SUB-001, 002, 003, 004 · API-PAY-001, 003
- **Components (5):** CMP-PAT-030, 031, 032, 033, 034

## M3 · D — Online Visit & Doctor — 11 pts · 22 h

- **Steps (11):** STEP-3B-01 → 3B-11
- **Features (14):** F-150, 151, 152, 153, 155, 157, 159, 310, 312, 327, 328, 329, 330, 331
- **APIs (11):** API-VISIT-001, 002, 003, 004, 005, 006, 007, 008 · API-NOTIF-003 · API-PROV-001 · API-PATIENT-010
- **Components (9):** CMP-PAT-040, 041, 042, 043 · CMP-DOC-001, 002, 003, 004, 005

## M4 · E — Nurse Home Visit — 12 pts · 24 h

The pilot-critical safety sequence (see `P4_nurse_safety_sequence.bpmn`).

- **Steps (12):** STEP-4A-01 → 4A-12
- **Features (17):** F-401, 405, 407, 415, 416, 417, 418, 430, 431, 432, 433, 434, 437, 441, 442, 443, 497
- **APIs (12):** API-FIELD-001 → 011 (+ 002) · API-AUTH-003
- **Components (11):** CMP-NUR-000 → 010

## M4 · G — Patient Self-Service (tracking & reports) — 9 pts · 18 h

- **Steps (9):** STEP-4B-01 → 4B-09
- **Features (11):** F-170, 171, 172, 174, 175, 176, 177, 178, 220, 221, 222
- **APIs (8):** API-FIELD-020, 021, 022, 023, 034 · API-PROV-010 · API-NOTIF-003 · API-PATIENT-030
- **Components (9):** CMP-PAT-050, 051, 052, 053, 054, 055, 056, 057 · CMP-SHL-005

## M4 · H — Admin / Ops (minimal) — 10 pts · 20 h

- **Steps (10):** STEP-4C-01 → 4C-10
- **Features (13):** F-147, 403, 404, 601, 602, 603, 606, 609, 610, 620, 651, 655, 656
- **APIs (11):** API-GW-010, 011, 013, 014, 015 · API-PAY-007 · API-PROV-020, 021, 022 · API-PATIENT-031, 032
- **Components (10):** CMP-ADM-001 → 006, 010, 011, 012, 043

## M5 · F — Pharmacy & Logistics — 17 pts · 34 h

Smart routing + cold-chain + nurse completion.

- **Steps (17):** STEP-5-01 → 5-16, 5-20
- **Features (28):** F-172, 330, 408, 410, 420, 422, 440, 450, 451, 452, 453, 454, 460, 465, 502, 503, 504, 505, 506, 515, 701–708
- **APIs (17):** API-PHARM-002 → 007 · API-FIELD-001, 030, 031, 033, 034, 040, 041, 042 · API-PROV-023 · API-NOTIF-002 · API-VISIT-006
- **Components (17):** CMP-PHM-001 → 006 · CMP-NUR-020 → 027 · CMP-ADM-012 · CMP-PAT-057 · CMP-DOC-004

## M5 · I — Safety Recovery — 3 pts · 6 h

Cold-chain replacement + batch recall (`P5`, `P6`; full flows in `Medica_Recovery_Flows.md`, RFLOW-01/02).

- **Steps (3):** STEP-R01-04, STEP-R02-01, STEP-R02-02
- **Features (3):** F-433, F-505, F-618
- **APIs (7):** API-FIELD-050, 051, 033, 021 · API-PHARM-004, 010, 011
- **Components (3):** CMP-ADM-041, CMP-ADM-042, CMP-NUR-022

## M6 · D — Doctor completion + finance — 11 pts · 22 h

- **Steps (11):** STEP-6-00 → 6-10
- **Features (27):** F-102, 301, 305, 311, 313, 314, 315, 317, 319, 329, 330, 340, 350, 356, 358, 470, 477, 485, 486, 495, 507, 508, 512, 513, 627, 628, 629
- **APIs (13):** API-VISIT-020, 021, 022, 024, 026 · API-PATIENT-040 · API-PROV-030 · API-AUTH-002 · API-PAY-020 → 024
- **Components (13):** CMP-DOC-000, 010 → 015, 020 · CMP-NUR-030, 031, 032 · CMP-PHM-010 · CMP-ADM-020

## M7 · G — Patient engagement & content — 9 pts · 18 h

- **Steps (9):** STEP-7-01 → 7-09
- **Features (21):** F-114, 116, 117, 132, 144, 154, 156, 158, 185, 186, 188, 189, 195, 196, 197, 199, 207, 209, 210, 223, 224
- **APIs (10):** API-PATIENT-050 → 055 · API-CRM-001 · API-VISIT-025 · API-SUB-010 · API-PAY-025
- **Components (9):** CMP-PAT-060 → 068

## M8 · H — Admin completion + finance/BI — 8 pts · 16 h

- **Steps (8):** STEP-8-01 → 8-08
- **Features (23):** F-604, 605, 607, 608, 615, 616, 617, 618, 619, 625, 626, 631, 632, 633, 640, 641, 644, 652, 653, 657, 660, 662, 663
- **APIs (8):** API-GW-020, 021, 022, 023 · API-PAY-030 · API-RPT-001, 002 · API-NOTIF-010
- **Components (8):** CMP-ADM-030 → 037

## M9 · J — Customer Service & Affiliate — 9 pts · 18 h

- **Steps (9):** STEP-9-01, 9-02, 9-08 → 9-14
- **Features (16):** F-443, 700, 703, 710, 761, 770, 771, 800–805, 807, 808, 809
- **APIs (8):** API-CRM-010, 011, 012, 020 → 023 · API-FIELD-010
- **Components (9):** CMP-CS-001, 002, 008 · CMP-AFF-001 → 004 · CMP-NUR-010 · CMP-ADM-040

---

*Points count distinct build steps; feature/API/component lists are de-duplicated within each cell (an id reused across cells is counted in each cell it appears in). Generated from the Build & Technical Specification — re-run if that file changes.*
