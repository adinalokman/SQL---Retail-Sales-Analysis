# Retail Sales Analysis — SQL Project

## Project Overview
This project analyzes retail sales transaction data using MySQL. It covers the full workflow a data analyst follows in a real job: setting up a database, importing raw data, cleaning it, exploring it, answering business questions, and applying more advanced SQL techniques (subqueries, CTEs, window functions, and joins).

**Level:** Beginner → Intermediate
**Database:** `p1_retail_db`
**Tool:** MySQL Workbench

## Dataset
The dataset (`SQL_-_Retail_Sales_Analysis_utf.csv`) contains ~2000 retail transaction records with the following fields: transaction ID, sale date, sale time, customer ID, gender, age, product category, quantity, price per unit, cost of goods sold (COGS), and total sale amount.

## Objectives
1. **Database Setup** — Create a database and a table structured to hold the retail sales data.
2. **Data Cleaning** — Identify duplicate or missing records.
3. **Exploratory Data Analysis (EDA)** — Understand the dataset's basic shape (customers, categories, etc.).
4. **Business Analysis** — Answer real business questions using SQL.
5. **Advanced SQL** — Apply subqueries, CTEs, window functions, and joins to answer more complex questions.

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
    quantity INT,
    price_per_unit FLOAT,
    cogs FLOAT,
    total_sale FLOAT
);
```

Data was then imported from the CSV file using MySQL Workbench's Table Data Import Wizard.

**Note:** Of the original 2000 records, 1987 were successfully imported. The remaining 13 failed to insert due to duplicate `transactions_id` values, which is expected behavior since it's set as a `PRIMARY KEY`.

**Note:** The `quantity` column was originally imported as `quantiy` (a typo in the source CSV) and later corrected:
```sql
ALTER TABLE retail_sales
RENAME COLUMN quantiy TO quantity;
```

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
   OR category IS NULL OR quantity IS NULL OR price_per_unit IS NULL
   OR cogs IS NULL OR total_sale IS NULL;
```
**Result:** 0 rows — no missing values. The dataset was clean and ready for analysis.

## 3. Exploratory Data Analysis (EDA)

```sql
SELECT COUNT(DISTINCT customer_id) AS total_customer_unique
FROM retail_sales;
-- Result: 155 unique customers

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
SELECT category, SUM(quantity) AS total_unit_sold
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

## 5. Advanced SQL

### Subquery: Transactions above the overall average sale
```sql
SELECT *
FROM retail_sales
WHERE total_sale > (SELECT AVG(total_sale) FROM retail_sales);
```
**Insight:** Above-average transactions tend to come from higher-priced items (Electronics/Clothing) and/or larger quantities purchased.

### CTE: Customers who spend more than the average customer
```sql
WITH customer_spending AS (
    SELECT customer_id, SUM(total_sale) AS total_spending
    FROM retail_sales
    GROUP BY customer_id
)
SELECT *
FROM customer_spending
WHERE total_spending > (SELECT AVG(total_spending) FROM customer_spending);
```

**Follow-up: how many customers are above vs below average?**
```sql
WITH customer_spending AS (
    SELECT customer_id, SUM(total_sale) AS total_spending
    FROM retail_sales
    GROUP BY customer_id
)
SELECT 
    (SELECT COUNT(*) FROM customer_spending WHERE total_spending > (SELECT AVG(total_spending) FROM customer_spending)) AS above_average_count,
    (SELECT COUNT(*) FROM customer_spending WHERE total_spending <= (SELECT AVG(total_spending) FROM customer_spending)) AS below_average_count;
```
| Above Average | Below Average |
|---|---|
| 61 | 94 |

**Insight:** Only 39% of customers (61 of 155) spend above the average — meaning the average is pulled upward by a smaller group of high-value customers. The majority (61%) actually spend below it, a right-skewed distribution. This shows why average alone can be misleading when describing a "typical" customer — median or distribution should be checked alongside it.

### Window Function: Ranking all customers by spending
```sql
SELECT 
    customer_id,
    SUM(total_sale) AS total_spending,
    RANK() OVER (ORDER BY SUM(total_sale) DESC) AS spending_rank
FROM retail_sales
GROUP BY customer_id;
```
**Insight:** Unlike `GROUP BY` + `LIMIT`, a window function ranks every customer (all 155) without dropping any rows — useful for building customer tiers (e.g. VIP/Gold/Regular) based on rank.

### Join: Comparing actual sales against category targets
A second table was created to simulate a target-setting scenario:
```sql
CREATE TABLE category_target (
    category VARCHAR(30),
    monthly_target FLOAT
);

INSERT INTO category_target (category, monthly_target) VALUES
('Clothing', 25000),
('Beauty', 22000),
('Electronics', 27000);
```

```sql
SELECT 
    r.category, 
    SUM(r.total_sale) AS actual_sale, 
    c.monthly_target,
    SUM(r.total_sale) - c.monthly_target AS variance
FROM retail_sales r
JOIN category_target c ON r.category = c.category
GROUP BY r.category, c.monthly_target;
```
| Category | Actual Sale | Monthly Target | Variance |
|---|---|---|---|
| Beauty | 286,790 | 22,000 | +264,790 |
| Clothing | 309,995 | 25,000 | +284,995 |
| Electronics | 311,445 | 27,000 | +284,445 |

**Learning note:** All categories appear to vastly exceed target — but this comparison isn't actually fair. `total_sale` here is summed across the *entire* dataset (spanning multiple years), while `monthly_target` represents a single month. A more accurate comparison would either annualize the target or filter `retail_sales` to a single month before comparing. This is a reminder that a query can run successfully and still produce a misleading comparison if the underlying business logic isn't checked.

---

## Key Takeaways
- Electronics leads in revenue; Clothing leads in units sold — two different, complementary views of category performance.
- Evening is by far the busiest sales period.
- A small group of high-value customers (e.g. Customer #3) contributes disproportionately to revenue; only 39% of customers spend above average.
- Customer demographics (age, gender) are fairly evenly distributed across categories.
- Comparisons across time periods (e.g. actual vs target) need matching time scopes to be meaningful.

## Tools Used
- MySQL / MySQL Workbench
- SQL: `CREATE`/`ALTER`/`INSERT`, `GROUP BY`, `CASE WHEN`, aggregate functions, subqueries, CTEs (`WITH`), window functions (`RANK() OVER`), `JOIN`

## Author
Nur Adina Faqiha
