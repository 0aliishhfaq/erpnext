# Travel Agency ERP Architecture and Implementation Plan

## Recommendation

Use a hybrid architecture:

- **ERPNext** for accounting, receivables/payables, bank reconciliation, purchasing, HR, and financial reporting.
- **Frappe CRM** for leads, deals, sales pipeline, and customer follow-ups.
- **A separate custom Frappe app** named `travel_agency` for ticketing operations, visa cases, Umrah/Hajj, tours, hotels, passengers, suppliers, and travel-specific workflows.
- **External booking providers** (GDS/NDC, consolidators, hotel suppliers, payment providers) behind adapters. The app must not assume that ERPNext itself is a flight inventory or ticketing system.

Keep this ERPNext fork close to upstream. Custom DocTypes, hooks, integrations, and business logic belong in the separate `travel_agency` app/repository, not directly inside ERPNext core.

## High-level architecture

```text
Travel Agency Portal / Staff Workspace
                 |
                 v
        Frappe CRM (Leads / Deals)
                 |
                 v
       travel_agency custom app
       |        |        |       |
       |        |        |       +-- Provider adapter layer
       |        |        |             |-- Existing GDS / consolidator
       |        |        |             |-- Optional second air provider
       |        |        |             |-- Hotel supplier
       |        |        |             |-- Payment / messaging providers
       |        |        |
       |        |        +-- Visa / Umrah / Tours / Hotels
       |        |
       |        +-- Passenger / PNR / Ticket / Refund / Exchange
       |
       v
 ERPNext (Customers, Items, Sales Invoices, Purchase Invoices,
          Payment Entries, Journals, taxes, bank reconciliation,
          cost centres, HR and financial reports)
```

Frappe CRM has a documented ERPNext integration for syncing customers, products/items, and quotations. Prefer installing CRM and ERPNext on the same Frappe site when practical; the integration also supports separate sites through authenticated API access. See https://docs.frappe.io/crm/erpnext.

## Functional scope

### 1. Shared customer and passenger records

- Customer / company profile and contact information.
- Separate passenger/traveller profile linked to a customer; one customer may book for many travellers.
- Passport details and expiry date with private file attachments and restricted roles.
- Emergency contact, nationality, date of birth, and other information only when operationally required.
- Deduplication and audit history for profile changes.

Passport scans and identity details are sensitive personal data. Use private files, least-privilege permissions, audit logs, retention rules, and avoid exposing passport data in routine list views or exports.

### 2. Air ticketing

- Flight request / enquiry and passenger list.
- Search and price results from an enabled provider, including offer expiry and fare conditions.
- Reservation / PNR record with provider, locator, booking status, itinerary, segments, and time limits.
- Ticket issuance record with ticket number, issue date, validating carrier, amount, taxes, commission, markup, supplier/consolidator, and attached e-ticket.
- Separate controlled workflows for void, refund, exchange/reissue, and cancellation.
- Store raw provider request/response references where permitted, with secrets and unnecessary personal data redacted.
- Reconcile each booking and ticket action to its financial entries.

Do not mark a booking as ticketed merely because a PNR exists. A held reservation, a confirmed reservation, and an issued e-ticket are different states.

### 3. Visa processing

- Visa enquiry and case pipeline (new, documents pending, submitted, under review, approved, rejected, passport returned, closed).
- Country/visa-type document checklist with configurable requirements.
- Applicant and family/group cases, appointment details, submission dates, expected return dates, fees, supplier/embassy costs, and service charges.
- Private document uploads, expiry reminders, comments, and an immutable status/activity timeline.
- No claim that the app itself issues visas or guarantees approval.

### 4. Umrah / Hajj and group packages

- Package catalogue, departure dates, group capacity, inclusions/exclusions, and terms.
- Passenger manifest, rooming list, family/group relationships, and traveller document checklist.
- Supplier contracts and cost heads for flights, hotels, transport, guides, and other services.
- Per-person pricing, child/room supplements, instalments, balance due, cancellations, and refunds.
- Operational checklists and group status board.

Hajj-specific workflows and quotas should remain configurable and must be validated against the agency's current official arrangements and authorized suppliers.

### 5. Tours and hotel bookings

- Itinerary builder, departure/return dates, travellers, inclusions, hotel/transport suppliers, and quote versions.
- Hotel booking / voucher records with confirmation numbers, dates, cancellation deadlines, room and guest details.
- Supplier payables, customer balances, change history, and voucher generation.

### 6. Agents, branches, and commissions

- Branches, sales staff, sub-agents, corporate clients, and supplier/consolidator accounts.
- Configurable service fees, markup, commission rules, agent credit limits, and settlement statements.
- Approval limits for discounts, refunds, write-offs, and manual ticket status overrides.
- Branch-, salesperson-, supplier-, and service-wise profitability.

## Consolidator PDF / Excel import

All customer-facing and internal forms/reports use the [international document and UI design standard](TRAVEL_AGENCY_DOCUMENT_DESIGN_STANDARD.md) and the [print format catalogue](docs/print-formats/README.md). Canonical import fields are defined in the [import data dictionary](docs/imports/CANONICAL_IMPORT_DATA_DICTIONARY.md). These documents specify the intended standard; they are not evidence that print formats or an importer have been implemented in a running site.

The initial supplier is a fictional development placeholder named **Demo Consolidator**. The real supplier name does not block the first release.

Staff download booking/ticket reports and supplier statements from the provider portal, then upload the files to a private import inbox. The import process must:

1. Accept XLSX/CSV and supported text-based PDF layouts.
2. Let staff choose report type, sheet and header row, then preview source rows and map columns through a versioned supplier/report mapping profile.
3. Validate PNRs, ticket numbers, passenger association, dates, currencies, and required amount fields.
4. Flag duplicates and invalid rows before any record is created.
5. Keep every imported row traceable to the original private file and sheet/page/row.
6. Require staff approval and make repeat uploads idempotent.
7. Keep supplier statement lines in reconciliation staging; do not auto-post them to the ERPNext general ledger.
8. Route scanned/image-only PDFs to manual review rather than silently trusting OCR output.

Do not scrape the supplier website, automate portal clicks, execute workbook macros/formulas, or store portal credentials. Use an API only if the provider later grants written authorization and supported API access.

Synthetic CSV templates live in `docs/sample-imports/`. They are examples only and must be replaced with mappings based on a redacted real export when available.

## Core custom DocTypes (initial proposal)

| DocType | Purpose |
| --- | --- |
| Traveller Profile | Traveller identity and preferences, linked to Customer |
| Travel Enquiry | Original request, source, assigned agent, and requirements |
| Travel Quotation | Versioned quote, price components, validity, and approval |
| Travel Booking | Master booking for one service or a package |
| Booking Passenger | Passenger-to-booking relationship |
| Air Reservation | PNR, provider, itinerary, locator, and status |
| Air Ticket | Issued ticket details and e-ticket reference |
| Ticket Service Case | Void, refund, exchange, or reissue request and provider status |
| Visa Case | Applicant/case workflow, checklist, fees, and deadlines |
| Travel Package | Package catalogue, itinerary, inclusions, and pricing |
| Group Departure | Departure, capacity, manifest, and operational status |
| Hotel Booking | Supplier confirmation, room, dates, and cancellation policy |
| Travel Supplier | Airline consolidator, hotel supplier, transport, visa or tour partner |
| Agent Commission | Commission calculation, approval, and settlement |
| Provider Transaction Log | Redacted integration event, correlation ID, result, and retry status |

Use standard ERPNext DocTypes for Customers, Suppliers where appropriate, Items/services, Sales Invoices, Purchase Invoices, Payment Entries, Journal Entries, Cost Centers, and accounting reports. Avoid duplicate custom ledgers if a standard ERPNext financial record already models the transaction.

## Accounting and business controls

- Create a finance mapping for each service type before enabling automatic posting.
- Model customer collection, supplier/consolidator payable, agency service fee, commission, markup, refund, and exchange adjustment separately.
- Confirm with the agency's accountant whether each ticket/package sale is treated as principal or agent revenue. Do not automatically recognize the entire ticket face value as agency revenue without that decision.
- Support PKR and required foreign currencies, exchange rates, realized/unrealized FX differences, and customer/supplier balances.
- Use a booking reference on every related Sales Invoice, Purchase Invoice, Payment Entry, refund, and journal adjustment.
- Configure tax templates and chart of accounts with a Pakistan-qualified accountant. FBR electronic invoicing and sales-tax applicability must be verified for the legal entity and taxpayer category; do not hard-code tax rates or assume that every travel service has identical treatment.
- Never store payment card numbers or CVVs. Use hosted checkout/tokenization from an approved payment provider.

## Booking provider strategy

Build provider-neutral interfaces such as:

- `AirSearchProvider`
- `AirPricingProvider`
- `AirBookingProvider`
- `AirTicketingProvider`
- `AirServicingProvider` (void / refund / exchange)
- `HotelBookingProvider`
- `PaymentProvider`

Each adapter must declare which operations it actually supports. Save the provider reference and transaction/correlation ID, preserve idempotency for write operations, retry only safe transient failures, and reconcile uncertain outcomes before retrying a booking or ticket issuance.

**Do not select a provider solely because a sandbox search API works.** Production access, ticketing authority, airline coverage, private/negotiated fares, payment/settlement, after-sales servicing, local consolidator arrangements, commercial terms, and support must be confirmed first.

Provider notes:

- Travelport documents GDS and NDC workflows and requires customer provisioning/credentials. NDC access also depends on carrier-level provisioning/registration. Booking, ticketing, refunds, exchanges, and voids have distinct workflows and can differ between GDS and NDC. See https://developer.travelport.com/docs/flights/guides/flights-general-guide and https://developer.travelport.com/docs/flights/guides/ticketing-guide.
- Amadeus Self-Service has dataset limitations (including exclusions and published-rate-only restrictions described in its FAQ); its FAQ says ticket issuance for Self-Service requires working with an airline consolidator. Treat it as a possible search/booking integration, not an assumed replacement for a fully provisioned agency ticketing arrangement. See https://admin.developers.amadeus.com/self-service/apis-docs/guides/developer-guides/faq/.
- Start by integrating the agency's existing GDS or consolidator workflow. Add another provider only when there is a proven commercial and operational need.

## Delivery phases and acceptance criteria

### Phase 0 — Requirements and provider discovery

Confirm:
- Current GDS/consolidator(s), PCC/office ID where applicable, contract, permitted API access, ticketing authority, and after-sales process.
- Air ticket sources and how agents currently handle PNRs, ticket issuance, voids, refunds, and exchanges.
- Visa countries/types, Umrah/tour package operations, hotel suppliers, branches, agents, and currencies.
- Current accounting treatment, tax registration, invoice requirements, and reporting needs.
- User roles, approval matrix, data migration scope, and document retention.

**Exit criteria:** signed-off workflow maps, provider capability matrix, accounting mapping, and MVP backlog.

### Phase 1 — Operational MVP (manual/provider-assisted ticketing)

Implement in the separate custom app:
- Customer and traveller profiles with private document handling.
- Enquiry, quotation, approvals, booking, PNR and e-ticket records.
- Visa cases, checklists, reminders, and document tracking.
- Umrah/tour packages, group departures, manifests, and hotel voucher records.
- Supplier, consolidator, sub-agent, commission, and branch records.
- ERPNext accounting posting, customer receipts, supplier payments, and refund adjustments.
- Role permissions, activity history, dashboards, printable quotation/voucher/receipt, and basic reports.

**Acceptance criteria:** staff can complete a full manual ticket/visa/package workflow from enquiry to collection and accounting reconciliation without maintaining duplicate spreadsheets.

### Phase 2 — Integrations

- Integrate the current production-authorized air provider using provider adapters.
- Add search/reprice/booking/ticketing only for operations contractually enabled by the provider.
- Add status sync, e-ticket retrieval, and separate void/refund/exchange workflows where supported.
- Integrate selected hotel supplier(s), payment gateway(s), and email/WhatsApp notifications.
- Add retries, idempotency, error queue, audit trail, and daily settlement reconciliation.

**Acceptance criteria:** sandbox and controlled production tests pass for each supported workflow; uncertain transactions are reconciled safely and do not generate duplicate bookings or charges.

### Phase 3 — Scale and customer/agent portals

- Customer portal for quotes, documents, payments, and vouchers.
- Agent / sub-agent portal with credit limits and settlement statements.
- Management dashboards, branch profitability, receivables ageing, supplier payables, and service-wise margins.
- More supplier APIs, multi-branch controls, automation, and analytics as justified by usage.

**Acceptance criteria:** portal permissions, data isolation, financial reconciliation, operational monitoring, and backup/restore tests pass.

## Security, quality, and release requirements

- Keep custom source in a separate app/repository and install it alongside a pinned, compatible ERPNext/Frappe release.
- Do not deploy directly from a moving `develop` branch. Test a compatible version pair on staging before production.
- No secrets, private keys, API credentials, passport scans, or customer exports in Git.
- Use least-privilege roles, private attachments, audit logs, and secure backups.
- Add automated tests for booking state transitions, money calculations, commissions, refunds, and accounting postings.
- Test install, migration, upgrade, backup, and restore procedures before releases.
- Keep a staging environment with anonymized or synthetic test data.

## Decisions required before implementation starts

1. Which GDS/consolidator do you currently use (Amadeus, Sabre, Travelport, local consolidator, or manual portal)?
2. Do you already have API credentials/contract permission, or do staff book through a desktop/website portal?
3. Which visa countries and Umrah/tour packages are highest priority?
4. How many branches, users, and sub-agents need access?
5. Which accounting/tax setup is currently used, and is the entity sales-tax registered?

These answers determine whether the first deliverable should be manual/provider-assisted operations or live air booking integration.
