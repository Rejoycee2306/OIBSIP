# Data Cleaning: Messy E-Commerce Sales Data

**Oasis Infobyte Internship | Data Analytics Track | Level 1, Task 3: Cleaning Data**

**Author:** Eseonu Rejoice Adaeze

**Date:** November 2026  

**Tech Stack:** Python, Pandas, NumPy, Jupyter Notebook  

## 1. Project Overview

Real-world data is rarely analysis-ready. This project takes a deliberately messy e-commerce sales dataset (103 order records) and turns it into a clean, consistent dataset of 100 unique orders, with every cleaning decision documented and justified.

**Issues found in the raw data:** inconsistent text labels, mixed date formats, corrupted numeric entries, negative values, missing values, wrong category assignments, duplicate orders, and totals that did not match quantity × price.

## 2. Dataset

- **Source:** [Kaggle: Messy E-Commerce Sales Data (kandeeldev)](https://www.kaggle.com/datasets/kandeeldev/messy-e-commerce-sales-data)
- **Raw size:** 103 rows × 11 columns
- **Cleaned size:** 100 rows × 11 columns
- **Order dates (after cleaning):** 5 Jan 2023 to 6 Nov 2025
- **Currency:** not stated in the source, so values are plain numbers

| Column Name | Data Type (Clean) | Description |
| :--- | :--- | :--- |
| `ID` | `str` | Unique identifier (`100` to `199`) |
| `Customer_Name` | `str` | Name of the customer (`Customer_100` – `Customer_199`) |
| `Order_ID` | `str` | Order ID for the transaction |
| `Order_Date` | `datetime64[us]` |  Date of the order |
| `Product` | `str` ) | Product sold |
| `Category` | `str`  | Product category (`Books`, `Clothing`, `Electronics`, `Home`, `Sports`) |
| `Quantity` | `Int64` | Number of units purchased per order |
| `Price` | `float64` | Unit price of the item sold |
| `Payment_Method` | `str` | Payment channel (`Cash on Delivery`, `PayPal`, `Bank Transfer`, `Credit Card`) |
| `Status` | `str` | Status of the transaction (`Shipped`, `Processing`, `Delivered`, `Cancelled`, `Returned`) |
| `Calculated_Total` | `Float64` | Recorded sales value (replaced by `Calculated_Total` after cleaning) |

## 3. Tools Used

Python · pandas · NumPy · Jupyter Notebook

## 4. Data Quality Report (Before Cleaning)

| Issue | Finding |
|---|---|
| Column headers | Leading spaces in `Customer_Name` and `Category` |
| Wrong data types | `Quantity`, `Price` and `Order_Date` stored as text |
| Missing values | Category (8), Quantity (5), Price (5), Total (14) |
| Negative values | Quantity (2), Price (1), Total (3) |
| Corrupted text | Price: `abd` (×2), `four hundred`, `300$`; Quantity: `4a` |
| Inconsistent labels | Category variants such as `electronic`, `ELECTRONICS`, `sports`; product `shoes` vs `Shoes` |
| Mixed/invalid dates | Mostly month/day/year, one `Jan 5 2023`, one invalid `abc` |
| Wrong categories | 9 rows where the category did not match the product |
| Inconsistent totals | 5 rows where Total ≠ Quantity × Price (always exactly 70%) |
| Duplicates | 1 exact duplicate row; 3 Order_IDs appearing twice |
| Outlier | A Blender priced at 10,000 (typical price about 350) |

## 5. Cleaning Steps and Decisions

### 5.1 Headers and text standardisation
- Stripped whitespace from all column names.
- Fixed `shoes` → `Shoes` (21 → 20 distinct products).
- Instead of only fixing spellings, **Category was derived from Product** using a fixed Product-to-Category mapping. This fixed spelling errors, missing categories (8) and wrong categories (9) in one step.

| Category | Products |
|---|---|
| Books | Science, Biography, Comics, Fiction |
| Clothing | Shoes, T-shirt, Jacket, Jeans |
| Home | Blender, Microwave, Lamp, Vacuum |
| Sports | Tennis Racket, Basketball, Yoga Mat, Football |
| Electronics | Smartphone, Smartwatch, Laptop, Headphones |

### 5.2 Dates
- Converted with `pd.to_datetime(errors='coerce', format='mixed')`, reading month first (day values reach 31, month values never exceed 12).
- The invalid entry `abc` (ID 192) became a missing date (NaT).

### 5.3 Numeric columns
- `300$` → 300 and `four hundred` → 400 (intended value is clear).
- `abd` (Price) and `4a` (Quantity) → NaN (intended value cannot be verified).
- Quantity cast to nullable integer (`Int64`); ID cast to text.

### 5.4 Negative values
- Negative quantities and the negative price were set to NaN, **not** converted with `abs()`, because a typing error cannot be told apart from a return or refund.

### 5.5 Missing values
- **Category:** filled via the product mapping.
- **Quantity and Price:** left as NaN, **not** imputed. Mean or median imputation would invent sales figures, and the recorded Total could not recover any of them.
- Records with missing values were kept and are excluded only from calculations that need them.

### 5.6 Total sales value
- Created `Calculated_Total = Quantity × Price` and dropped the original `Total`.
- All 5 mismatched totals were exactly 70% of Quantity × Price (a possible 30% discount), but with no discount column this could not be confirmed. `Calculated_Total` is therefore a **gross value**.

### 5.7 Duplicates
- Removed repeated Order_IDs with `drop_duplicates(subset=['Order_ID'], keep='first')`: 103 → 100 rows.

### 5.8 Outliers (IQR method)

| Column | Lower bound | Upper bound | Outliers found |
|---|---|---|---|
| Quantity | -2.50 | 9.50 | 0 |
| Price | -412.04 | 1,427.93 | 1 (ID 117: 10,000) |
| Calculated_Total | -1,488.33 | 4,338.44 | 2 (ID 149: 4,481.90; ID 194: 4,722.70) |

- **Blender price 10,000:** treated as a data-entry error (about 30× the typical price) and replaced with the median of the remaining Blender prices (**336.02**). The median was chosen because it is not pulled by extreme values (with the outlier, mean 1,603 vs median 354).
- **High totals (4,481.90 and 4,722.70):** retained, as they come from valid 5-unit orders.

## 6. Before vs After Summary

| Aspect | Before | After |
|---|---|---|
| Rows × columns | 103 × 11 | 100 × 11 |
| Column headers | Leading spaces in 2 | All trimmed |
| Distinct categories | 9 inconsistent labels + 8 missing | 5 consistent, 0 missing |
| Wrong category assignments | 9 | 0 |
| Distinct products | 21 | 20 |
| Order_Date | Text, mixed formats, 1 invalid | Datetime (YYYY-MM-DD), 1 missing |
| Quantity | Text; negatives and `4a` | Numeric; 8 missing |
| Price | Text; `$`, words, `abd`, negative, outlier | Numeric; 8 missing; outlier fixed |
| Total | 14 missing, 3 negative, 5 inconsistent | `Calculated_Total`: 0 negative, 0 inconsistent, 15 missing |
| Duplicates | 1 exact + 3 repeated Order_IDs | 0 |
| Complete records | n/a | 85 of 100 |

Null counts in some columns rose (for example Quantity 5 → 8) because invalid entries were deliberately converted to missing values rather than guessed.

## 7. Validation

- Every product maps to exactly one category (Home 25, Books 23, Sports 21, Clothing 17, Electronics 14).
- `Calculated_Total` equals Quantity × Price in every row (checked with `np.isclose`).
- No negative, zero or non-numeric values remain in Quantity or Price.
- The saved CSV was re-run against the raw file and matches the notebook output exactly.

## 8. Limitations

- `Calculated_Total` is a gross value and may overstate revenue for the 5 orders whose original Total reflected a 30% reduction.
- 15 of 100 records have at least one missing value and must be excluded or handled in analyses that depend on those fields.
- The Price for ID 117 (336.02) is an estimate; its Quantity is missing, so no revenue figure is affected.
- Currency is not stated in the source dataset.

## 9. Repository Structure

```
DataAnalytics-L1-CleaningData/
├── README.md
├── DataCleaning_Ecommerce.ipynb
├── cleaned_ecommerce_dataset.csv
├── Data_cleaning_documentation.pdf
├── messy_ecommerce_sales_data.csv
```
