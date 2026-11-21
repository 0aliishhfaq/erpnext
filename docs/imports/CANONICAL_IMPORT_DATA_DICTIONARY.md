# Canonical Import Data Dictionary — Consolidator Reports

**Initial supplier profile:** Demo Consolidator (fictional placeholder only).  
**Purpose:** Normalize downloaded portal reports into a supplier-neutral import staging model. These are internal canonical fields, not claims about a provider's exported headers.

## General rules

- Store raw source values and normalized values side by side.
- Dates use ISO 8601 `YYYY-MM-DD`; timestamps use ISO 8601 with offset/timezone where known.
- Currency uses ISO 4217 three-letter codes.
- Country uses ISO 3166-1 alpha-2. Airport/location codes use IATA codes where assigned.
- Amounts use decimal data types, never binary floating point for financial calculations.
- Preserve ticket numbers, locators and supplier references exactly as supplied (including leading zeroes).
- A field absent from the source stays empty/unknown. Never invent a code, fare, date, status or ticket number.
- Map external headings to these canonical names using a versioned profile for each supplier + report type + layout version.

## Ticket / booking report fields

| Canonical field | Type | Required to stage? | Validation / notes |
| --- | --- | --- | --- |
| supplier_profile | Link / text | Yes | Demo Consolidator until replaced with actual supplier master |
| source_report_id | Text | Recommended | Report ID as supplied |
| source_report_date | Date | Recommended | ISO 8601 date |
| source_file_hash | Text | Yes | Cryptographic hash calculated by importer, not user-supplied |
| source_sheet_name | Text | When XLSX | Preserve worksheet name |
| source_page_number | Integer | When PDF | Page containing source row, where available |
| source_row_number | Integer | When tabular | Original row number |
| internal_booking_reference | Link / text | No | Internal Travel Booking identifier |
| supplier_booking_reference | Text | Recommended | Supplier's own booking/transaction reference |
| airline_pnr | Text | No | Airline locator only when clearly identified |
| supplier_pnr | Text | No | Consolidator locator kept separate from airline PNR |
| reservation_status_source | Text | No | Raw supplier status |
| ticket_status_source | Text | No | Raw supplier status |
| passenger_reference | Link / text | Recommended | Internal traveller ID, if safely matched |
| passenger_name_source | Text | Recommended | Preserve exact source spelling/order |
| passenger_type | Enum | No | ADT / CHD / INF only when explicitly supplied |
| origin_iata | Text | No | Validate as a known location code; unknown values require review |
| destination_iata | Text | No | Validate as a known location code; unknown values require review |
| marketing_carrier_iata | Text | No | Preserve source marketing carrier |
| operating_carrier_iata | Text | No | Separate for codeshare where supplied |
| flight_number | Text | No | Preserve source value |
| departure_local_datetime | ISO datetime | No | Airport-local time; include timezone/offset separately |
| departure_timezone | IANA timezone | No | Required when converting to a global timestamp |
| arrival_local_datetime | ISO datetime | No | Airport-local time; include timezone/offset separately |
| arrival_timezone | IANA timezone | No | Required when converting to a global timestamp |
| ticket_document_number | Text | No | Exact issuer value; do not reformat |
| ticket_issue_date | Date | No | Only if confirmed in source |
| validating_carrier_iata | Text | No | Preserve source value |
| booking_class | Text | No | Preserve exact source value |
| cabin | Text | No | Preserve source value; normalize only with a documented mapping |
| baggage_allowance_source | Text | No | Preserve segment/passenger context |
| currency_code | ISO 4217 | No | Never infer from symbol alone |
| customer_total_amount | Decimal | No | Source amount and source currency |
| supplier_cost_amount | Decimal | No | Source amount and source currency |
| airline_tax_amount | Decimal | No | Only where independently identified |
| agency_service_fee_amount | Decimal | No | Do not infer from gross fare |
| agency_markup_amount | Decimal | No | Do not infer from gross fare |
| payment_status_source | Text | No | Raw supplier status only |
| source_document_filename | Text | No | Original attachment name |
| import_notes | Text | No | Manual review reason / notes |

## Supplier statement fields

| Canonical field | Type | Required to stage? | Validation / notes |
| --- | --- | --- | --- |
| supplier_profile | Link / text | Yes | Supplier master profile |
| statement_reference | Text | Recommended | Supplier's statement ID |
| statement_date | Date | Yes | ISO 8601 |
| statement_period_start | Date | No | ISO 8601 |
| statement_period_end | Date | No | ISO 8601 |
| transaction_reference | Text | Recommended | Supplier transaction reference |
| transaction_date | Date | Yes | ISO 8601 |
| transaction_type_source | Text | No | Raw source transaction type |
| supplier_booking_reference | Text | No | Supplier booking reference |
| airline_pnr | Text | No | Airline PNR only where identified |
| ticket_document_number | Text | No | Exact ticket number if present |
| passenger_name_source | Text | No | Personal data; restrict visibility |
| currency_code | ISO 4217 | Yes | Required for any amount |
| debit_amount | Decimal | Yes | Non-negative value |
| credit_amount | Decimal | Yes | Non-negative value |
| running_balance_amount | Decimal | No | Preserve statement sign convention in mapping |
| due_date | Date | No | ISO 8601 |
| transaction_description_source | Text | No | Preserve original text |
| source_sheet_name | Text | When XLSX | Preserve source sheet |
| source_page_number | Integer | When PDF | Preserve source page where available |
| source_row_number | Integer | When tabular | Preserve source row |

## Duplicate and import policy

- Prefer supplier transaction reference + supplier profile + report type as a stable source key.
- For tickets, use supplier profile + exact ticket document number + passenger association as a duplicate candidate.
- For reservations, use supplier profile + supplier locator / airline PNR as a duplicate candidate; keep airline PNR and supplier locator separately.
- A passenger name by itself is not a unique identifier.
- Re-uploading the same file hash must not create a second import batch of financial/operational records without an explicit, audited override.
- A changed report can contain corrections; stage changes for review instead of silently overwriting a ticket or booking history.
- Supplier statement rows remain reconciliation items. They are not journal entries and must not auto-post to the general ledger.

## Export templates

Use the header-only CSV templates in this directory. They contain no real passenger, ticket, booking, supplier, or accounting data.
