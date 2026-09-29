# Retail Sales Analysis — SQL Project

## Project Overview
This project analyzes retail sales transaction data using MySQL. It covers the full workflow a data analyst follows in a real job: setting up a database, importing raw data, cleaning it, exploring it, and answering business questions through SQL queries.

**Level:** Beginner
**Database:** `p1_retail_db`
**Tool:** MySQL Workbench

## Dataset
The dataset (`SQL_-_Retail_Sales_Analysis_utf.csv`) contains 2000 retail transaction records with the following fields: transaction ID, sale date, sale time, customer ID, gender, age, product category, quantity, price per unit, cost of goods sold (COGS), and total sale amount.

## Objectives
1. **Database Setup** — Create a database and a table structured to hold the retail sales data.
2. **Data Cleaning** — Identify duplicate or missing records.
3. **Exploratory Data Analysis (EDA)** — Understand the dataset's basic shape (customers, categories, etc.).
4. **Business Analysis** — Answer real business questions using SQL.

---

## 1. Database Setup

```sql
CREATE DATABASE p1_retail_db;
USE p1_retail_db;

CREATE TABLE retail_sales
(
    transactions_id INT PRIMARY KEY,
    sale_date DATE,
    sale_time TIME,
    customer_id INT,
    gender VARCHAR(15),
    age INT,
    category VARCHAR(30),
    quantiy INT,
    price_per_unit FLOAT,
    cogs FLOAT,
    total_sale FLOAT
);
```

Data was then imported from the CSV file using MySQL Workbench's Table Data Import Wizard.

**Note:** Of the original 2000 records, 1987 were successfully imported. The remaining 13 failed to insert due to duplicate `transactions_id` values, which is expected behavior since it's set as a `PRIMARY KEY`.

## 2. Data Cleaning

Checked for duplicate transaction IDs remaining in the table:

```sql
SELECT transactions_id, COUNT(*) AS jumlah
FROM retail_sales
GROUP BY transactions_id
HAVING COUNT(*) > 1;
```
**Result:** 0 rows — no duplicates remained after import.

Checked for missing (NULL) values across all columns:

```sql
SELECT * FROM retail_sales
WHERE transactions_id IS NULL OR sale_date IS NULL OR sale_time IS NULL
   OR customer_id IS NULL OR gender IS NULL OR age IS NULL
   OR category IS NULL OR quantiy IS NULL OR price_per_unit IS NULL
   OR cogs IS NULL OR total_sale IS NULL;
```
**Result:** 0 rows — no missing values. The dataset was clean and ready for analysis.

## 3. Exploratory Data Analysis (EDA)

```sql
-- Total unique customers
SELECT COUNT(DISTINCT customer_id) AS total_customer_unique
FROM retail_sales;
-- Result: 155 unique customers

-- Product categories available
SELECT DISTINCT category FROM retail_sales;
-- Result: Beauty, Clothing, Electronics
```

## 4. Business Analysis & Key Insights

### Q1: Which category generates the most revenue?
```sql
SELECT category, SUM(total_sale) AS total_revenue
FROM retail_sales
GROUP BY category
ORDER BY total_revenue DESC;
```
| Category | Total Revenue |
|---|---|
| Electronics | 311,445 |
| Clothing | 309,995 |
| Beauty | 286,790 |

**Insight:** Electronics leads in revenue, but only narrowly ahead of Clothing — the three categories are fairly balanced.

### Q2: What is the average customer age per category?
```sql
SELECT category, ROUND(AVG(age), 1) AS average_age
FROM retail_sales
GROUP BY category;
```
| Category | Avg Age |
|---|---|
| Beauty | 40.4 |
| Clothing | 41.9 |
| Electronics | 41.6 |

**Insight:** Average customer age is consistent (~40-42) across all categories — no single category skews toward a particular age group.

### Q3: Which time of day sees the most transactions?
```sql
SELECT 
    CASE 
        WHEN HOUR(sale_time) < 12 THEN 'Morning'
        WHEN HOUR(sale_time) BETWEEN 12 AND 17 THEN 'Afternoon'
        ELSE 'Evening'
    END AS shift,
    COUNT(*) AS total_transaction
FROM retail_sales
GROUP BY shift
ORDER BY total_transaction DESC;
```
| Shift | Transactions |
|---|---|
| Evening | 1062 |
| Morning | 548 |
| Afternoon | 377 |

**Insight:** Over half of all transactions happen in the evening — a key window for staffing and promotions.

### Q4: Who are the top 5 spending customers?
```sql
SELECT customer_id, SUM(total_sale) AS total_expenses
FROM retail_sales
GROUP BY customer_id
ORDER BY total_expenses DESC
LIMIT 5;
```
| Customer ID | Total Spend |
|---|---|
| 3 | 38,440 |
| 1 | 30,750 |
| 5 | 30,405 |
| 2 | 25,295 |
| 4 | 23,580 |

**Insight:** Customer #3 is the top spender by a clear margin — a strong candidate for a loyalty/retention program.

### Q5: Which category sells the most units (not revenue)?
```sql
SELECT category, SUM(quantiy) AS total_unit_sold
FROM retail_sales
GROUP BY category
ORDER BY total_unit_sold DESC;
```
| Category | Units Sold |
|---|---|
| Clothing | 1780 |
| Electronics | 1682 |
| Beauty | 1533 |

**Insight:** Clothing sells the most units despite Electronics generating more revenue — Electronics has a higher price point per unit.

### Q6: How does spending compare between male and female customers?
```sql
SELECT gender, COUNT(*) AS total_transaction, SUM(total_sale) AS total_revenue
FROM retail_sales
GROUP BY gender;
```
| Gender | Transactions | Revenue |
|---|---|---|
| Male | 975 | 445,120 |
| Female | 1012 | 463,110 |

**Insight:** Female customers make up a slightly larger share of both transactions and revenue, though the gap is small — a fairly balanced customer base.

---

## Key Takeaways
- Electronics leads in revenue; Clothing leads in units sold — two different, complementary views of category performance.
- Evening is by far the busiest sales period.
- A small group of high-value customers (e.g. Customer #3) contributes disproportionately to revenue.
- Customer demographics (age, gender) are fairly evenly distributed across categories.

## Tools Used
- MySQL / MySQL Workbench
- SQL (CREATE, GROUP BY, CASE WHEN, aggregate functions, subqueries)

## Author
Nur Adina Faqiha
