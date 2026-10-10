# Solar Database Read-Only Audit Report

**Audit date:** 2026-10-10  
**Scope:** Solar Service Management Mobile Application  
**Audit mode:** Read-only

No project files were changed during the audit. No SQL was executed, no database connection was made, and no JSON was imported.

## A. Executive summary

Overall recommendation: **DO NOT APPROVE**

The design has good structural foundations:

- 56 proposed tables.
- All tables contain the five mandatory audit columns.
- Every table and column is documented.
- All primary keys use `GENERATED ALWAYS AS IDENTITY`.
- No triggers are defined.
- Offline billing and payment constraints are represented.
- Path A/Path B onboarding, notifications, site visits, catalogue, service, and complaints have proposed data models.

Approval is blocked by:

1. `DDL_SCHEMA.md` cannot be executed in its documented order because two foreign keys reference `customer_details` before that table is created.
2. The generated city dataset is materially unreliable: 838 source cities could not be mapped and were silently assigned to the first district in their state.
3. Subsidy slabs cannot deterministically calculate the documented subsidy.
4. Visit-charge slabs overlap at 10, 25, and 50 km.
5. Reference imports are neither atomic nor rerunnable.
6. `01_schema.sql` and `02_functions.sql` do not exist, so most business controls remain planned rather than implemented.
7. Dynamic dropdown SQL remains an unimplemented and potentially unsafe design.
8. Several BRD/design conflicts need product-owner decisions.

---

## B. File inventory

### Reviewed

| Area | Files |
|---|---|
| Requirements | `Solar-Docs/BRD.md`, `Solar-Docs/Notes.txt`, `.cursorrules` |
| Database design | `Solar-DB/DB_REQUIREMENTS.md`, `Solar-DB/DB-Design.md`, `Solar-DB/DDL_SCHEMA.md`, `Solar-DB/DB-Setup.sql` |
| Reference documentation | `Solar-DB/reference-data/SUMMARY.md` |
| JSON | All 15 files under `Solar-DB/reference-data/json/` |
| Sources | All 3 files under `Solar-DB/reference-data/_source/` |
| Scripts | `generate-reference-data.mjs`, `patch-ddl-comments.mjs` |
| SQL | All 16 files under `Solar-DB/reference-data/sql/` |

### Missing

- `Solar-Docs/BRD.pdf`
- `Solar-Docs/सोलर अॅप डेव्हलपमेंट.pdf`
- `Solar-DB/SUMMARY.md`

A similarly named file exists at `Solar-DB/reference-data/SUMMARY.md`, but it is only a reference-data summary.

### Planned canonical files that are absent

- `Solar-DB/01_schema.sql`
- `Solar-DB/02_functions.sql`
- `Solar-DB/03_seed_data.sql`

Therefore, the reviewed schema is a proposal, not an implemented database.

---

## C. Requirement-to-table coverage matrix

| Requirement | Proposed coverage | Status |
|---|---|---|
| OTP, lockout, PIN, biometric | `user_details`, `log_user_sessions`, `log_otp_attempts` | Schema proposed; functions missing |
| User roles and technicians | `user_details` + 1:1 `technician_details` | Covered |
| Path A/Path B onboarding | `admin_registered`, registration source, customer/site/order tables | Covered but semantics conflict |
| Calculator and subsidy | Calculator estimates, parameters, subsidy slabs | Structurally covered; rules ambiguous |
| Leads and quotations | Lead, quote header and lines | Covered |
| All site-visit purposes | Site visits, office locations, charge slabs | Covered; charging rules ambiguous |
| Customer/site/KYC | Customer, site, site access, KYC | Covered |
| Equipment and warranty | Equipment, warranty | Covered |
| AMC and free service | AMC, milestones, service requests | Covered; “exactly four” conflicts with configurable count |
| Complaint management | Ticket, status history, media, messages | Covered |
| Technician jobs | Assignments, checklist, media, sign-off, spares | Covered |
| Offline billing/payment | Invoice, lines, receipts, milestones | Covered; no gateway |
| Product catalogue/enquiry | Products, media, enquiries | Covered |
| Orders/follow-ups | Orders, lines, follow-ups | Covered |
| Notifications | Inbox and delivery logs | Covered; delivery functions/worker missing |
| Dynamic dropdown API | Reference categories, values and config | Tables/seeds exist; function missing |
| API/function logging | API and SQL logs | Tables exist; functions missing |
| Full India PIN coverage | Only 70 representative PINs | Not covered |
| Push delivery runtime | Delivery queue + token columns | Worker not implemented |
| PDF generation, FCM, routing | URLs/status columns only | External implementation missing |

---

## D. DDL audit

### Aggregate result

| Check | Result |
|---|---:|
| Tables | 56 |
| Table comments | 56/56 |
| Column comments | All columns documented |
| Mandatory audit columns | 56/56 |
| Identity primary keys | 56/56 |
| Triggers | 0 |
| Foreign keys | Approximately 179 |
| Secondary/partial indexes | Approximately 42 |
| Check constraints | Approximately 63 |
| Duplicate constraint names | None detected |

### Table-by-table review

#### Reference tables

| Table | Result |
|---|---|
| `ref_data_categories` | Valid purpose/comments; `category_code` uses full unique rather than soft-delete-aware partial unique. |
| `ref_data` | Good generic-value model, partial unique, parent relation and JSONB index. Category correctness is not enforceable by dependent tables. |
| `ref_dropdown_config` | Structurally documented; free-form identifiers and `extra_where_sql` create SQL-safety risk. `SQL` source type has no clear source-query column. |
| `ref_countries` | Valid India root record model. |
| `ref_states` | Valid state/country relationship and indexes. |
| `ref_districts` | Valid structure and partial business uniqueness. Source quality needs correction. |
| `ref_cities` | `state_id` is denormalized without verifying it matches the selected district. Generated business data is critically unreliable. |
| `ref_pincodes` | PIN format enforced; city/district/state consistency is not. Dataset is not nationwide. |
| `ref_discoms` | State/district model is reasonable; one combined TPADL/Jamshedpur record is questionable. |
| `ref_calculator_params` | Valid generic parameter model; values lack region/effective-date versioning. |
| `ref_subsidy_slabs` | Basic range checks exist; overlapping bounds and amount semantics are unresolved. |
| `ref_amc_plans` | Free visit entitlement supported; radius lacks a nonnegative check. Prices have no effective dates. |
| `ref_checklist_items` | Appropriate composite partial unique and enum checks. |
| `ref_office_locations` | Valid origin model; no unique partial constraint for one default office. |
| `ref_visit_charge_slabs` | Basic checks exist; no exclusion constraint prevents overlapping bands. |

#### Users, logs and notifications

| Table | Result |
|---|---|
| `user_details` | Good identity master; `account_status_ref_data_id` is nullable. `admin_registered` meaning conflicts with Path A terminology. |
| `technician_details` | Correct 1:1 extension through unique `user_id`; technician role itself is enforced only by planned functions. |
| `log_api_execution` | Useful audit model; full payload storage risks persisting OTPs, tokens and PII. |
| `log_sql_functions` | Same payload-redaction and retention concern. |
| `log_user_sessions` | Good session/device model; token retention and cleanup policy are unspecified. |
| `log_otp_attempts` | OTP hash and cooldown fields are appropriate. |
| `notification_details` | Polymorphic entity references have no FK or paired-null check. |
| `log_notification_delivery` | Suitable push queue/audit model; worker is absent. |

#### Calculator, sales and catalogue

| Table | Result |
|---|---|
| `calculator_estimate_details` | Appropriate calculation snapshot; common lookup indexes are absent. |
| `lead_details` | `mobile_no` lacks a 10-digit CHECK; raw PIN and PIN FK may conflict. |
| `quote_details` | Appropriate header; queue/filter indexes are sparse. |
| `quote_line_details` | Appropriate owned child. |
| `site_visit_details` | Comprehensive model, but many optional parents have no valid-combination constraint. Charge and receipt fields can become inconsistent. |
| `product_details` | Strong catalogue model and partial product-code uniqueness. |
| `product_media_details` | Appropriate product child and ordering index. |
| `product_enquiry_details` | Good model; customer FK is placed before `customer_details` creation in document order. |

#### Customer and assets

| Table | Result |
|---|---|
| `customer_details` | Appropriate 1:1 user extension; GSTIN has no format validation. |
| `site_details` | Good domain coverage; geo consistency and operational indexes are missing. |
| `user_site_access_details` | Appropriate bridge; no partial single-default-site constraint. |
| `kyc_document_details` | File-size constraint exists; reject-comment enforcement remains planned only. |
| `equipment_details` | Flexible equipment model; no active serial-number unique index is evident. |
| `warranty_details` | Appropriate child and date check. |

#### Service, complaints and jobs

| Table | Result |
|---|---|
| `amc_contract_details` | Appropriate AMC instance; bidirectional invoice link relies on functions. |
| `service_milestone_details` | Supports configurable sequence, but table documentation still states four visits elsewhere. |
| `service_request_details` | Good service model; queue indexes are sparse. |
| `complaint_ticket_details` | Appropriate ticket header; common status/technician queue indexes are missing. |
| `ticket_status_history_details` | Correct lifecycle child; ticket index should be explicit. |
| `ticket_media_details` | Correct child; four-file maximum is planned function logic only. |
| `ticket_message_details` | Appropriate ticket chat child. |
| `job_assignment_details` | Good technician queue; commissioning allows no parent. Pre-customer site visits cannot create a job if `site_id` is unavailable because job `site_id` is mandatory. |
| `job_checklist_response_details` | Appropriate checklist child. |
| `job_media_details` | Appropriate evidence child. |
| `job_signoff_details` | Appropriate signature/OTP confirmation child. |
| `spare_part_request_details` | Appropriate lightweight request model. |

#### Billing, orders and follow-ups

| Table | Result |
|---|---|
| `invoice_details` | Correct offline register concept; total/paid/balance consistency is function-only. |
| `invoice_line_details` | Appropriate optional bill lines. |
| `payment_details` | Customer is denormalized but not constrained to match invoice customer. |
| `payment_milestone_details` | Suitable planned instalment split; relationship to partial receipt rows is unclear. |
| `order_details` | Appropriate fulfilment model and useful indexes. |
| `order_line_details` | Appropriate order child. |
| `followup_details` | Selected parent must be present, but unrelated parent columns are not forced null. |

### Executable-order defect

`DDL_SCHEMA.md` adds `fk_enq_customer` and `fk_lead_converted_customer` before creating `customer_details`.

Impact: copying/running SQL fences in order fails with “relation does not exist.”

### Index review

The original scope requests strategic indexes on foreign keys and frequently searched fields. With approximately 179 FKs but only 42 secondary indexes, this requirement is not met.

Important missing index candidates include:

- Customer/site ownership and status.
- Service request status/technician/date.
- Complaint status/severity/technician.
- Ticket child parent IDs.
- Invoice customer/site/status/date.
- Payment invoice/customer/date.
- AMC site/status/expiry.
- Subsidy state/connection/effective period.
- KYC verification queue.
- Equipment site/product.

---

## E. JSON-by-JSON results

All 15 files contain valid JSON. Total generated records: 2,954.

| JSON file | Count | Intended purpose | Result |
|---|---:|---|---|
| `ref_countries.json` | 1 | Country master | Pass |
| `ref_states.json` | 36 | States and UTs | Pass |
| `ref_districts.json` | 784 | District hierarchy | Warning: questionable names and provenance |
| `ref_cities.json` | 1,679 | City/district cascade | **Fail:** large-scale wrong district assignment |
| `ref_pincodes.json` | 70 | Representative PIN lookup | **Fail for nationwide use:** limited and inherits city errors |
| `ref_discoms.json` | 76 | State/district utilities | Warning: questionable TPADL/Jamshedpur record |
| `ref_data_categories.json` | 38 | Generic category headers | Pass |
| `ref_data.json` | 187 | Generic dropdown values | Pass |
| `ref_dropdown_config.json` | 47 | Single dropdown API metadata | Warning: dynamic SQL and DISCOM filter ambiguity |
| `ref_calculator_params.json` | 11 | Calculator and service defaults | Warning: static/unversioned business assumptions |
| `ref_subsidy_slabs.json` | 5 | Central/state subsidy rules | **Fail:** overlaps and ambiguous amount meaning |
| `ref_amc_plans.json` | 3 | AMC plans and free entitlements | Warning: pricing validity/24-month annual-price ambiguity |
| `ref_checklist_items.json` | 12 | Technician checklist templates | Pass |
| `ref_office_locations.json` | 1 | Visit distance origin | **Fail for production:** placeholder address |
| `ref_visit_charge_slabs.json` | 4 | Distance-based visit fees | **Fail:** overlapping inclusive bands |

### JSON shape observations

- Audit timestamps are omitted intentionally and rely on database defaults.
- Geo JSON includes convenience names not present in tables; SQL strips them.
- `ref_data.json` includes `category_code` and `extra_json`; generated SQL omits them.
- Subsidy JSON includes `connection_type_code`, which is not a table column.
- These JSON files therefore are not direct table-import payloads despite being described as one file per table.

---

## F. Cross-file reference integrity

### Valid technical relationships

- All state `country_id` values resolve.
- District `state_id` values resolve.
- City/pincode/DISCOM references resolve numerically.
- Every `ref_data.category_id` resolves.
- Generic IDs and category/code pairs are unique.
- Dropdown category codes resolve.
- Subsidy connection IDs resolve to connection-type values.
- Office city/PIN IDs resolve.
- AMC/checklist IDs and codes are unique.

### Critical semantic failure: city mapping

The generator silently substitutes the first state district when it cannot infer a city’s district:

```javascript
const dlist = (districtsByState.get(stateName) || []).map((d) => d.district_name);
let distName = matchDistrict(dlist, c.name);
if (!distName && dlist.length) distName = dlist[0];
```

Examples:

- Kalyan-Dombivali → Ahilyanagar
- Osmanabad → Ahilyanagar
- Aurangabad → Ahilyanagar
- Allahabad → Agra
- Guwahati → Bajali
- Jamshedpur → Bokaro
- Shillong → East Garo Hills

Approximately 838 of 1,221 source-city rows used this fallback.

Because these references are numerically valid, ordinary FK validation will not detect the corruption.

### Geo coverage

- Only 70 PIN codes are provided.
- Several states/UTs depend on district-name placeholder cities.
- The city source contains duplicate business names.
- Source versions, checksums, retrieval dates and licence records are absent.

---

## G. Dropdown and SQL safety

### DISCOM district filtering

`DISCOMS_BY_DISTRICT` defines both:

- `parent_filter_column = district_id`
- `extra_where_sql = (district_id IS NULL OR district_id = :parent_value)`

If the planned generic function applies both, the resulting logic excludes statewide rows:

```sql
district_id = :parent_value
AND (district_id IS NULL OR district_id = :parent_value)
```

This contradicts the requirement that district dropdowns also return statewide utilities such as MSEDCL.

### Dynamic SQL risks

`ref_dropdown_config` permits:

- Table names
- Value/label columns
- Filter columns
- Sort columns
- Raw `extra_where_sql`
- `source_type = 'SQL'`

No implementation of `fn_get_dropdown_data` exists, so identifier whitelisting, `%I` quoting, bind parameters and authorization cannot be verified.

Recommended control:

- Whitelist source tables and columns.
- Use `format('%I', identifier)`.
- Bind values with `EXECUTE ... USING`.
- Replace free-form predicates with structured operators.
- Do not expose metadata maintenance to ordinary admins.
- Add category/source-specific CHECK constraints.

---

## H. Import order, identity mapping and rerun risks

### Identity handling

Generated SQL uses explicit identity values with `OVERRIDING SYSTEM VALUE`, which is correct for `GENERATED ALWAYS AS IDENTITY`.

Each file resets the associated sequence with `setval(MAX(id))`. This works only after a complete successful load.

### Import order

The intended order is generally valid:

1. Country
2. States
3. Districts
4. Cities
5. PINs/DISCOMs
6. Generic categories/data/config
7. Calculator/subsidy/AMC/checklists
8. Offices
9. Visit-charge slabs

### Atomicity and idempotency

`00_load_all.sql` uses `ON_ERROR_STOP`, but has no transaction.

Consequences:

- Earlier files remain committed after a later failure.
- Large chunked files can be partially committed.
- Failed files may never reset their identity sequences.
- A rerun fails on primary keys/business unique keys.
- Concurrent inserts can race sequence resets.
- There is no documented truncate/upsert/migration policy.

### Documentation drift

`reference-data/SUMMARY.md` still says SQL files `01–13`, although files 14 and 15 exist.

The documented `01_schema.sql` prerequisite does not exist.

---

## I. Conflicts and product-owner decisions

1. **Path A catalogue access**
   - BRD: OTP → browse catalogue.
   - Design: catalogue requires `admin_registered = TRUE`.
   - Decide exactly when self-registered users become catalogue-eligible.

2. **Meaning of `admin_registered`**
   - The name implies admin action, but comments say self-registration functions also set it true.
   - Decide whether this should represent account completion, admin creation, or authorization.

3. **Free service visits**
   - Some text says exactly four.
   - Other text says configurable globally/per AMC.
   - Decide the canonical rule and migration behavior when plans change.

4. **Registration source codes**
   - BRD uses `SELF_APP` / `ADMIN_POST_INSTALL`.
   - Seeds use `REG_SELF_APP` / `REG_ADMIN_POST_INSTALL`.

5. **Session table naming**
   - BRD uses `user_session_details`.
   - Design uses `log_user_sessions`.

6. **Subsidy semantics**
   - Is `subsidy_amount` a per-kW rate, fixed slab amount, or incremental rate?
   - How is ₹78,000 cap represented?
   - How are exact boundary values handled?
   - Should the active zero Maharashtra placeholder participate in calculations?

7. **Visit-charge boundaries**
   - Define half-open bands, e.g. `0 <= km < 10`, `10 <= km < 25`.
   - Decide pricing beyond 999 km.
   - Decide purpose-specific pricing precedence.

8. **AMC price**
   - For a 24-month plan, is `annual_price = 17999` annual or total contract price?

9. **PIN coverage**
   - Decide whether all-India PIN lookup is a v1 requirement.

10. **Technician pre-customer visits**
    - Decide whether a technician job can exist without `site_details`, or whether pre-customer assignments remain only in `site_visit_details`.

11. **Receipt evidence**
    - Decide whether `charges_received_ind = TRUE` requires `payment_id`, or whether a date alone is sufficient.

12. **Seed deployment model**
    - Decide whether `03_seed_data.sql` remains canonical or the `reference-data/sql` directory replaces it.

---

## J. Prioritized findings

### Critical

| Finding | Evidence | Impact | Recommended action |
|---|---|---|---|
| Wrong DDL execution order | Customer FKs precede `customer_details` in `DDL_SCHEMA.md` | Schema creation fails | Move both ALTER statements after customer creation or to the final deferred-FK section |
| Corrupted city/district mapping | Generator uses `dlist[0]` fallback; hundreds of incorrect cities | Wrong addresses, DISCOMs, routing and visit charges | Remove fallback; add verified alias map; fail generation on unmatched cities |
| Subsidy rules are nondeterministic | Overlapping 0–2 and 2–3 ranges; amount semantics unclear; active zero placeholder | Wrong financial estimates | Redesign rule representation and approve formula before implementation |

### High

| Finding | Evidence | Impact | Recommended action |
|---|---|---|---|
| Import is non-atomic/non-idempotent | Plain inserts, no transaction/upsert | Partial or unrecoverable seed state | Add transaction and explicit rerun strategy |
| Visit bands overlap | 0–10, 10–25, 25–50, 50–999 all inclusive | Multiple prices match boundary distances | Use non-overlapping half-open ranges and exclusion constraint |
| DDL is not an implementation | `01_schema.sql`, `02_functions.sql` missing | Business controls cannot be verified | Do not deploy until approved executable files exist |
| Dynamic dropdown SQL not proven safe | Raw table/column/predicate metadata; function missing | SQL injection or malformed queries | Implement strict identifier whitelist and bound values |
| Statewide DISCOM filtering ambiguous | Parent filter plus OR predicate | Statewide utilities can disappear | Use one filtering mechanism and test Maharashtra district results |
| Insufficient indexes | ~179 FKs vs ~42 secondary indexes | Slow admin queues and customer lookups | Add workload-driven indexes before production |
| Full payload logging | Request/response JSONB logs | PII, OTP, token leakage | Define redaction and retention policy in functions |
| Hard-coded DB password | `DB-Setup.sql` contains a plaintext password | Credential disclosure | Replace with deployment secret/psql variable |
| Active placeholder office | `"Configure street address in admin"` | Incorrect distance fees | Require verified office address/coordinates before activation |
| Soft-delete vs full unique constraints | Many business keys use full UNIQUE | Cannot recreate logically deleted codes/numbers | Convert selected keys to partial active uniques |

### Medium

- Geo city/district/state combinations are not cross-validated.
- Payment customer may differ from invoice customer.
- Notification entity type and ID may become orphaned.
- Follow-up rows may populate more than one parent.
- Site visits have many optional parents without combination rules.
- Lead mobile has no format CHECK.
- Customer GSTIN has no format CHECK.
- Only 70 PINs are available.
- Calculator parameters and AMC prices lack effective dates.
- TPADL/Jamshedpur DISCOM record needs authoritative verification.
- One-default-office and one-default-site rules are function-only.
- `patch-ddl-comments.mjs` removes comments globally before regenerating from regex-matched tables and lacks an atomic write/completeness guard.
- Source files lack version/provenance metadata.
- JSON shape differs from direct table shape without a formal manifest.

### Low

- `SUMMARY.md` says files 01–13 instead of 01–15.
- An unused legacy district source is present without documentation.
- `DB_REQUIREMENTS.md` labels both `fn_get_*` and `fn_log_*` as “Logging Functions.”
- Notes refer to an outdated `Solar-DB/BRD.md` path and 10 modules rather than 13.
- Some table-count wording in design documentation is stale.
- Several comment descriptions still refer to exactly four milestones despite configurable counts.

---

## Final recommendation

**DO NOT APPROVE**

Minimum approval conditions:

1. Correct the city/district generation process and regenerate dependent PIN data.
2. Resolve subsidy and visit-charge boundary semantics.
3. Correct DDL creation order.
4. Decide Path A/`admin_registered` behavior.
5. Add transactional, rerunnable seed handling.
6. Produce ordered `01_schema.sql` and required function definitions.
7. Review missing operational indexes and sensitive-log handling.
8. Replace production placeholders and verify questionable utility records.

No remediation was performed as part of this audit.
