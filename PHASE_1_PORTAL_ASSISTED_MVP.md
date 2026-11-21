# Phase 1 MVP — Portal-Assisted Travel Agency Operations

## Confirmed operating model

The agency buys/issues tickets through an external consolidator using that consolidator's website/desktop portal. For development and screenshots, use the fictional placeholder name **Demo Consolidator**; the real supplier name is intentionally not required yet. The agency does **not** currently have API credentials or authorized API access. Staff can download booking/ticket reports as PDF and Excel files.

Therefore Phase 1 is a **portal-assisted, staff-operated workflow**:

- Staff search/book/issue/refund/exchange in the consolidator's own portal.
- Staff record the resulting PNR, ticket, amount, supplier reference, and supporting documents in this ERP.
- Staff can upload consolidator PDF and Excel downloads into an import inbox. The app parses supported files into a staging preview, maps columns, flags duplicates and invalid rows, and requires staff approval before creating/updating ERP records.
- The system manages enquiries, quotations, approval, passenger records, payment collection, supplier payable, document control, after-sales requests, and reconciliation.
- There is no direct live flight search, booking, ticket issuance, refund, exchange, or portal automation in Phase 1.

Do not use browser scraping, reverse-engineered endpoints, credential sharing, or automated clicks against the consolidator portal. The approved first integration is staff-initiated upload of downloaded PDF/XLSX/CSV reports. Only add a live API integration if the provider gives explicit written authorization and supported API access.

## Phase 1 goal

Use the [International Document & UI Design Standard](TRAVEL_AGENCY_DOCUMENT_DESIGN_STANDARD.md) and [Print Format Catalogue](docs/print-formats/README.md) as the presentation baseline. These are design specifications, not implemented/rendered Print Format records yet.

Give staff one reliable place to manage an air-ticket customer from enquiry to issued e-ticket and financial settlement, while also supporting visa cases and basic Umrah/tour package records. Keep flight-provider work outside the app until authorized access exists.

## In-scope modules

### A. Shared customer and passenger records

- ERPNext Customer remains the financial/customer master.
- A separate Traveller Profile stores passenger information and links to Customer.
- Support one customer booking for multiple travellers and one traveller appearing in multiple bookings.
- Capture only necessary identity/travel data; protect passport number and private document attachments with role permissions.
- Keep passport scans out of public attachments, list views, routine exports, and notification messages.
- Record consent/authorization for handling passenger documents where the agency's process requires it.

### B. Air ticketing — manual portal assisted

Recommended workflow:

1. Enquiry received and assigned to an agent.
2. Staff records requested route, dates, passenger count, cabin, baggage needs, and flexibility.
3. Staff obtains a quote from the consolidator portal and records provider, quoted fare, currency, expiry/time limit, fare conditions, baggage, cancellation/change rules, and notes.
4. Quote is sent to customer. Discount or margin outside configured limits requires approval.
5. Customer accepts and pays the required advance (or receives approved credit).
6. Staff books in the consolidator portal. The ERP records PNR/locator, provider, booking time limit, itinerary/segments, and portal reference.
7. Staff separately confirms that ticket issuance has completed in the portal.
8. Staff records each issued ticket number, passenger association, airline/validating carrier, issue date, base fare/taxes/fees, consolidator cost, agency service fee/markup, and uploads the e-ticket to a private attachment area.
9. Staff verifies passenger name, itinerary, ticket status, and amount against the consolidator portal before marking the ticket issued.
10. ERPNext invoice/receipt and supplier payable are reconciled to the booking reference.

A PNR or reservation confirmation must never automatically imply that an e-ticket has been issued.

### C. Air booking and ticket statuses

Keep reservation and ticket states distinct.

**Reservation status:**
- Draft
- Quoted
- Awaiting Customer
- Awaiting Payment / Credit Approval
- Ready to Book
- Booked / PNR Held
- Expired
- Cancelled

**Ticket status:**
- Not Issued
- Issuance Pending
- Issued
- Void Requested
- Voided
- Refund Requested
- Partially Refunded
- Refunded
- Exchange Requested
- Exchanged
- Rejected / Failed (with explanation)

Any transition to Issued, Voided, Refunded, or Exchanged requires a staff member to capture evidence/reference and the appropriate approval. Never auto-mark a financial outcome solely from a user-entered note.

### D. After-sales / service cases

Use one service-case record per void, refund, exchange/reissue, cancellation, or name correction request:

- Original booking, PNR and ticket(s).
- Customer request and timestamp.
- Consolidator portal reference and status.
- Airline/consolidator penalty, refund estimate, actual refund amount, currency, and fees.
- Approval trail and supporting documents.
- Customer refund due, supplier credit due, amount received, and accounting reference.
- Completion evidence and final communication.

Do not promise or calculate a guaranteed airline refund before the provider confirms its rules and amount.

### E. Visa operations

- Visa enquiry and applicant/case records.
- Configurable country and visa-type document checklist.
- Document pending, ready for submission, submitted, under review, approved, rejected, passport returned, and closed statuses.
- Appointment/submission date, expected update date, fees, supplier/embassy charges, service fee, and reminders.
- Private passport/financial documents and strict role-level access.

The system tracks agency workflow; it does not represent itself as an embassy or guarantee approval.

### F. Umrah / tours — basic operational MVP

- Package catalogue, departure, itinerary, inclusions/exclusions, capacity, and supplier cost components.
- Group booking, passenger manifest, rooming list, and instalment/balance tracking.
- Hotel/transport/visa/air service supplier references and private vouchers.
- Cancellation and refund requests with approval and accounting linkage.

Hajj-specific quotas and regulated processes remain out of scope until the agency's authorized operating arrangements are confirmed.

### G. Finance and reconciliation

Use standard ERPNext financial documents; do not create a second ledger.

For each booking, link:
- Customer quotation / Sales Order where adopted.
- Sales Invoice and Payment Entry.
- Supplier/consolidator Purchase Invoice or payable record.
- Supplier Payment Entry.
- Credit note/refund or adjustment documents.
- Cost Center / branch / salesperson dimensions as agreed.

Keep these amounts separate:
- Customer sale amount.
- Consolidator/supplier cost.
- Agency service fee.
- Markup/commission.
- Airline taxes/fees.
- Refund/penalty/exchange difference.

**Accounting policy must be signed off by the agency accountant before automatic posting.** Whether the agency recognizes ticket gross value or only its commission/service fee depends on its principal/agent arrangement and applicable rules. Do not guess.

### H. Consolidator file import (PDF / Excel)

Use a controlled import inbox with a fictional, configurable supplier profile named **Demo Consolidator** until the real supplier name is needed.

Supported file paths:
- XLSX: parse workbook sheets and rows; let staff select the correct sheet and header row.
- CSV: parse rows with delimiter/encoding detection, then preview before import.
- PDF with selectable text and a supported, stable report layout: extract text/table candidates and map fields for review.
- Scanned/image-only PDF: do not silently OCR and import. Show a clear “manual review required” state; staff may key the values or a later explicitly enabled OCR process may propose values with confidence and a human verification step.

Import workflow:
1. Staff downloads the report from the consolidator portal and uploads it to a private Import Batch.
2. Select report type: Booking Report, Ticket Report, or Supplier Statement.
3. Select supplier profile (initially Demo Consolidator) and file type; select sheet/header where applicable.
4. Preview extracted rows and source columns. A mapping profile proposes field matches; staff can correct the mapping.
5. Validate data types, required fields, currency, dates, PNR/ticket identifiers, passenger names, and booking references.
6. Detect duplicates using supplier + source reference + PNR/ticket number + report date; do not treat a matching passenger name alone as a unique key.
7. Show row-level outcomes: Ready, Warning, Duplicate, Invalid, or Manual Review Required. Keep source file, page/sheet and row references for traceability.
8. Staff approves the batch and eligible rows. Import must be idempotent: re-uploading the same file/report must not create duplicate bookings, tickets, or ledger postings.
9. Create/update operational records first. Financial documents remain draft or require a separately authorized finance action; imported supplier statement lines must not post journals automatically.
10. Store the import user, timestamps, source-file hash, mapping profile version, row decisions, errors, and audit history.

Security and correctness:
- Store original files as private attachments with least-privilege access and retention rules.
- Reject password-protected or unsupported files with a clear message; never ask users to upload consolidator credentials or OTPs.
- Treat file contents as untrusted. Do not execute Excel macros, formulas, embedded scripts, or external links.
- Apply upload size/page/row limits and safe parser timeouts; validate file signatures rather than trusting file extensions alone.
- Extract only fields needed for operations and reconciliation; do not expose passport data in report imports unless genuinely required.
- Never mark a ticket Issued from a PDF row alone unless required ticket identifiers and a staff verification step are satisfied.
- Preserve original values alongside normalized values so a staff member can audit every transformation.

### I. Staff controls

- Roles: Owner/Admin, Ticketing Agent, Visa Officer, Package Operations, Accounts, Branch Manager, Read-only Auditor.
- Approval limits for discounts, credit sales, ticket issuance override, void/refund/exchange, write-offs, and supplier payment.
- Audit log for sensitive field updates and all status/financial changes.
- Branch, salesperson, and booking filters.
- No staff member may erase or overwrite an issued-ticket history; corrections are append-only with a reason and approval.

## Proposed custom DocTypes

| DocType | Purpose |
| --- | --- |
| Traveller Profile | Passenger profile linked to ERPNext Customer |
| Travel Enquiry | Customer request and assignment |
| Travel Quotation | Versioned offer, expiry, fare conditions and approval |
| Travel Booking | Booking master and service category |
| Booking Passenger | Many-to-many traveller/booking relation |
| Air Reservation | PNR/locator, portal provider, time limit and itinerary |
| Air Ticket | Individual ticket number and issue evidence |
| Ticket Service Case | Void/refund/exchange/reissue request |
| Visa Case | Applicant checklist, deadlines, fees and status |
| Travel Package | Package definition and price components |
| Group Departure | Departure date, capacity, manifest and rooming |
| Hotel Booking | Supplier reference, room, dates and voucher |
| Travel Supplier | Consolidator, hotel, transport, visa and tour supplier |
| Agent Commission | Commission calculation, approval and settlement |
| Provider Activity Log | Manual portal reference and (later) authorized integration events |
| Import Batch | Uploaded file, supplier profile, report type, hash, parser status, approval and audit trail |
| Import Batch Row | Extracted row, source location, mapping, validation results and import decision |
| Import Mapping Profile | Versioned column/header mapping for Demo Consolidator report formats |

Standard ERPNext DocTypes remain the source of truth for customer/supplier accounting, invoices, payment entries, journals, currencies, taxes, and financial statements.

## First build sequence

### Sprint A — foundation and access control
- Create a separate custom Frappe app repository named `travel_agency` (do not place its source inside ERPNext core).
- Pin a tested compatible Frappe/ERPNext version pair for development and staging; do not deploy from moving `develop`.
- Install ERPNext and Frappe CRM on the same Frappe site where possible.
- Configure company, branch, currencies, fiscal year, cost centers, roles, and private file storage.
- Create Traveller Profile, Travel Enquiry, Travel Quotation, Travel Booking, and Booking Passenger.
- Add basic tests for permissions, document linking, and state transitions.

### Sprint B — consolidator document import foundation
- Add a configurable fictional supplier profile named Demo Consolidator.
- Create Import Batch, Import Batch Row, and versioned Import Mapping Profile records.
- Support XLSX and CSV preview/mapping/validation.
- Support text-based PDF report extraction for the agreed sample layout; scanned PDFs go to manual review.
- Add duplicate detection, source traceability, approval, idempotent re-import, and error downloads.
- Add sample CSV templates that staff can open in Excel.
- Test malformed files, unexpected columns, duplicate rows, partial failures, access permissions, and safe re-import.

### Sprint C — portal-assisted air ticket operations
- Create Air Reservation, Air Ticket, and Ticket Service Case.
- Implement separate PNR and ticket status lifecycles.
- Add a staff checklist to record the external portal action and verify data before status changes.
- Add private e-ticket attachment, print/email templates, and booking/ticket search.
- Add approval and audit trail for issuance, void, refund, and exchange.
- Test duplicate PNR/ticket detection and multi-passenger booking support.

### Sprint D — accounting, visa and package basics
- Link each booking to ERPNext customer invoices, receipts, supplier payables, and payments.
- Add finance mapping after accountant review.
- Add Visa Case, document checklist, reminders, and status dashboard.
- Add Travel Package, Group Departure, passenger manifest, rooming list, and supplier vouchers.
- Test customer balance, supplier balance, commission, profit calculation, cancellation, and refund cases.

## Acceptance tests for Phase 1

13. Customer-facing and internal print formats follow the document catalogue, use unambiguous dates/currency/airport codes, and pass A4 print/pagination tests.
14. Original issuer e-ticket documents remain unaltered; agency summaries are clearly labelled and never impersonate official tickets or visas.
15. Sensitive fields are omitted or permission-filtered by document audience.
1. Staff can create a customer enquiry and prepare multiple quote versions.
2. A quote can be approved, accepted, expired, or rejected with an audit trail.
3. One booking supports multiple passengers and retains a stable booking reference.
4. Staff can record a PNR from the consolidator portal without the system claiming an e-ticket exists.
5. A ticket can only become Issued after required ticket number, passenger match, issue evidence, and authorization are recorded.
6. Duplicate ticket numbers and suspicious duplicate PNR entries are flagged.
7. A refund/exchange case preserves original booking/ticket history and records estimated vs confirmed amounts separately.
8. Passport/e-ticket attachments are private and unavailable to unauthorized roles.
9. Customer receipt, supplier payable/payment, and agency fee/cost values reconcile to the same booking reference.
10. Visa and Umrah/tour bookings can be tracked from enquiry to completion with reminders and document permissions.
11. Reports show outstanding customer balances, supplier balances, ticket sales, service fees, refunds, and branch/agent performance.
12. Backup/restore and migration tests pass on staging.
13. XLSX/CSV files can be previewed, mapped, validated, and approved without directly posting ledger entries.
14. Text-based supported PDF reports can be extracted into a reviewable preview; scanned PDFs are clearly routed to manual review.
15. Re-uploading the same report does not create duplicate operational or financial records.
16. Every imported field can be traced to its source file, page/sheet, and row where available.

## Explicitly out of scope for this first phase

- Live flight search or fare repricing through API.
- Automated booking or ticket issuance.
- Portal scraping, reverse-engineered APIs, password vaulting, or robotic browser automation.
- Automated void/refund/exchange submission.
- Customer self-service flight purchasing.
- Blind, unattended OCR/import of scanned PDFs.
- Automatic posting of supplier statement rows to the general ledger.
- Hajj quota management or unverified regulatory automation.

## Phase 2 integration gate

Only consider API automation once the consolidator confirms in writing:
- A documented and supported API or approved export.
- Permission for search, PNR creation, ticket issuance, and each servicing operation separately.
- Production credentials and certification/sandbox process.
- Supplier settlement and ticketing authority.
- API rate limits, support contacts, and incident process.
- Data processing/security terms suitable for passport and traveller information.

An authorized read-only booking/status import may be the first integration; booking and ticketing writes can follow after idempotency and reconciliation have been tested.

## Operational information still to collect

- Consolidator legal/product name and portal name (do not share username/password/OTP).
- A representative, redacted sample of each downloadable PDF/Excel booking report and supplier statement, with all real passenger, passport, contact, PNR, ticket, and payment data removed.
- Whether PDF files contain selectable text or are scanned images; which Excel sheets and header rows carry the data.
- Settlement method (prepaid wallet, credit line, per-ticket invoice, or periodic statement) and statement fields.
- Current ticket fields and statement formats used by the consolidator.
- How the agency settles with the consolidator: prepaid wallet, credit line, per-ticket invoice, or periodic statement.
- Number of branches, ticketing staff, accounts staff, visa staff, and agents.
- Current customer payment methods and refund approval policy.
- Accountant-approved principal/agent revenue treatment and Pakistan tax setup.

## Implementation ownership and code layout

The ERPNext fork is an upstream application mirror. Keep it that way. Put custom DocTypes, hooks, fixtures, reports, workflows, and tests in a separate `travel_agency` Frappe app repository. Use the ERPNext fork only for upstream tracking and reviewed core patches. This keeps upgrades maintainable and avoids mixing a business-specific product with upstream ERPNext source.
