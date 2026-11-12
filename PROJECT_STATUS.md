# Solar MobileApp — Consolidated Project Status

**As of:** 2026-11-12  
**Repos:** Solar-Docs / Solar-DB / Solar-API / Solar-UI  
**Architecture:** Flutter UI → Node.js/Express thin API → PostgreSQL `solar_app`  
**Evidence basis:** authored SQL/API/Flutter source, SQLScripts ITs, npm/flutter test reports, Solar-local-backups deploy/E2E packages. Features are **not** marked complete merely because a table, function file, or screen stub exists.

**Related:** [BRD.md](BRD.md) · [DB_AUDIT_REPORT.md](DB_AUDIT_REPORT.md) · [M8_CLOSEOUT.md](M8_CLOSEOUT.md)

---

## Overall completion estimate

| Metric | Value |
|---|---|
| **Overall estimate** | **~58%** |
| Method | Equal weight across 13 modules. Each module scored 0–100 using four evidence legs (SQL 25%, API 25%, Flutter 25%, Tests+Deploy evidence 25%). Module score = sum of legs. Overall = arithmetic mean of module scores. |
| Confidence | Medium — M8 has strong live E2E proof; several modules have code without formal SolarDB/API deploy reports. |

### Score legend (per leg)

| Score | Meaning |
|---|---|
| 0–15 | Absent or schema-only |
| 16–40 | Authored / partial path; not end-to-end |
| 41–70 | Usable slice; gaps vs BRD or weak deploy proof |
| 71–100 | Strong implementation + tests; deploy/E2E where required |

---

## Module-wise status

### M1 Authentication / Profile — **~70%**

| Area | Status |
|---|---|
| Completed | PIN register / login / change-PIN; profile get/update; session via `log_user_sessions` (JWT hash); Flutter login + account screens |
| Partial | BRD OTP/SMS/auto-read/biometric not implemented (PIN used instead); full BRD profile preferences unverified |
| Pending | OTP auth, OTP lockout flows, biometric quick login |
| SQL | `fn_create_pin_user`, `fn_create_pin_session`, `fn_update_user_pin`, `fn_get/update_user_profile` present |
| API | `POST /api/auth/register\|login\|change-pin`, `POST /api/account`, `/update` |
| Flutter | Login, change PIN, account |
| Tests | `auth.test.js`, `account.test.js`, Flutter auth/account tests; M8 E2E used live login successfully |
| Deploy | Live in M8 API environment (indirect); dedicated auth deploy package not separately archived |
| Decisions | **OTP vs PIN** product decision unresolved vs BRD |

### M2 Calculator / Visit Charging — **~25%**

| Area | Status |
|---|---|
| Completed | Visit charge quoting SQL `fn_get_site_visit_charge` used from service booking |
| Partial | Charging embedded in M5 booking only — not a Module-2 product surface |
| Pending | Feasibility calculator UI/API, subsidy engine, PDF quote, lead capture from calculator |
| SQL | Charge fn present; calculator/lead/quote `fn_*` absent (tables exist in schema) |
| API | No calculator routes |
| Flutter | No calculator feature folder |
| Tests | DB rules test for visit charge; results not re-run this closeout |
| Deploy | Charge/AMC helpers implied live from prior M7 preserve notes |
| Decisions | Calculator parameters / subsidy versioning gaps noted in DB audit |

### M3 Site KYC — **~75%**

| Area | Status |
|---|---|
| Completed | KYC get/create/update SQL; API + media; customer KYC + admin review screens; image/PDF handling |
| Partial | Create-path notification comments may be stale; full BRD vault completeness not separately E2E’d |
| Pending | Deeper BRD edge cases; formal KYC-only live deploy report |
| SQL | `fn_get/create/update_kyc_document_details` |
| API | `POST /api/kyc`, `/create`, `/update` + `/media/kyc` |
| Flutter | `SiteKycScreen`, `AdminKycReviewScreen`, preview/compressor |
| Tests | API + Flutter KYC suites; DB rules test present |
| Deploy | Present in running API tree; dedicated KYC deploy archive not found |

### M4 Assets / Installation — **~45%**

| Area | Status |
|---|---|
| Completed | Site list/update via installations API; Flutter installation list with KYC/AMC hooks |
| Partial | Equipment mainly via Path B wizard, not standalone CRUD; some dashboard copy still “coming soon” |
| Pending | Barcode/QR vault UX, warranty vault UX, equipment/order standalone APIs |
| SQL | `fn_get/update_site_details`; equipment/order write fns largely absent outside Path B create |
| API | `/api/installations`, `/update` — no `/api/equipment` |
| Flutter | Installation screens yes |
| Tests | `installations.test.js`, installation Flutter tests |
| Deploy | Unverified formal package |

### M5 AMC / Preventive Service — **~60%**

| Area | Status |
|---|---|
| Completed | Service request list/create/update/complete; coverage API; visit charge integration; AMC **get** |
| Partial | No authored AMC create/update admin APIs; admin “service information” BRD slice incomplete |
| Pending | AMC renewal UX, signed service report downloads, scheduler-driven free-service lapse automation |
| SQL | Service + visit update + charge + `fn_get_amc_contract_details`; AMC write fns absent |
| API | `/api/service-requests*`, `/api/service-coverage` |
| Flutter | `service/`, `coverage/` |
| Tests | API + Flutter + DB booking/AMC tests present |
| Deploy | Implied live with AMC helpers in prior DB preserve notes |

### M6 Complaints — **~5%**

| Area | Status |
|---|---|
| Completed | Schema tables only |
| Partial | Dashboard “Complaints” → coming soon |
| Pending | Full BRD complaint lifecycle (raise, media, status, admin console, OTP resolve) |
| SQL | No complaint `fn_*` authored |
| API | None |
| Flutter | None |
| Tests | None |
| Deploy | N/A |

### M7 Technician — **~85%**

| Area | Status |
|---|---|
| Completed | Technician CRUD/list + job assign/list/update SQL; admin + technician APIs; Flutter admin hub + job screens; SolarDB install evidence in M7 backups |
| Partial | Map/route optimization not evidenced; no separate M7 API/Flutter E2E closeout package like M8 |
| Pending | Complaint-linked specialty jobs (depends on M6); map optimization |
| SQL | Six technician/job functions; SolarDB preserve lists them live |
| API | `/api/admin/technicians*`, `/api/technician/jobs*` + `/media/jobs` |
| Flutter | `technician/` feature |
| Tests | API technicians/jobs tests; Flutter technician tests; restore ITs `12_`/`13_`/`14_technician` / live `15a`/`15b` scripts exist |
| Deploy | DB: `Solar-DB/backups/SolarDB_pre_m7_deploy_*` + preserve notes. API/Flutter formal deploy report: not found |

### M8 Offline Billing — **~92%**

| Area | Status |
|---|---|
| Completed | Six invoice/payment SQL **on SolarDB**; API deployed; Flutter billing wired; restore ITs passed; API 147; Flutter 156; live E2E 23/23; fixtures cleaned |
| Partial | Android/Windows native builds failed; Android device verification pending |
| Pending | Native Android verification; any BRD extras beyond offline register |
| SQL | Six fns deployed (OIDs 41176–41186) |
| API | Invoices + payments customer/admin routes live |
| Flutter | Admin + customer billing screens |
| Tests | See [M8_CLOSEOUT.md](M8_CLOSEOUT.md) |
| Deploy | Full package under `Solar-local-backups/m8_*` |
| Decisions | Offline-only (no Pay-now) — aligned with BRD/domain rules |

### M9 Dropdowns — **~80%**

| Area | Status |
|---|---|
| Completed | `fn_get_dropdown_data` + `POST /api/dropdowns`; Flutter cascading address/dropdowns; injection guards |
| Partial | Admin UI to maintain reference masters not evidenced (seed/reference-data pipeline instead) |
| Pending | Admin master-data maintenance screens if required by BRD ops |
| SQL | Dropdown fn present (logs to `log_sql_functions` without shared `fn_log_function_execution`) |
| API | Yes |
| Flutter | Shared + address selection |
| Tests | API dropdown cases; Flutter dropdown/address tests; SQL `05_test_fn_get_dropdown_data.sql` |
| Deploy | Foundational; used by live API |

### M10 Logging / Sessions — **~55%**

| Area | Status |
|---|---|
| Completed | Log tables present; API writes `log_api_execution`; login sessions hashed JWT; M8 probes confirmed audit rows |
| Partial | Shared `fn_log_function_execution` **absent**; ad-hoc SQL logging; no admin audit viewer |
| Pending | OTP attempt flows; push delivery logging end-to-end; centralized redaction audit |
| SQL | Tables yes; `fn_log_*` absent |
| API | Embedded in routes |
| Flutter | N/A |
| Tests | Indirect via route tests / M8 verify |
| Deploy | Tables live (M8 audit evidence) |

### M11 Catalogue / Enquiry — **~55%**

| Area | Status |
|---|---|
| Completed | Published product browse; enquiry create/list/action; Flutter catalogue + enquiry list |
| Partial | Admin catalogue CRUD/publish missing; some SQL comments on notifications may be stale |
| Pending | Admin product master/media/publish; `NTF_CATALOGUE_NEW` on publish |
| SQL | `fn_get_product_details`, enquiry create/get/update; admin product write fns absent |
| API | `/api/catalogue/*` customer paths |
| Flutter | Catalogue + enquiry |
| Tests | `catalogue.test.js`, Flutter catalogue tests |
| Deploy | Present in running API; formal archive not found |
| Decisions | **`admin_registered` / Path A catalogue visibility conflict** — do not invent resolution (BRD + DB audit + `.cursorrules`) |

### M12 Admin / Path B — **~50%**

| Area | Status |
|---|---|
| Completed | Path B `fn_create_user_details` + `POST /api/admin/customers/create` + multi-step Flutter wizard |
| Partial | Not a full BRD back-office console; admin surfaces fragmented (KYC, technicians, billing) |
| Pending | Catalogue admin, complaints admin, lead queues, generic customer list, many 12.x items |
| SQL | Path B create present |
| API | Admin customer create + other module admin routes |
| Flutter | Registration wizard + module-specific admin hubs |
| Tests | `adminCustomers.test.js`, wizard tests; Path B SQL ITs `10_`/`11_` |
| Deploy | Pre-dump for `fn_create_user_details` exists; full console deploy N/A |

### M13 Notifications — **~55%**

| Area | Status |
|---|---|
| Completed | Inbox get/update; create helper used by billing/jobs/Path B/KYC update; Flutter inbox |
| Partial | Push delivery (FCM/APNs) not evidenced; event matrix incomplete vs BRD (no complaint/catalogue-publish emitters) |
| Pending | Push pipeline; deep links for all entities; full event catalogue |
| SQL | `fn_create/get/update_notification_details` |
| API | `POST /api/notifications`, `/update` |
| Flutter | Notification inbox |
| Tests | API + Flutter inbox tests (analyze warnings in notification test helpers); DB rules test |
| Deploy | Live with M8 API process; helper MD5 stable across M7/M8 |

---

## Score table

| Module | SQL | API | Flutter | Tests+Deploy | **Module %** |
|---|---:|---:|---:|---:|---:|
| M1 Auth/Profile | 75 | 80 | 75 | 50 | **70** |
| M2 Calculator/Visit | 40 | 10 | 5 | 45 | **25** |
| M3 KYC | 80 | 80 | 80 | 60 | **75** |
| M4 Assets/Install | 45 | 50 | 55 | 30 | **45** |
| M5 AMC/Service | 65 | 70 | 65 | 40 | **60** |
| M6 Complaints | 10 | 0 | 0 | 0 | **5** |
| M7 Technician | 90 | 85 | 85 | 80 | **85** |
| M8 Billing | 95 | 95 | 90 | 90 | **92** |
| M9 Dropdowns | 85 | 85 | 80 | 70 | **80** |
| M10 Logging | 50 | 60 | 10 | 55 | **55** *(Flutter N/A → scored low intentionally; backend-weighted)* |
| M11 Catalogue | 55 | 60 | 65 | 40 | **55** |
| M12 Admin/Path B | 55 | 55 | 55 | 35 | **50** |
| M13 Notifications | 70 | 60 | 60 | 45 | **55** |
| **Overall mean** |  |  |  |  | **~58** |

*(M10 Flutter leg reflects “no dedicated UI required”; alternative backend-only reweight would raise M10 ~65 and overall ~59. Reported overall uses the stated four-leg method.)*

---

## Pending tasks (priority order)

1. **Resolve product decisions:** OTP vs PIN (M1); `admin_registered` / Path A catalogue visibility (M11/M12).
2. **Android toolchain:** install NDK `28.2.13676358`; enable Windows Developer Mode if Windows builds needed; obtain emulator or device; run Android native M8 verification.
3. **M6 Complaints** end-to-end (SQL → API → Flutter) — largest functional gap.
4. **M2 Calculator** (if in scope) or explicitly defer with BRD note.
5. **Shared `fn_log_function_execution`** + standardize SQL audit logging (M10).
6. **Admin catalogue CRUD/publish** (M11) and AMC create/update (M5).
7. **Push notification delivery** pipeline (M13).
8. **M7 API/Flutter formal deploy + E2E package** (parity with M8 closeout rigor).
9. **Standalone equipment/order admin APIs** (M4) beyond Path B create.
10. **Refresh generated `DATABASE_OBJECTS_INDEX.md`** after approved export (currently stamped 2026-10-10).

---

## Blockers and unresolved decisions

| ID | Item | Impact |
|---|---|---|
| D1 | OTP (BRD) vs PIN (implemented) | M1 compliance / onboarding |
| D2 | `admin_registered` vs Path A catalogue browse | M11/M12 authorization |
| B1 | Android NDK missing | APK build / device verify |
| B2 | Windows symlink / Developer Mode | Windows desktop build |
| B3 | No Android device/emulator in last verify | Native mobile E2E |
| G1 | Complaints module not started in app layer | M6 |
| G2 | Calculator app layer not started | M2 |
| G3 | `fn_log_function_execution` missing | M9/M10 consistency |

---

## Release-readiness

| Track | Status |
|---|---|
| **Local SolarDB + API offline billing (M8)** | **Ready for continued internal use** on `127.0.0.1` with documented rollback |
| **M7 technician (DB)** | Functions present on SolarDB per M7 preserve; API/UI code present; formal E2E package weaker than M8 |
| **Play Store / production mobile release** | **Not ready** — Android build blocked; native device verification pending; OTP/catalogue decisions open; M6/M2 gaps |
| **Full BRD feature release** | **Not ready** — overall ~58%; multiple modules partial/pending |

### Do / do-not for release claims

- **Do** claim M8 SQL+API+live-API E2E verified on Windows host against SolarDB.
- **Do not** claim Android APK build passed.
- **Do not** claim Android emulator/device verification passed.
- **Do not** claim production deploy.

---

## Evidence index (non-exhaustive)

- M8 closeout: [M8_CLOSEOUT.md](M8_CLOSEOUT.md)
- M8 packages: `Solar-local-backups/m8_pre_deploy_20261112_163433/`, `m8_api_deploy_20261112_164340/`, `m8_flutter_e2e_20261112/`
- M7 DB backups: `Solar-DB/backups/SolarDB_pre_m7_deploy_*`, `m7_preserve_after.txt`
- Authored SQL: `Solar-DB/DB-Objects/SQLFunctions/`
- API mounts: `Solar-API/src/app.js`
- Flutter features: `Solar-UI/lib/features/`
