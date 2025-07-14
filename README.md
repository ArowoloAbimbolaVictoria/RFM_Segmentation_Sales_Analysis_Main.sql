# RFM_Segmentation_Sales_Analysis_Main.sql
## 1. Introduction

Understanding sales performance is very important to making smarter business decisions. I analyzed the sales data to uncover trends, identify top-performing products and customers, and segment customers based on behavior using RFM analysis.

### Problem Statement

The objective was to analyze sales performance data to uncover trends, identify top-performing products and customers, and segment customers based on purchasing behavior using the RFM (Recency, Frequency, Monetary) model. This empowers the business to make smarter decisions around marketing, customer retention, product strategy, and regional focus.

### Data Preparation:

The dataset (sales_data_sample.csv) contains sales transactions across multiple years, with fields like order date, country, customer name, deal size, product line, and sales amount.

* Loaded the CSV file into a SQL-compatible environment.
* Cleaned and explored the data:
* Verified formats for dates and numeric fields
* Checked for missing or inconsistent values
* Ensured data types were query-friendly (e.g., dates, decimals, etc.)

Used SQL to perform:
* Yearly and monthly sales trend analysis
* Product and country-based performance analysis
* RFM-based customer segmentation
* Top customer and city performance insights

### Key Insights

* Sales grew steadily from 2003 to 2005
* Q4 months consistently had the highest sales, revealing seasonal spikes
* USA, France, and UK were top-performing countries
* Classic Cars product line drove the most revenue
* Top 10 customers contributed significantly to total revenue
* RFM analysis revealed a strong group of VIP and loyal customers

**The Goal** is to Provide business intelligence on sales trends, customer behavior, and product performance to improve marketing, sales strategies, and customer retention.

---

## 2. Understanding Sales Performance

To understand the business landscape, I worked with one key dataset: `sales_data_sample`. It includes information like `OrderDate`, `CustomerName`, `Country`, `ProductLine`, `DealSize`, `Status`, and `Sales`.

I started by asking the following:

* How are we doing year over year?
* Are there any seasonal sales trends?
* Which products and customers are driving our revenue?
* Can we use RFM analysis to segment our customer base?

---

## 3. Data and Some SQL Queries Used

### 3.1 Yearly Sales Trend

```sql
SELECT YEAR_ID,
       SUM(SALES) AS total_sales
FROM sales_data_sample
GROUP BY YEAR_ID
ORDER BY YEAR_ID;
```

✅ Sales increased year over year, with 2005 performing the best.

---

### 3.2 Monthly Trend and Seasonality

```sql
SELECT YEAR_ID,
       MONTH_ID,
       SUM(SALES) AS monthly_sales
FROM sales_data_sample
GROUP BY YEAR_ID, MONTH_ID
ORDER BY YEAR_ID, MONTH_ID;
```

✅ Seasonality observed: Q4 consistently performed best.

---

### 3.3 Sales by Country

```sql
SELECT COUNTRY,
       SUM(SALES) AS total_sales
FROM sales_data_sample
GROUP BY COUNTRY
ORDER BY total_sales DESC;
```

✅ USA led in total sales, followed by France and UK.

---

### 3.4 Sales by Deal Size & Status

```sql
SELECT DEALSIZE, STATUS, SUM(SALES) AS total_sales
FROM sales_data_sample
GROUP BY DEALSIZE, STATUS
ORDER BY DEALSIZE, STATUS;
```

✅ Medium and Large deals were the biggest contributors.

---

### 3.5 Best-Selling Product Lines

```sql
SELECT PRODUCTLINE, SUM(SALES) AS total_sales
FROM sales_data_sample
GROUP BY PRODUCTLINE
ORDER BY total_sales DESC;
```

✅ Classic Cars outperformed other product lines.

---

### 3.6 Average Sales Per Product Line

```sql
SELECT PRODUCTLINE,
       SUM(SALES) AS total_sales,
       AVG(SALES) AS avg_sales_per_order
FROM sales_data_sample
GROUP BY PRODUCTLINE
ORDER BY total_sales DESC;
```

✅ High-value orders were seen in Classic Cars and Vintage Cars.

---

### 3.7 Top Customers

```sql
SELECT CUSTOMERNAME, SUM(SALES) AS total_sales
FROM sales_data_sample
GROUP BY CUSTOMERNAME
ORDER BY total_sales DESC
LIMIT 10;
```

✅ Top 10 customers accounted for a large chunk of sales.

---

### 3.8 Top Cities in USA

```sql
SELECT CITY, COUNTRY, SUM(SALES) AS total_sales
FROM sales_data_sample
WHERE COUNTRY = 'USA'
GROUP BY CITY, COUNTRY
ORDER BY total_sales DESC
LIMIT 10;
```

✅ New York, Chicago, and Los Angeles led US city sales.

---

### 3.9 RFM Analysis: Who is our best customer?

```sql
-- Full RFM logic using CTEs and NTILE
WITH rfm AS (
    SELECT
        CUSTOMERNAME,  
        SUM(sales) AS MonetaryValue,  
        AVG(sales) AS AvgMonetaryValue,  
        COUNT(ordernumber) AS Frequency,  
        MAX(STR_TO_DATE(ORDERDATE, '%m/%d/%Y')) AS Last_Order_Date,
        DATEDIFF(
            (SELECT MAX(STR_TO_DATE(ORDERDATE, '%m/%d/%Y')) FROM sales_data_sample WHERE ORDERDATE IS NOT NULL),
            MAX(STR_TO_DATE(ORDERDATE, '%m/%d/%Y'))
        ) AS Recency
    FROM sales_data_sample
    GROUP BY CUSTOMERNAME
),
rfm_calc AS (
    SELECT
        CUSTOMERNAME,
        Recency,
        Frequency,
        MonetaryValue,
        NTILE(4) OVER (ORDER BY Recency DESC) AS rfm_recency,
        NTILE(4) OVER (ORDER BY Frequency) AS rfm_frequency,
        NTILE(4) OVER (ORDER BY MonetaryValue) AS rfm_monetary
    FROM rfm
)
SELECT
    CUSTOMERNAME,
    Recency,
    Frequency,
    MonetaryValue,
    CONCAT(rfm_recency, rfm_frequency, rfm_monetary) AS RFM_Score,
    (rfm_recency + rfm_frequency + rfm_monetary) AS RFM_Total,
    CASE
        WHEN (rfm_recency + rfm_frequency + rfm_monetary) >= 10 THEN 'VIP'
        WHEN (rfm_recency + rfm_frequency + rfm_monetary) BETWEEN 7 AND 9 THEN 'Loyal'
        WHEN (rfm_recency + rfm_frequency + rfm_monetary) BETWEEN 4 AND 6 THEN 'At Risk'
        ELSE 'Churned'
    END AS Customer_Segment_by_total
FROM rfm_calc
ORDER BY RFM_Total DESC;
```

💎 RFM helped group customers into actionable segments like **VIP**, **Loyal**, **At Risk**, and **Churned**.
This is where it gets powerful. Now you know who to retain, reward, and re-engage.

---

### 3.10 Orders Per Country

```sql
WITH customer_orders AS (
    SELECT CUSTOMERNAME, COUNTRY, COUNT(DISTINCT ORDERNUMBER) AS num_orders
    FROM sales_data_sample
    GROUP BY CUSTOMERNAME, COUNTRY
)
SELECT COUNTRY, ROUND(AVG(num_orders), 2) AS avg_orders_per_customer
FROM customer_orders
GROUP BY COUNTRY
ORDER BY avg_orders_per_customer DESC;
```

✅ Average orders per customer were highest in top-performing countries.
More orders per customer = higher lifetime value. Some countries outperform others.


---

## 4. Data Analysis & Key Findings

### 4.1 Seasonal and Yearly Trends

* Q4 consistently showed a surge in sales across all years
* 2005 had the highest overall revenue

### 4.2 Best Markets

* USA, France, and UK were top 3 countries by revenue
* USA cities like NY and LA drove significant value

### 4.3 Product Performance

| Product Line     | Total Sales | Avg Order Value |
| ---------------- | ----------- | --------------- |
| Classic Cars     | Highest     | High            |
| Vintage Cars     | Second      | High            |
| Trucks and Buses | Lower       | Lower           |

### 4.4 Customer Segmentation via RFM

| Segment | Description                        |
| ------- | ---------------------------------- |
| VIP     | Frequent, recent, high spenders    |
| Loyal   | Frequent but may not be recent     |
| At Risk | Not recent, average frequency      |
| Churned | Low frequency, low recent activity |

---

## 5. Recommendations

### 5.1 Customer Strategy

* **Retarget At-Risk customers** with discounts and loyalty perks
* **Reward VIPs** to maintain brand loyalty
* **Re-engage Churned** customers with “We Miss You” campaigns

### 5.2 Product Strategy

* Focus marketing around **Classic Cars**
* Promote slower-moving products with bundle offers

### 5.3 Regional Strategy

* Explore city-level campaigns in top-performing regions
* Consider localizing inventory based on regional demand

---

## 6. Conclusion

This project shows how SQL can help drive sales decisions. By analyzing trends, customer behavior, and product performance, businesses can:

* Increase retention with targeted strategies
* Optimize product mix
* Make smarter regional investment decisions

✅ Data-driven decisions = better growth and higher ROI

💡 "It’s not just about collecting data, it’s about connecting the dots."

---

## 7. Thank You
