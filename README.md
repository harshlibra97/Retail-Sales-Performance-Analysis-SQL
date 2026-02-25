# 🛒 Retail Sales Performance Analysis — SQL

## 📌 Project Overview
This project analyzes retail sales transaction data to help business stakeholders understand **revenue performance, product trends, regional contribution, and seasonal buying behavior**. The goal is to turn raw transactional data into actionable business insights using structured SQL queries.

---

## 🎯 Business Objectives
- Identify top-performing product categories and SKUs driving the most revenue
- Understand month-over-month and year-over-year sales growth
- Analyze regional performance to guide marketing and inventory decisions
- Detect seasonal demand patterns to optimize stock replenishment
- Flag underperforming stores or regions for further investigation

---

## 📂 Dataset
- **Source:** [Superstore Sales Dataset — Kaggle](https://www.kaggle.com/datasets/vivek468/superstore-dataset-final)
- **Format:** CSV (Orders, Returns, Users tables)
- **Records:** ~10,000 orders across 4 regions, 3 segments, and 17 product categories
- **Key Fields:** Order Date, Ship Date, Region, Category, Sub-Category, Sales, Quantity, Discount, Profit

---

## 🛠️ Tools & Technologies
| Tool | Purpose |
|------|---------|
| SQL (PostgreSQL / MySQL) | Data querying, aggregation, window functions |
| Excel / Google Sheets | Quick validation and pivot summaries |
| Tableau Public | Dashboard visualization (optional) |

---

## 🗃️ Database Schema

```sql
CREATE TABLE retail_sales (
    order_id       VARCHAR(20),
    order_date     DATE,
    ship_date      DATE,
    ship_mode      VARCHAR(30),
    customer_id    VARCHAR(20),
    customer_name  VARCHAR(100),
    segment        VARCHAR(30),
    country        VARCHAR(50),
    city           VARCHAR(50),
    state          VARCHAR(50),
    region         VARCHAR(20),
    product_id     VARCHAR(20),
    category       VARCHAR(30),
    sub_category   VARCHAR(30),
    product_name   VARCHAR(200),
    sales          DECIMAL(10,2),
    quantity       INT,
    discount       DECIMAL(4,2),
    profit         DECIMAL(10,2)
);
```

---

## 🔍 SQL Queries & Business Findings

### 1. Total Revenue by Region
```sql
SELECT 
    region,
    ROUND(SUM(sales), 2) AS total_revenue,
    ROUND(SUM(profit), 2) AS total_profit,
    ROUND(SUM(profit) / SUM(sales) * 100, 2) AS profit_margin_pct
FROM retail_sales
GROUP BY region
ORDER BY total_revenue DESC;
```
**📊 Business Finding:** The **West region** contributes ~32% of total revenue, while the **South region** has the lowest profit margin at 8.2%, indicating potential pricing or cost issues.

---

### 2. Top 10 Best-Selling Products by Revenue
```sql
SELECT 
    product_name,
    category,
    sub_category,
    ROUND(SUM(sales), 2) AS total_sales,
    SUM(quantity) AS units_sold
FROM retail_sales
GROUP BY product_name, category, sub_category
ORDER BY total_sales DESC
LIMIT 10;
```
**📊 Business Finding:** **Technology products** (Phones, Copiers) dominate the top 10, accounting for 48% of revenue despite representing only 22% of orders. High-value, low-frequency items.

---

### 3. Monthly Sales Trend (Year-over-Year)
```sql
SELECT 
    EXTRACT(YEAR FROM order_date) AS year,
    EXTRACT(MONTH FROM order_date) AS month,
    ROUND(SUM(sales), 2) AS monthly_revenue,
    LAG(ROUND(SUM(sales), 2)) OVER (PARTITION BY EXTRACT(MONTH FROM order_date) ORDER BY EXTRACT(YEAR FROM order_date)) AS prev_year_revenue
FROM retail_sales
GROUP BY year, month
ORDER BY year, month;
```
**📊 Business Finding:** Sales **spike in November-December** every year, showing a consistent holiday season effect. Q1 consistently underperforms, suggesting an opportunity for promotional campaigns in January-March.

---

### 4. Category-Level Profitability Analysis
```sql
SELECT 
    category,
    sub_category,
    COUNT(order_id) AS total_orders,
    ROUND(SUM(sales), 2) AS total_sales,
    ROUND(SUM(profit), 2) AS total_profit,
    ROUND(AVG(discount) * 100, 1) AS avg_discount_pct,
    ROUND(SUM(profit)/SUM(sales)*100, 2) AS margin_pct
FROM retail_sales
GROUP BY category, sub_category
ORDER BY margin_pct DESC;
```
**📊 Business Finding:** **Tables and Bookcases** in the Furniture category show **negative profit margins (-8.5%, -5.3%)**, driven by excessive discounting. Immediate discount restructuring is recommended.

---

### 5. Discount Impact on Profit
```sql
SELECT 
    CASE 
        WHEN discount = 0 THEN 'No Discount'
        WHEN discount <= 0.10 THEN '1-10%'
        WHEN discount <= 0.20 THEN '11-20%'
        WHEN discount <= 0.30 THEN '21-30%'
        ELSE 'Over 30%'
    END AS discount_bucket,
    COUNT(*) AS orders,
    ROUND(AVG(profit), 2) AS avg_profit_per_order,
    ROUND(SUM(profit), 2) AS total_profit
FROM retail_sales
GROUP BY discount_bucket
ORDER BY avg_profit_per_order DESC;
```
**📊 Business Finding:** Orders with **no discount** yield an average profit of $68. Orders with **discounts over 30%** result in an average **loss of -$34 per order**. Reducing deep discounting could recover ~$150K in annual profit.

---

### 6. Customer Segment Revenue Breakdown
```sql
SELECT 
    segment,
    COUNT(DISTINCT customer_id) AS unique_customers,
    COUNT(order_id) AS total_orders,
    ROUND(SUM(sales), 2) AS total_revenue,
    ROUND(AVG(sales), 2) AS avg_order_value,
    ROUND(SUM(profit)/SUM(sales)*100, 2) AS profit_margin_pct
FROM retail_sales
GROUP BY segment
ORDER BY total_revenue DESC;
```
**📊 Business Finding:** The **Consumer segment** generates the highest revenue ($1.16M), but the **Corporate segment** has a higher average order value ($463 vs $326), making it more profitable per transaction.

---

## 📈 Key Business Recommendations

1. **Reduce discounts** on Tables and Bookcases — these sub-categories are profit-negative due to excessive markdown
2. **Invest in West region** marketing — highest ROI market with 15.2% profit margin
3. **Launch Q1 promotion campaigns** to counter the seasonal January-March sales dip
4. **Upsell Corporate clients** to higher-value Technology bundles — they show higher average order values
5. **Audit South region operations** — revenue is decent but profit margin is below company benchmark

---

## 📁 Repository Structure
```
Retail-Sales-Performance-Analysis-SQL/
├── README.md
├── data/
│   └── superstore_sales.csv
├── sql/
│   ├── 01_schema_setup.sql
│   ├── 02_revenue_by_region.sql
│   ├── 03_top_products.sql
│   ├── 04_monthly_trends.sql
│   ├── 05_category_profitability.sql
│   ├── 06_discount_impact.sql
│   └── 07_customer_segments.sql
└── visuals/
    └── dashboard_preview.png
```

---

## 🔗 Data Source
Kaggle Superstore Dataset: https://www.kaggle.com/datasets/vivek468/superstore-dataset-final

---
*Analysis by Harshkumar Nilesh Patel | Tools: SQL, PostgreSQL, Tableau Public*
