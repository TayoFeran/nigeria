# 🇳🇬 Nigeria FAAC Allocation Intelligence Platform

> **Making Nigeria's public financial data accessible, comparable, and interpretable — across every presidential administration since 2020.**

---

## Overview

The **Federation Account Allocation Committee (FAAC)** distributes oil and tax revenues to Nigeria's 36 states and the FCT every month. These numbers are public record — but they are buried in government PDFs and Excel files that most Nigerians never see, let alone understand.

This project changes that.

It is an end-to-end data platform that:

- Ingests monthly FAAC Excel reports published by the **National Bureau of Statistics (NBS)**
- Cleans, merges, and loads them into a **Microsoft Fabric Warehouse** via an automated PySpark pipeline
- Will power a **public-facing Power BI dashboard** that enables any Nigerian — regardless of technical background — to answer questions like:
  - *How much has my state received since 2020?*
  - *Is the current government distributing more or less than its predecessors?*
  - *Which states benefit most from oil derivation, and by how much?*

This is civic data infrastructure. Built in the open.

---

## Table of Contents

- [Why This Project Matters](#why-this-project-matters)
- [Project Architecture](#project-architecture)
- [Repository Structure](#repository-structure)
- [Data Source & Coverage](#data-source--coverage)
- [Pipeline Walkthrough](#pipeline-walkthrough)
- [Getting Started](#getting-started)
  - [Prerequisites](#prerequisites)
  - [Step 1 — Set Up Microsoft Fabric](#step-1--set-up-microsoft-fabric)
  - [Step 2 — Prepare Your Files](#step-2--prepare-your-files)
  - [Step 3 — Run the Pipeline](#step-3--run-the-pipeline)
  - [Step 4 — Verify the Output](#step-4--verify-the-output)
- [Output Schema](#output-schema)
- [Known Limitations](#known-limitations)
- [Roadmap](#roadmap)
- [Contributing](#contributing)
- [Author](#author)
- [License](#license)

---

## Why This Project Matters

Nigeria's revenue allocation system is complex by design. The FAAC sharing formula accounts for population, land mass, equality of states, and a 13% derivation principle for oil-producing states. For most Nigerians, the monthly FAAC communiqué is either ignored or misunderstood.

The result: public discourse about government spending is driven by opinion rather than evidence.

This platform is a direct response to that gap. By presenting the data in a structured, visual, and contextualised format — broken down by state, by administration, and by revenue type — it enables citizens, journalists, researchers, and policymakers to engage with the numbers on their own terms.

---

## Project Architecture

```
NBS Excel Reports (Source)
        │
        ▼
┌─────────────────────────────┐
│   Microsoft Fabric          │
│   Lakehouse (tayo_feran_lh) │
│                             │
│   Files/Spreedsheets/       │
│   Nigeria/FAAC/Refined/     │
│   ├── 2020/                 │
│   ├── 2021/                 │
│   ├── 2022/                 │
│   ├── 2023/                 │
│   └── 2024/                 │
└────────────┬────────────────┘
             │  PySpark Pipeline
             ▼
┌─────────────────────────────┐
│   Microsoft Fabric          │
│   Warehouse (tayo_feran_wh) │
│                             │
│   dbo.faac_monthly_income   │
└────────────┬────────────────┘
             │  Power BI DirectQuery (Planned)
             ▼
┌─────────────────────────────┐
│   Public Power BI Dashboard │
│   (In Development)          │
└─────────────────────────────┘
```

**Technology Stack**

| Layer | Technology |
|---|---|
| Data Storage | Microsoft Fabric Lakehouse (OneLake) |
| Data Warehouse | Microsoft Fabric Warehouse |
| Pipeline / ETL | PySpark (Fabric Notebook) |
| File Format | Excel (.xlsx) — NBS published format |
| Visualisation | Power BI (planned) |
| Version Control | GitHub |

---

## Repository Structure

```
faac-nigeria/
│
├── README.md                        ← You are here
│
├── pipeline/
│   └── faac_pipeline.py             ← Main PySpark ingestion pipeline
│
├── docs/
│   ├── SETUP_GUIDE.md               ← Full step-by-step setup instructions
│   ├── DATA_DICTIONARY.md           ← Column definitions and data quality notes
│   └── CHANGELOG.md                 ← Version history and update log
│
└── assets/
    └── architecture_diagram.png     ← System architecture (see above)
```

---

## Data Source & Coverage

| Attribute | Detail |
|---|---|
| **Primary Source** | National Bureau of Statistics (NBS) — [nigerianstat.gov.ng](https://nigerianstat.gov.ng) |
| **Report Type** | Monthly FAAC Disbursement Reports (Excel format) |
| **Coverage** | January 2020 — July 2024 |
| **Granularity** | Monthly, per state (37 beneficiaries including FCT) |
| **Gap** | NBS ceased publishing Excel reports after July 2024. See [Known Limitations](#known-limitations). |
| **File Naming Convention** | `{YYYY} {Mon} Disbursement.xlsx` (e.g. `2020 Apr Disbursement.xlsx`) |
| **Sheet Used** | `Income` — contains gross inflows per state |

---

## Pipeline Walkthrough

The pipeline (`pipeline/faac_pipeline.py`) executes the following steps in sequence:

**1. Path Verification**
Before any processing begins, the pipeline confirms that each year folder exists at the expected Lakehouse path and reports how many Excel files it finds per year. If any folder is missing, it reports this clearly and skips that year rather than failing silently.

**2. File Discovery**
For each year in scope (2020–2024), the pipeline uses Python's `os.walk()` to list all `.xlsx` files in the year folder. This approach is used deliberately — Spark's native file listing API (`wholeTextFiles`) is incompatible with Microsoft Fabric's OneLake storage layer.

**3. Month Extraction**
The pipeline parses the disbursement month from the filename. It supports both full month names (`January`) and standard abbreviations (`Jan`, `Apr`, `Dec`), making it resilient to minor naming inconsistencies across years.

**4. Excel Parsing**
Each file is read using `pandas` with `openpyxl`. The pipeline:
- Locates the `Income` sheet via case-insensitive matching
- Reads from row 1 (the header row) with no rows skipped
- Filters to the 37 valid state beneficiaries, excluding subtotal and summary rows
- Cleans numeric columns (removes thousand separators, converts dashes to zero)

**5. Enrichment**
Four metadata columns are added to every row:
- `disbursement_date` — first day of the disbursement month, in `DATE` format
- `year` and `month_number` — integer extracts for easier slicing in Power BI
- `president` — the head of state in office at the time of disbursement

**6. Merge and Write**
All monthly DataFrames are unioned into a single Spark DataFrame, sorted by date and state, and written to `tayo_feran_wh.dbo.faac_monthly_income` in overwrite mode. If the direct warehouse write fails, the pipeline automatically falls back to saving a Delta table in the Lakehouse.

---

## Getting Started

For the complete setup guide with screenshots and troubleshooting, see [`docs/SETUP_GUIDE.md`](docs/SETUP_GUIDE.md).

A condensed version follows below.

### Prerequisites

| Requirement | Notes |
|---|---|
| Microsoft Fabric workspace | Free trial available at [fabric.microsoft.com](https://fabric.microsoft.com) |
| Fabric Lakehouse | Named `tayo_feran_lh` or update `LAKEHOUSE_NAME` in the pipeline |
| Fabric Warehouse | Named `tayo_feran_wh` or update `WAREHOUSE_NAME` in the pipeline |
| NBS FAAC Excel files | Manually downloaded and cleaned — see Data Source section |
| Python (local, optional) | Only needed if running outside Fabric |

### Step 1 — Set Up Microsoft Fabric

1. Sign in to [fabric.microsoft.com](https://fabric.microsoft.com)
2. Create or open your workspace
3. Confirm you have a **Lakehouse** (`tayo_feran_lh`) and a **Warehouse** (`tayo_feran_wh`) provisioned

### Step 2 — Prepare Your Files

Upload your cleaned FAAC Excel files into the Lakehouse following this exact folder structure:

```
tayo_feran_lh/
└── Files/
    └── Spreedsheets/
        └── Nigeria/
            └── FAAC/
                └── Refined/
                    ├── 2020/
                    │   ├── 2020 Jan Disbursement.xlsx
                    │   ├── 2020 Feb Disbursement.xlsx
                    │   └── ... (one file per month)
                    ├── 2021/
                    ├── 2022/
                    ├── 2023/
                    └── 2024/
```

> **Important:** Each Excel file must contain a sheet named `Income` (capital I). The first row of that sheet must be the column header row. See [`docs/DATA_DICTIONARY.md`](docs/DATA_DICTIONARY.md) for the expected column names.

### Step 3 — Run the Pipeline

1. In your Fabric workspace, open `tayo_feran_lh`
2. Click **New** → **Notebook**
3. In the left panel, click **Add Lakehouse** and select `tayo_feran_lh`
4. Copy the full contents of `pipeline/faac_pipeline.py` into the first notebook cell
5. Click **Run All**

The pipeline will print a live progress log to the notebook output. A successful run looks like:

```
=================================================================
  FAAC Monthly Income Pipeline
  Source:  /lakehouse/default/Files/Spreedsheets/Nigeria/FAAC/Refined
  Target:  tayo_feran_wh.dbo.faac_monthly_income
=================================================================

🔍 Verifying folder structure...

   ✅ 2020  →  found (12 Excel files)
   ✅ 2021  →  found (12 Excel files)
   ✅ 2022  →  found (12 Excel files)
   ✅ 2023  →  found (12 Excel files)
   ✅ 2024  →  found (7 Excel files)

📁 Processing year: 2020
   📄 2020 Jan Disbursement.xlsx  →  January 2020 ... ✅ 37 rows
   📄 2020 Feb Disbursement.xlsx  →  February 2020 ... ✅ 37 rows
   ...

✅ Successfully written to tayo_feran_wh.dbo.faac_monthly_income
```

### Step 4 — Verify the Output

In your Fabric Warehouse, run the following SQL to confirm the data loaded correctly:

```sql
-- Row count by year — expect ~37 rows × 12 months per year
SELECT
    year,
    COUNT(DISTINCT month_number)    AS months_loaded,
    COUNT(*)                        AS total_rows,
    SUM(total_gross_amount)         AS total_allocated_naira
FROM dbo.faac_monthly_income
GROUP BY year
ORDER BY year;
```

```sql
-- Verify all 37 states are present
SELECT DISTINCT state
FROM dbo.faac_monthly_income
ORDER BY state;
```

---

## Output Schema

The pipeline writes to `tayo_feran_wh.dbo.faac_monthly_income` with the following structure:

| Column | Data Type | Description |
|---|---|---|
| `state` | VARCHAR | State name (e.g. `Lagos`, `Rivers`, `FCT`) |
| `lg_count` | INT | Number of local governments in the state |
| `statutory_allocation` | BIGINT | Statutory allocation in Naira |
| `oil_derivation` | BIGINT | 13% derivation for oil-producing states (zero for non-oil states) |
| `exchange_gain_difference` | BIGINT | Allocation from FX gains on dollar-denominated revenues |
| `total_ecology_fund` | BIGINT | Ecology and derivation fund component |
| `gross_vat_allocation` | BIGINT | Share of Value Added Tax pool |
| `others_income` | BIGINT | Miscellaneous income components |
| `total_gross_amount` | BIGINT | Sum of all income components — the headline allocation figure |
| `year` | INT | Disbursement year (e.g. `2020`) |
| `month_number` | INT | Disbursement month as integer (1 = January, 12 = December) |
| `month_name` | VARCHAR | Disbursement month as text (e.g. `April`) |
| `disbursement_date` | DATE | First day of disbursement month (e.g. `2020-04-01`) |
| `president` | VARCHAR | President in office at time of disbursement |
| `source_file` | VARCHAR | Original filename (for traceability and auditing) |

> All monetary values are in **Nigerian Naira (NGN)**, stored as whole numbers (no decimals). Do not aggregate across years without considering the effect of Naira devaluation — see [`docs/DATA_DICTIONARY.md`](docs/DATA_DICTIONARY.md) for guidance.

---

## Known Limitations

**1. NBS publication gap (August 2024 – present)**
The National Bureau of Statistics stopped publishing monthly FAAC Excel reports after July 2024. Data for subsequent months must be sourced manually from PDF communiqués published by the Federal Ministry of Finance at [finance.gov.ng](https://finance.gov.ng). A future pipeline update will automate extraction from these PDFs.

**2. Nominal vs. real values**
All monetary figures are nominal Naira. The dramatic increase in FAAC allocations from mid-2023 onwards is substantially driven by Naira devaluation following the removal of the fuel subsidy in June 2023 and the unification of the FX market — not by a proportional increase in real revenue. Any cross-administration comparison must account for this.

**3. Pre-2020 data**
NBS Excel reports are only reliably available from 2020. Earlier years (1999–2019) are available in PDF format only and require manual extraction or OCR processing. A separate historical dataset covering annual totals from 1999 to 2024 is maintained alongside this project.

**4. Pipeline run mode**
The pipeline currently runs in `overwrite` mode — each execution replaces the entire warehouse table. This is appropriate for the current dataset size. An incremental (append-only) mode is planned for the roadmap.

---

## Roadmap

| Phase | Description | Status |
|---|---|---|
| **Phase 1** | PySpark ingestion pipeline — Lakehouse to Warehouse | ✅ Complete |
| **Phase 2** | Deduction sheet pipeline (`dbo.faac_monthly_deductions`) | 🔲 Planned |
| **Phase 3** | Pre-2020 historical data integration (PDF extraction) | 🔲 Planned |
| **Phase 4** | Dim_Calendar and Dim_State dimension tables | 🔲 Planned |
| **Phase 5** | Power BI data model and DAX measures | 🔲 Planned |
| **Phase 6** | Public-facing Power BI dashboard — administration comparisons | 🔲 Planned |
| **Phase 7** | Automated monthly refresh (Fabric Data Pipeline) | 🔲 Planned |

---

## Contributing

This project is open to collaboration, particularly from data professionals, journalists, and civic technologists working on Nigerian public finance.

If you identify data quality issues, have access to historical NBS reports, or want to contribute to the Power BI dashboard layer, please open an issue or submit a pull request.

**Reporting a data discrepancy:** If a figure in the output does not match the source NBS report, please open an issue and include the source file name, the state, the month, and the expected vs. actual value.

---

## Author

**Tayo Feran**
Data Analyst — 6 years in Nigeria's financial sector
Microsoft Fabric · Power BI · PySpark · SQL

---

## License

This project is licensed under the MIT License. The underlying FAAC data is published by the National Bureau of Statistics and is in the public domain.
