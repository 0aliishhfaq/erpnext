# Phase 1 MVP — Portal-Assisted Travel Agency Operations

## Confirmed operating model

The agency buys/issues tickets through a local or other ticket consolidator using that consolidator's website/desktop portal. The agency does **not** currently have API credentials or authorized API access.

Therefore Phase 1 is a **portal-assisted, staff-operated workflow**:

- Staff search/book/issue/refund/exchange in the consolidator's own portal.
- Staff record the resulting PNR, ticket, amount, supplier reference, and supporting documents in this ERP.
- The system manages enquiries, quotations, approval, passenger records, payment collection, supplier payable, document control, after-sales requests, and reconciliation.
- There is no direct live flight search, booking, ticket issuance, refund, exchange, or portal automation in Phase 1.

Do not use browser scraping, reverse-engineered endpoints, credential sharing, or automated clicks against the consolidator portal. Only add an integration if the provider gives explicit written authorization and supported API/export access.

## Phase 1 goal

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

### H. Staff controls

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

Standard ERPNext DocTypes remain the source of truth for customer/supplier accounting, invoices, payment entries, journals, currencies, taxes, and financial statements.

## First build sequence

### Sprint A — foundation and access control
- Create a separate custom Frappe app repository named `travel_agency` (do not place its source inside ERPNext core).
- Pin a tested compatible Frappe/ERPNext version pair for development and staging; do not deploy from moving `develop`.
- Install ERPNext and Frappe CRM on the same Frappe site where possible.
- Configure company, branch, currencies, fiscal year, cost centers, roles, and private file storage.
- Create Traveller Profile, Travel Enquiry, Travel Quotation, Travel Booking, and Booking Passenger.
- Add basic tests for permissions, document linking, and state transitions.

### Sprint B — portal-assisted air ticket operations
- Create Air Reservation, Air Ticket, and Ticket Service Case.
- Implement separate PNR and ticket status lifecycles.
- Add a staff checklist to record the external portal action and verify data before status changes.
- Add private e-ticket attachment, print/email templates, and booking/ticket search.
- Add approval and audit trail for issuance, void, refund, and exchange.
- Test duplicate PNR/ticket detection and multi-passenger booking support.

### Sprint C — accounting, visa and package basics
- Link each booking to ERPNext customer invoices, receipts, supplier payables, and payments.
- Add finance mapping after accountant review.
- Add Visa Case, document checklist, reminders, and status dashboard.
- Add Travel Package, Group Departure, passenger manifest, rooming list, and supplier vouchers.
- Test customer balance, supplier balance, commission, profit calculation, cancellation, and refund cases.

## Acceptance tests for Phase 1

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

## Explicitly out of scope for this first phase

- Live flight search or fare repricing through API.
- Automated booking or ticket issuance.
- Portal scraping, reverse-engineered APIs, password vaulting, or robotic browser automation.
- Automated void/refund/exchange submission.
- Customer self-service flight purchasing.
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
- Whether it offers downloadable booking reports, CSV/XLS exports, ticket PDFs, email notifications, or an approved API.
- Current ticket fields and statement formats used by the consolidator.
- How the agency settles with the consolidator: prepaid wallet, credit line, per-ticket invoice, or periodic statement.
- Number of branches, ticketing staff, accounts staff, visa staff, and agents.
- Current customer payment methods and refund approval policy.
- Accountant-approved principal/agent revenue treatment and Pakistan tax setup.

## Implementation ownership and code layout

The ERPNext fork is an upstream application mirror. Keep it that way. Put custom DocTypes, hooks, fixtures, reports, workflows, and tests in a separate `travel_agency` Frappe app repository. Use the ERPNext fork only for upstream tracking and reviewed core patches. This keeps upgrades maintainable and avoids mixing a business-specific product with upstream ERPNext source.
