# WealthWizard raw synthetic dataset

Generated to match the raw schema (RAW_USER, RAW_CATEGORY, RAW_TRANSACTION, RAW_BUDGET, RAW_INSIGHT).
Total records: **~128,000+** across 7 files, spanning `.csv`, `.json`, and `.txt`.

## Files

| File | Format | Rows | Entity |
|---|---|---|---|
| `raw_user.csv` | CSV | 5,000 | User |
| `raw_category.json` | JSON | 20 | Category (master data) |
| `raw_budget.csv` | CSV | 8,000 | Budget |
| `raw_transaction_batch1.csv` | CSV | 50,000 | Transaction |
| `raw_transaction_batch2.json` | JSON | 30,000 | Transaction |
| `raw_transaction_batch3.txt` | Pipe-delimited TXT | 20,000 | Transaction |
| `raw_insight.json` | JSON | 15,000 | Insight |

Transactions are deliberately split across 3 formats/files — this mimics how raw data
usually lands from different source systems (a scheduled CSV export, an API dump in JSON,
and a legacy pipe-delimited flat-file feed).

## Dirty data injected (roughly 10–25% of rows per file, varies by file)

- **Negative amounts** — violates FR-01.1's "expense amount cannot be negative" rule
- **Currency-prefixed amounts** — e.g. `Rs.4521.0` stored as text instead of a clean number
- **Missing/empty amounts** — blank string or `N/A`
- **Inconsistent date formats** — mixes `YYYY-MM-DD`, `DD-MM-YYYY`, `DD/MM/YYYY`, `MM/DD/YYYY`, `YYYY/MM/DD` in the same file
- **Future-dated transactions** — violates FR-01.1's "date cannot be future-dated" rule
- **Orphan foreign keys** — `category_id = "C999"` that doesn't exist in `raw_category.json`
- **Missing `user_id`** on some transactions/budgets/insights
- **Duplicate `user_id` / exact duplicate transaction rows** — simulates re-ingestion
- **Leading/trailing whitespace** in text fields (name, description)
- **Invalid emails** (`invalid-email` instead of a real address)
- **Malformed budget periods** — e.g. `2026-13`, blank, or `Q1-2026` instead of `YYYY-MM`
- **Blank/unknown insight types**
- **`NULL` string literal** used instead of true nulls in the TXT file (typical of flat-file exports)

## Suggested use

- Load all three transaction files into a single `raw_transaction` staging table to
  practice schema-on-read / format normalization (CSV + JSON + pipe-delimited → common shape)
- Use the dirty rows to build your FR-01.4 validation & error-handling module, FR-02.5
  recalculation logic (orphan category handling), and FR-03.1 overspend/negative-amount rules
- `ingested_at` on every row lets you simulate incremental/raw-layer loads before building
  your cleaned/staging layer on top
