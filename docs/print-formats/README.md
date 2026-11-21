# Travel Agency — Professional Print Format Catalogue

This directory defines the print document catalogue and implementation mapping for the Travel Agency app. The full design language, international data rules, required content and compliance boundaries are in [TRAVEL_AGENCY_DOCUMENT_DESIGN_STANDARD.md](../../TRAVEL_AGENCY_DOCUMENT_DESIGN_STANDARD.md).

## Standard layout

- A4 portrait by default; A4 landscape for wide operational reports; US Letter configurable.
- Consistent agency header, unique document number, booking reference, issue/revision date, status, footer, page number and support contacts.
- Deep-navy / teal visual identity, neutral backgrounds, high contrast, consistent typography and spacing.
- Values are stored canonically (ISO codes and ISO 8601 dates/timestamps); labels are localized for display.
- Use grayscale-safe tables and repeat table headers across pages.
- Do not create official-looking airline, government, embassy or supplier documents. Keep original supplier-issued documents intact.

## Document formats

| ID | Print format | Page | Data source / ERPNext record | Primary audience |
| --- | --- | --- | --- | --- |
| D01 | Travel Quotation | A4 portrait | Travel Quotation | Customer |
| D02 | Booking Confirmation & Travel Itinerary | A4 portrait | Travel Booking + Air Reservation + Air Ticket | Customer |
| D03 | E-ticket Pack / Agency Cover Sheet | A4 portrait + original attachment | Original issuer e-ticket + agency cover | Passenger |
| D04 | Visa Case Summary & Document Checklist | A4 portrait | Visa Case | Applicant / staff |
| D05 | Umrah / Tour Package Quotation & Confirmation | A4 portrait | Travel Package + Group Departure + Travel Booking | Customer / group leader |
| D06 | Hotel / Ground Service Voucher | A4 portrait | Hotel Booking / service record | Guest / supplier |
| D07 | Proforma / Sales Invoice / Receipt / Credit Note | A4 portrait | Standard ERPNext Sales Invoice / Payment Entry / Credit Note | Customer / accounts |
| D08 | Void / Refund / Exchange Service Case | A4 portrait | Ticket Service Case | Customer / operations / accounts |
| D09 | Supplier Statement / Agent Settlement Statement | A4 landscape | Import Batch + reconciliation / settlement records | Accounts / agent |
| D10 | Passenger Manifest / Rooming List | A4 landscape | Group Departure + Booking Passenger | Restricted operations / supplier |

## Template implementation policy

1. Implement these as Print Format records in the separate `travel_agency` Frappe app (except D07, which should extend/configure standard ERPNext financial print formats without duplicating the ledger).
2. Keep the ERPNext fork close to upstream. Do not put agency-specific DocTypes, print formats or hooks directly in ERPNext core.
3. Every print format must have a preview fixture with synthetic data and page-break testing.
4. Do not expose passport numbers, dates of birth, private contact details, or passenger manifests in documents unless required for the recipient and authorized by role.
5. D03 must attach the authentic supplier/airline-issued e-ticket receipt and mandatory notices. The agency cover sheet must be labelled "AGENCY SUMMARY — NOT AN AIRLINE TICKET".
6. D07 tax content and invoice numbering are configured only after the agency accountant approves the legal entity's tax treatment and jurisdictional requirements.
7. Every document must be available as a controlled PDF and must retain its document number, source record, revision and generated timestamp.

## Status

This is a design catalogue. The Print Format records and rendered PDFs have not yet been implemented or tested in a running Frappe site.
