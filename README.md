# Retail Performance Analysis Pipeline

**Week 5 | Python · Pandas · Matplotlib · Seaborn · Jupyter Notebook**

A production-grade Python data pipeline built to ingest, validate, and analyze retail transaction data — combining automated data quality gates and quarantine routing with a four-pillar executive analysis across $2.81M in revenue, surfacing insights on category profitability, omnichannel performance, and promotional effectiveness.

---

## Project Overview

| | |
|---|---|
| **Tools** | Python, Pandas, Matplotlib, Seaborn, Jupyter Notebook |
| **Domain** | Retail / Omnichannel E-Commerce |
| **Dataset** | 12,575 retail transactions (`retail_store_sales.csv`) |
| **Total Revenue Analyzed** | $2,810,794.13 |
| **Average Transaction Value** | $223.52 |

---

## Pipeline Architecture

```
raw_incoming/          ← Landing zone for raw CSV transaction files
clean_warehouse/       ← Validated records ready for analysis
quarantine_zone/       ← Isolated anomalous records for IT review
logs_and_reports/      ← Timestamped pipeline run summaries (.txt)
```

The pipeline never stalls — clean records flow to the warehouse while flagged records are isolated in parallel, allowing analysis to proceed without waiting for data remediation.

---

## Data Quality Gates

### Gate 1 — Missing Value Auditing
```python
missing_id_condition = df['Transaction ID'].isnull()
quarantine_rows.update(df[missing_id_condition].index)
print(f"-> Gate 1: Flagged {missing_id_count} records missing a 'Transaction ID'.")
```
Scans for missing values in critical identity columns. Captures exact index positions of flagged records so data teams can trace ingestion errors back to their source.

### Gate 2 — Type Safety & Format Enforcement
```python
numeric_prices = pd.to_numeric(df['Price Per Unit'], errors='coerce')
corrupt_price_condition = numeric_prices.isnull() & df['Price Per Unit'].notnull()
quarantine_rows.update(df[corrupt_price_condition].index)
print(f"-> Gate 2: Flagged {corrupt_price_count} records containing non-numeric pricing data.")
```
Forces corrupt alphabetical characters or hidden strings in price columns to register as errors using `pd.to_numeric(errors='coerce')` — preventing broken data types from reaching downstream BI models.

### Gate 3 — Business Logic & Boundary Validation
```python
numeric_qty = pd.to_numeric(df['Quantity'], errors='coerce')
negative_qty_condition = numeric_qty <= 0
quarantine_rows.update(df[negative_qty_condition].index)
print(f"-> Gate 3: Flagged {negative_qty_count} records violating inventory logic (Qty <= 0).")
```
Establishes operational boundaries based on real-world business rules. Isolates any transaction with zero or negative quantity — values that violate inventory logic and corrupt aggregation totals.

### Automated Routing
```python
clean_df = df.drop(index=quarantine_list)
quarantine_df = df.iloc[quarantine_list]

clean_df.to_csv('clean_warehouse/clean_store_sales.csv', index=False)
quarantine_df.to_csv('quarantine_zone/quarantined_sales_records.csv', index=False)
```

### Automated Audit Logging
```python
with open(report_path, 'w') as f:
    f.write(f"Total Transactions Ingested : {total_raw}\n")
    f.write(f"Clean Records Logged        : {clean_count}\n")
    f.write(f"Quarantined Records Isolated: {quarantine_count}\n")
    f.write(f"Pipeline Ingestion Success  : {success_rate:.2f}%\n")
```
Generates a timestamped `.txt` run log on every execution — recording total ingested, clean count, quarantine count, and pipeline status (SUCCESS / WARNING).

---

## The Four Analysis Pillars

### Pillar 1 — Executive KPIs
```python
total_revenue = df_clean['Total Spent'].sum()       # $2,810,794.13
total_orders  = df_clean['Transaction ID'].nunique() # 12,575
avg_order_value = df_clean['Total Spent'].mean()    # $223.52
```

| Metric | Value |
|---|---|
| Total Revenue | $2,810,794.13 |
| Total Transactions | 12,575 |
| Average Transaction Value (ATV) | $223.52 |

---

### Pillar 2 — Product Category Performance
```python
category_analysis = df_clean.groupby('Category').agg(
    Units_Sold=('Quantity', 'sum'),
    Total_Revenue=('Total Spent', 'sum')
).sort_values(by='Total_Revenue', ascending=False)
```

| Category | Units Sold | Revenue | Share |
|---|---|---|---|
| Butchers | 27,472 | $591,858.91 | 21.06% |
| Patisserie | 25,920 | $569,720.61 | 20.27% |
| Beverages | 25,835 | $565,496.06 | 20.12% |
| Milk Products | 24,736 | $543,003.54 | 19.32% |
| Produce | 24,379 | $540,715.01 | 19.24% |

> **Key Insight:** The revenue spread between the highest (Butchers) and lowest (Produce) category is only $51,143.90 — demonstrating healthy, diversified consumer demand with no dangerous single-category dependency.

---

### Pillar 3 — Channel Distribution
```python
channel_analysis = df_clean.groupby('Location').agg(
    Order_Count=('Transaction ID', 'count'),
    Revenue=('Total Spent', 'sum')
)
channel_analysis['Revenue_Contribution_%'] = (
    channel_analysis['Revenue'] / df_clean['Total Spent'].sum()
) * 100
```

| Channel | Orders | Revenue | Contribution |
|---|---|---|---|
| Online | 6,368 | $1,429,915.22 | 50.87% |
| In-Store | 6,207 | $1,380,878.91 | 49.13% |

> **Key Insight:** Near-perfect omnichannel equilibrium. The digital storefront yields +161 more orders and captures an extra $49K, but both channels are critically important — infrastructure and marketing investment must remain equally split.

---

### Pillar 4 — Promotional Impact Analysis
```python
discount_analysis = df_clean.groupby('Discount Applied')['Total Spent'].mean()
```

| Discount Status | Average Order Value |
|---|---|
| No Discount | $222.82 |
| Discount Applied | $224.22 |
| **Net Lift** | **+$1.40** |

> **Key Insight:** Current flat discounts fail to drive larger basket sizes. A $1.40 ATV lift means customers are buying the same volume regardless of promotion — the business is giving up margin on sales that would have happened at full price anyway.

---

## Recommendations

**1. Overhaul the Discount Strategy**
Pause flat, blanket discounts. Replace with threshold-based promotions — e.g., *"Spend $250 and get 10% off"* — to actively push ATV past the $223 baseline rather than rewarding existing spend.

**2. Cross-Promote Using the Butchers Anchor**
Leverage the high traffic of the top-performing Butchers department. Bundle high-margin Beverages or Produce items with top-selling meat products to boost lagging category profitability without new acquisition spend.

**3. Optimise Omnichannel Logistics**
Maintain the 50/50 capital split between channels, but implement Click & Collect (Buy Online, Pick Up In-Store) to bridge both. This exposes digital shoppers to in-store displays and reduces last-mile delivery costs.

---

## Files

| File | Description |
|---|---|
| `retail_performance_analysis.ipynb` | Full Jupyter Notebook — pipeline + analysis + visualisations |
| `retail_store_sales.csv` | Raw transaction dataset (if included) |

---

## Author

**Refilwe Molelu** — Business Intelligence Analyst

- Portfolio: [refilwe-molelu.netlify.app](https://refilwe-molelu.netlify.app)
- LinkedIn: [linkedin.com/in/refilwe-molelu-713379241](https://www.linkedin.com/in/refilwe-molelu-713379241)# Retail-Performance-Analysis-Pipeline
