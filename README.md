
# Sales Data Analysis & Power BI Dashboard

A data analytics and business intelligence portfolio project demonstrating data preparation, validation, exploratory analysis, KPI development and interactive reporting using **Python, Pandas, PySpark, SQL, Databricks and Microsoft Power BI**.

The repository brings together three analytical components:

1. **Power BI Sales Performance Dashboard:** A two-page interactive report analysing Superstore sales performance from 2015 to 2018.
2. **Databricks Sales Analytics:** A PySpark and SQL workflow for validating, transforming and analysing Superstore retail data, with preparation of a Delta table for downstream reporting.
3. **Python Sales Data Analysis:** Data cleaning, aggregation and exploratory analysis of a separate historical electronics sales dataset.

**Dataset distinction:** The Databricks notebook and Power BI dashboard work with Superstore sales data, although their specific source versions and results should be verified before direct comparison. The original Python sales analysis uses a different electronics sales dataset. Results from the electronics dataset should not be combined with Superstore results.

---

## 1. Power BI Sales Performance Dashboard

The Power BI report provides an executive-level view of sales performance and a detailed breakdown of customer segments, product categories, geographical regions and monthly trends.

The dashboard was developed using **Microsoft Power BI Service**, with interactive slicers, KPI cards, bar charts, a line chart, a doughnut chart and a matrix visual.

### Dashboard Preview

#### Executive Sales Summary

![Executive Sales Summary](screenshots/01-executive-sales-summary.png)

#### Market Performance Overview

![Market Performance Overview](screenshots/02-market-performance.png)

### Executive KPIs

| KPI | Result |
|---|---:|
| Total Sales | €2,261,536.78 |
| Total Orders | 4,922 |
| Average Sales per Order | €459.48 |
| Largest Customer Segment | Consumer |
| Largest Sales Region | West |
| Largest Product Category | Technology |

*The currency and KPI values above reflect the Power BI report as displayed.*

### Page 1: Executive Sales Summary

The executive page provides a high-level view of commercial performance through KPI cards, regional comparisons, customer segmentation and annual sales trends.

**Key findings**

- Consumer customers generated approximately €1.15 million in sales, the largest contribution among customer segments.
- The West region generated approximately €0.71 million, followed by the East at €0.67 million.
- Sales increased between 2016 and 2018, with 2018 recording the highest annual sales in the report.

### Page 2: Market Performance Overview

The market performance page enables more detailed exploration through year, segment and shipping-mode filters, category and subcategory comparisons, a sales matrix and monthly trend visualisations.

**Key findings**

- Technology generated €827,455.87, representing 36.59% of total sales.
- Furniture generated €728,658.58, representing 32.22%.
- Office Supplies generated €705,422.33, representing 31.19%.
- Phones and Chairs were the leading subcategories by sales.
- November recorded the highest aggregated month-of-year sales, followed by December and September.

### Business Questions

1. How did annual sales change between 2015 and 2018?
2. Which customer segments contributed the most sales?
3. Which geographical regions generated the highest sales?
4. Which product categories and subcategories were the strongest performers?
5. How did sales vary throughout the calendar year?
6. How did monthly trends differ across customer segments?

### Analytical Interpretation

The dashboard reveals differences in customer segment contributions, regional sales distribution and product category performance.

The findings support further investigation into geographical demand, product mix and seasonal sales patterns. However, the descriptive analysis does not establish why these differences occurred or demonstrate changes in profitability.

### Power BI Data Model

![Power BI Semantic Model](screenshots/03-semantic-model.png)

The Power BI model contains the `Superstore_BI_Notebook` table, including order information, sales values, product categories, customer segments, geographical attributes and shipping information.

The model also includes date-related calculated fields and an average-sales-per-order measure.

### Dashboard Demonstration

A recorded walkthrough of the completed two-page Power BI Sales Performance Dashboard (2015–2018), demonstrating interactive filtering, dashboard navigation, executive KPIs, sales trends, customer segmentation and product performance.

[![Watch the Power BI Dashboard Demonstration](screenshots/02-market-performance.png)](https://youtu.be/V4LyME-_Wus)

**[Watch the full dashboard demonstration on YouTube](https://youtu.be/V4LyME-_Wus)**

---

## 2. Databricks Sales Analytics — PySpark & SQL

### Overview

This component demonstrates a structured sales analytics workflow using **Databricks, Apache Spark (PySpark), Spark SQL and Delta Lake**.

The notebook analyses a US retail Superstore dataset containing **9,800 sales line items**.

Its objective is to prepare reliable analytical data, validate source assumptions, calculate business KPIs and examine sales performance across time, geography, customer segments, product categories and shipping modes.

The workflow places particular emphasis on **data quality, metric accuracy and reproducibility**.

**Notebook:** [Superstore_BI_Notebook_Improved.py](notebooks/Superstore_BI_Notebook_Improved.py)

### 2.1 Source Data and Schema Validation

The workflow begins by loading the registered Databricks source table:

```python
sales_raw = spark.table("train")
```

The notebook defines 18 required source columns and validates that they are present before continuing.

These include:

- Order and shipping dates
- Order and customer identifiers
- Customer segments
- Product categories and subcategories
- Geographical attributes
- Shipping modes
- Sales values

The schema is inspected using `printSchema()`.

**Purpose:** Detect missing required fields before executing downstream transformations and analytical queries.

### 2.2 Data Preparation and Transformation

PySpark transformations prepare the dataset for analysis.

The workflow includes:

- Removing identifier fields not required in the presentation dataset.
- Converting order and shipping dates into date types.
- Casting sales values to a numeric type.
- Deriving order year and year-month attributes.
- Retaining the original number of sales line items.

Example:

```python
sales_clean = (
    sales_raw
    .drop("Row ID", "Product ID", "Postal Code", "Customer ID")
    .withColumn("Order Date", F.to_date(F.col("Order Date")))
    .withColumn("Ship Date", F.to_date(F.col("Ship Date")))
    .withColumn("Sales", F.col("Sales").cast(DoubleType()))
    .withColumn("Order Year", F.year(F.col("Order Date")))
    .withColumn("Order Month", F.date_format(F.col("Order Date"), "yyyy-MM"))
)
```

The workflow checks that the transformed dataset contains the same number of rows as the source.

This protects against unintended record loss during preparation.

### 2.3 Data Quality and Validation

The notebook implements several data-quality checks before analytical reporting.

| Validation | Purpose |
|---|---|
| Required-column validation | Detect missing source fields |
| Row-count reconciliation | Confirm that preparation preserves records |
| Null-value inspection | Identify missing data |
| Exact-row duplicate detection | Flag potential duplicate records for review |
| Non-positive sales checks | Identify unusual or invalid sales values |
| Shipping-date validation | Detect shipping dates earlier than order dates |
| Distinct-order counting | Separate orders from product line items |

An important modelling distinction is that **one order may contain multiple product line items**.

Therefore, repeated order IDs do not automatically represent duplicate records.

Exact-row duplicates are reported for investigation rather than removed without review.

The notebook also calculates source-level summaries including distinct orders, distinct customer names, first and last order dates, and total recorded sales.

### 2.4 KPI Development Using Spark SQL

After preparation, the notebook creates a temporary SQL view:

```python
sales_clean.createOrReplaceTempView("sales_data")
```

Spark SQL is then used to calculate business performance metrics.

#### Headline KPIs

```sql
SELECT
    COUNT(*) AS line_items,
    COUNT(DISTINCT `Order ID`) AS orders,
    ROUND(SUM(Sales), 2) AS total_sales,
    ROUND(
        SUM(Sales) / COUNT(DISTINCT `Order ID`),
        2
    ) AS sales_per_order
FROM sales_data
```

The query distinguishes:

- **Line items:** Individual product records.
- **Orders:** Unique order identifiers.
- **Total sales:** Sum of recorded sales values.
- **Sales per order:** Total sales divided by distinct orders.

This distinction prevents overstating the number of customer orders when an order contains multiple products.

### 2.5 Business Performance Analysis

The notebook contains SQL queries addressing the following business questions.

| Analysis | Business Question |
|---|---|
| Regional sales | Which geographical regions contribute the most sales? |
| Category performance | Which product categories and subcategories generate the most sales? |
| Monthly sales | How does sales performance change over time? |
| Customer analysis | Which customers account for the highest recorded sales? |
| State-level performance | Which states contribute the most sales? |
| Shipping modes | How are distinct orders distributed across shipping methods? |
| Average order value | How does average order value differ across customer segments? |
| Segment trends | How do monthly sales patterns vary by customer segment? |

#### Chronological Monthly Analysis

The notebook uses year-month values rather than grouping all Januaries, Februaries or other calendar months together.

This preserves the chronological sales trend across multiple years.

#### Correct Average Order Value

Average order value is calculated by first aggregating product line items into individual order totals.

The notebook then calculates the average of those order totals by customer segment.

This avoids confusing average line-item sales with average order value.

### 2.6 Preparing Data for Power BI

The notebook includes a workflow for publishing a prepared analytical dataset as a versioned Delta table.

The preparation selects and standardises reporting fields such as:

- `order_id`
- `order_date`
- `ship_date`
- `ship_mode`
- `segment`
- `region`
- `category`
- `sub_category`
- `sales`
- `order_year`
- `order_month`

The intended output table is:

```text
superstore_sales_clean_v2
```

The workflow uses Delta format and includes a safeguard to prevent overwriting an existing table of the same name.

After publication, the code is designed to validate:

1. The number of published rows against the prepared dataset.
2. The number of distinct orders against the source summary.
3. The total recorded sales against the source summary.

These checks are intended to confirm consistency between the prepared and published datasets.

**Integration note:** The notebook defines a Delta-table publishing workflow suitable for downstream Power BI reporting. The repository does not currently establish that the displayed Power BI report was connected directly to this Delta table or refreshed automatically from Databricks.

### 2.7 Analytical Limitations

The available sales dataset supports descriptive analysis of recorded sales, orders, customers, products, shipping methods and time patterns.

It does not contain sufficient information to establish:

- Profitability or profit margins
- Marketing return on investment
- Advertising conversion rates
- Inventory performance
- Product return rates
- Causal business impact

Accordingly, findings are presented as descriptive analytical results rather than verified commercial improvements.

### Databricks Skills Demonstrated

- Apache Spark and PySpark DataFrames
- Spark SQL queries and aggregations
- Source schema validation
- Data-quality checks
- Data-type conversion and transformation
- Row-count and KPI reconciliation
- Distinct-order and line-item modelling
- Date-based analytical preparation
- Delta Lake table publication logic
- Data preparation for business intelligence reporting

---

## 3. Python Sales Data Analysis

The original analysis was developed using Python and Jupyter Notebook to investigate historical electronics sales data.

This analysis uses a separate dataset from the Superstore Power BI and Databricks components.

### Data Preparation

The notebook demonstrates:

- Combining monthly CSV files into a consolidated dataset.
- Removing missing values and repeated header rows.
- Converting quantity and price columns to numeric types.
- Converting order dates into datetime values.
- Deriving month, weekday and city attributes.
- Preparing sales values for aggregation and visualisation.

### Exploratory Analysis

The notebook investigates several business questions:

1. Which months generated the highest and lowest sales?
2. How did sales vary across months?
3. Which cities recorded the most purchases?
4. Which products were purchased most frequently?
5. How did purchases vary by weekday?
6. Which products appeared together in customer orders?

### Tools and Methods

- Python
- Pandas
- Matplotlib
- Seaborn
- Jupyter Notebook
- Data cleaning and type conversion
- Grouping and aggregation
- Exploratory data visualisation

### Python Analysis Limitations

The notebook represents exploratory work rather than a production analytics pipeline. Its analysis should be interpreted within the context of the underlying dataset.

Potential extensions include automated data validation, additional product-level analysis and formal evaluation of forecasting or recommendation approaches.

---

## 4. Technologies and Skills

| Area | Technologies / Methods |
|---|---|
| Programming | Python, SQL |
| Distributed data processing | Apache Spark, PySpark |
| Data analytics platform | Databricks |
| Data preparation | Pandas, PySpark, CSV, Excel |
| Data validation | Schema checks, null analysis, duplicate detection, reconciliation |
| Exploratory analysis | Jupyter Notebook, Matplotlib, Seaborn |
| Business intelligence | Microsoft Power BI Service |
| Data modelling | Power BI semantic model, analytical tables |
| Calculations | DAX measures, calculated fields, Spark SQL |
| Data storage | Delta Lake |
| Reporting | KPI design, interactive filtering, trend analysis |
| Documentation | GitHub, Markdown |

---

## 5. Repository Structure

```text
sales_data_analysis/
├── Sales_Data/
├── notebooks/
│   └── Superstore_BI_Notebook_Improved.py
├── screenshots/
│   ├── 01-executive-sales-summary.png
│   ├── 02-market-performance.png
│   └── 03-semantic-model.png
├── sales_data_analysis.ipynb
├── combined_sales_data.csv
├── cleaned_df.xlsx
├── README.md
└── LICENSE
```

The `notebooks/` directory contains the Databricks notebook source export. The original Jupyter notebook and electronics sales files remain in the repository root.

---

## 6. Reproducing the Analysis

### 6.1 Python Sales Analysis

1. Download or clone the repository.
2. Install the required Python libraries: `pandas`, `matplotlib` and `seaborn`.
3. Open `sales_data_analysis.ipynb` in Jupyter Notebook.
4. Update any local file paths to match the downloaded repository.
5. Run the notebook cells in sequence.

### 6.2 Databricks Sales Analytics

1. Create or access a Databricks workspace with Spark compute available.
2. Import `notebooks/Superstore_BI_Notebook_Improved.py` as a Databricks notebook.
3. Register the corresponding Superstore dataset as a table named `train`, or update the notebook's source-table reference.
4. Confirm that the source contains the expected fields and data types.
5. Run the notebook cells in sequence.
6. Review schema checks, data-quality results and KPI outputs.
7. If publishing the Delta table, confirm the target catalogue, schema, permissions and output-table name before running the publication step.
8. Review the final row-count, order-count and sales-total reconciliation results.

The notebook uses Databricks-specific functionality, including `spark`, `%sql` and `display()`. It is not intended to run unchanged as a standalone local Python script.

The original source dataset and an active Databricks environment are required to reproduce the workflow.

### 6.3 Power BI Report

The repository includes screenshots of the Power BI dashboard and semantic model, together with a recorded dashboard demonstration.

The original editable Power BI report and full DAX definitions are not currently distributed in this repository.

The screenshots and demonstration video provide evidence of the completed dashboard.

The Databricks notebook documents a related Superstore analytical workflow, but a reproducible end-to-end Databricks-to-Power BI integration is not currently included.

---

## 7. Overall Conclusion

This portfolio project demonstrates three complementary approaches to sales analytics:

**Power BI:** Executive KPI reporting, customer segmentation, regional comparisons, product performance analysis and interactive business intelligence dashboards.

**Databricks:** Structured data validation, PySpark transformations, SQL-based KPI calculations, analytical modelling and preparation of a versioned Delta table for downstream reporting.

**Python:** Data preparation, exploratory analysis, aggregation and visualisation of a separate historical electronics sales dataset.

The Databricks component strengthens the project's focus on data quality and reliable KPI definitions, while the Power BI dashboard demonstrates how sales performance can be communicated through interactive visual reporting.

Together, these components provide practical evidence of working with analytical datasets, validating source information, defining business metrics, identifying patterns and presenting findings to support further business investigation.

---

**Project type:** Personal data analytics and business intelligence portfolio project.

**Primary technologies:** Python · SQL · PySpark · Databricks · Power BI · Pandas · DAX
