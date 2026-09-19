# AML Transaction Monitoring — No-Code (Excel) Version

A working AML transaction monitoring build using **only spreadsheet
formulas — no Python, no SQL**. This is a companion project to a SQL
version of the same case: same typologies, same detection logic,
same scoring approach, expressed entirely through Excel formulas
instead of code.

**Why this version exists:** many compliance teams — especially at
smaller institutions, or for one-off investigations — still do
first-pass transaction review in Excel. Building the same analytical
logic here demonstrates that the underlying AML thinking (typology
recognition, threshold-setting, false-positive tradeoffs) isn't
dependent on a particular tool, and it's the format many AML analyst
job postings still expect day-to-day.

## What's in this repo

```
AML_Transaction_Monitoring.xlsx    ← the main workbook (open this)
README.md
data/
  transactions.csv                 ← raw transaction data, standalone CSV
  customers.csv                    ← customer/account reference data, standalone CSV
  alert_summary_output.csv         ← computed risk-scoring results, exported as CSV
screenshots/
  transactions_highlighted.png
  alert_summary.png
```

The CSVs are provided so the underlying data and results are viewable
directly on GitHub without opening Excel, and so the same data could
be dropped into another tool without re-typing anything.

## Workbook structure

| Tab | Purpose |
|---|---|
| `README` | Methodology and how to read the workbook (same content as this file, for anyone who only opens the workbook) |
| `Transactions` | 190 synthetic transactions + 4 helper flag columns |
| `Alert Summary` | Per-account risk scoring, built entirely from formulas referencing the Transactions sheet |

Open it in Excel, Google Sheets, or LibreOffice Calc — click any
cell to see the live formula that produced it.

## Typologies detected

Same four as the SQL version:

1. **Structuring** — deposits just under the $10,000 CTR reporting
   threshold, repeated.
2. **Rapid in-out / pass-through** — a large inbound wire followed
   quickly by an outbound wire of nearly the same amount.
3. **High velocity** — an unusual number of transactions for one
   account in a single day.
4. **Round-dollar wires to higher-risk jurisdictions** — repeated,
   suspiciously round wire amounts to a country on a higher-risk list.

Plus a flat scoring bump for PEP (Politically Exposed Person)
status, reflecting standard enhanced due diligence (EDD) policy.

## Formulas used (all standard, no add-ins)

- **`COUNTIFS` / `SUMIFS`** — counting/summing transactions matching
  multiple conditions per account (e.g., deposits between
  $8,000–$9,999).
- **`SUMPRODUCT`** — matching an inbound wire to a corresponding
  outbound wire for the same account, within a date window and
  amount tolerance — a cross-row comparison `COUNTIFS` alone can't
  express.
- **`MAXIFS`** — finding each account's single busiest day.
- **`INDEX`/`MATCH`** — pulling customer name and PEP status across
  sheets.
- **Nested `IF`** — combining typology hits into a risk score and a
  plain-English disposition (HIGH / MEDIUM / LOW).
- Conditional formatting highlights every suspicious row directly in
  the `Transactions` tab, so the trail from raw data to risk score is
  visible without leaving the sheet.

## Results

Running the formulas against the synthetic data correctly identifies
all four planted suspicious accounts, with zero false positives among
the seven normal accounts included specifically to test that:

| Account | Customer | Typology | Risk Score | Disposition |
|---|---|---|---|---|
| 1010 | Alex Rowe | Structuring (4 deposits) | 30 | MEDIUM |
| 1020 | Priya Nair | Rapid in-out | 25 | MEDIUM |
| 1040 | Omar Haddad | Round-dollar wires (4) | 20 | MEDIUM |
| 1030 | Daniel Kim | High velocity (14 txns/day) | 15 | LOW |
| 1120 | Grace Ito | PEP (EDD) | 10 | LOW |

![Transactions with suspicious rows highlighted](screenshots/transactions_highlighted.png)

![Alert Summary risk scoring](screenshots/alert_summary.png)

## A documented simplification

The SQL version's structuring query uses a true **rolling 7-day
window** (any 3+ qualifying deposits within any 7-day span). Pure
Excel formulas can't easily express a rolling window without array
formulas or a helper table, so this version flags 3+ qualifying
deposits across the account's **full history** instead — a
deliberate, disclosed trade-off. Documenting a limitation like this,
rather than glossing over it, is itself part of how a compliance
analyst would present this kind of work.

## Limitations

- Synthetic data only.
- Rules are illustrative, not exhaustive — a production TMS screens
  for many more typologies.
- Thresholds ($8,000 floor, $1,000 round-dollar granularity, 10-txn
  velocity cap) are example policy values, not statistically derived.

## About

Built by Biswajit Das, CAMS-certified compliance analyst, as a
portfolio piece — a deliberately different toolset from the SQL
version of the same project, to show the underlying logic transfers
across tools. 
