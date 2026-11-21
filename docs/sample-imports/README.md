# Demo Consolidator — sample import files

These CSV files are **synthetic templates only**. They do not represent a real consolidator's report format, live booking, PNR, passenger, ticket number, statement, or settlement.

## Files

- `demo-consolidator-ticket-report-template.csv`: one example row for a ticket/booking report. Map each source provider's actual columns to the import fields; do not assume these headers will match the real portal export.
- `demo-consolidator-supplier-statement-template.csv`: one example row for a supplier statement. Statement rows must remain staging records until accounts staff reconcile and approve them.

## Import rules

1. Ask the consolidator for a representative report and statement export only when needed; use redacted samples with real passenger, passport, PNR, ticket, contact, and payment details removed.
2. Create a versioned mapping profile for each actual report type/layout. Never identify a supplier format from the fictional placeholder alone.
3. Preview and validate rows before import. Keep the original values and source file/page/sheet/row reference.
4. Detect duplicates using stable supplier references and PNR/ticket identifiers. A passenger name alone is not a unique key.
5. Re-uploading a report must be idempotent and must not create duplicate tickets, bookings, invoices, or payments.
6. Supplier statement rows are not journal entries. They require reconciliation and an authorized accounts action before financial posting.
7. XLSX formulas/macros, external links, and embedded scripts must never execute during import.
8. Text-based PDFs can be parsed only for supported layouts. Scanned PDFs must go to manual review unless a separately approved OCR workflow is implemented with human verification.

Do not put real personal data or supplier credentials into these repository templates or Git.
