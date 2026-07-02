# Changelog

All notable changes to this project are documented here.
Format follows [Keep a Changelog](https://keepachangelog.com/en/1.0.0/).

---

## [1.0.0] — 2025

### Added
- Initial PySpark ingestion pipeline (`pipeline/faac_pipeline.py`)
- Reads monthly FAAC Excel files (NBS format) from Microsoft Fabric Lakehouse
- Parses `Income` sheet with case-insensitive sheet name matching
- Extracts month from filename — supports full names and 3-letter abbreviations
- Filters to 37 valid state beneficiaries; discards subtotal and summary rows
- Cleans numeric columns: removes thousand separators, converts dashes to zero
- Enriches each row with: `disbursement_date`, `year`, `month_number`, `month_name`, `president`, `source_file`
- Writes merged output to `tayo_feran_wh.dbo.faac_monthly_income` in overwrite mode
- Fallback to Lakehouse Delta table if direct warehouse write fails
- Path verification block at startup — reports file count per year before processing begins
- Uses `os.walk()` for file listing (compatible with OneLake; replaces broken `wholeTextFiles` approach)
- Coverage: January 2020 – July 2024 (55 monthly files, 2,035 rows)
- Full documentation: README, SETUP_GUIDE, DATA_DICTIONARY, CHANGELOG

### Known Issues
- Deduction sheet not yet ingested
- Pre-2020 data not yet included
- Pipeline runs in full overwrite mode only (no incremental load)

---

## [Planned] — Phase 2

- Deduction sheet pipeline → `dbo.faac_monthly_deductions`
- Net allocation view: income minus deductions

## [Planned] — Phase 3

- PDF extraction pipeline for pre-2020 historical data (1999–2019)
- Integration with annual totals seed dataset

## [Planned] — Phase 4

- `Dim_Calendar` table with fiscal year logic (April-start), Nigerian public holidays
- `Dim_State` table with geopolitical zone, oil-producing flag, LGA count

## [Planned] — Phase 5 & 6

- Power BI data model and DAX measures
- Public-facing dashboard: cross-administration allocation comparison
