# 🧸 Maven Toys — Sales Performance Analysis

> A end-to-end data analysis project covering data cleaning, SQL analysis, and dashboard reporting.

![Project Banner](https://img.shields.io/badge/Status-Completed-brightgreen) ![Tools](https://img.shields.io/badge/Tools-MySQL%20%7C%20DBeaver%20%7C%20Power%20BI-blue) ![Type](https://img.shields.io/badge/Type-Portfolio%20Project-orange)

---

## 📌 Project Overview

This project analyzes **21 months of sales data (January 2022 – September 2023)** from Maven Toys, a fictional toy retail chain operating **50 stores** across 4 location types and selling products across **5 categories**.

The goal is to uncover actionable business insights around revenue, profitability, product performance, and store efficiency — and deliver them through a clean dashboard and presentation report.

---

## 🎯 Business Questions

1. Which product categories generate the most revenue and profit?
2. Which individual products are the top and bottom performers?
3. Which store locations are the most efficient?
4. What does the revenue and profit trend look like over time?
5. Which products should be discontinued or restocked?

---

## 🗂️ Dataset

| Table | Description |
|---|---|
| `sales` | Transaction-level sales records |
| `products` | Product details including price and cost |
| `stores` | Store information and location type |
| `inventory` | Stock on hand per product per store |
| `calendar` | Date dimension table |

> **Note:** Raw data contained formatting issues — `Product_Price` and `Product_Cost` were stored as text with `$` prefixes (e.g. `"$15.99"`), requiring cleaning before analysis.

---

## 🛠️ Tools & Tech Stack

| Tool | Purpose |
|---|---|
| **MySQL (XAMPP)** | Database hosting and querying |
| **DBeaver** | SQL editor and database management |
| **Power BI Desktop** | Interactive dashboard |
| **PowerPoint** | Final presentation report |

---

## ⚙️ Workflow

### Step 1 — Data Cleaning (SQL)

The raw data had `Product_Price` and `Product_Cost` stored as TEXT with `$` symbols, causing revenue and profit calculations to return zero.

**Fix:**
```sql
-- Remove $ symbol and clean whitespace
UPDATE products
SET 
    Product_Price = CAST(REPLACE(TRIM(Product_Price), '$', '') AS DECIMAL(10,2)),
    Product_Cost  = CAST(REPLACE(TRIM(Product_Cost),  '$', '') AS DECIMAL(10,2));

-- Change column type to numeric
ALTER TABLE products 
    MODIFY COLUMN Product_Price DECIMAL(10,2),
    MODIFY COLUMN Product_Cost  DECIMAL(10,2);
```

---

### Step 2 — Building the Master Table (SQL)

All analysis tables were derived from a central `master_table` joining sales, products, and stores.

```sql
CREATE TABLE master_table AS
SELECT
    s.Sale_ID,
    s.Date,
    s.Store_ID,
    st.Store_Location,
    s.Product_ID,
    p.Product_Name,
    p.Product_Category,
    s.Units,
    s.Units * p.Product_Price AS Revenue,
    s.Units * (p.Product_Price - p.Product_Cost) AS Profit
FROM sales s
JOIN products p ON s.Product_ID = p.Product_ID
JOIN stores st ON s.Store_ID = st.Store_ID;
```

---

### Step 3 — Analysis Tables (SQL)

The following tables were created for reporting:

| Table | Description |
|---|---|
| `master_table` | Core fact table with revenue & profit per transaction |
| `most_profitable_category` | Total revenue per product category |
| `total_profit` | Total gross profit per product category |
| `profit_margin_category` | Profit margin % per product category |
| `best_performing_store` | Total revenue per store location type |
| `distribution_units` | Units sold per product with % of total |
| `monthly_revenue_trend` | Revenue and profit aggregated by month |
| `revenue_store_category` | Revenue and profit broken down by store × category |
| `least_profitable_category` | Bottom performers by units sold |

---

### Step 4 — Dashboard (Power BI)

Built in Power BI — connected directly from MySQL via native connector.
 
![Dashboard Preview](Maven%20Toys%20Assets%201.png)
![Dashboard Preview](Maven%20Toys%20Assets%202.png)

**Visuals included:**
- **Page 1 — Overview:** KPI cards, monthly revenue & profit trend line chart, revenue share donut chart, total revenue by store, revenue & profit by product category
- **Page 2 — Products:** Top 10 / Bottom 10 most sold products, units sold by category table, total store by category


---

### Step 5 — Report (PowerPoint)

A presentation-ready report was built summarizing findings and recommendations, structured as:

1. Cover
2. Executive Summary
3. Revenue & Profit Trends
4. Category Performance
5. Product Deep Dive
6. Store Performance
7. Recommendations

---

## 📊 Key Findings

- **Electronics** has the highest profit margin at **44.57%** — nearly double the next category — despite only having 3 SKUs.
- **Art & Crafts** drives the highest unit volume (**29.87% of all units sold**), with 3 products in the Top 10 Most Sold.
- **Toys** generates the highest absolute gross profit (**$1.08M**) on strong volume, though at the thinnest margin (21.2%).
- **Downtown stores** account for **56.9% of total revenue** across 29 of 50 locations.
- **Airport stores** (only 3 locations) generate **$430K revenue per store** — the highest efficiency across all location types.

---

## 💡 Recommendations

| Category | Action | Reasoning |
|---|---|---|
| **Toys** | Increase stock for Lego Bricks, Action Figure, Animal Figures | Top 3 in units sold within category |
| **Art & Crafts** | Increase stock for PlayDoh Can, Barrel O' Slime, Magic Sand | 3 products in Top 10 overall |
| **Art & Crafts** | Discontinue Playfoam | Only 4,158 units in 21 months (~4.0 units/store/month) |
| **Electronics** | Expand SKU variety, especially wireless audio | Highest margin (44.57%), Colorbuds is #1 bestseller overall |
| **Games** | Hold & monitor, restock only when depleted | Strong timeless titles, healthy margin (30.27%) |
| **Sports & Outdoors** | Discontinue Mini Basketball Hoop | Only 2,647 units in 21 months (~2.5 units/store/month) |
| **Store Expansion** | Prioritize Airport locations | Highest revenue per store at $430K avg |

---

## 🛠️ Tools Used
 
| Tool | Purpose |
|---|---|
| **Excel** | Initial data inspection, boolean cleaning, date formatting |
| **MySQL + DBeaver** | Data import, type fixing, exploratory analysis |
| **Power BI** | Dashboard and visualization |
 
---

📁 Repository Structure

maven-toys-analysis/
│
├── sql/
│   ├── 01_data_cleaning.sql
│   ├── 02_master_table.sql
│   ├── 03_analysis_tables.sql
│   └── 04_monthly_trend.sql
│
├── report/
│   └── MavenToys_Analysis.pptx
│
└── README.md

---
 
## 🔗 About This Portfolio
 
This is **Project #1** of my Data Analyst portfolio, focusing on:
- Multi-table SQL analysis with JOINs and window functions
- Data cleaning across Excel and SQL
- Translating data findings into business recommendations
- Dashboard storytelling in Power BI
---
 
*Dataset: Toy Store E-Commerce Database — Maven Analytics*
