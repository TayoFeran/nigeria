# Setup Guide

## Nigeria FAAC Allocation Intelligence Platform
### Complete Step-by-Step Instructions — From Zero to Running Pipeline

---

This guide walks you through every step required to get this project running — including setting up GitHub, configuring Microsoft Fabric, uploading your data, and executing the pipeline for the first time. No prior data engineering experience is assumed.

---

## Contents

1. [What You Will Need](#1-what-you-will-need)
2. [Set Up Your GitHub Repository](#2-set-up-your-github-repository)
3. [Configure Microsoft Fabric](#3-configure-microsoft-fabric)
4. [Upload Your FAAC Files to the Lakehouse](#4-upload-your-faac-files-to-the-lakehouse)
5. [Create and Run the Notebook](#5-create-and-run-the-notebook)
6. [Verify the Output in the Warehouse](#6-verify-the-output-in-the-warehouse)
7. [Troubleshooting](#7-troubleshooting)

---

## 1. What You Will Need

Before you begin, confirm you have the following:

| Item | Required | Notes |
|---|---|---|
| A GitHub account | ✅ Yes | Free at [github.com](https://github.com) |
| A Microsoft Fabric workspace | ✅ Yes | Free trial at [fabric.microsoft.com](https://fabric.microsoft.com) |
| Your cleaned FAAC Excel files | ✅ Yes | One `.xlsx` file per month, structured as described in this guide |
| A modern web browser | ✅ Yes | Chrome or Edge recommended for Fabric |
| Git (optional) | ⬜ Optional | Only needed if pushing code from your local machine |

---

## 2. Set Up Your GitHub Repository

GitHub is where your code lives. It also serves as your professional portfolio — anyone you share the link with can read your code, your documentation, and your project history.

### Step 2.1 — Create a GitHub Account

If you do not already have one:

1. Go to [github.com](https://github.com)
2. Click **Sign up**
3. Enter your email, create a password, and choose a username
   - Choose a professional username — this will appear in your portfolio URL (e.g. `github.com/tayo-feran`)
4. Verify your email address

### Step 2.2 — Create the Repository

1. Once signed in, click the **+** icon in the top-right corner
2. Select **New repository**
3. Fill in the details:
   - **Repository name:** `faac-nigeria` (or `nigeria-faac-intelligence`)
   - **Description:** `End-to-end data platform for Nigeria's monthly FAAC allocation data — NBS source to Microsoft Fabric Warehouse.`
   - **Visibility:** Select **Public** (this is your portfolio — it needs to be visible)
   - Tick **Add a README file** — this creates the default file we will replace
4. Click **Create repository**

Your repository is now live at `https://github.com/YOUR-USERNAME/faac-nigeria`

### Step 2.3 — Upload the Project Files

You will upload three things: the README, the pipeline script, and the docs folder.

**Option A — Upload via GitHub website (no Git required)**

1. On your repository page, click **Add file** → **Upload files**
2. Drag and drop `README.md` from your computer and click **Commit changes**
3. Repeat for the pipeline file:
   - Click **Add file** → **Create new file**
   - In the filename box, type `pipeline/faac_pipeline.py` — GitHub will automatically create the `pipeline/` folder
   - Paste the full pipeline code into the editor
   - Click **Commit new file**
4. Repeat for each file in the `docs/` folder:
   - `docs/SETUP_GUIDE.md`
   - `docs/DATA_DICTIONARY.md`
   - `docs/CHANGELOG.md`

**Option B — Push from your computer using Git**

If you have Git installed, open a terminal in your project folder and run:

```bash
git init
git add .
git commit -m "Initial commit — FAAC pipeline and documentation"
git remote add origin https://github.com/YOUR-USERNAME/faac-nigeria.git
git push -u origin main
```

---

## 3. Configure Microsoft Fabric

### Step 3.1 — Sign In to Fabric

1. Go to [fabric.microsoft.com](https://fabric.microsoft.com)
2. Sign in with your Microsoft account
3. You will land on the Fabric home screen

### Step 3.2 — Confirm Your Workspace

1. In the left navigation, click **Workspaces**
2. Select **My workspace** (or the workspace where your Lakehouse and Warehouse live)
3. Confirm you can see two items:
   - `tayo_feran_lh` — this is your **Lakehouse** (file storage)
   - `tayo_feran_wh` — this is your **Warehouse** (SQL query layer)

If either is missing, you will need to create it:
- To create a Lakehouse: click **New** → **Lakehouse** → name it `tayo_feran_lh`
- To create a Warehouse: click **New** → **Warehouse** → name it `tayo_feran_wh`

> **Note:** If you name your Lakehouse or Warehouse differently, update the `LAKEHOUSE_NAME` and `WAREHOUSE_NAME` variables at the top of `faac_pipeline.py` before running.

---

## 4. Upload Your FAAC Files to the Lakehouse

### Step 4.1 — Understand the Required Folder Structure

Your files must be organised in exactly this structure inside the Lakehouse:

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
                    │   ├── 2020 Mar Disbursement.xlsx
                    │   └── ... (continue for all 12 months)
                    ├── 2021/
                    │   └── ... (12 files)
                    ├── 2022/
                    │   └── ... (12 files)
                    ├── 2023/
                    │   └── ... (12 files)
                    └── 2024/
                        └── ... (7 files — January through July)
```

> **File naming:** The pipeline reads the month from the filename. Use the format `{YYYY} {Mon} Disbursement.xlsx` exactly as shown above. Both full month names (`January`) and three-letter abbreviations (`Jan`) are supported.

### Step 4.2 — Create the Folder Structure

1. In your workspace, click on `tayo_feran_lh` to open the Lakehouse
2. In the left panel, click on **Files**
3. Click **New subfolder** and create `Spreedsheets`

   > Note: The folder name `Spreedsheets` (with the double `e`) matches what is already in your Lakehouse — keep this spelling consistent.

4. Open `Spreedsheets` and create subfolder `Nigeria`
5. Open `Nigeria` and create subfolder `FAAC`
6. Open `FAAC` and create subfolder `Refined`
7. Open `Refined` and create subfolders for each year: `2020`, `2021`, `2022`, `2023`, `2024`

### Step 4.3 — Upload Your Excel Files

1. Open the `2020` folder
2. Click **Upload** → **Upload files**
3. Select all 12 monthly Excel files for 2020
4. Wait for the upload to complete
5. Repeat for each year folder

**Verify the upload:** After uploading, click into any year folder and confirm your `.xlsx` files are visible. If you see files but the Fabric preview looks strange (showing `#REF!` errors and unfamiliar column names), this is a known Fabric rendering issue with multi-sheet Excel files. Your actual data is intact — the pipeline reads the file directly using pandas, bypassing the preview.

---

## 5. Create and Run the Notebook

### Step 5.1 — Create a New Notebook

1. In your Fabric workspace, click on `tayo_feran_lh` to open the Lakehouse
2. Click **Open notebook** → **New notebook**
   - Alternatively: in the workspace, click **New** → **Notebook**
3. A new notebook opens with one empty cell

### Step 5.2 — Attach the Lakehouse

This step is critical — without it, the notebook cannot access your files.

1. In the left panel of the notebook, you will see an **Explorer** section
2. Click **Add Lakehouse**
3. Select **Existing Lakehouse** → find and select `tayo_feran_lh`
4. Click **Add**

You should now see `tayo_feran_lh` in the left panel with a file tree showing your folders.

### Step 5.3 — Paste the Pipeline Code

1. Click inside the first (empty) code cell
2. Open `pipeline/faac_pipeline.py` from this repository
3. Select all the code (Ctrl+A) and copy it (Ctrl+C)
4. Click back in the notebook cell and paste (Ctrl+V)

### Step 5.4 — Run the Pipeline

1. Click **Run All** in the notebook toolbar (or press Shift+Enter to run the current cell)
2. The notebook will start a Spark session — this takes approximately 60–90 seconds on first run
3. Once the session is ready, the pipeline begins executing and prints a live log

**What a successful run looks like:**

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

─────────────────────────────────────────────────────────────────
🔗 Merging 55 monthly datasets...
   Total rows merged: 2,035
   Date range: 2020-01-01 to 2024-07-01
   States: 37

💾 Writing to warehouse: tayo_feran_wh.dbo.faac_monthly_income

✅ Successfully written to tayo_feran_wh.dbo.faac_monthly_income

=================================================================
  ✅ PIPELINE COMPLETE
=================================================================
  Files processed successfully : 55
  Files failed / skipped       : 0
  Total rows in final table    : 2,035
=================================================================
```

**Estimated run time:** 5–10 minutes depending on Fabric capacity tier.

---

## 6. Verify the Output in the Warehouse

### Step 6.1 — Open the Warehouse

1. Return to your workspace
2. Click on `tayo_feran_wh` to open the Warehouse
3. In the left panel, expand **Schemas** → **dbo** → **Tables**
4. You should see `faac_monthly_income` listed

### Step 6.2 — Run Verification Queries

Click **New SQL query** and run the following:

```sql
-- Check 1: Row count by year
-- Expected: ~444 rows per year (37 states × 12 months)
-- 2024 will be lower: 37 states × 7 months = 259 rows
SELECT
    year,
    COUNT(DISTINCT month_number)    AS months_loaded,
    COUNT(*)                        AS total_rows,
    FORMAT(SUM(total_gross_amount), 'N0') AS total_allocated_naira
FROM dbo.faac_monthly_income
GROUP BY year
ORDER BY year;
```

```sql
-- Check 2: Confirm all 37 states are present
SELECT
    state,
    COUNT(*) AS months_of_data
FROM dbo.faac_monthly_income
GROUP BY state
ORDER BY state;
```

```sql
-- Check 3: Top 5 states by total allocation (2020–2024)
SELECT TOP 5
    state,
    FORMAT(SUM(total_gross_amount), 'N0') AS total_allocation_naira
FROM dbo.faac_monthly_income
GROUP BY state
ORDER BY SUM(total_gross_amount) DESC;
```

If all three queries return sensible results, your pipeline has completed successfully.

---

## 7. Troubleshooting

### "Folder does not exist" for all years

**Cause:** The `BASE_PATH` in the pipeline does not match your actual Lakehouse folder structure.

**Fix:**
1. In your Lakehouse, click through the folder tree until you reach the year folders
2. Note the exact path — paying close attention to capitalisation and spelling
3. Update the `BASE_PATH` variable at the top of `faac_pipeline.py`:
   ```python
   BASE_PATH = "/lakehouse/default/Files/YOUR/EXACT/PATH/HERE"
   ```

### "No 'Income' sheet found" for some files

**Cause:** The sheet tab in that specific Excel file is named differently (e.g. `income`, `INCOME`, `Sheet1`).

**Fix:**
1. Open the flagged file in Excel on your computer
2. Check the sheet tab name at the bottom of the screen
3. The pipeline will print the available sheet names in the error message — use that to confirm
4. Rename the tab to `Income` and re-upload the file

### Pipeline completes but table is not in the Warehouse

**Cause:** The direct warehouse write failed and the pipeline fell back to a Lakehouse Delta table.

**Fix:** Check the pipeline output for the line:
```
🔄 Falling back: saving as Delta table in Lakehouse...
```
If this appears, the data was written as a Delta table to the Lakehouse instead. You can either:
- Reference it directly in Power BI from the Lakehouse
- Or create a shortcut from the Warehouse pointing to the Lakehouse Delta table

### Some months show fewer than 37 rows

**Cause:** The source Excel file for that month may have had a state row that could not be matched to the known state name list.

**Fix:** Open `source_file` column in the output to identify which file is affected. Open that file, find the unmatched row, and check if the state name is spelled unusually (e.g. `Nassarawa` instead of `Nasarawa`). Report it as an issue on GitHub if you cannot resolve it.

---

*For issues not covered here, open a GitHub issue with the full error message and the filename that caused it.*
