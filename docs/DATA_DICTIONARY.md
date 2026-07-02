# Data Dictionary

## Nigeria FAAC Allocation Intelligence Platform
### Column Definitions, Source Notes & Analytical Guidance

---

## Source Document

Each row in `dbo.faac_monthly_income` originates from the **Income sheet** of a monthly NBS FAAC Disbursement Report.

- **Publisher:** National Bureau of Statistics (NBS), Federal Republic of Nigeria
- **Original format:** Microsoft Excel (.xlsx), one file per month
- **URL:** [nigerianstat.gov.ng/elibrary](https://nigerianstat.gov.ng/elibrary) — search "FAAC"
- **Coverage in this dataset:** January 2020 – July 2024

---

## Table: `dbo.faac_monthly_income`

### Identifiers & Dimensions

| Column | Data Type | Source | Description |
|---|---|---|---|
| `state` | VARCHAR | NBS (`Beneficiaries` column) | Name of the state or federal territory. 37 distinct values: Nigeria's 36 states plus the FCT (Federal Capital Territory, Abuja). |
| `year` | INT | Derived from filename | The calendar year of disbursement. Range: 2020–2024. |
| `month_number` | INT | Derived from filename | The month of disbursement as an integer. 1 = January, 12 = December. |
| `month_name` | VARCHAR | Derived from filename | The month of disbursement as a full text name (e.g. `April`). |
| `disbursement_date` | DATE | Derived | First day of the disbursement month in ISO format (`YYYY-MM-DD`). Used as the primary date key for time intelligence in Power BI. |
| `president` | VARCHAR | Derived from date | The President of the Federal Republic of Nigeria in office at the time of disbursement. Values: `Buhari`, `Tinubu`. Note: the transition occurred in May 2023. |
| `source_file` | VARCHAR | Derived from filename | The original Excel filename from which this row was extracted (e.g. `2020 Apr Disbursement.xlsx`). Used for traceability and audit. |

---

### Revenue Components (Monetary — Nigerian Naira, whole numbers)

All monetary columns are stored in **Nigerian Naira (NGN)** as `BIGINT` (whole numbers, no decimal places). Values in the source NBS files are already in Naira — no conversion is applied.

| Column | NBS Column Name | Description |
|---|---|---|
| `statutory_allocation` | `Statutory Allocation` | The primary revenue share distributed to states from the Federation Account. This is derived mainly from oil receipts, company income tax, and customs duties, split according to the constitutional revenue sharing formula. This is the largest component for most states. |
| `oil_derivation` | `Oil Derivation` | The 13% derivation payment constitutionally mandated for oil-producing states under Section 162(2) of the 1999 Constitution. **Non-oil-producing states receive zero.** The six primary beneficiaries are: Delta, Rivers, Akwa Ibom, Bayelsa, Ondo, and Edo. |
| `exchange_gain_difference` | `Exchange Gain Difference` | Revenue credited to states arising from the difference between the exchange rate used to book oil receipts and the prevailing market rate. This component grew substantially from mid-2023 following the unification of Nigeria's FX market. |
| `total_ecology_fund` | `Total Ecology Fund` | Allocations from the Ecological Fund, which supports environmental remediation projects. Distributed across states based on ecological vulnerability factors. Often zero for many states in a given month. |
| `gross_vat_allocation` | `Gross VAT Allocation` | Each state's share of the Value Added Tax pool. VAT is collected nationally and redistributed: 50% to states equally, 30% by population, 20% by derivation. This is a significant and growing component as non-oil tax revenues increase. |
| `others_income` | `Others Income` | Miscellaneous income items including solid minerals revenue, stamp duties, and other non-standard receipts. Typically small and irregular. |
| `total_gross_amount` | `Total Gross Amount` | The **sum of all income components** for a given state in a given month. This is the headline allocation figure — the total amount credited to the state before any deductions. **Use this column as the primary measure for allocation analysis.** |

---

### Derived Financial Indicator

| Column | Description |
|---|---|
| `lg_count` | The number of Local Government Areas (LGAs) in the state, as published by NBS. This is a static reference field — it does not change month to month. It is included as a denominator for per-LGA allocation calculations. Range: 8 (Bayelsa) to 44 (Kano). |

---

## Critical Analytical Notes

### 1. Nominal vs. Real Values

All figures are **nominal Naira**. They are not adjusted for inflation.

The FAAC pool roughly doubled in nominal terms between 2022 and 2024. A significant portion of this increase is not due to higher oil production or improved revenue collection — it reflects:

- **Fuel subsidy removal (June 2023):** Prior to this, petroleum revenues were substantially netted off at source to fund the subsidy. Removal meant the full gross amount flowed into the Federation Account.
- **FX market unification (June 2023):** Nigeria maintained an artificially low official exchange rate for years. Unifying the FX market meant that dollar-denominated oil revenues, when converted to Naira at the market rate, produced a much larger Naira figure — even if the volume of oil sold was unchanged.

**Implication for cross-year analysis:** A ₦10 billion allocation in 2020 represents significantly more real purchasing power than ₦10 billion in 2024. For accurate inter-year comparison, analysts should deflate values using the Consumer Price Index (CPI) published by NBS, or express values in USD using the prevailing official exchange rate.

### 2. Cross-Administration Comparison

This dataset covers two administrations:

| President | Period in dataset |
|---|---|
| Muhammadu Buhari | January 2020 – May 2023 |
| Bola Ahmed Tinubu | May 2023 – July 2024 |

The `president` column assigns each disbursement to the president in office at the time. The transition month (May 2023) is assigned to Tinubu, as he was inaugurated on 29 May 2023 and the June 2023 disbursement reflected post-inauguration policy.

**Do not compare raw totals between administrations without normalising for time period length and the structural economic changes of mid-2023.**

### 3. Oil Derivation — Who Benefits

The 13% derivation is exclusive to oil-producing states. In this dataset, states that consistently receive non-zero `oil_derivation` values are:

- **Delta** — typically the largest beneficiary
- **Rivers**
- **Akwa Ibom**
- **Bayelsa**
- **Ondo**
- **Edo**
- **Imo** (smaller amounts)
- **Abia** (occasional, smaller amounts)

This structural advantage means that comparison of `total_gross_amount` across states without separating out derivation income can be misleading. Consider analysing `statutory_allocation + gross_vat_allocation` as a "derivation-neutral" measure when comparing oil and non-oil states.

### 4. FCT (Federal Capital Territory)

The FCT — Abuja — is included as one of the 37 beneficiaries. It is administered by the Federal Government directly and does not have an elected governor. Its allocation is managed by the FCT Administration. Its per-capita allocation is typically lower than most states due to the direct federal funding it receives through other channels.

### 5. Deductions

This table contains **gross income only**. FAAC allocations are subject to deductions before states receive their net credit, including:

- Refunds to the Federation Account
- External debt service obligations
- Contractors' debt deductions
- Paris Club loan deductions (historical)

The deduction data is contained in the `Deduction` sheet of each source Excel file and is **not yet ingested** by this pipeline. It is listed as Phase 2 in the project roadmap. Analysts should note that `total_gross_amount` does not equal the actual cash transferred to the state.

---

## State Reference Table

| State | Geopolitical Zone | Oil Producing | LGAs |
|---|---|---|---|
| Abia | South East | Marginal | 17 |
| Adamawa | North East | No | 21 |
| Akwa Ibom | South South | Yes | 31 |
| Anambra | South East | No | 21 |
| Bauchi | North East | No | 20 |
| Bayelsa | South South | Yes | 8 |
| Benue | North Central | No | 23 |
| Borno | North East | No | 27 |
| Cross River | South South | Marginal | 18 |
| Delta | South South | Yes | 25 |
| Ebonyi | South East | No | 13 |
| Edo | South South | Yes | 18 |
| Ekiti | South West | No | 16 |
| Enugu | South East | No | 17 |
| FCT | — | No | 6 |
| Gombe | North East | No | 11 |
| Imo | South East | Marginal | 27 |
| Jigawa | North West | No | 27 |
| Kaduna | North West | No | 23 |
| Kano | North West | No | 44 |
| Katsina | North West | No | 34 |
| Kebbi | North West | No | 21 |
| Kogi | North Central | No | 21 |
| Kwara | North Central | No | 16 |
| Lagos | South West | No | 20 |
| Nasarawa | North Central | No | 13 |
| Niger | North Central | No | 25 |
| Ogun | South West | No | 20 |
| Ondo | South West | Yes | 18 |
| Osun | South West | No | 30 |
| Oyo | South West | No | 33 |
| Plateau | North Central | No | 17 |
| Rivers | South South | Yes | 23 |
| Sokoto | North West | No | 23 |
| Taraba | North East | No | 16 |
| Yobe | North East | No | 17 |
| Zamfara | North West | No | 14 |

---

*This document is maintained alongside the pipeline code. If a column definition changes due to NBS format changes, this dictionary should be updated in the same commit.*
