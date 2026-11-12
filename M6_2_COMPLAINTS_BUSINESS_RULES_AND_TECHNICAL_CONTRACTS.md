# M6.2 — Complaints Business Rules & Technical Contracts

**Date:** 2026-11-12  
**Updated:** 2026-11-12 — Phase A **generator source** Conditional GO implemented (baseline + gates). Live DB migration **not** approved/executed. U1–U18 / OPEN-T1…T9 unchanged.  
**Type:** Documentation / contracts (+ generator implementation status)  
**Related:** [M6_1_COMPLAINTS_ASSESSMENT.md](M6_1_COMPLAINTS_ASSESSMENT.md) · [BRD.md](BRD.md) §6 / 12.7 / 13.2 · Solar-DB DDL / M7 job SQL · Solar-API jobs/technicians routes

**Legend:** **Approved (U*)** · **Closed technical (OPEN-T*)** · **Evidence** · **Contract** · **Pending execution** (seed/impl not applied)

**Documentation closure ≠ implementation.** Closing OPEN-T items records decisions only. Live U1 seed apply, COMPLAINT job create (U15), SQL/API/Flutter remain **unimplemented**.

---

## 0. Scope of this update

- Preserves product-approved decisions **U1–U18** (including U17a/b/c) without reinterpretation.  
- Closes **OPEN-T1…T9** per explicit user approval (business items T1/T2/T3/T7/T9) and accepted technical defaults (T4/T5/T6/T8).  
- Phase A **generator** work (Conditional GO) may update Solar-DB `reference-data/` artifacts under the M6 gate; **live DB migration remains separately approved**.  
- **Does not** apply live seed, DDL, SQL function, API, or Flutter changes without explicit approval.

---

## 1. Approved decision register (authoritative) — U1–U18 unchanged

| ID | Decision (approved) |
|---|---|
| **U1** | Add status `TKT_CLOSED` (label **Closed**). Relabel `TKT_RESOLVED` → **Resolved – Awaiting User Confirmation**. Seed **not applied yet**. |
| **U2** | User reopen: `TKT_CLOSED` → `TKT_RAISED`. Admin reassigns. **No** automatic COMPLAINT job on reopen. |
| **U3** | Technician advances mid-stepper (`TKT_EN_ROUTE` / `TKT_ON_SITE` / `TKT_PARTS`) through **job actions**. Admin may **override** mid-status. Exact action→status map: **OPEN-T1 CLOSED** (§1A). |
| **U4** | **Customer** and **Admin** may raise. **Technician** cannot raise. |
| **U5** | **Admin may resolve** to `TKT_RESOLVED` with **mandatory remarks**. Technician resolve remains primary (R2 / U8 / U16). |
| **U6** | **Admin may reopen anytime** (including after Day 15) with **mandatory remarks**. **Admin may force-close** from `TKT_RESOLVED` → `TKT_CLOSED` (with remarks). |
| **U7** | Prefer owning **Customer** app user for CONFIRM_CLOSE / Customer reopen. **Admin may confirm close** only when **no linked customer app user** exists. Detection rule: **OPEN-T2 CLOSED** (§1A). |
| **U8** | Technician may `RESOLVE` from **any post-assignment open** status (`TKT_ASSIGNED`…`TKT_PARTS`, and mid-states). Not from `TKT_RAISED` / `TKT_CLOSED`. |
| **U9** | Duplicate `RESOLVE` while already `TKT_RESOLVED` → **HTTP 409**. |
| **U10** | On closure (`TKT_CLOSED`): notify **Customer** (when eligible), **all eligible Admins**, and **assigned Technician** via existing `NTF_COMPLAINT_UPDATE` (fan-out). |
| **U11** | On Customer/Admin reopen: `resolved_on` = NULL; `assigned_technician_id` = NULL; `closed_on` = NULL; status → `TKT_RAISED`; **preserve** history/job rows. |
| **U12** | Admin reassignment resets ticket status to **`TKT_ASSIGNED`**. |
| **U13** | Prior active COMPLAINT job → **`JOB_REJECTED`** with **mandatory Admin reason**, subject to M7 validation/extension. See §3 inspection (reject path not implemented today). Storage: **OPEN-T4/T8 CLOSED** (§1A). |
| **U14** | Customer reopen through **end of calendar Day 15**; Day 1 = local date of `closed_on` in `organization_details.time_zone_name`. |
| **U15** | **M7 COMPLAINT job creation** is the **single** field-work assignment mechanism (sets ticket assignee + `TKT_ASSIGNED`). **Not implemented yet.** |
| **U16** | COMPLAINT job **COMPLETE** automatically sets ticket to **`TKT_RESOLVED`** (+ `resolved_on`). No OTP. Edge cases: **OPEN-T5 CLOSED** (§1A). |
| **U17a** | **Defer** numeric SLA; UI/API may **display severity** only. |
| **U17b** | **Defer** detailed spare-parts workflow; reuse `spare_part_request_details` later. |
| **U17c** | Complaint attachment **maximum must be configurable** (prefer `ref_business_parameters`). Spec: **OPEN-T3 CLOSED** (§1A). Parameter **not seeded yet**. |
| **U18** | **Do not** add `technician_id` to `ticket_status_history_details`. Preserve technician assignment history via **M7 `job_assignment_details`** rows. |

### Confirmed product rules (derived)

| # | Rule |
|---|---|
| R1 | Only Admin assigns/reassigns via U15 job create; technicians cannot self-assign. |
| R2 | Technician marks `TKT_RESOLVED` (no OTP); also via U16 job COMPLETE; Admin may resolve with remarks (U5). |
| R3 | Prefer Customer CONFIRM_CLOSE → `TKT_CLOSED`; Admin force-close allowed from `TKT_RESOLVED` (U6); U7 Admin close fallback if customer user ineligible (OPEN-T2). |
| R4/U10 | Closure notifies Customer + Admins + assigned Technician (`NTF_COMPLAINT_UPDATE`). |
| R5–R7/U14 | Customer reopen within window → `TKT_RAISED`; after window Customer cannot; Admin reopen anytime (U6). |

---

## 1A. OPEN-T1…T9 closure register

**Status:** All nine **CLOSED** in documentation (2026-11-12).  
**Distinction:** *User-approved* = explicit product approval of recommended options. *Technical default accepted* = accepted implementation defaults under U1–U18 (not new business rules).

| ID | Class | Closed decision |
|---|---|---|
| **OPEN-T1** | User-approved | **ACCEPT** → keep `TKT_ASSIGNED`. **START** → `TKT_ON_SITE`. **COMPLETE** → `TKT_RESOLVED` (U16). **`TKT_EN_ROUTE`** and **`TKT_PARTS`**: **Admin override only** in v1. **No** new Technician job actions in v1. |
| **OPEN-T2** | User-approved | `customer_details.user_id` remains **NOT NULL**. “No linked customer app user” (U7) means linked user **ineligible to receive notifications**: `active_ind IS NOT TRUE` **or** `ACCT_SUSPENDED`. **No** NULL `user_id` interpretation; **no** schema change. |
| **OPEN-T3** | User-approved | Parameter `COMPLAINT_MEDIA_MAX_COUNT`, `value_type=NUMERIC`, `numeric_value=4`, `admin_manageable_ind=TRUE`. Enforce in complaint create. **Fail closed** if parameter missing/unseeded — **no** silent hardcoded fallback. **Seed not applied in this task.** |
| **OPEN-T4** | Technical default accepted | Admin reassignment rejects prior COMPLAINT job → `JOB_REJECTED` with **mandatory** Admin reason in `job_assignment_details.accept_reject_reason`. **Do not** enable Technician-initiated rejection. |
| **OPEN-T5** | Technical default accepted | COMPLETE if ticket `TKT_CLOSED` → **HTTP 409**. If already `TKT_RESOLVED` → allow job COMPLETE; **do not** insert another ticket resolution history row. Explicit duplicate RESOLVE still **409** (U9). |
| **OPEN-T6** | Technical default accepted | Admin RESOLVE only from post-assignment open statuses `TKT_ASSIGNED`…`TKT_PARTS`; **mandatory remarks**. Force-close remains separate (U6). |
| **OPEN-T7** | User-approved | On reopen: notify **eligible Customer** + **all eligible Admins**. **Do not** notify previous Technician by default. **Do not** use cleared `assigned_technician_id` as recipient. |
| **OPEN-T8** | Technical default accepted | U13 reason: `accept_reject_reason` on the rejected job **and** copy into `ticket_status_history_details.remarks` for the reassignment transition. **No** new DB column. |
| **OPEN-T9** | User-approved | **Defer** chat/messaging UI to post-core. Core phase: `tel:` dialer to assigned Technician `user_details.mobile_no` when assignee exists. **No** call-history table. |

### Noted tension (documented, not silently “fixed”)

- **U3** states technicians advance mid-stepper statuses including `TKT_EN_ROUTE` / `TKT_PARTS` through job actions.  
- **OPEN-T1** (user-approved) states v1 job actions only auto-map ACCEPT/START/COMPLETE, and **`TKT_EN_ROUTE` / `TKT_PARTS` are Admin override only**; no new tech job actions.  
- **Resolution for implementers:** apply **OPEN-T1** as the v1 action→status contract; Admin override covers `TKT_EN_ROUTE` / `TKT_PARTS`. U3’s Admin override clause remains valid. U3 wording in §1 is **not rewritten**.

---

## 2. Status model (approved target vs current seed)

### 2.1 Current seed (evidence — unchanged on disk)

Source: `Solar-DB/reference-data/json/ref_data.json` / `09_ref_data.sql`.

| Code | Seed label today |
|---|---|
| `TKT_RAISED` … `TKT_PARTS` | As seeded |
| `TKT_RESOLVED` | Resolved & Closed |
| — | **No** `TKT_CLOSED` |

### 2.2 Approved target (U1 — not seeded)

| Code | Approved label | sort (proposed) |
|---|---|---|
| `TKT_RAISED` | Ticket Raised | 1 |
| `TKT_ASSIGNED` | Assigned to Engineer | 2 |
| `TKT_EN_ROUTE` | Technician En-Route | 3 |
| `TKT_ON_SITE` | On-Site Inspection | 4 |
| `TKT_PARTS` | Parts Replaced | 5 |
| `TKT_RESOLVED` | Resolved – Awaiting User Confirmation | 6 |
| `TKT_CLOSED` | Closed | 7 |

### 2.3 Required seed changes (document only — **do not apply** in this task)

**U1:**

1. Relabel `TKT_RESOLVED` → `Resolved – Awaiting User Confirmation`.  
2. Insert `TKT_CLOSED` / `Closed`.  
3. Regenerate via `reference-data` pipeline.

**U17c / OPEN-T3:**

4. Seed `ref_business_parameters` row: `COMPLAINT_MEDIA_MAX_COUNT`, NUMERIC, `4`, `admin_manageable_ind=TRUE` (via generator; `BUSINESS_PARAMETERS` currently `[]`).

### 2.4 Key transitions (approved + OPEN-T1)

```text
Raise (Customer|Admin) --> TKT_RAISED
Admin COMPLAINT job create (U15) --> TKT_ASSIGNED + assigned_technician_id
Tech ACCEPT --> ticket stays TKT_ASSIGNED (OPEN-T1)
Tech START --> TKT_ON_SITE (OPEN-T1)
Admin override --> TKT_EN_ROUTE | TKT_ON_SITE | TKT_PARTS (OPEN-T1)
Tech RESOLVE (U8) OR Admin RESOLVE+remarks (U5/OPEN-T6) OR job COMPLETE (U16/OPEN-T5)
    --> TKT_RESOLVED + resolved_on
Customer CONFIRM_CLOSE | Admin force-close (U6) | Admin close if U7/OPEN-T2 ineligible customer user
    --> TKT_CLOSED + closed_on
Customer REOPEN (U14) | Admin REOPEN (U6)
    --> TKT_RAISED; U11 resets; notify Customer+Admins only (OPEN-T7)
Admin reassign (U12/U13/U15): prior job JOB_REJECTED + accept_reject_reason
    + history remarks (OPEN-T4/T8); new job; ticket TKT_ASSIGNED
```

---

## 3. Technical inspection findings (evidence — still true; impl pending)

### 3.1 Does `fn_create_job_assignment_details` accept `COMPLAINT`?

| | |
|---|---|
| **Finding** | **No.** Only `SERVICE_REQUEST` or `SITE_VISIT`. |
| **Evidence** | `Solar-DB/DB-Objects/SQLFunctions/solar_app.fn_create_job_assignment_details.sql` ~151–156; contract MD Phase 1 excludes complaint. |
| **Schema** | `chk_job_type` **allows** `COMPLAINT` (`01_schema.sql`). |

**Pending (U15):** extend create — **not done**.

### 3.2 Duplicate active-job predicate

Active = `active_ind IS TRUE` and status ∈ `JOB_QUEUED`, `JOB_ACCEPTED`, `JOB_RESCHEDULED`, `JOB_IN_PROGRESS`. **`JOB_REJECTED` excluded.** Evidence: create fn ~301–310 / ~394–397. Supports U13/OPEN-T4 after Admin reject is implemented.

### 3.3 Job history preserves technicians

`job_assignment_details.technician_id` NOT NULL per row; U18 relies on keeping rows (including `JOB_REJECTED`).

### 3.4 Job create updates parent

SR/visit updated today; COMPLAINT N/A until U15 implemented.

### 3.5 COMPLETE → complaint resolve

Not implemented today (SR/visit only). U16 + OPEN-T5 are **contracts for future** update fn work.

### 3.6 Media parameter (OPEN-T3)

Table can store NUMERIC parameters; `BUSINESS_PARAMETERS = []` in `generate-reference-data.mjs` ~671. Spec closed; **seed pending**. Create must **fail closed** if missing.

### 3.7 Notification fan-out

`fn_create_notification_details` one recipient/call; callers loop. Types: `NTF_COMPLAINT_NEW`, `NTF_COMPLAINT_UPDATE` only. Entity `TICKET`.

### 3.8 API / Flutter

No complaint routes; dashboard placeholder only. Job routes: `/api/technician/jobs`, admin technicians/assign.

---

## 4. Authorization & lifecycle reconciliation

### 4.1 Roles

| Action | Customer | Technician | Admin |
|---|---|---|---|
| Raise | Yes (U4) | No | Yes (U4) |
| Assign / reassign (U15) | No | No | Yes |
| Mid-status via job (OPEN-T1) | No | ACCEPT/START/COMPLETE maps only | Override `EN_ROUTE`/`ON_SITE`/`PARTS` |
| Resolve | No | Yes (U8; U16 COMPLETE) | Yes + remarks (U5/OPEN-T6) |
| Confirm close | Yes if eligible user | No | U7/OPEN-T2 if customer user ineligible; or force-close U6 |
| Reopen | Yes within U14 | No | Anytime + remarks (U6) |
| Chat | Deferred (OPEN-T9) | Deferred | Deferred |
| Call (`tel:`) | When assignee present (OPEN-T9) | — | When assignee present |

### 4.2 Technician vs Admin resolve

| | Technician | Admin |
|---|---|---|
| From | `TKT_ASSIGNED`…`TKT_PARTS` (U8) | Same (OPEN-T6) |
| Remarks | Optional unless later required | **Mandatory** (U5) |
| Duplicate | **409** (U9) | **409** (U9) |
| Job COMPLETE | Sync to RESOLVED (U16); OPEN-T5 edge cases | N/A |

### 4.3 Close / reopen / force-close

Unchanged from U2/U6/U11/U14. Reopen notifications: **OPEN-T7**.

### 4.4 U7 / OPEN-T2 eligibility

- Schema: `customer_details.user_id` **NOT NULL** (`01_schema.sql`).  
- **Eligible customer user:** `active_ind IS TRUE` and account status **not** `ACCT_SUSPENDED` (align with `fn_create_notification_details` recipient checks).  
- **Ineligible:** inactive or suspended → Customer cannot CONFIRM_CLOSE / Customer reopen; Admin may confirm close (U7).  
- Do **not** use `user_id IS NULL`.

### 4.5 Notifications

| Event | Type | Recipients |
|---|---|---|
| Created | `NTF_COMPLAINT_NEW` | Admins; tech if assigned |
| Assign / status / resolve | `NTF_COMPLAINT_UPDATE` | Customer, admins, assigned tech |
| Closure | `NTF_COMPLAINT_UPDATE` | Customer + Admins + assigned Technician (U10) |
| Reopen | `NTF_COMPLAINT_UPDATE` | **Eligible Customer + eligible Admins only** (OPEN-T7) |
| Job queued | `NTF_JOB_ASSIGNED` | Technician |

### 4.6 Mandatory Admin remarks / reasons

| Action | Storage |
|---|---|
| Admin RESOLVE | `ticket_status_history_details.remarks` |
| Admin force-close / reopen | history `remarks` |
| Admin reassign / `JOB_REJECTED` | `accept_reject_reason` **and** history `remarks` (OPEN-T4/T8) |

---

## 5. Mid-stepper & M7 mapping (U3 + OPEN-T1 CLOSED)

| Job action | Ticket status (v1) |
|---|---|
| ACCEPT | Keep `TKT_ASSIGNED` |
| START | `TKT_ON_SITE` |
| COMPLETE | `TKT_RESOLVED` (U16; OPEN-T5) |
| RESCHEDULE | Job only; no automatic ticket mid-status change unless later specified |
| Admin override | May set `TKT_EN_ROUTE`, `TKT_ON_SITE`, `TKT_PARTS` (+ remarks) |
| New tech actions | **None** in v1 |

---

## 6. Fifteen-day Customer reopen (U14)

Unchanged. Admin reopen (U6) bypasses window.

---

## 7. History (U11 / U18)

Unchanged. Technician identity via jobs only. No history `technician_id` column.

---

## 8. Decision status summary

| Register | Status |
|---|---|
| U1–U18 | **Approved** (business); seed/impl **pending** where noted |
| OPEN-T1…T9 | **Closed** in documentation (§1A) |
| Phase A generator source | **Done (Conditional GO)** — `stable/ref_data_id_baseline.json` (187), empty overrides, natural-key `COMPLAINT_MEDIA_MAX_COUNT` SQL, forward-only `setval`, fail-closed until live `TICKET_STATUSES.TKT_CLOSED` id |
| Final M6 `ref_data` regenerate | **Blocked** — awaiting live-captured `TKT_CLOSED` id in `ref_data_id_overrides.json` |
| U1 seed apply (live DB) | **Pending** — separate migration approval; **not** executed |
| U15 COMPLAINT job create | **Pending implementation** |
| U17c parameter seed | Generator emit ready; **live apply pending** (separate DB approval) |

**No remaining OPEN-T items.**  
**Remaining work is live migration approval + app implementation**, not further OPEN-T product questions.

---

## 9. Phase-wise implementation dependencies

| Phase | Work | Depends on |
|---|---|---|
| **A — Seed** | Generator: baseline/overrides/gates + canonical param SQL (**done**). Live: migrate `TKT_CLOSED` / relabel / apply param (**pending separate approval**) | Live id capture → override → full regen |
| **B — Raise / get** | Create/get complaint; fail-closed media max; U4 raise; `NTF_COMPLAINT_NEW` | A |
| **C — Assign** | Extend job create for **COMPLAINT (U15)**; reassign U12/U13 + OPEN-T4/T8 | A |
| **D — Field + resolve** | OPEN-T1 maps; U16 COMPLETE; U5/U8/U9; OPEN-T5/T6 | C |
| **E — Close / reopen** | U10; U14; U6; U7/OPEN-T2; U11; OPEN-T7; OPEN-T9 `tel:` | A |
| **F — Deferred** | U17a/b; chat (OPEN-T9 deferred) | Core done |

---

## 10. Planned function surface (not implemented)

- `fn_create_complaint_ticket_details` / `fn_get_complaint_ticket_details` / `fn_update_complaint_ticket_details`  
- Extend `fn_create_job_assignment_details`, `fn_update_job_assignment_details`  
- Reuse `fn_create_notification_details`, `fn_get_dropdown_data`

---

## 11. Conflicts with older docs

| Older | This contract |
|---|---|
| BRD OTP to resolve | Superseded — no OTP |
| BRD single “Resolved & Closed” | Split U1 |
| Prior OPEN-T1…T9 open list | **Closed** §1A |
| U18 history.technician_id proposal | Rejected — use jobs |

---

## 12. Confirmation

- Documentation updated to close OPEN-T1…T9.  
- **U1–U18 decision text preserved** (pointers updated only to closed OPEN-T refs).  
- Phase A **generator source** Conditional GO implemented under Solar-DB `reference-data/` (no live DB connection; no invented `TKT_CLOSED` id).  
- **No** live migration; **no** API/Flutter; **no** `02_functions.sql`; **no** commit/push in this task unless separately requested.  
- **Documentation / generator readiness does not mean live deployment is complete.**
