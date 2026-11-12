# M6.1 — Complaints Module Requirement & Existing Implementation Assessment

**Date:** 2026-11-12  
**Updated:** 2026-11-12 — Cross-ref: [M6_2_COMPLAINTS_BUSINESS_RULES_AND_TECHNICAL_CONTRACTS.md](M6_2_COMPLAINTS_BUSINESS_RULES_AND_TECHNICAL_CONTRACTS.md) holds **U1–U18** (approved) and **OPEN-T1…T9** (documentation-closed). Docs closure ≠ seed/impl. This assessment remains a **point-in-time gap scan**; where it conflicts with M6.2, **M6.2 wins**.  
**Type:** Assessment only (no code/SQL/DB changes)  
**Related:** [BRD.md](BRD.md) § Module 6 / 12.7 / 13.2 · [M6_2_COMPLAINTS_BUSINESS_RULES_AND_TECHNICAL_CONTRACTS.md](M6_2_COMPLAINTS_BUSINESS_RULES_AND_TECHNICAL_CONTRACTS.md) · [PROJECT_STATUS.md](PROJECT_STATUS.md) (M6 ~5%) · [DB_AUDIT_REPORT.md](DB_AUDIT_REPORT.md)

---

## 1. Confirmed BRD requirements

### Documented (BRD evidence)

| Area | Source | Requirement |
|---|---|---|
| Raise ticket | BRD §6.1 | Customer/admin rapid escalation for faults / zero generation / damage |
| Categories | BRD §6.1 | Dropdown: Zero Generation / Inverter Red Light; Physical Panel Damage; Structure Loose / Water Leakage; Generation Lower Than Expected; Billing / App Connection Issue |
| Severity | BRD §6.1 | Low, Medium, High, Emergency (e.g. Sparking/Fire Risk) |
| Description + media | BRD §6.1 | Text + up to **4** photos/videos (e.g. inverter error screens) |
| Ticket number | BRD §6.1 | Auto unique ID pattern `TKT-YYYY-nnnn` |
| Lifecycle stepper | BRD §6.2 | Raised → Assigned to Engineer → Technician En-Route → On-Site Inspection → Parts Replaced → Resolved & Closed |
| Communication | BRD §6.2 | Call or chat with assigned engineer |
| Resolution OTP | BRD §6.2 | Customer provides **4-digit OTP** on-site for technician to mark Resolved (**superseded by M6.2** — no OTP; see M6.2 U5/U8/U16) |
| Technician queue | BRD §7.1 | Job queue includes emergency breakdown tickets alongside routine services |
| Admin console | BRD §12.7 | Allocate engineers, move stepper, approve spare-part requests, monitor SLA/severity (especially Emergency) |
| Tables mapped | BRD §12.7 | `complaint_ticket_details`, `ticket_status_history_details`, `spare_part_request_details`, `job_assignment_details` |
| Visit charging | BRD §2.3 / domain rules | Complaint visits are chargeable unless free entitlement applies; purpose includes complaint |
| Notifications | BRD §13.2 | `New complaint` → Admin; technician when assigned. `Complaint updates` → Customer + admin; technician if assigned (status, message, resolved/closed, spare-part decision) |
| Deep link | BRD §13.1 | `entity_type` + `entity_id` includes ticket |
| Checklist | BRD / DDL | `ref_checklist_items.job_type` may be `COMPLAINT` |

### Not documented as concrete rules (do not invent)

| Topic | Evidence of gap |
|---|---|
| **SLA hours / response / resolution timers** | BRD §12.7 says “monitor SLA/severity” but gives **no** numeric SLAs, clocks, breach actions, or tables. `ref_data.extra_json` comment allows optional “SLA hours” — **no seeded values** found for severities. |
| **Who may raise** (customer only vs admin vs both) | BRD §6.1 customer-facing; §12.7 admin manage — exact raise roles not spelled as a matrix. |
| **Resolved vs Closed** as separate statuses | BRD stepper ends at “Resolved & Closed”; schema has both `resolved_on` and `closed_on` and a single `TKT_RESOLVED` code labeled “Resolved & Closed”. |
| **Telephony / “Call”** | BRD §6.2 “Call or chat” — no dialer/VoIP integration specified. |
| **OTP delivery channel for resolution OTP** | BRD requires OTP to resolve; Module 1 OTP SMS vs in-app display for complaint OTP is **not** specified for M6. |
| **Spare-part approval workflow detail** | Mentioned in §12.7; detailed states/rules not expanded in Module 6. |
| **Escalation matrix beyond Emergency severity** | No escalation ladder, auto-reassign, or timeout rules documented. |

---

## 2. Existing DB capabilities

### Tables (authored + exported)

| Table | Evidence |
|---|---|
| `solar_app.complaint_ticket_details` | `SQLScripts/01_schema.sql`, `DB-Docs/DDL_SCHEMA.md`, `DB-Docs/Tables/solar_app.complaint_ticket_details.md`, `DB-Objects/Tables/solar_app.complaint_ticket_details.sql` (export 2026-10-10) |
| `solar_app.ticket_status_history_details` | Same sources |
| `solar_app.ticket_media_details` | Same; `CHECK (media_type IN ('PHOTO','VIDEO'))` |
| `solar_app.ticket_message_details` | Same; `is_internal` for dispatcher notes |

### Header columns (`complaint_ticket_details`)

`complaint_ticket_id`, `ticket_no` (UNIQUE), `site_id`, `customer_id`, `raised_by_user_id`, `assigned_technician_id` → `technician_details`, category/severity/status → `ref_data`, `description`, `resolution_otp_hash`, `otp_expires_on`, `resolved_on`, `closed_on`, standard audit columns (`active_ind`, `created_by/updated_by`, `created_on/updated_on`).

### Indexes / FKs

- Indexes: `idx_tkt_tech_status_created`, `idx_tkt_customer_site` (and matching history/media/message indexes).
- FKs: site, customer, raiser user, technician, category/severity/status `ref_data`.
- Related: `job_assignment_details.complaint_ticket_id` (`fk_job_tkt`), `spare_part_request_details.complaint_ticket_id` (`fk_spr_tkt`), `followup_details.complaint_ticket_id`, `site_visit_details.complaint_ticket_id` (`fk_sv_ticket` added after complaint table).

### Reference data (seeded in `reference-data/`)

| Category | Codes (from `reference-data/json/ref_data.json`) |
|---|---|
| `COMPLAINT_CATEGORIES` | `CMP_ZERO_GEN`, `CMP_PANEL_DMG`, `CMP_STRUCTURE`, `CMP_LOW_GEN`, `CMP_BILLING` |
| `COMPLAINT_SEVERITIES` | `SEV_LOW`, `SEV_MEDIUM`, `SEV_HIGH`, `SEV_EMERGENCY` |
| `TICKET_STATUSES` | `TKT_RAISED`, `TKT_ASSIGNED`, `TKT_EN_ROUTE`, `TKT_ON_SITE`, `TKT_PARTS`, `TKT_RESOLVED` |
| Visit purpose | `VISIT_COMPLAINT` |
| Notifications | `NTF_COMPLAINT_NEW`, `NTF_COMPLAINT_UPDATE` |
| Dropdown config | `COMPLAINT_CATEGORIES`, `COMPLAINT_SEVERITIES`, `TICKET_STATUSES` in `ref_dropdown_config` |

### Relationships to other modules

| Link | Status |
|---|---|
| Customer / site | Required FKs on ticket header |
| Technician | Optional `assigned_technician_id` → `technician_details` |
| Job | `job_assignment_details` supports `job_type = 'COMPLAINT'` + required `complaint_ticket_id` (table CHECK) |
| Site visit | Optional `complaint_ticket_id`; purpose `VISIT_COMPLAINT` |
| Service request | **No direct FK** ticket ↔ SR (parallel workflows) |
| Spare parts | Optional `complaint_ticket_id` on `spare_part_request_details` |
| Notifications | Types seeded; enqueue via `fn_create_notification_details` (exists) |
| Sessions / audit | Standard JWT-hash session pattern + `log_api_execution` / `log_sql_functions` expected for future fns; OTP hash fields on ticket (not `log_otp_attempts` unless product decides otherwise) |

### DB audit notes (historical)

`Solar-Docs/DB_AUDIT_REPORT.md` § Service/complaints/jobs: ticket model appropriate; earlier note that some indexes were missing — **current DDL includes** tech/status and customer/site indexes.

---

## 3. Missing SQL functions

**No authored complaint `fn_*` files** under `DB-Objects/SQLFunctions/` or contracts under `DB-Docs/SQLFunctions/`.  
**None registered** in `SQLScripts/02_functions.sql`.

Expected application functions (by project naming convention — **not yet specified as approved contracts**):

| Likely function | Purpose |
|---|---|
| `fn_create_complaint_ticket_details` | Raise ticket, ticket_no, history row, media, `NTF_COMPLAINT_NEW` |
| `fn_update_complaint_ticket_details` | Status stepper, assign technician, OTP resolve, timestamps |
| `fn_get_complaint_ticket_details` | Customer history / admin list / detail + history/media/messages |
| (optional) message/media helpers or included in create/update/get | Chat + attachments |
| Extension of job create | Accept `job_type = 'COMPLAINT'` |

**Existing helper that M6 should reuse (not recreate):**

- `fn_create_notification_details(jsonb)` — inbox enqueue  
- `fn_create/update/get_job_assignment_details` — **today reject COMPLAINT** (see §4)  
- `fn_get_site_visit_charge` / visit update — complaint visit charging  
- `fn_get_dropdown_data` — categories/severities/statuses  

---

## 4. M7 integration requirements

### What already exists

| Object | Role for M6 |
|---|---|
| `technician_details` | Assignee target |
| `job_assignment_details` | Field job linked to complaint |
| `fn_create_technician_details`, `fn_update/get_technician_details` | Technician master |
| `fn_create_job_assignment_details` | Admin assign job |
| `fn_update_job_assignment_details` | Accept / reschedule / start / complete + checklist/media/signoff |
| `fn_get_job_assignment_details` | Technician queue |
| Flutter `technician/` + API `/api/admin/technicians*`, `/api/technician/jobs*` | M7 UI/API |

### Critical gap — do not duplicate; **extend**

`fn_create_job_assignment_details.sql` currently:

```text
job_type must be SERVICE_REQUEST or SITE_VISIT
```

Table CHECK already allows `COMPLAINT` with `complaint_ticket_id NOT NULL`.  
**M6 must extend job create (and likely get/update filters)** to support `COMPLAINT` without a parallel assignment stack.

### Sign-off / OTP

- `job_signoff_details.otp_verified_ind` documented for complaint resolution OTP success.
- Ticket stores `resolution_otp_hash` / `otp_expires_on`.
- **Unresolved:** whether resolve OTP is verified only in ticket update, only in job complete, or both must agree.

### Avoid duplicating M7 / M5

- Do **not** invent a second technician assignment table.
- Do **not** force complaints through `service_request_details` (no FK; BRD treats breakdown tickets separately from preventive SR).
- Reuse job queue UI patterns; add complaint-specific deep links and OTP resolve UX.

---

## 5. Missing API endpoints

**Solar-API:** zero matches for `complaint` / `COMPLAINT` under `src/` and `test/`.  
`src/app.js` has no complaint mounts.

Likely thin proxy routes (pattern from M7/M8 — **not implemented**):

| Suggested route | Backing function |
|---|---|
| `POST /api/complaints` | get (customer/admin scoped) |
| `POST /api/complaints/create` or `/api/admin/complaints/create` | create |
| `POST /api/complaints/update` / admin update | status, assign, message, resolve |
| Media static under `/media/...` | similar to KYC/jobs |

---

## 6. Missing Flutter screens

| Evidence | Finding |
|---|---|
| `lib/features/dashboard/models/dashboard_module.dart` | Module title **Complaints** — placeholder: “coming soon” |
| `lib/features/dashboard/presentation/dashboard_shell.dart` | Copy mentions complaints among unimplemented service areas |
| `test/widget_test.dart` | Expects find text `Complaints` (dashboard tile only) |
| `lib/features/**` | **No** `complaints` feature folder, models, or APIs |

Missing relative to BRD: raise ticket, lifecycle/status stepper, media attach, chat, customer history, admin allocate/SLA view, technician complaint-job handling / OTP resolve.

---

## 7. Unresolved business decisions

1. **SLA definition** — What clocks, thresholds, and breach actions apply per severity (especially Emergency)?  
2. **Resolved vs Closed** — One status (`TKT_RESOLVED`) vs two-phase close after `resolved_on`.  
3. **Raise roles** — Customer self-raise only, admin on behalf, or both?  
4. **Resolution OTP channel** — SMS (`log_otp_attempts`) vs in-app display vs both; expiry duration.  
5. **Job vs ticket ownership of resolve** — Ticket update with OTP vs job complete with `otp_verified_ind` (or dual write).  
6. **“Call” engineer** — In-app only (show mobile) vs true telephony.  
7. **Spare-part approval** for complaints — reuse generic spare-part flow or complaint-specific rules.  
8. **Billing/App category (`CMP_BILLING`)** — Field visit required or desk-only path?  
9. **Media limits** — BRD “up to 4”; DDL says limit from business parameters — confirm parameter code/value.  
10. **Login OTP vs complaint OTP** — M1 is PIN today; complaint still needs resolution OTP per BRD (orthogonal, but delivery stack may be shared).

---

## 8. Recommended M6 implementation phases

### Phase A — Foundations (after decisions 1–5 minimum)

1. Approve function contracts for create/get/update complaint.  
2. Confirm media limit parameter + ticket_no generator rules.  
3. Extend `fn_create_job_assignment_details` (+ docs/tests) for `COMPLAINT`.

### Phase B — Customer raise + history

1. SQL create/get + media.  
2. API + Flutter raise form (dropdowns via `fn_get_dropdown_data`) + list/detail.  
3. `NTF_COMPLAINT_NEW` to admin.

### Phase C — Admin allocate + lifecycle

1. Assign technician; status history; optional job create `COMPLAINT`.  
2. Admin console list/detail/stepper.  
3. `NTF_COMPLAINT_UPDATE` on status/assign.

### Phase D — Technician execution + OTP resolve

1. Complaint jobs appear in existing job queue.  
2. Checklist `job_type = COMPLAINT` templates as needed.  
3. Resolution OTP + signoff; ticket → `TKT_RESOLVED`; notifications.

### Phase E — Chat, spare parts, SLA (if decided)

1. Ticket messages (`is_internal`).  
2. Spare-part approval linkage.  
3. SLA monitoring only after product defines metrics.

---

## 9. Dependencies and risks

| Risk / dependency | Detail |
|---|---|
| M7 job create gap | Schema allows COMPLAINT; **SQL create rejects it** — must extend before field workflow |
| Notification helper | `fn_create_notification_details` available; push delivery still incomplete (M13) |
| Visit charging | Complaint visits use existing charge stack; needs purpose `VISIT_COMPLAINT` wiring when visits created |
| Media storage | Pattern exists (`/media/kyc`, `/media/jobs`); complaint media path not created |
| No tests | Zero complaint SQL/API/Flutter tests — greenfield quality risk |
| Overlap with SR | Product must keep complaint ≠ service request to avoid dual workflows |
| Schema vs app lag | Tables + seeds ~ready; **application layer ~0%** — matches PROJECT_STATUS ~5% |

---

## Evidence index (exact names)

### Docs
- `Solar-Docs/BRD.md` (Modules 6, 7.1, 12.7, 13.2)
- `Solar-Docs/DB_AUDIT_REPORT.md`
- `Solar-Docs/PROJECT_STATUS.md`
- `.cursorrules` (visit purposes; notification coverage)

### Schema / seed
- `Solar-DB/SQLScripts/01_schema.sql`
- `Solar-DB/DB-Docs/DDL_SCHEMA.md`
- `Solar-DB/DB-Docs/Tables/solar_app.complaint_ticket_details.md` (+ ticket_* siblings)
- `Solar-DB/DB-Objects/Tables/solar_app.complaint_ticket_details.sql`
- `Solar-DB/reference-data/json/ref_data.json`, `ref_data_categories.json`, `ref_dropdown_config.json`
- `Solar-DB/reference-data/sql/08_ref_data_categories.sql`, `09_ref_data.sql`, `10_ref_dropdown_config.sql`

### M7 (reuse / extend)
- `Solar-DB/DB-Objects/SQLFunctions/solar_app.fn_create_job_assignment_details.sql` (rejects COMPLAINT today)
- `solar_app.fn_update_job_assignment_details.sql`, `solar_app.fn_get_job_assignment_details.sql`
- `Solar-API/src/routes/jobs.js`, `technicians.js`; `Solar-UI/lib/features/technician/`

### Absent (confirmed by search)
- No `*complaint*` under `SQLFunctions/`, `Solar-API/src`, `Solar-UI/lib` (except dashboard placeholder strings)
- No complaint entries in `02_functions.sql`
- No complaint API tests

### Flutter placeholder only
- `Solar-UI/lib/features/dashboard/models/dashboard_module.dart`
- `Solar-UI/test/widget_test.dart` (tile text)
