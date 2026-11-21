# ERPNext Customization Guide

This repository is a fork of [frappe/erpnext](https://github.com/frappe/erpnext).
Keep the upstream ERPNext application as close to upstream as possible and place
business-specific functionality in a separate Frappe custom app.

## Branch strategy

- `develop`: upstream tracking branch. Keep this aligned with `frappe/erpnext`.
- `customization/initial-setup`: project setup and customization guidance.
- `feature/<area>-<short-name>`: create a separate branch for each actual feature.
- Do not deploy the moving `develop` branch to production. Choose a supported,
  tested ERPNext/Frappe release pair for production.

## Golden rule: do not customize ERPNext core directly

Create a separate Frappe app for business-specific work. For example:

```bash
bench new-app business_custom
bench --site <site-name> install-app business_custom
```

Use that app for custom DocTypes, fields, workflows, reports, integrations,
permissions, hooks, fixtures, and server/client scripts. This keeps custom work
reviewable and makes upstream updates much safer.

## Development environment

Before installing dependencies, check the version requirements in
`pyproject.toml` and use a compatible Frappe Framework branch. At the time this
guide was added, the ERPNext manifest declares:

- Python: `>=3.14`
- Frappe: `>=17.0.0-dev,<18.0.0`

These are repository declarations, not proof that a particular machine or
deployment has been tested. Confirm the matching Bench, Frappe, database,
Node.js, and package-manager versions from the official Frappe installation
documentation before creating the environment.

To fetch this fork into an existing Bench:

```bash
bench get-app --branch develop https://github.com/0aliishhfaq/erpnext
bench --site <site-name> install-app erpnext
```

Use a disposable development site first. Never test migrations or new business
logic directly against production data.

## Upstream update workflow

1. Keep a remote named `upstream` pointing to
   `https://github.com/frappe/erpnext.git`.
2. Fetch upstream changes and review release notes and breaking changes.
3. Merge or rebase upstream changes into the tracking branch in a controlled PR.
4. Run the relevant Python, JavaScript, migration, and integration tests.
5. Merge custom feature branches only after review and staging verification.

Example local remote setup:

```bash
git remote add upstream https://github.com/frappe/erpnext.git
git fetch upstream
```

Do not force-push shared branches or overwrite upstream history.

## Pakistan-specific configuration checklist

The target business and its legal/tax requirements have not yet been specified.
Before configuring a Pakistan deployment, verify the current requirements with a
qualified local tax/accounting professional and the relevant authorities.
In particular, validate:

- Company registration details, fiscal year, chart of accounts, and multi-company needs.
- Sales tax, withholding tax, tax periods, and invoice formats.
- Any FBR/POS/e-invoicing integration requirements that apply to the business.
- PKR currency, bank accounts, payment methods, and bank reconciliation.
- User roles, approval limits, audit trail, and data retention.

Do not hard-code tax rates, legal rules, or credentials into ERPNext core or
source control. Prefer configurable records and a separate integration app.

## Quality and release checklist

- Add automated tests for every custom feature and important business rule.
- Test install, upgrade, migrate, backup, and restore procedures.
- Review permissions and role access with least privilege.
- Keep credentials and private customer data out of Git.
- Take and verify backups before migrations.
- Validate the full workflow on staging with representative data.
- Record the deployed ERPNext, Frappe, custom-app, and database versions.

## Requirements needed before business features are implemented

A tailored ERP cannot be designed safely from the repository alone. The next
implementation scope should identify the business type, required modules,
end-to-end workflows, user roles, integrations, reporting needs, and applicable
tax/compliance rules. Implement each agreed feature in the separate custom app,
not by replacing upstream ERPNext files.
