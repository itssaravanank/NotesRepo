## Summary for PO / Lead Review

---

### ✅ Confirmed & Ready

| Item | Detail |
|---|---|
| **Customers in scope** | `CustId` values: `1085`, `1090`, `1096`, `2132` — Elevate customers |
| **Eligibility gate** | `LDBillingFilesInd = 'Y'` in `LASCommon.dbo.lac_CustomerProfile` |
| **No existing SP for gate** | A new `LASCommon` SP must be created to read `LDBillingFilesInd` by `CustId` at runtime |
| **File format** | Plain text `.txt`, UTF-8, one record per line, 80 characters per line |
| **File name** | `BillingFile-{RunName}-{CustomerId}.txt` |
| **Fixed-width layout** | Columns: LoanNo (pos 1–10, left), TaxId (pos 11–35, left), Amount (pos 47–56, right-justified integer cents), ELDDate (pos 57–62, `MMDDYY`) |
| **Amount to Pay** | `CASE InstallmentInd WHEN '1' THEN FirstInstallAmt WHEN '2' THEN SecondInstallAmt WHEN '3' THEN ThirdInstallAmt WHEN '4' THEN FourthInstallAmt` from `tbl_cta_ReportedItems` |
| **ELD Due Date source** | `NextELDDate` from `tbl_cta_ReportedItems` formatted as `MMDDYY` |
| **API path** | Classic TAX API remoting path (same as `GetExportPayItems`) |
| **Export is user-initiated** | Button click only — no automatic generation |
| **Button visibility** | Shown only when `LDBillingFilesInd = 'Y'`; hidden otherwise |

---

### ❓ Open Questions for PO / Lead

**Q4 — Which items are included in the file?**
AC4 states *"one row per reported item in that TAR job — no items are omitted"*. Does this mean:
- **(a) All items** regardless of `DoNotPayInd` flag, or
- **(b) Pay items only** where `DoNotPayInd <> 'Y'`?

---

### ⚠️ Assumptions Made (need sign-off)

**A1 — Q4 assumed: All items included**
Based on AC4 wording "no items are omitted", the billing file will include all rows in `tbl_cta_ReportedItems` for that TAR run (`CTARunName + CTARunDate + CustId`), regardless of `DoNotPayInd`. If Pay items only is intended, the SP and service need a filter.

**A2 — Amount format: integer cents**
The sample files show amounts like `66447` meaning `$664.47`. The value will be formatted as `(AmountToPay * 100)` cast to a right-justified integer with no decimal point, right-padded to 10 characters.

**A3 — TaxId field: formatted (`TaxId`) not unformatted (`UnformattedTaxId`)**
The sample files show values like `1275800010007` and `11.029.20.32.0042` which include formatting characters. The implementation will use the `TaxId` column (formatted). If `UnformattedTaxId` is required instead, please confirm.

**A4 — DROPPED items excluded**
Rows where `DROPPED = 'Y'` in `tbl_cta_ReportedItems` will be excluded. This is the standard filter used across all TAR export queries.

---

Once Q4 is confirmed and assumptions A1–A4 are signed off, implementation can begin immediately.











## All Open Questions for PO / Lead

---

**Q1 — Which items are included in the billing file?**
AC4 states *"one row per reported item in that TAR job — no items are omitted"*. Should the file include:
- **(a) All items** regardless of whether they are marked Do Not Pay, or
- **(b) Pay items only** where `DoNotPayInd <> 'Y'`?

---

**Q2 — Amount format in the file**
The sample files show amounts like `387743` meaning `$3,877.43` (no decimal point, stored as cents). The database stores the amount as dollars with 2 decimal places (e.g. `3877.43`). Is it correct to multiply by 100 and remove the decimal point when writing to the file?
- **(a) Yes** — multiply by 100, write as integer cents (matches sample files), or
- **(b) No** — a different format is expected?

---

**Q3 — Which Tax ID column to use**
There are two Tax ID columns in the database (Table Name : db_TaxAmtReports.dbo.tbl_cta_ReportedItems):
- `TaxId` — formatted, may include dots/dashes (e.g. `11.029.20.32.0042`)
- `UnformattedTaxId` — raw digits only (e.g. `1102920320042`)

The sample files show formatted values with dots for customer `1090`. Should the implementation use:
- **(a) `TaxId`** — formatted version (matches sample files), or
- **(b) `UnformattedTaxId`** — raw digits only?

---

**Q4 — Placement of the "Billing File" button in the UI**
The Disbursements tab already has existing export links — *Export Pay Items*, *Export Do Not Pay Items*, and *Export Disbursement Amount*. Where should the new **Billing File** button be placed?
- **(a) Alongside the existing export links** in the same export section, or
- **(b) In a separate/distinct location** on the page (e.g. its own row, near the page header, or near the job summary)?

A screenshot or mockup from the designer would be ideal. If none exists, option (a) is assumed.

---

Once these four questions are answered, all requirements are fully defined and implementation can begin immediately.