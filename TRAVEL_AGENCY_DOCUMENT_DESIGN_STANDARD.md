# Travel Agency — International Document & UI Design Standard

**Status:** Design baseline for the Travel Agency product  
**Applies to:** Web forms, dashboards, quotations, booking confirmations, itineraries, vouchers, invoices/receipts, visa case packs, manifests, settlement reports, import/export files  
**Initial supplier placeholder:** Demo Consolidator (fictional; replaceable configuration, not a real vendor)

> This is an internationalization and interoperability design baseline, not a claim of certification by IATA, ISO, OpenTravel, an airline, a government, or a payment provider. Legal/tax text and supplier-specific fields must be reviewed before production.

## 1. Design direction

Build a calm, premium B2B travel operations product: clear hierarchy, generous whitespace, consistent alignment, high legibility, and print-first documents. It should look credible to international customers and suppliers without impersonating airline, GDS, government, embassy, or hotel supplier documents.

### Visual system

- Primary brand ink: deep navy `#142B4E`; neutral text: `#253247`; secondary text: `#667085`; canvas: white / very light cool grey.
- Accent: restrained teal `#087E8B`; action blue: `#2457C5`.
- Semantic colors: success `#16794B`, warning `#9A6700`, danger `#B42318`, informational `#175CD3`. Never communicate status by color alone; add text and/or an icon.
- Use a consistent 8 px spacing scale, 12-column desktop grid, 16–24 px cards, 8–12 px control radii, and aligned labels/values.
- Use a licensed, locally bundled sans-serif font with safe fallbacks (Inter, Arial, sans-serif). Do not depend on remote font downloads for printed documents.
- Keep icons functional and consistent; do not use emoji in formal documents.
- Tables use stable column alignment: text left, codes centred where useful, amounts right-aligned with currency visible.
- Prefer sentence case labels, clear action verbs, and plain English. Allow translation-ready labels and content; English is the initial default, Urdu can be an additional locale.

### Layout and print rules

- Default business document page: A4 portrait. Use A4 landscape for passenger manifests, rooming lists, supplier statements, and wide operational tables. Support US Letter as a configurable page size where a client requires it.
- Default print margins: 12–15 mm, with a clear header/footer safe area.
- Header: agency logo, legal trading name, registered/business address, phone/email, website, tax registration fields only when applicable.
- Document heading: document title, unique document number, booking reference, issue date, revision/version, and status.
- Footer: page number, generated timestamp with timezone, booking/document reference, contact details, and a short appropriate disclaimer.
- Do not split a passenger row, itinerary segment, totals block, signature block, or critical warning across pages. Repeat table headers on new pages.
- Documents must remain useful in grayscale printing; do not rely on colored backgrounds to carry meaning.
- Include a clear "DRAFT", "QUOTE", "PAID", "UNPAID", "CANCELLED", or "VOID" status label where relevant. Do not use "TICKETED" unless ticket issuance has been verified.

## 2. International data conventions

Use standardized machine values and localize only the human-readable presentation.

| Data | Canonical representation | Validation / presentation |
| --- | --- | --- |
| Date | ISO 8601 `YYYY-MM-DD` | Display, for example, `21 Nov 2026`; avoid ambiguous `01/05/26` |
| Timestamp | ISO 8601 with offset or timezone | Show local airport/venue time and identify timezone when ambiguity is possible |
| Currency | ISO 4217 three-letter code | Show code with amounts (e.g. `PKR 125,000.00`, `USD 450.00`); use configured minor units |
| Country | ISO 3166-1 alpha-2 | Store code and display country name |
| Airport/location | IATA three-letter code where assigned | Store code + city/airport display name; validate against an authoritative code source |
| Airline | IATA two-character designator where assigned | Store marketing carrier and operating carrier separately when codeshare applies |
| Flight number | Carrier designator + flight number | Preserve source representation; never infer a missing flight number |
| Booking reference | Supplier/airline locator as returned | Preserve exact value and provider namespace; distinguish supplier locator from airline PNR |
| Ticket document number | Exact value issued by airline/consolidator | Do not manufacture or reformat; preserve leading zeros and issuer |
| Passenger name | Exact source/passport travel-document name | Do not reorder, autocorrect, translate, or infer identity |
| Phone | E.164 where country code is known | Preserve extension separately; validate but do not silently rewrite |
| Language | BCP 47 tag where needed | Example `en`, `ur-PK`, `ar-SA` |
| Time zone | IANA timezone identifier | Example `Asia/Karachi`; airport-local schedule time is also displayed |

### Air travel data rules

- For each flight segment record: segment sequence, marketing carrier, operating carrier when different, flight number, departure airport/code, arrival airport/code, departure local date/time, arrival local date/time, terminals when supplied, booking class/cabin when supplied, segment status, baggage allowance and source.
- For connections, show each segment separately and compute connection duration only when the required time-zone data is known.
- Store itinerary timestamps with their actual timezone/offset. Display each departure/arrival in the local time of its airport and label dates clearly.
- Record PNR/locator, ticket number, EMD number, airline record locator, consolidator reference, and internal booking number as distinct fields. Do not put all references in one generic "ticket no." field.
- Preserve fare basis, ticketing time limit, validating carrier, fare rules, included baggage, and source/capture timestamp when the portal supplies them.
- Record a ticket as issued only after staff verifies the ticket number and authoritative evidence. A booking confirmation or PNR alone is not proof of ticket issuance.
- Mandatory airline ticket notices and the original ticket issuer's passenger itinerary receipt must be passed through in the original provider-issued form. The agency may add a clearly branded cover page; it must not fabricate an airline ticket, boarding pass, barcode, ticket stock, or an official airline notice.

### Money and accounting display

- Always show currency codes, not a bare `$` symbol or assumed currency.
- Keep customer sale, consolidator/supplier cost, airline taxes/fees, agency service fee, markup/commission, discounts, and refunds as separate components.
- Show subtotal, tax (with tax name/rate only after accountant configuration), total, amount paid, balance due, and payment due date.
- Store original transaction currency and base-currency equivalent separately, with exchange rate, rate date, and source.
- Do not assume the ticket's gross face value is agency revenue. Revenue recognition and taxes require an accountant-approved principal/agent policy.

## 3. Standard document catalogue

### D01 — Travel Quotation (A4 portrait)

**Purpose:** Offer and price proposal; not proof of booking or ticket issuance.

**Layout**
1. Agency identity + "TRAVEL QUOTATION".
2. Quote number, revision, issue date/time, expiry date/time, sales agent, branch, status.
3. Customer name/company and contact information.
4. Service summary: Air / Visa / Umrah / Tour / Hotel / Other.
5. Air itinerary table where applicable: segment, airline, flight, route, departure/arrival local dates and times, cabin, baggage.
6. Traveller count and passenger names only where required.
7. Price breakdown by currency: base/supplier price, taxes/fees, agency service fee, markup, discount, total payable.
8. Inclusions, exclusions, fare/visa/package conditions, cancellation/change rules, and assumptions.
9. Payment terms, validity warning, prepared-by and approval fields.
10. Customer acceptance/signature area where operationally required.

**Mandatory notice:** "Quote only — subject to availability and supplier confirmation. No booking or ticket is confirmed until the relevant provider confirms it."

### D02 — Booking Confirmation & Travel Itinerary (A4 portrait)

**Purpose:** Agency's summary of an operational booking.

**Layout**
1. Agency identity + "BOOKING CONFIRMATION & ITINERARY".
2. Internal booking reference, service type, booking date, booking status, lead agent.
3. Customer/traveller details; show only identity data needed for the trip.
4. Supplier name/profile and supplier reference; clearly identify Demo Consolidator only in demo mode.
5. Airline PNR / record locator and ticket issue status as distinct values.
6. One row per flight segment with local dates/times, airport codes and full names, marketing/operating carriers, terminals, cabin, baggage and connection details when known.
7. Ticket number per passenger only when issued; show "Not issued" otherwise.
8. Payment summary and balance due, separated from supplier information.
9. Check-in/travel instructions only from verified source data.
10. Emergency/support contact, amendment history and source timestamps.

**Mandatory notice:** "This is an agency booking summary. The airline/consolidator's original ticket and travel notices remain the authoritative travel documents."

### D03 — Passenger Itinerary Receipt / E-ticket pack

**Purpose:** Deliver the authentic travel document to the passenger.

- Attach and deliver the original issuer-generated electronic ticket receipt and mandatory notices without altering their contents.
- Agency cover sheet may show internal reference, support contacts, payment status, and a summary of passenger/itinerary data.
- Clearly label agency cover sheet "AGENCY SUMMARY — NOT AN AIRLINE TICKET".
- Never create a boarding-pass barcode, airline ticket number, airline ticket stock, ticket coupon, or official ticket notice.
- If the issuer supplies a reissued ticket, archive the prior version and show the current issuer-sourced document clearly.

### D04 — Visa Case Summary & Document Checklist (A4 portrait)

**Purpose:** Agency's workflow summary only; never an official visa or embassy application form.

**Layout**
1. Agency identity + "VISA CASE SUMMARY".
2. Case number, applicant, destination country, visa category, intended travel dates, assigned visa officer.
3. Case status, submission/appointment dates, expected follow-up date.
4. Configurable country/visa-type checklist: required, received, verified, submitted, returned/expired.
5. Fees: government/embassy/supplier charge, agency service fee, currency, payment status.
6. Missing/expired document warnings, comments, activity log and contact.
7. Applicant acknowledgement/consent fields only where the business process and applicable law require them.

Do not claim approval likelihood or promise a decision date. Checklist content must be maintained against current official destination-country sources and reviewed by an authorized staff member.

### D05 — Umrah / Hajj / Group Package Quotation & Confirmation (A4 portrait)

**Purpose:** Package offer and traveller confirmation.

**Layout**
1. Agency identity + "GROUP PACKAGE" / "UMRAH PACKAGE".
2. Package code/name, departure/return dates, group leader, departure city, sales contact.
3. Itinerary by day/date, city, hotel, ground transport and included services.
4. Hotel summary: property name, room type, occupancy, check-in/out dates, meal plan when confirmed.
5. Flight details as D02; ticket status separately.
6. Per-person price, adult/child/infant or room supplements only where configured, inclusions/exclusions, taxes, payment instalments and balance due.
7. Passenger manifest reference; avoid distributing passport numbers in group copies.
8. Cancellation/refund terms, supplier dependencies, emergency/support details.
9. Traveller acknowledgement and revision history.

### D06 — Hotel / Ground Service Voucher (A4 portrait)

**Purpose:** Agency-issued service voucher linked to supplier confirmation.

**Layout**
1. Agency identity + "HOTEL VOUCHER" or "GROUND SERVICE VOUCHER".
2. Voucher number, internal booking reference, supplier confirmation number, issue date, status.
3. Lead guest and traveller count; guest names as required by supplier.
4. Property/provider name, full address, country, contact, check-in/out dates and local times.
5. Room/occupancy, meal plan, transfers, pick-up/drop-off location and inclusions.
6. Supplier confirmation status, payment/settlement status where helpful.
7. Special instructions and supplier emergency contact.
8. Clear statement that the voucher is valid only subject to supplier confirmation; include cancellation policy.

### D07 — Sales Invoice, Proforma, Receipt & Credit Note

**Purpose:** Financially and legally controlled accounting documents.

- Prefer ERPNext's standard Sales Invoice, Payment Entry/receipt and Credit Note workflows; do not create duplicate custom financial ledgers.
- Separate proforma/quotation from tax invoice and paid receipt.
- Display legal entity, invoice number, issue date, tax identity only when applicable, buyer details, service description, booking reference, currency, line items, subtotal, configured taxes, total, paid, balance and payment reference.
- Use accountant-approved tax labels, tax rates, invoice numbering, mandatory notices, and FBR/local integration fields. These vary by legal entity and jurisdiction and must not be hard-coded into a global template.
- Credit notes/refunds reference the original invoice and booking, show original/refunded amount, reason and approval reference.

### D08 — Refund / Void / Exchange Service Case

**Purpose:** Customer request and controlled after-sales audit trail.

- Original booking, PNR, ticket number(s), passenger, route and original issue date.
- Request type, request received date/time, requested action and reason.
- Provider/consolidator reference, request status, supplier-confirmed penalty/fee and supplier-confirmed refund/exchange amount.
- Customer impact, amount paid/refund due, supplier credit/payable effect, approvals and accounting references.
- Estimated and confirmed values must be clearly separated.
- Completion evidence and customer notification record.
- Never label an estimate as a confirmed airline refund.

### D09 — Supplier Statement / Agent Settlement Statement (A4 landscape)

**Purpose:** Reconciliation and commercial reporting.

- Statement number, supplier/agent, account reference, currency, period start/end, issue date.
- Opening balance, debit/credit transaction rows, booking/PNR/ticket reference, transaction date, description, amount, matched/unmatched status, running balance, closing balance.
- Separate reconciliation differences, unapplied credits, disputed items, and approval.
- Include an explicit "For reconciliation — not a tax invoice" note where appropriate.
- Imported rows remain staging data until accounts staff reconcile and authorize posting.

### D10 — Passenger Manifest / Rooming List (A4 landscape)

**Purpose:** Operations-only group list.

- Package/group reference, leader, departure date, service provider and preparation timestamp.
- Passenger name, internal traveller reference, room/roommate assignment, service allocation and operational notes.
- Passport numbers, dates of birth, phone numbers and medical/accessibility data must be omitted unless essential to the use case and explicitly permissioned.
- Generate a restricted internal version and, when needed, a supplier-specific minimum-data version.
- Include version number and access classification; never send broad passenger manifests to all customers.

## 4. Web form and dashboard standard

- Responsive desktop/tablet/mobile layout; common actions in a consistent header/action bar.
- Forms group fields into logical sections with clear required/optional indicators, examples, and inline validation.
- Use searchable link fields for customers, travellers, suppliers, currencies, airports and employees; avoid free-text duplicates for master data.
- Long operations use sections/tabs: Overview, Passengers, Itinerary, Documents, Pricing, Payments, Supplier, Activity.
- Persistent identifiers and status remain visible while scrolling.
- Use status text plus icon/label; never color-only meaning. Show warnings adjacent to the affected field.
- Confirm destructive/financial actions with a modal explaining the impact; require a reason and approval for controlled actions.
- Keep document preview beside import mapping. Show source column, destination field, validation, transformed value, and row-level result.
- Use keyboard-accessible controls, visible focus, labels and adequate contrast. Target WCAG 2.2 AA for the web UI; validate with automated and human testing rather than claiming conformance without audits.

## 5. Import/export standard for Demo Consolidator (PDF / Excel)

The real supplier report format is not known. Do not pretend one generic template is the vendor's official schema.

Create a canonical internal schema and configurable mapping profiles per supplier/report type:
- `ticket_report`
- `booking_report`
- `supplier_statement`
- `refund_exchange_report`

### Import steps

1. Private upload and file-type/size/security validation.
2. Source file retained with a cryptographic hash and uploader/time.
3. Staff selects report type, supplier profile (Demo Consolidator initially), worksheet and header row where applicable.
4. Parser proposes column mappings; staff confirms/corrects the mapping.
5. Show original and normalized values side by side.
6. Validate PNR, ticket number, dates, currency, numeric amounts, passenger association, and supplier transaction reference.
7. Detect duplicate rows and near-duplicates. Never dedupe on passenger name alone.
8. Stage row outcomes: Ready, Warning, Duplicate, Invalid, Manual Review Required.
9. Staff reviews and approves eligible records; financial postings require a separate accounts action.
10. Keep source page/sheet/row, mapping version, import user, approval and error history.

### File safety

- XLSX only; never execute formulas, macros, embedded scripts, or external links. Use values-only extraction where possible.
- CSV encoding/delimiter must be detected and previewed.
- Text-based PDFs may be parsed for supported layouts. Scanned/image-only PDFs go to manual review; any future OCR must be an explicit feature with confidence display and human verification.
- Reject encrypted/password-protected files with a clear instruction; never request a portal password or OTP.
- Enforce upload/page/row limits and parser timeouts.
- Keep files private and do not commit real booking reports to Git.

## 6. Accessibility, privacy and localization

- Target WCAG 2.2 AA for interactive user journeys. WCAG 2.2 was also approved as ISO/IEC 40500:2025; conformance requires testing.
- Support translated labels and right-to-left layout for Urdu/Arabic without changing canonical stored values.
- Keep names and document numbers in their source-script/original form; provide a separate display translation only when explicitly reviewed.
- Store airport-local times and timezone metadata to prevent ambiguity.
- Minimize passport and identity data; use least privilege, private attachments, audit logs, retention/deletion policies, and controlled exports.
- Never include secrets, passport scans, or customer booking data in Git, test fixtures, error logs, or public URLs.

## 7. QA / design acceptance checklist

- Print tests on A4 portrait, A4 landscape, and US Letter.
- Browser/PDF output checked for pagination, repeated table headers, logo scaling, page count, legibility, grayscale and long passenger/route names.
- Amounts retain currency codes and decimal precision; totals reconcile.
- Airport/airline codes validate against maintained reference sources; unknown codes remain reviewable instead of guessed.
- Date/time output is unambiguous across timezone boundaries and daylight-saving regions.
- Passenger name and ticket number match the source exactly.
- Quote, booking, PNR, ticket, invoice and supplier references remain distinct.
- Accessibility checks include keyboard-only flows, focus, labels, contrast, validation messages and screen-reader review.
- Import tests include duplicate uploads, changed column order, missing columns, corrupted files, formula cells, encrypted PDFs, scanned PDFs, partial failures and retry behaviour.
- No document claims to be an official airline, government, embassy or supplier document unless it is the original document issued by that authority/provider.

## 8. Standards and source references

Use these references as design/data conventions; purchase/licensing requirements and applicable editions must be checked before production integrations.

- **ISO 8601 date/time:** https://www.iso.org/iso-8601-date-and-time-format.html
- **ISO 4217 currency codes:** https://www.iso.org/iso-4217-currency-codes.html
- **ISO 3166-1 country codes:** https://www.iso.org/standard/72482.html
- **IATA airline and airport code search:** https://www.iata.org/codes
- **IATA location identifier database:** https://www.iata.org/en/publications/manuals/airline-airport-location-coding-databases/airport-location-identifier-database/
- **IATA passenger distribution, NDC, One Order and ticketing standards:** https://developer.iata.org/en/passenger/
- **IATA e-ticket itinerary receipt and notices:** https://www.iata.org/en/fmc-documents/a4938a2d-e11c-44f5-b88f-dd1e548829ef/
- **IATA BSP data interchange (DISH):** https://www.iata.org/en/publications/bspdish/
- **OpenTravel messaging / schemas:** https://opentravel.org/opentravel-messaging-business-functionality/
- **WCAG 2.2:** https://www.w3.org/TR/wcag/
- **ISO/IEC 40500:2025:** https://www.iso.org/standard/91029.html
