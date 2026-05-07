# SHEIN Product Intelligence — Data Cleaning & Feature Engineering

**Dataset:** [SHEIN E-Commerce Product Data — Kaggle](https://www.kaggle.com/datasets/oleksiimartusiuk/e-commerce-data-shein/data)  
**Stack:** Python · Pandas · Matplotlib · VS Code · Jupyter (.ipynb)
**Scope:** Raw scraped product data → cleaned analytical dataset with engineered features

---

## Project Overview

This project takes a raw, multi-category SHEIN product dataset scraped from one of the world's largest fast-fashion e-commerce platforms and transforms it into a clean, analysis-ready dataset through systematic data cleaning, text parsing, and feature engineering. The primary deliverable is a validated dataset of ~70k products with engineered metrics suitable for product performance comparison, pricing tier analysis, and value efficiency ranking.

The data cleaning work here is the portfolio centrepiece — not dashboards or ML models. The raw dataset reflects real-world scraping artifacts: inconsistent text formats, mixed numeric and string fields, extreme price outliers, and duplicates. Resolving these issues cleanly and documenting each decision is the analytical skill being demonstrated.

---

## Dataset Structure

### Raw Data

The dataset consists of **21 CSV files across different SHEIN product categories**, each with slight schema variations due to scraping inconsistencies.

Typical raw file structure includes:

- Product title (sometimes duplicated across fields)
- Product URL
- Category ranking fields
- Price (string format with currency symbols)
- Discount (in inconsistent formats)
- Selling proposition (text-based sales indicator)

Each file contains ~3,000–4,000 records, with occasional missing or null-heavy fields depending on category.

### Key Data Quality Issues in Raw Data

- Inconsistent column naming across files
- Duplicate product title fields
- Mixed numeric and string formats in price
- Non-standard discount formatting (e.g. "-28%", "28% off")
- Sparse and partially missing selling data
- Embedded structured data inside text fields
---

## Data Cleaning Steps

**1. Deduplication**  
Identified and removed ~2,876 duplicate product entries. Duplicates arose from category overlap in the original scrape — the same product appearing in multiple category exports.

**2. Price field standardisation**  
The `price` column was stored as mixed-format strings (e.g. `"$12.99"`, `"12,99"`, `"12.99 USD"`). Cleaned to a consistent `float64` dtype by stripping currency symbols, normalising decimal separators, and coercing unparseable values to null before dropping.

**3. Discount field standardisation**  
The `discount` column contained string values like `"30% off"` and `"-30%"`. Stripped to a consistent numeric percentage as `float64`.

**4. Null handling**  
Missing values in `price`, `discount`, and `selling_proposition` were assessed. Rows with null `price` were dropped (cannot be used in any downstream pricing analysis). Null discount treated as zero-discount rather than dropped, as absence of a discount tag is itself meaningful.

**5. Outlier removal — price**  
The raw price distribution was heavily right-skewed with extreme outliers (likely data entry errors or mislabelled currency). Prices above the 99th percentile (~$123.99) were capped. Validated using histogram comparison before and after, and manual inspection of the top 1% of values.

---

## Feature Engineering

Five new columns were created to enrich the analytical layer beyond the raw scraped fields:

**`units_sold`**  
Extracted from the `selling_proposition` text field which contained strings like `"10k+ sold recently"`, `"500+ sold"`, `"5.3k sold"`. Parsed using regex to extract the numeric component and convert abbreviated formats (`k` multiplier) to integer values.

**`units_sold_log`**  
Log transformation (`log(units_sold + 1)`) applied to correct for the strong right skew in the raw sales distribution. Used in downstream comparative analysis and as input to the value score calculation.

**`price_category`**  
Grouped products into pricing tiers based on the cleaned price distribution: Budget (bottom quartile), Mid-range (interquartile range), and Premium (top quartile). Allows segment-level analysis without continuous price as the grouping variable.

**`has_discount`**  
Binary flag: 1 if the product carries any non-zero discount, 0 otherwise. Simple indicator useful for comparing discounted vs full-price product performance.

**`value_score`**  
Custom engineered metric:
value_score = log(units_sold) / (price + 1)
Measures how many log-units of sales volume a product delivers per dollar of price — a proxy for sales efficiency relative to price point. Higher value_score indicates a product that sells well despite (or because of) low price. **Important:** this is a comparative analytical metric, not a revenue or profit measure. It is intended for ranking products within a category, not for estimating actual retail value.

---

## Outlier Handling

Price outliers were identified using quantile analysis:

- 95th percentile: ~$89.99
- 99th percentile: ~$123.99
- Raw max: values in the thousands (likely scraping or currency errors)

Values above the 99th percentile were capped rather than dropped, preserving row count while eliminating distortion in distribution visualisations and aggregate statistics. Validated using before/after histogram comparison saved to `/visuals/`.

---

### Cleaned Data

The final dataset contains **10 engineered analytical features**, designed for downstream analysis:

| Column | Description |
|--------|-------------|
| price | Cleaned numeric price (float) |
| discount | Standardised discount percentage |
| selling_proposition | Original sales text field |
| source_file | Origin CSV file identifier |
| color-count | Extracted categorical/count-based feature |
| units_sold | Parsed numeric estimate of product sales |
| units_sold_log | Log-transformed sales to reduce skew |
| price_category | Budget / Mid-range / Premium segmentation |
| has_discount | Binary discount indicator |
| value_score | Custom efficiency metric (sales vs price) |

The cleaned dataset expands beyond the raw schema because it transforms unstructured e-commerce text fields into structured analytical variables through feature engineering.

## Key Findings

- Price distribution was strongly right-skewed before cleaning; after capping, median price settled at approximately $15–18, consistent with SHEIN's mass-market positioning
- Units sold distribution required log transformation — raw values were dominated by a small number of viral products with 100k+ sales dwarfing the majority of the catalogue
- Low-priced products dominate the top of the value_score ranking, confirming that SHEIN's commercial strength is in high-volume, low-margin products rather than premium pricing
- Approximately 60–65% of products carry a discount flag, suggesting discount is the norm rather than a promotional exception across the catalogue

---

## Honesty Note

What this project demonstrates:

✅ Systematic data cleaning on a real scraped dataset  
✅ Text parsing to extract structured values from free-form strings  
✅ Log transformation for skew correction  
✅ Custom metric engineering for comparative product analysis  
✅ Outlier handling with documented rationale  

What this project does not claim:

❌ Predicted real revenue or profit  
❌ Preserved raw transaction-level ground truth in engineered metrics  
❌ Validated value_score against actual SHEIN business performance data  

The `value_score` metric is an analytical construct for comparative ranking within this dataset. It is not a financial metric and should not be interpreted as one.

---

## Visualisations

**Price Distribution — Before Cleaning**  
![Price Before](visuals/price_before.png)

**Price Distribution — After Cleaning**  
![Price After](visuals/price_after.png)

**Units Sold Distribution**  
![Units Sold](visuals/units_sold.png)

**Value Score Distribution**  
![Value Score](visuals/value_score.png)


## Repository Structure

```text
shein-product-cleaning/
├── README.md
├── data/
│   ├── raw/
│   │   └── https://www.kaggle.com/datasets/oleksiimartusiuk/e-commerce-data-shein/data
│   │       (CSVs not included)
│   └── cleaned/
│       └── shein_clean.csv    ← 70,292 rows × 19 columns
├── notebooks/
│   └── 01_shein_cleaning.ipynb
├── visuals/
│   ├── price_before.png
│   ├── price_after.png
│   ├── units_sold.png
│   └── value_score.png
└── .gitignore
```

## Tools

| Tool | Purpose |
|---|---|
| Python + Pandas | Data loading, cleaning, feature engineering |
| Matplotlib | Distribution visualisations |
| Jupyter Notebook | Exploratory analysis and documentation |

---

## Data Source

SHEIN E-Commerce Product Dataset  
[kaggle.com/datasets/oleksiimartusiuk/e-commerce-data-shein](https://www.kaggle.com/datasets/oleksiimartusiuk/e-commerce-data-shein/data)


## Potential Use Cases

- Price segmentation analysis across categories
- Discount strategy evaluation
- Product demand proxy modeling
- Identifying high value-for-money products

- 
