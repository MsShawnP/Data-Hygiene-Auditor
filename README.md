# Data Hygiene Auditor

A command-line tool that audits Excel files for common data quality issues and generates three reports from a single run: a visual HTML report, a sortable Excel findings file, and a PDF suitable for client delivery.

Built for consultants and data teams who need to show stakeholders *what's wrong* and *why it matters* — not just dump raw statistics.

## What It Detects

**Mixed Formats** — Identifies dates, phone numbers, and currency values stored in inconsistent formats within the same column. For example, `2023-01-15` alongside `Jan 15, 2023` and `01/15/2023` in one date field. The auditor recognizes 6 date patterns, 7 phone patterns, and 6 currency patterns.

**Misused Fields** — Flags data stored in the wrong column: reference codes in name fields, free text in currency columns, invalid email addresses, and mixed boolean representations (`Y/N` vs `1/0` vs `Active/Inactive` in the same field).

**Placeholder Floods** — Detects test values (`Test`, `N/A`, `TBD`, `000-000-0000`) that persisted into production, as well as suspiciously repeated values that may indicate defaults that were never updated.

**Phantom Duplicates** — Finds records that appear different on the surface due to casing, whitespace, or punctuation but represent the same entity after normalization. ID columns are automatically excluded from matching, so two records with different surrogate keys but identical content are still caught.

**Completeness Baseline** — Every field receives a null/missing analysis with severity rating, including detection of whitespace-only values that look populated but carry no data.

## Design Decisions

- **Automatic field classification.** Each column is inferred as one of: date, phone, currency, email, ID, name, categorical, freetext, or zipcode. The correct validation rules are applied based on the inferred type — date-format checks don't run against phone fields, and vice versa.
- **Severity ratings on every finding.** High, Medium, and Low thresholds are calibrated to the issue type (e.g., >30% format inconsistency = High; 10–30% = Medium).
- **Plain-English impact statements.** Every finding includes a "Why this matters" explanation written for non-technical stakeholders who need to understand the business consequence, not just the statistic.
- **Multi-sheet support.** All sheets in a workbook are audited independently.

## Output

A single run produces three files:

| Output | Use Case |
|--------|----------|
| `*_audit_report.html` | Visual walkthrough with severity badges, format distribution tables, null-completeness bars, and impact explanations. Designed to present to a client or stakeholder. |
| `*_audit_findings.xlsx` | One row per issue. Frozen header, auto-filter, severity color-coding. Built for the person who needs to work through the fixes. |
| `*_audit_report.pdf` | Same substance as the HTML in a format suitable for email attachments and formal deliverables. |

## Usage

```
pip install -r requirements.txt
python audit.py --input <file.xlsx> --output <report_directory>
```

### Options

| Flag | Description |
|------|-------------|
| `--input`, `-i` | Path to the Excel file to audit (required) |
| `--output`, `-o` | Directory for generated reports (required) |
| `--json` | Also output the raw findings as structured JSON |

### Example

```
python audit.py --input sample_messy_data.xlsx --output ./reports
```

```
Auditing: sample_messy_data.xlsx
Running analysis...
  Generating HTML report...
    → ./reports/sample_messy_data_audit_report.html
  Generating Excel findings...
    → ./reports/sample_messy_data_audit_findings.xlsx
  Generating PDF report...
    → ./reports/sample_messy_data_audit_report.pdf

Audit complete: 59 issues found
  High: 23 | Medium: 20 | Low: 16
```

## Sample Data

`generate_sample.py` creates a deliberately messy 30-row, 2-sheet Excel file that exercises every detection category. It serves as both a demo and a regression test:

```
python generate_sample.py
python audit.py --input sample_messy_data.xlsx --output ./reports
```

## Requirements

- Python 3.8+
- pandas
- openpyxl
- reportlab

## License

Proprietary — Lailara LLC
