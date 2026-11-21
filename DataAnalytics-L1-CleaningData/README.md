# Retail Store Sales: Data Cleaning

A complete, documented cleaning of a deliberately messy retail sales dataset, turning **12,575 raw transactions** into an analysis-ready dataset of **11,852 rows**. Every decision is justified and verified.

**Oasis Infobyte Internship (OIBSIP) · Data Analytics · Level 1, Task 3: Cleaning Data**

---

## Table of Contents

- [Project Overview](#project-overview)
- [Dataset](#dataset)
- [Tools and Libraries](#tools-and-libraries).
- [Problems Found](#problems-found)
- [Cleaning Workflow](#cleaning-workflow)
- [Before vs After Summary](#before-vs-after-summary)
- [Key Decisions](#key-decisions)
- [Project Structure](#project-structure)
- [How to Run](#how-to-run)
- [Limitations](#limitations)
- [License and Attribution](#license-and-attribution)
- [Author](#author)

---

## Project Overview

Real-world data is rarely clean. This project takes a retail store sales dataset full of duplicates, missing values, inconsistent labels, wrong data types and incorrect totals, and cleans it step by step using **Python and pandas**.

The focus is not only on *what* was fixed, but on *why* each fix is safe. Wherever possible, missing values were **recovered using rules found in the data** rather than filled with averages.

## Dataset

| | |
|---|---|
| **Dataset used** | Advance Retail Store Sales Data (Kaggle), a synthetic dirty retail dataset for practice |
| **Published by** | Brian.C (Kaggle) |
| **Original dataset** | Dirty Retail Store Sales dataset by ahmedmohamed2003 ([original on Kaggle](https://www.kaggle.com/datasets/ahmedmohamed2003/retail-store-sales-dirty-for-data-cleaning)) |
| **License** | [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/) |
| **Size (raw)** | 12,575 rows × 11 columns |
| **Period** | 1 January 2022 to 18 January 2025 |

The dataset used is a remixed and extended version of the original by ahmedmohamed2003, with repeated records, varied representations of similar values and mismatched numbers added on purpose to simulate real data problems. The data is **synthetic**.


The dataset contains retail transaction records with 11 fields:

| Column | Description |
|---|---|
| `Transaction ID` | Identifier assigned to a transaction |
| `Customer ID` | Identifier assigned to a customer |
| `Category` | Product category |
| `Item` | Item or product identifier/name |
| `Price Per Unit` | Price of one unit |
| `Quantity` | Number of units purchased |
| `Total Spent` | Total amount recorded for the transaction |
| `Payment Method` | Payment method used |
| `Location` | Sales channel/location |
| `Transaction Date` | Date of the transaction |
| `Discount Applied` | Indicates whether a discount was applied |

The cleaned dataset covers **1 January 2022 to 18 January 2025**, based on the notebook's final date statistics.

## Tools and Libraries

- **Python** — programming language
- **Jupyter Notebook** — interactive development and documentation
- **pandas** — data loading, inspection, transformation, missing-value handling, deduplication and export
- **NumPy** — numerical operations and tolerance-aware validation
- **CSV** — cleaned dataset output format

## Problems Found

| Issue | Where | Finding |
|---|---|---|
| Duplicate rows | All columns | 125 exact duplicates |
| Missing values | Item, Price Per Unit, Quantity, Total Spent, Discount Applied | 7,248 blanks in total |
| Inconsistent text | Payment Method, Location | 15 spellings for 3 values; 9 for 2 values |
| Mixed boolean formats | Discount Applied | `True`, `true`, `T`, `1`, `False`, `false`, `F`, `0` |
| Wrong data types | Transaction Date, Quantity, Discount Applied | Dates as text, whole numbers as decimals, booleans as text |
| Incorrect totals | Total Spent | 1,121 rows where Total ≠ Price × Quantity |

## Cleaning Workflow

### 1. Load and preserve the source data

The raw dataset was loaded into `df_raw`. A separate working copy (`df`) was created so that the original data remained available for comparison.

### 2. Inspect data quality

The initial inspection covered:

- Dataset shape and column names
- Data types
- Missing-value counts
- Exact duplicate rows
- Unique categorical values
- Descriptive statistics and numeric ranges
- Consistency between `Total Spent` and `Price Per Unit × Quantity`

### 3. Remove duplicates

The notebook identified 125 duplicate rows and 125 repeated `Transaction ID`–`Customer ID` combinations. Duplicate removal was performed using that two-column combination, keeping the first occurrence and resetting the index.

The dataset decreased from 12,575 to 12,450 rows.

### 4. Standardise categorical values

Inconsistent text representations were normalised:

- `Payment Method`: 15 representations reduced to `Cash`, `Credit Card` and `Digital Wallet`.
- `Location`: 9 representations reduced to `Online` and `In-store`.

Whitespace and casing were handled before mapping the values to standard labels.

### 5. Standardise `Discount Applied`

The following representations were mapped:

- `true`, `T`, `1` → `True`
- `false`, `F`, `0` → `False`

The column was converted to pandas nullable Boolean dtype. Missing values were preserved because the available fields did not establish whether a discount had been applied.

### 6. Correct data types

The notebook applied the following types:

| Column(s) | Final in-memory dtype |
|---|---|
| Transaction IDs, customer IDs and text fields | `string` |
| `Price Per Unit` | `float64` |
| `Quantity` | `Int64` (nullable integer) |
| `Total Spent` | `Float64` (nullable float) |
| `Transaction Date` | `datetime64` |
| `Discount Applied` | `boolean` (nullable Boolean) |

### 7. Recover missing unit prices

After duplicate removal, 607 `Price Per Unit` values were missing. For these records, the notebook calculated:

`Total Spent / Quantity`

A calculated value was used only if it matched a unit price already observed in the dataset. All 607 values passed this check and were recovered.

### 8. Recover missing item names

There were 1,205 missing `Item` values after duplicate removal. The notebook tested whether each `Category`–`Price Per Unit` combination corresponded to exactly one item. The test passed for the observed data.

Missing item names were then recovered using that pair. No `Unknown` item values remained.

### 9. Handle incomplete sales records

There were 598 records where both `Quantity` and `Total Spent` were missing. Neither value could be calculated from the other, so the records were removed rather than assigning unsupported values.

The dataset decreased from 12,450 to 11,852 rows.

### 10. Recalculate `Total Spent`

After recovering missing prices and removing records with both quantity and total missing, 1,121 rows had totals inconsistent with the multiplication of unit price and quantity.

The column was recalculated:

```python
df["Total Spent"] = df["Price Per Unit"] * df["Quantity"]
```

A second validation using `numpy.isclose()` returned zero mismatched rows.

### 11. Detect and review outliers

The Interquartile Range (IQR) method was applied to `Price Per Unit`, `Quantity` and `Total Spent`.

It flagged 60 records. Every flagged record had:

- `Price Per Unit` = 41
- `Quantity` = 10
- `Total Spent` = 410

These values were within the observed business ranges (unit price 5–41, quantity 1–10 and total spent up to 410). They were retained because they represented valid maximum-value purchases, not data-entry errors.

## Before vs After Summary

| Quality check | Before | After |
|---|---:|---:|
| Rows | 12,575 | 11,852 |
| Columns | 11 | 11 |
| Exact duplicate rows | 125 | 0 |
| Missing `Item` | 1,220 | 0 |
| Missing `Price Per Unit` | 614 | 0 |
| Missing `Quantity` | 606 | 0 |
| Missing `Total Spent` | 606 | 0 |
| Missing `Discount Applied` | 4,202 | 3,953 |
| Payment method labels | 15 | 3 |
| Location representations labels | 9 | 2 |
| Rows where Total ≠ Price × Quantity | 1,121* | 0 |

\* Measured after duplicates and unrecoverable rows were removed.
The remaining 3,953 missing `Discount Applied` values are intentional. They were not imputed because the dataset did not provide a reliable basis for doing so.

**Missing values by column**

| Column | Before | After |
|---|---|---|
| Item | 1,220 | 0 |
| Price Per Unit | 614 | 0 |
| Quantity | 606 | 0 |
| Total Spent | 606 | 0 |
| Discount Applied | 4,202 | 3,953 |

**Data types**

| Column | Before | After |
|---|---|---|
| Quantity | float64 | Int64 |
| Transaction Date | text | datetime |
| Discount Applied | text | boolean (allows blanks) |


## Key Decisions

1. **Discount information remains incomplete.** Missing values mean unknown, not necessarily `False`.
2. **598 rows were removed** because both quantity and total spent were missing. This avoids inventing transaction values but reduces the available records.
3. **Item recovery relies on a uniqueness assumption:** in the observed data, `Category` plus `Price Per Unit` identified one item. This relationship should be rechecked if the dataset changes.
4. **Fix totals before outliers.** Wrong totals would have created false outliers.
5. **Total Spent was recalculated for all retained rows.** This assumes the intended business rule is exactly unit price multiplied by quantity, without additional taxes, discounts or fees.
6. **Duplicate removal relies on the composite key** `Transaction ID` + `Customer ID`. This should match the source system's transaction-identity rules.
7. **Keep valid extremes.** The 60 flagged rows are real maximum-value orders, not errors.


## Project Structure

```
DataAnalytics-L1-RetailDataCleaning/
├── README.md                              # this file
├── Task1_DataCleaning.ipynb               # cleaning notebook (code, outputs, observations)
├── retail_store_sales.csv                 # raw dataset (CC BY-SA 4.0, see License and Attribution)
├── cleaned_retail_store_dataset.csv       # cleaned output: 11,852 rows × 11 columns
└── Data_cleaning_documentation.pdf       # full documentation report
```

## How to Run

1. **Get the data.** Use the included `retail_store_sales.csv`, or download *Advance Retail Store Sales Data* by Brian.C from Kaggle and save it under that name in the same folder as the notebook.
2. **Install the requirements:**
   ```bash
   pip install pandas numpy jupyter
   ```
3. **Run the notebook:**
   ```bash
   jupyter notebook Task1_DataCleaning.ipynb
   ```
   Run all cells from top to bottom. The last cell writes `cleaned_retail_store_dataset.csv`.

## Limitations

- **Discount Applied** is still about 33% blank. Any discount analysis must exclude or separately report these blanks.
- **598 rows were removed.** They contained valid Category, Payment Method, Location and Date values that are now excluded.
- **Price and Quantity were treated as the reliable fields** when Total Spent was recalculated. The checks support this, but the original totals cannot be independently verified.
- **The data is synthetic.** Findings describe this dataset, not a real store.
- The valid ranges used in the range check (Price 5 to 41, Quantity 1 to 10) come from the data itself, not from an external business rule.

## License and Attribution

- **Original dataset:** Dirty Retail Store Sales dataset by **ahmedmohamed2003** (Kaggle).
- **Extended version used here:** *Advance Retail Store Sales Data* by **Brian.C** (Kaggle), licensed under [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/).
- **Changes made:** the data was cleaned as described in this repository (duplicates removed, labels standardised, data types corrected, missing values recovered or removed, totals recalculated).
- **Cleaned dataset:** `cleaned_retail_store_dataset.csv` is an adaptation of the data above and is therefore shared under the same **CC BY-SA 4.0** license.

## Author

**Eseonu Rejoice Adaeze**
Data Analytics Intern, Oasis Infobyte

LinkedIn: www.linkedin.com/in/eseonu-rejoice-adaeze-999488413

## Acknowledgements

- Original dataset by **ahmedmohamed2003** and extended version by **Brian.C**, both on Kaggle.
- Project completed as part of the **Oasis Infobyte Internship (OIBSIP)**, Data Analytics track.

