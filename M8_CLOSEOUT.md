# M8 Offline Billing — Closeout

**Module:** M8 Offline Billing (invoice + payment register)  
**Closeout date:** 2026-11-12  
**Scope closed:** M8.1–M8.6 implementation, SolarDB SQL deploy, Solar-API deploy, live API E2E verification  
**Overall project tracker:** [PROJECT_STATUS.md](PROJECT_STATUS.md)

## 1. Delivered capability

Offline admin-maintained invoice and payment register (no customer Pay-now / gateway). Customers have read-only visibility of their own invoices and payments.

### SQL (SolarDB / `solar_app`) — deployed 2026-11-12

| Function | OID (SolarDB) | body_md5 |
|---|---|---|
| `fn_create_invoice_details(jsonb)` | 41176 | `ce890214360512549754273a0172994f` |
| `fn_update_invoice_details(jsonb)` | 41178 | `1cfd03fb3f1d34f9b299da54fa47bf45` |
| `fn_get_invoice_details(jsonb)` | 41180 | `69a1648b4b14ac184fa985f5e9f4310d` |
| `fn_create_payment_details(jsonb)` | 41182 | `feede7bed87e9acb4ef121f8e581783b` |
| `fn_update_payment_details(jsonb)` | 41184 | `cd748a65734a9793be6251ae77ee63c9` |
| `fn_get_payment_details(jsonb)` | 41186 | `bf61d832b53dbc463ee147ccf2520330` |

- Authored sources: `Solar-DB/DB-Objects/SQLFunctions/solar_app.fn_*invoice*_details.sql`, `solar_app.fn_*payment*_details.sql`
- Contracts: `Solar-DB/DB-Docs/SQLFunctions/`
- Registered in `Solar-DB/SQLScripts/02_functions.sql` (loader **not** executed for this deploy; files installed individually in one transaction)
- Owner: `solar_app_user`; EXECUTE privileges follow PostgreSQL defaults (`PUBLIC` + owner)
- Installed bodies matched current source (`BODY_MATCH` 6/6)

### SQL hardening rules (verified)

- Invoice status sync: unpaid / partial / paid from payment totals
- Draft payment rejected (`INV_DRAFT` → 409)
- Cancelled / inactive invoice payment rejected
- Currency locked after any `payment_details` history (including cancelled)
- Customer get excludes draft / cancelled / inactive invoices
- Partial / full payment transitions and cancel → balance recalculation
- Overpayment and concurrent-style overpay protection (409)
- Ownership / role enforcement (admin write; customer read-own; cross-customer denied)
- `NTF_BILLING_UPDATE` failure rolls back mutating work
- Existing invoice/payment integrity preserved on rejected updates

### API (Solar-API) — deployed to local SolarDB environment

| Method | Path | Role |
|---|---|---|
| `POST` | `/api/invoices` | Session list/detail |
| `POST` | `/api/admin/invoices/create` | Admin create |
| `POST` | `/api/admin/invoices/update` | Admin update |
| `POST` | `/api/payments` | Session list/detail |
| `POST` | `/api/admin/payments/create` | Admin create |
| `POST` | `/api/admin/payments/update` | Admin update |

- Sources: `src/db/invoices.js`, `src/db/payments.js`, `src/routes/invoices.js`, `src/routes/payments.js`, mounts in `src/app.js`
- Auth: Bearer → JWT hash only; client `user_id` / `role` not trusted
- Audit: `log_api_execution` with redacted payloads
- Health during deploy verify: `{"status":"UP","database":"UP"}` on `127.0.0.1:3000`

### Flutter (Solar-UI) — billing feature wired

Admin:

- `AdminBillingHubScreen`
- `InvoiceRegisterScreen`, `InvoiceFormScreen`, `InvoiceDetailScreen`
- `PaymentListScreen`, `PaymentFormScreen`, `PaymentDetailScreen`

Customer:

- `CustomerBillsScreen` (read-only invoices)
- Payment list/detail without create actions

Clients: `HttpInvoiceApi`, `HttpPaymentApi` via `ApiBaseUrl.fromEnvironment()` (`--dart-define=API_BASE_URL=...`).

## 2. Backup and rollback packages

| Package | Path |
|---|---|
| Pre-deploy SolarDB backup + rollback | `Solar-local-backups/m8_pre_deploy_20261112_163433/` |
| Full dump | `.../SolarDB_pre_m8_deploy_20261112_163433.dump` |
| SHA256 | `699062D64B9D5C8660B6D0C7FC694E5C4C5C7AC141BB6148F138F22FD6B2B055` |
| Function rollback (pre-state absent → DROP) | `.../rollback/01_rollback_m8_functions.sql` |
| Full DB restore doc | `.../rollback/02_full_database_restore.md` |
| API / Flutter rollback docs | `.../rollback/03_api_rollback.md`, `04_flutter_rollback.md` |
| SQL deploy report | `.../deploy_m8_20261112/DEPLOY_REPORT.md` |
| API deploy preserve + report | `Solar-local-backups/m8_api_deploy_20261112_164340/` |
| Flutter / live E2E package | `Solar-local-backups/m8_flutter_e2e_20261112/` |

Pre-deploy capture: all six M8 functions were **absent** on SolarDB; rollback after install is DROP (not restore of prior bodies).

## 3. Test evidence

| Suite | Result | Notes |
|---|---|---|
| Restore IT `14_test_m8_1_invoice_register_restore.sql` | **PASSED** on `SolarDB_restore_check` | Guard refuses other DBs |
| Restore IT `15_test_m8_2_payment_register_restore.sql` | **PASSED** on `SolarDB_restore_check` | Guard refuses other DBs |
| `npm test` (Solar-API) | **147 passed / 0 failed** | Includes invoices + payments |
| `npm run check` | Exit 0 | Syntax check includes invoice/payment modules |
| Flutter billing widget tests | **14 passed** | `test/features/billing/billing_screens_test.dart` |
| Full Flutter test suite | **156 passed** | |
| Live API E2E | **23/23 passed** | Flutter client contracts → live API → SolarDB |
| M8 E2E fixture cleanup | **Confirmed** | leftover users/customers/invoices/payments/notifications = 0 |
| SolarDB M8 test records after cleanup | **None** | `invoice_details` / `payment_details` counts 0 |

Live E2E covered: admin login; invoice with one line + one tax snapshot; detail; customer visibility; offline payment; status/balance sync; customer updated view; payment cancel + recalc; customer write 403; cross-customer 403 (`customer_id` spoof) / 404 (foreign `invoice_id`); draft payment + overpayment 409.

## 4. Remaining limitations (explicit)

These are **not** claimed as passed:

1. **Android APK build blocked** — missing NDK `28.2.13676358` (`flutter build apk --debug` failed).
2. **Windows desktop build blocked** — Developer Mode / symlink support required (`flutter build windows` failed).
3. **No Android emulator or physical Android device** was available during verification.
4. **Android native device verification remains pending** — live E2E used Windows host → `http://127.0.0.1:3000` only.

Documented URL guidance (when Android is available):

- Emulator: `http://10.0.2.2:3000`
- Physical device: host LAN address (example observed: `http://192.168.31.172:3000`)

## 5. Related docs

- Function contracts: `Solar-DB/DB-Docs/SQLFunctions/solar_app.fn_*invoice*_details.md`, `solar_app.fn_*payment*_details.md`
- DB deploy note: `Solar-DB/DB-Docs/M8_DEPLOYMENT.md`
- API deploy note: `Solar-API/M8_API_DEPLOYMENT.md`
- Flutter notes: `Solar-UI/README.md` (M8 section)
