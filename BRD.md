# Business Requirements Document (BRD)
## Solar Service Management Mobile Application (Solar-App)

---

## 1. Executive Summary & Product Vision

### 1.1 Objective
The Solar Service Management Mobile Application is a comprehensive digital platform designed to streamline solar operations—from Play Store self-onboarding via the product catalogue, through orders and installation, to AMC/complaint servicing. **Billing and payment collection happen offline**; the app only lets **admin record and update** that information (customers do not pay inside the app).

### 1.2 Target Audience & Core Personas
1. **End Customer:** Residential, society, or commercial solar installation owners who browse the catalogue, place enquiries, track orders/installation, schedule service, and raise complaints. They may **view** recorded billing/payment status; they do **not** pay in-app.
2. **Field Technician / Service Engineer:** On-ground technicians who receive ticket assignments, complete preventive maintenance checklists, capture site proof, and request spare parts.
3. **Sales & Survey Agent:** Field representatives who capture lead details, execute site feasibility assessments, generate instant quotes, and follow up product enquiries.
4. **System Administrator:** Back-office operators who maintain the catalogue, register users (including **post-installation** onboarding), manage orders through installation, **enter offline billing and payment**, and maintain service information per customer/product/installation.

---

## 2. Core Modules & Detailed Feature Matrix

---

### Module 1: Authentication & User Profile Management

#### 1.1 Mobile Number & OTP Authentication Screen
* **Functionality:** Secure passwordless access for customers and field agents using mobile OTP verification.
* **Feature Breakdown:**
  * **Country Code & Phone Input:** Standard 10-digit mobile number input with validation.
  * **OTP Dispatch & Resend Logic:** Automatic SMS trigger with a 60-second cooldown timer.
  * **Auto-Read OTP:** SMS auto-fill permission handling on Android devices.
  * **PIN/Biometric Quick Login:** Option to set a 4-digit PIN or enable Fingerprint/FaceID for subsequent app launches.
* **Business Rules:**
  * Unregistered mobile numbers will be redirected to the Lead Registration / catalogue enquiry pipeline (Onboarding Path A).
  * Users created by admin after installation (Onboarding Path B) can log in immediately with the registered mobile.
  * Max 3 invalid OTP attempts allowed per hour before temporary lockout (15 minutes).

#### 1.3 Two onboarding paths (mandatory)

**Path A — Customer self-onboards from Play Store**
1. Download app → OTP → browse **product catalogue** → **place enquiry**.
2. Admin/sales convert enquiry → lead/order and process through **installation / commissioning**.
3. **Billing and payment are collected offline**; admin later **records** amounts, dates, and payment references in the app against the customer/order.
4. Admin also maintains **service** data (free visits, AMC, history) against that customer, site, product, and installation.

**Path B — Admin registers the user after installation**
1. Installation is completed in the field **without** the customer having used the app.
2. Admin **creates the user** (mobile, profile, role Customer), customer, site, equipment (linked to catalogue product where known), and order in **Installed/Commissioned** status.
3. Admin **enters offline billing and payment** records and **service information** (milestones, AMC, completed visits).
4. Customer later logs in with OTP on the same mobile and sees sites, service, tickets, and recorded billing/payment (read-only).

#### 1.2 User Profile & Role Switching Screen
* **Functionality:** Displays active user attributes, associated site details, and language preferences.
* **Feature Breakdown:**
  * **Multi-Property/Site Selector:** Switch between different installed sites (e.g., "Home - 5kW" vs "Factory - 50kW") under one customer account.
  * **Profile Editing:** Update secondary contact numbers, communication email, and emergency contacts.
  * **App Preferences:** Toggle language (English/Marathi/Hindi) and master **push notification** on/off.
  * **Notification inbox (Module 13):** Bell/list of in-app notifications with unread badge; tap opens the related product, enquiry, ticket, service, or order.

---

### Module 2: Interactive Solar Calculator & Lead Capture

#### 2.1 Feasibility Calculator Screen
* **Functionality:** An interactive, instant estimation engine for potential leads and customers considering capacity expansion.
* **Feature Breakdown:**
  * **Monthly Electricity Bill Slider:** Interactive slider ranging from ₹500 to ₹1,000,000+.
  * **Rooftop Area Input:** Numerical input for available shade-free area (sq. ft. or sq. m.).
  * **Connection Type Selector:** Radio selection (`Residential Individual`, `Housing Society`, `Commercial/Industrial`).
  * **Instant Estimation Summary Card:**
    * Recommended Solar Capacity (in kW).
    * Estimated Project Cost (Gross).
    * Estimated Government Subsidy Amount (DBT calculation under PM Surya Ghar Scheme).
    * Net Investment Cost & Estimated Monthly / Annual Savings.
    * Payback Period (in years/months) and CO₂ Reduction Metrics.
* **Business Rules:**
  * Subsidy calculations must automatically apply current state/central slab rules (e.g., 3kW cap for maximum residential subsidy).
  * Area validation: Minimum 100 sq. ft. per 1 kW system required; display warning if rooftop area is insufficient for target bill reduction.

#### 2.2 Instant Quote Generation & Lead Conversion Screen
* **Functionality:** Converts calculator estimates into a formal PDF quotation and captures lead parameters.
* **Feature Breakdown:**
  * **Lead Details Form:** Full Name, Installation Pincode, Preferred Contact Time.
  * **Schedule Site Visit:** Date & slot. Purpose defaults to enquiry/feasibility. Charge is calculated from office distance (Module 2.3); pre-sale visits are typically chargeable unless a ₹0 slab applies.

#### 2.3 Site visits (all types) — free vs chargeable
* **Functionality:** Every field visit is a `site_visit` row: enquiry/feasibility, installation, servicing, complaint, or commissioning.
* **Feature Breakdown:**
  * **Purpose** from `SITE_VISIT_PURPOSES`.
  * **Assign** any authorised app user (technician, sales, admin/other). Technician execution may also create a job.
  * **Distance from office:** km from configured office lat/long to site/pincode GPS.
  * **Charge:** Configurable distance slabs (`ref_visit_charge_slabs`). Enquiry, installation, complaint, and extra service visits are chargeable unless a free entitlement applies.
  * **Free servicing:** Count and optional km radius are configurable globally (`FREE_SVC_VISIT_COUNT`, `FREE_VISIT_RADIUS_KM`) and per AMC plan (`free_visit_count`, `free_visit_radius_km`). Consuming a free milestone/AMC credit sets `is_free_entitlement_ind`.
  * **Status:** Scheduled → Assigned → In progress → Completed / Cancelled / No-show.
  * **Charges received:** Admin records offline collection (`charges_received_ind` + optional invoice/payment). No in-app gateway.
* **Business Rules:**
  * Charge amount is computed in SQL functions (no triggers) from office + slab + remaining free credits.
  * Completing a visit does not automatically mean money was received; that is a separate update.
  * **PDF Quote Preview & Download:** One-click generation of branded PDF quote with itemized component breakdown (Panels, Inverter, Structure, Balance of System).

---

### Module 3: Customer Onboarding & Site KYC Management

#### 3.1 Customer Profile Setup Screen
* **Functionality:** Captures detailed technical and legal attributes for sanctioning and installation.
* **Feature Breakdown:**
  * **Consumer Account Number Details:** Electricity Provider (DISCOM) selection, Consumer Number, Meter Number, Sanctioned Load (kW).
  * **Address & Geolocation Tagging:** Address fields with auto-capture of latitude/longitude coordinates via mobile GPS for technician routing.

#### 3.2 Document Upload & Verification Screen
* **Functionality:** Upload required KYC and DISCOM approval documents.
* **Feature Breakdown:**
  * **Document Checklist:**
    1. Aadhaar Card / PAN Card (Identity Proof).
    2. Latest Electricity Bill (Address & Sanctioned Load Proof).
    3. Property Ownership Proof / NOC (Tax receipt or agreement).
    4. Roof Site Photos (Pre-installation).
  * **Camera & Gallery Integration:** Direct photo capture with automatic image compression (max 2MB per upload).
  * **Verification Status Badge:** Real-time badge indicating document review state (`Pending Verification`, `Approved`, `Action Required / Rejected`).
* **Business Rules:**
  * Rejection requires mandatory comment entry by admin (e.g., "Electricity bill unreadable, re-upload").
  * **Path B:** If admin registers the customer after installation, KYC may be uploaded later; site and equipment can be created first so service can start.

---

### Module 4: Asset & Installation Tracking

#### 4.1 Site Equipment Register Screen
* **Functionality:** Digital twin of installed solar equipment for warranty and servicing tracking.
* **Feature Breakdown:**
  * **Solar Panel Specs:** Brand, Model, Serial Numbers (Barcode/QR Code scanner input), Total Quantity, Individual Panel Wattage.
  * **Inverter Specs:** Inverter Serial Number, Capacity (kW), Phase Type (Single Phase / 3 Phase), Monitoring Dongle ID.
  * **Structure & Battery Details:** Mounting Structure Type (GI High-Rise, Rooftop Flush), Battery Bank details (if Hybrid/Off-grid).
  * **Warranty Card Vault:** Digital repository of panel linear performance warranties (25 years) and inverter warranties (5–10 years).
  * **Catalogue product link:** Where known, each asset is linked to the catalogue `product` so service plans follow the sold/installed SKU.
  * **Admin (both onboarding paths):** Create/update equipment and commissioning date after installation without waiting for the customer to use the app.

---

### Module 5: Preventive Service Management & AMC Operations

#### 5.1 Service Milestone Dashboard Screen
* **Functionality:** Tracks free service schedules and active Annual Maintenance Contracts (AMC).
* **Feature Breakdown:**
  * **Free Service Timeline:** Configurable number of free servicing visits post-commissioning (default 4; not hard-coded).
  * **AMC Status Banner:** Active plan details (`Basic Cleaning`, `Comprehensive Breakdown Included`), Expiry Date, and **Request renewal** (creates a follow-up / service request for admin — **not** in-app payment).
  * **Service History List:** Log of past completed services with downloadable Service Reports signed by field engineers.

#### 5.2 Service Request & Booking Screen
* **Functionality:** Customer requests a routine cleaning or preventive health checkup.
* **Feature Breakdown:**
  * **Service Type Selection:** Routine Panel Cleaning, Complete System Inspection, Inverter Health Check.
  * **Preferred Slot Selector:** Select date and morning/afternoon time window.
  * **Technician Allocation Tracker:** Displays assigned technician name, photo, phone number, and real-time arrival status.
* **Business Rules:**
  * Free services must be claimed within 30 days of the due milestone date; otherwise, status updates to `Lapsed`.

#### 5.3 Admin — Service Information per Customer
* **Functionality:** Admin maintains service setup **against each customer**, aligned to **product, installation/site, and order**.
* **Feature Breakdown:**
  * Create/adjust free servicing milestones from `commissioning_on` (count from AMC plan or `FREE_SVC_VISIT_COUNT`).
  * Attach or change AMC plan, start/expiry.
  * Record completed or historical services (type, date, technician, report) that happened offline or before the customer had the app (typical Path B).
  * Link service rows to catalogue product and/or order when relevant.
* **Business Rules:**
  * Customers can view and book (Module 5.1–5.2); they cannot overwrite admin-entered history except by raising a new request.
  * Path B customers should see a populated service dashboard on first login.

---

### Module 6: Complaint Ticketing & Breakdown Resolution

#### 6.1 Raise Complaint Ticket Screen
* **Functionality:** Rapid escalation path for system faults, zero generation, or physical damage.
* **Feature Breakdown:**
  * **Issue Category Dropdown:** (`Zero Generation / Inverter Red Light`, `Physical Panel Damage`, `Structure Loose / Water Leakage`, `Generation Lower Than Expected`, `Billing / App Connection Issue`).
  * **Severity Indicator:** Low, Medium, High, Emergency (e.g., Sparking/Fire Risk).
  * **Problem Description & Media Attachment:** Text box + ability to attach up to 4 photos/videos of error codes on inverter screens.
  * **Auto-Ticket Number Generation:** Instant creation of unique tracking ID (e.g., `TKT-2026-0892`).

#### 6.2 Ticket Lifecycle & Live Status Screen
* **Functionality:** End-to-end audit trail of complaint resolution.
* **Feature Breakdown:**
  * **Status Stepper:** `Ticket Raised` $\rightarrow$ `Assigned to Engineer` $\rightarrow$ `Technician En-Route` $\rightarrow$ `On-Site Inspection` $\rightarrow$ `Parts Replaced` $\rightarrow$ `Resolved & Closed`.
  * **In-App Communication:** Call or chat directly with assigned service engineer.
  * **Customer OTP Resolution Verification:** Customer must provide a 4-digit OTP to the technician on-site to officially mark the ticket as `Resolved`.

---

### Module 7: Field Technician Workspace (Mobile Workflow)

#### 7.1 Job Queue & Task List Screen
* **Functionality:** Daily dashboard for field engineers showing assigned routine services and emergency breakdown tickets.
* **Feature Breakdown:**
  * **Map Route Optimization:** Overview map displaying all assigned site visits ordered by distance/time priority.
  * **Job Acceptance:** Accept/Reschedule job assignments with reason capture.

#### 7.2 Service Execution & Inspection Checklist Screen
* **Functionality:** Digital job sheet completed on-site before sign-off.
* **Feature Breakdown:**
  * **Mandatory Checklist Items:**
    * Panel VOC / ISC Voltage measurement checks.
    * AC/DC Distribution Box wiring & Earthing check.
    * Physical panel cleaning verification.
    * Inverter firmware & Wi-Fi logger check.
  * **Before / After Photo Capture:** Mandatory upload of clean panel photo and inverter running status screen.
  * **Customer Signature Pad:** On-screen digital signature field for customer sign-off.

---

### Module 8: Offline Billing & Payment Register (Admin-maintained)

There is **no in-app payment gateway**, no UPI/card checkout, and no customer “Pay now” flow. Collection happens **outside** the app. The app stores what admin types in after the fact.

#### 8.1 Customer — Billing & Payment View (read-only)
* **Functionality:** Registered customers can see recorded bills and payments for their sites/orders (amount, date, status Paid/Partial/Unpaid, offline method, reference such as cheque/UTR).
* **Business Rules:**
  * Customers cannot create, edit, or delete billing/payment rows.
  * Absence of records is a valid empty state until admin enters them.

#### 8.2 Admin — Record / Update Offline Billing
* **Functionality:** Admin captures a bill against a customer, order, site, AMC, or spare request (categories unchanged: installation, AMC, spares, cleaning).
* **Feature Breakdown:**
  * Header: bill number (manual or generated), date, taxable/total amounts, optional GST split if known, status.
  * Line items optional (description, qty, rate) when admin has a paper invoice to transcribe.
  * Link to `order` when the bill is for Path A fulfilment or Path B post-install registration.
* **Business Rules:**
  * GST/HSN fields are optional; offline bills may be a simple total.
  * No statutory e-invoice / IRN in v1.

#### 8.3 Admin — Record / Update Offline Payment
* **Functionality:** Admin logs money received outside the app.
* **Feature Breakdown:**
  * Method: Cash, Cheque, Bank transfer / NEFT/RTGS, UPI collected outside app, Other.
  * Instrument reference (cheque number, UTR, receipt number), amount, paid-on date, remarks.
  * Partial payments allowed (running `paid_amount` / `balance_amount` on the bill).
* **Business Rules:**
  * Every payment row is `is_offline_ind = TRUE`. Gateway IDs are not used.
  * Updating payment must recalculate bill status (Unpaid / Partial / Paid) inside SQL functions.

---

### Module 9: Dynamic Master & Dropdown Configuration Engine

#### 9.1 Unified Dynamic Dropdown System
* **Functionality:** Core architecture requirement where ALL dropdown selections across the application are fed by a single generic API endpoint (`fn_get_dropdown_data`).
* **Feature Breakdown:**
  * **Single Source Config:** State lists, City lists, DISCOM names, Complaint Categories, Equipment Brands, and Service Types managed via centralized database reference tables (`ref_data_categories` and `ref_data`).
  * **Cascading / Dependent Dropdowns:**
    * Selecting a State (e.g., *Maharashtra*) dynamically filters the District/City dropdown without app updates.
    * Selecting a District dynamically populates the local Electricity DISCOM branches (e.g., *MSEDCL*, *Tata Power*).
* **Business Rules:**
  * All dropdown responses must be cached locally in the Flutter app state to allow smooth offline-first dropdown rendering.

---

### Module 10: System Logging, Sessions & Audit Trail

#### 10.1 Centralized Database Logging & Audit Engine
* **Functionality:** Full observability of system interactions and exception tracking.
* **Feature Breakdown:**
  * **API & SQL Execution Logging:** Every API call passes `api_endpoint` and `logged_in_user_id` into database logging tables (`log_api_execution`, `log_sql_functions`).
  * **User Session Auditing:** Captures user login history, IP address, device model, and active session tokens in `user_session_details`.
  * **Graceful Exception Mapping:** All database execution errors return standardized JSON payloads, enabling friendly error UI popups (e.g., *"System under scheduled maintenance, please retry in 5 minutes"* instead of raw crash screens).

---

### Module 11: Product Catalogue & Product Enquiry

#### 11.1 Admin Product Catalogue Management Screen
* **Functionality:** Administrators maintain a sellable / related-product catalogue (panels, inverters, structures, batteries, AMC add-ons, accessories, and other solar-related SKUs). Newly published products become visible to registered app users without a mobile app release.
* **Feature Breakdown:**
  * **Product Master Form:** Product code, name, category, short and detailed description, specification attributes, list price (INR), HSN/SAC, thumbnail, and publish/unpublish toggle.
  * **Media Gallery:** Multiple photos/videos/spec-sheets per product (camera/gallery or file upload).
  * **Display Order:** Sort order so featured products appear first in the catalogue list.
* **Business Rules:**
  * Draft products (`Unpublished`) are visible only to administrators.
  * Publishing a product **enqueues** `NTF_CATALOGUE_NEW` for registered customers (Module 13).
  * Soft-deleted or unpublished products must not appear in the customer catalogue.
  * Product code must be unique among active catalogue items.

#### 11.2 Registered User Catalogue & Product Detail Screen
* **Functionality:** Registered customers and field/sales users browse published products and open a product detail page.
* **Feature Breakdown:**
  * **Catalogue List:** Filter by category; cards show thumbnail, name, short description, and list price.
  * **Product Detail:** Full description, specification table, media gallery, and price.
* **Business Rules:**
  * Unregistered users (`admin_registered = FALSE`) are not shown the catalogue; they are directed to OTP registration / lead capture first.
  * Prices displayed are indicative list prices; a formal quote may still be generated via the calculator/quote flow.

#### 11.3 Place Product Enquiry Screen
* **Functionality:** From product detail, a registered user submits an enquiry that sales/admin can follow up (and optionally convert to a lead).
* **Feature Breakdown:**
  * **Enquiry Form:** Message / requirement, preferred contact window, optional pincode/site context.
  * **Enquiry Tracking:** Unique enquiry number and status (`Open`, `In Progress`, `Converted to Lead`, `Closed`).
* **Business Rules:**
  * Enquiries are always tied to one catalogue product and the logged-in user.
  * Closing or converting an enquiry requires a sales/admin actor; conversion may create or link a `lead` / **order** without deleting the enquiry history.
  * **Path A:** Enquiry → order processed through installation; **do not** require billing/payment in the app before installation. Admin records billing/payment later (Module 8).

---

### Module 12: Back-office Admin Console

Admin does **not** use a separate copy of operational data. The console reads and updates the same transaction tables as the mobile app, with `ROLE_ADMIN` (and assigned sales agents where noted). This module defines the admin **screens and duties**; new tables are only introduced where the existing model had no entity (orders and follow-up tasks).

#### 12.1 User & Role Management Screen
* **Functionality:** Create/update customers, technicians, sales agents, and other admins; suspend or reactivate accounts; maintain technician employee code and availability.
* **Feature Breakdown:**
  * Search by mobile, name, role; filter active / suspended.
  * Assign `USER_ROLES`; set **registration source** (`SELF_APP` vs `ADMIN_POST_INSTALL`).
  * **Path B wizard:** After installation, admin creates user + customer + site + equipment (product-linked) + commissioned order in one guided flow, then billing and service (12.8–12.9 / 5.3).
* **Business Rules:**
  * Role changes and suspend/reactivate are admin-only SQL functions.
  * Soft-delete (`active_ind = FALSE`) hides the user from login; suspend keeps the row visible in admin with login blocked.
  * Path B users are `admin_registered = TRUE` so they skip the lead-only funnel on first OTP. Admin-created technicians/sales/admins are also `admin_registered = TRUE`.

#### 12.2 Catalogue Maintenance Screen
* **Functionality:** Same capabilities as Module 11.1 (create/edit/publish products and media). Listed here as the admin entry point for catalogue.

#### 12.3 Order Management Screen
* **Functionality:** After a quote is accepted or an enquiry/lead is converted to a commercial job, admin tracks the **order** through fulfilment (confirmed → material → installed → commissioned / cancelled).
* **Feature Breakdown:**
  * Order list with status filter, customer/site, source (quote / enquiry / manual).
  * Order detail: line items (catalogue product and/or free-text), amounts, linked quote/lead/enquiry, assigned owner.
  * Status updates with remarks.
* **Business Rules:**
  * An order is distinct from an **offline bill** (Module 8) and from installed `equipment_details`.
  * Path A: order can reach Installed/Commissioned **before** any bill/payment row exists.
  * Path B: admin may create the order already in Installed/Commissioned.
  * Manual orders may be created by admin without a prior quote.

#### 12.4 Follow-up / Task Management Screen
* **Functionality:** Admin and sales capture scheduled follow-ups against a lead, product enquiry, order, ticket, service request, or customer (call, visit, WhatsApp, email).
* **Feature Breakdown:**
  * My / team task list: overdue, due today, upcoming.
  * Log outcome and optionally schedule the next follow-up.
* **Business Rules:**
  * A follow-up is always owned by an assigned user.
  * Completing a follow-up does not by itself change lead/ticket status; that remains an explicit status update on the parent entity.

#### 12.5 Lead, Enquiry & KYC Queues
* **Functionality:** Work lists for open leads, open product enquiries, and KYC documents pending verification (Module 3.2 reject-comment rule still applies).
* **Feature Breakdown:** Assign owner, change status, convert enquiry → lead → order.

#### 12.6 Service Request & Service Master Screen
* **Functionality:** Allocate/reschedule bookings **and** (Module 5.3) create/update service master data per customer: milestones, AMC, historical visits, product/order links.
* **Maps to:** `service_request_details`, `service_milestone_details`, `amc_contract_details`, `job_assignment_details`, plus `equipment_details` / `product_details` / `order_details`.

#### 12.7 Complaint Ticket Management Screen
* **Functionality:** Allocate engineers, move the ticket stepper, approve spare-part requests, monitor SLA/severity (especially Emergency).
* **Maps to:** `complaint_ticket_details`, `ticket_status_history_details`, `spare_part_request_details`, `job_assignment_details`.

#### 12.8 Offline Billing & Payment Entry
* **Functionality:** Module 8.2–8.3 — admin transcribes offline bills and payments. Customers only view (8.1).
* **Maps to:** `invoice_details`, `invoice_line_details`, `payment_details`, `payment_milestone_details` (offline register, **not** a payment gateway).

#### 12.9 Reference Data
* **Functionality:** Maintain dropdowns and geo/DISCOM/AMC/catalogue masters via `ref_*` / `ref_data` (Module 9), including **office locations** and **visit charge slabs**. Warehouse stock remains out of v1.

#### 12.10 Site visit queue
* **Functionality:** Module 2.3 — create/assign/complete visits; compute charge from office distance; mark charges received offline.
* **Maps to:** `site_visit_details`, `ref_office_locations`, `ref_visit_charge_slabs`, optional `job_assignment_details` when the assignee is a technician.

---

### Module 13: Notifications (in-app inbox + push)

There is **no** separate notification product. Business functions **enqueue** a row in `notification_details` (and optional push delivery log). **No database triggers.** OTP SMS stays in `log_otp_attempts` (Module 1 / 10), not this module.

#### 13.1 In-app inbox
* **Functionality:** Each user (customer, admin, sales, technician) sees notifications addressed to them; mark read / mark all read.
* **Business Rules:**
  * Respect `user_details.push_notifications_enabled` for **push** only; in-app inbox is still written so the user sees history when they open the app.
  * Suspended or inactive users are not sent push; inbox rows are not created for inactive users.
  * Deep-link payload must name `entity_type` + `entity_id` (product, enquiry, ticket, service request, order, invoice, follow-up, lead).

#### 13.2 Event catalogue (v1 mandatory)

| Event | Who is notified | When (SQL function, not a trigger) |
| --- | --- | --- |
| New catalogue product | All **registered** customers (push on preferred); sales optional | Admin **publishes** a product (`is_published` becomes true) |
| Product enquiry | **Admin** and **sales** | Customer submits enquiry |
| Enquiry / order status | **Customer** | Status change or conversion to lead/order |
| Service reminder | **Customer** (site access); assigned technician if any | Free-service milestone approaching (due − 7 days and due day); AMC expiry − 30 / − 7 days |
| Service request | **Admin**; **technician** when assigned; **customer** on assign/complete | Create / assign / complete SR |
| New complaint | **Admin**; **technician** when assigned | Ticket created |
| Complaint updates | **Customer** and **admin**; technician if assigned | Status change, new ticket message, resolved/closed, spare-part decision |
| Job assigned | **Technician** | Job created/assigned |
| Site visit assigned | **Assignee** (tech/sales/other); **customer** if linked | `assigned_to_user_id` set |
| Site visit completed | **Customer**, **admin** | Status Completed (chargeable vs free in body) |
| KYC reviewed | **Customer** | Approve or reject (reject includes comment) |
| Path B account ready | **Customer** | Admin finishes post-install registration |
| Order / installation status | **Customer**; **admin** on exception/cancel | Order status update |
| Offline bill / payment recorded | **Customer** (view-only) | Admin creates/updates invoice or payment |
| Follow-up due | Assigned **admin/sales** | Due today / overdue (batch) |
| Spare-part request | **Admin** | Technician requests parts |

#### 13.3 Push delivery
* Device `push_token` is stored on the active `log_user_sessions` row. API/worker reads pending `log_notification_delivery` (`PUSH` / `PENDING`) and calls FCM (or equivalent). Failures stay on the log; inbox row remains.
* Reminders are created by scheduled calls to `fn_create_service_reminder_notifications` / `fn_create_followup_due_notifications` (external cron or pg_cron), never by triggers.

---

## 3. Implementation Roadmap & Module Breakdown

To maintain an agile execution flow, modules can be picked up sequentially for implementation in the following order:

```text
 Phase 1: Foundation Setup
 ├── Module 10: System Logging & Session Tables
 ├── Module 9: Dynamic Dropdown Engine (ref_data)
 ├── Module 13: Notification inbox & delivery log
 └── Module 1: Authentication & User Details

 Phase 2: Core Customer & Onboarding Pipeline
 ├── Module 2: Solar Calculator & Lead Management
 ├── Module 11: Product Catalogue & Product Enquiry
 ├── Module 3: Customer Profile & KYC Document Upload
 └── Module 4: Asset & Equipment Registry

 Phase 3: Service Engine & Technician Workflow
 ├── Module 5: Preventive Service & AMC Management
 ├── Module 6: Complaint Ticketing System
 └── Module 7: Field Technician Mobile App

 Phase 4: Admin Controls
 ├── Module 12: Back-office Admin Console (incl. Path B registration)
 └── Module 8: Offline Billing & Payment Register (admin entry, customer view)