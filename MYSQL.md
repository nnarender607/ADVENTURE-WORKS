# SQL Data Analysis & Database Engineering - Adventure Works


## 📊 Overview
The MySQL database component powers the backend data engineering and metric aggregation for the Adventure Works portfolio project. Using **MySQL Workbench**, structured tables (`dimcustomer`, `dimdate`, `dimproduct`, `factinternetsales`) and optimized SQL queries were executed to perform relational lookups, compute financial metrics, and format aggregations for downstream reporting.

---

## 🛠️ Complete SQL Script & Implementation

```sql
create database adventure_works_cycle;
use adventure_works_cycle;

select * from dimcustomer;
select* from dimdate;
select * from dimproduct;
select * from dimproductcategory;
select * from dimproductsubcategory;
select * from dimsalesterritory;
select * from factinternetsales;
select * from factinternetsalesnew;

#Q 0.Union of Fact Internet sales and Fact internet sales new

CREATE VIEW FactInternetSales_All AS
SELECT *
FROM FactInternetSales
UNION
SELECT *
FROM Fact_Internet_Sales_new;

select count(*) from factinternetsales_All;

SELECT * FROM FactInternetSales_All;

CREATE OR REPLACE VIEW vw_sales_with_product AS
SELECT
    f.SalesOrderNumber,
    p.EnglishProductName AS ProductName,
    f.UnitPrice,
    f.OrderQuantity,
    f.SalesAmount
FROM FactInternetSales_All f
LEFT JOIN DimProduct p
    ON f.ProductKey = p.ProductKey;
    
    select  * from vw_sales_with_product;

# Q.2.Lookup the Customerfullname from the Customer and Unit Price from Product sheet to Sales sheet.
CREATE OR REPLACE VIEW vw_sales_customer AS
SELECT
    f.SalesOrderNumber,
    f.SalesOrderLineNumber,
    f.OrderDateKey,
    c.CustomerKey,
    CONCAT_WS(' ', c.FirstName, c.LastName) AS CustomerFullName,
    c.Gender,
    c.EnglishOccupation,
    p.ProductKey,
    p.EnglishProductName AS ProductName,
    p.unitPrice,
    f.OrderQuantity,
    f.UnitPrice,
    f.SalesAmount
FROM FactInternetSales_All f
LEFT JOIN DimCustomer c
    ON f.CustomerKey = c.CustomerKey
LEFT JOIN DimProduct p
    ON f.ProductKey = p.ProductKey;
    
SELECT Gender, COUNT(*), SUM(SalesAmount), COUNT(SalesAmount)
FROM vw_sales_customer
GROUP BY Gender;

SELECT
    COALESCE(NULLIF(TRIM(Gender), ''), 'Unknown') AS Gender,
    CONCAT(CAST(COALESCE(SUM(SalesAmount), 0) / 1000000 AS DECIMAL(12,2)), ' M') AS TotalSales
FROM vw_sales_customer
GROUP BY COALESCE(NULLIF(TRIM(Gender), ''), 'Unknown');

select * from vw_sales_customer;


# Q3 A)Year
SELECT
    d.FullDateAlternateKey AS OrderDate,
    d.CalendarYear AS Year
FROM FactInternetSales_All f
JOIN DimDate d
    ON f.OrderDateKey = d.DateKey
ORDER BY d.FullDateAlternateKey;
# B. Monthno, C.MonthFullName
SELECT
    d.FullDateAlternateKey AS OrderDate,
    d.MonthNumberOfYear AS MonthNo,
    d.EnglishMonthName AS MonthName
FROM FactInternetSales_All f
JOIN DimDate d
    ON f.OrderDateKey = d.DateKey
ORDER BY d.FullDateAlternateKey;

SELECT
    STR_TO_DATE(f.OrderDateKey, '%Y%m%d') AS OrderDate,
    MONTHNAME(STR_TO_DATE(f.OrderDateKey, '%Y%m%d')) AS MonthFullName
FROM FactInternetSales_All f
ORDER BY OrderDate;
# D. Quarter
SELECT
    d.FullDateAlternateKey AS OrderDate,
    d.CalendarQuarter AS Quarter
FROM FactInternetSales_All f
JOIN DimDate d
    ON f.OrderDateKey = d.DateKey
ORDER BY d.FullDateAlternateKey;
# E.YearMonth
SELECT
    d.FullDateAlternateKey AS OrderDate,
    DATE_FORMAT(d.FullDateAlternateKey, '%Y-%b') AS YearMonth
FROM FactInternetSales_All f
JOIN DimDate d
    ON f.OrderDateKey = d.DateKey
ORDER BY d.FullDateAlternateKey;
# F.WeekdayNo
SELECT
    d.FullDateAlternateKey AS OrderDate,
    d.DayNumberOfWeek AS WeekdayNo
FROM FactInternetSales_All f
JOIN DimDate d
    ON f.OrderDateKey = d.DateKey
ORDER BY d.FullDateAlternateKey;
# G. WeekdayName
SELECT
    d.FullDateAlternateKey AS OrderDate,
    d.EnglishDayNameOfWeek AS WeekdayName
FROM FactInternetSales_All f
JOIN DimDate d
    ON f.OrderDateKey = d.DateKey
ORDER BY d.FullDateAlternateKey;
# H. Financial Month
SELECT
    d.FullDateAlternateKey AS OrderDate,
    CONCAT(
        'FM',
        CASE
            WHEN d.MonthNumberOfYear >= 4
                THEN d.MonthNumberOfYear - 3
            ELSE d.MonthNumberOfYear + 9
        END
    ) AS FinancialMonth
FROM FactInternetSales_All f
JOIN DimDate d
    ON f.OrderDateKey = d.DateKey
ORDER BY d.FullDateAlternateKey;

# I. Financial Quarter
SELECT
    d.FullDateAlternateKey AS OrderDate,
    CONCAT('FQ', d.FiscalQuarter) AS FinancialQuarter
FROM FactInternetSales_All f
JOIN DimDate d
    ON f.OrderDateKey = d.DateKey
ORDER BY d.FullDateAlternateKey;


CREATE OR REPLACE VIEW vw_sales_with_datefields AS
SELECT
    f.SalesOrderNumber,
    f.SalesOrderLineNumber,
    d.FullDateAlternateKey AS OrderDate,
    d.CalendarYear AS Year,
    d.MonthNumberOfYear AS MonthNo,
    d.EnglishMonthName AS MonthFullName,
    d.CalendarQuarter AS Quarter,
    DATE_FORMAT(d.FullDateAlternateKey, '%Y-%b') AS YearMonth,
    d.DayNumberOfWeek AS WeekdayNo,
    d.EnglishDayNameOfWeek AS WeekdayName,
    CONCAT(
        'FM',
        CASE
            WHEN d.MonthNumberOfYear >= 4 THEN d.MonthNumberOfYear - 3
            ELSE d.MonthNumberOfYear + 9
        END
    ) AS FinancialMonth,
    CASE
        WHEN d.MonthNumberOfYear BETWEEN 4 AND 6   THEN 'FQ1'
        WHEN d.MonthNumberOfYear BETWEEN 7 AND 9   THEN 'FQ2'
        WHEN d.MonthNumberOfYear BETWEEN 10 AND 12 THEN 'FQ3'
        ELSE 'FQ4'
    END AS FinancialQuarter,
    f.SalesAmount,
    f.OrderQuantity
FROM FactInternetSales_All f
JOIN DimDate d
    ON f.OrderDateKey = d.DateKey;

SELECT *
FROM vw_sales_with_datefields;


# Q.4.Calculate the Sales amount uning the columns(unit price,order quantity,unit discount)
SELECT
    f.UnitPrice,
    f.OrderQuantity,
    f.UnitPriceDiscountPct AS UnitDiscount,
    (f.UnitPrice * f.OrderQuantity)
      - (f.UnitPrice * f.OrderQuantity * f.UnitPriceDiscountPct)
      AS SalesAmount
FROM FactInternetSales_All f;

# Total Sales
SELECT  
    concat(cast(SUM((f.UnitPrice * f.OrderQuantity)
            - (f.UnitPrice * f.OrderQuantity * f.UnitPriceDiscountPct)) / 1000000
        AS DECIMAL(12,2)),' M'
    ) AS TotalSales_Million
FROM FactInternetSales_All f;

# Q.5.Calculate the Productioncost uning the columns(unit cost ,order quantity)
SELECT
    f.SalesOrderNumber,
    f.ProductKey,
    f.OrderQuantity,
    f.productStandardCost AS UnitCost,
    (f.productStandardCost * f.OrderQuantity) AS ProductionCost
FROM FactInternetSales_All f;

 
# Q.6.Calculate the profit.
    SELECT
    f.SalesOrderNumber,
    f.OrderQuantity,
    f.UnitPrice,
    f.productstandardcost,
CAST(((f.UnitPrice * f.OrderQuantity)
            - (f.UnitPrice * f.OrderQuantity * f.UnitPriceDiscountPct)
            - (f.productStandardCost * f.OrderQuantity))
        AS DECIMAL(12,2)
    ) AS Profit
    FROM FactInternetSales_All f;


# Q.7. Total Profit
SELECT
    CONCAT(CAST(SUM(
            (f.UnitPrice * f.OrderQuantity)
            - (f.UnitPrice * f.OrderQuantity * f.UnitPriceDiscountPct)
            - (COALESCE(f.productStandardCost, 0) * f.OrderQuantity)
        ) / 1000000 AS DECIMAL(12,2)),
        ' M'
    ) AS TotalProfit
FROM FactInternetSales_All f;

# Q.8 Yearwise Sales
SELECT
    YEAR(STR_TO_DATE(f.OrderDateKey, '%Y%m%d')) AS Year,
    concat(CAST(SUM((f.UnitPrice * f.OrderQuantity)
            - (f.UnitPrice * f.OrderQuantity * f.UnitPriceDiscountPct))/1000000 AS DECIMAL (12,2)),' M'
    ) AS YearwiseSales
FROM FactInternetSales_All f
GROUP BY Year
ORDER BY Year;

# Q.9. Monthwise Sales
SELECT
    MONTHNAME(order_date) AS MonthFullName,
    CONCAT(CAST(SUM(UnitPrice * OrderQuantity * (1 - UnitPriceDiscountPct)) / 1000000
            AS DECIMAL(12,2)),' M'
    ) AS MonthwiseSales
FROM (
    SELECT
        *,
        STR_TO_DATE(OrderDateKey, '%Y%m%d') AS order_date
    FROM FactInternetSales_All) f
GROUP BY
    MONTH(order_date),
    MONTHNAME(order_date)
ORDER BY
    MONTH(order_date);


# Q.10. Quarterwise Sales
    SELECT
    CONCAT('Q', qtr) AS Quarter,
    CONCAT(CAST(
            SUM(UnitPrice * OrderQuantity * (1 - UnitPriceDiscountPct)) / 1000000
            AS DECIMAL(12,2)
        ),' M'
    ) AS QuarterwiseSales
FROM (
    SELECT
        UnitPrice,
        OrderQuantity,
        UnitPriceDiscountPct,
        QUARTER(STR_TO_DATE(OrderDateKey, '%Y%m%d')) AS qtr
    FROM FactInternetSales_All
) f
GROUP BY qtr
ORDER BY qtr;

#Q.11. Yearwise Sales and ProductionCost
SELECT
    YEAR(STR_TO_DATE(f.OrderDateKey, '%Y%m%d')) AS Year,
    -- Sales Amount 
    CONCAT(ROUND(SUM((f.UnitPrice * f.OrderQuantity)
                - (f.UnitPrice * f.OrderQuantity * f.UnitPriceDiscountPct)
            ) / 1000000,2),' M'
    ) AS SalesAmount,
    -- Production 
    CONCAT(ROUND(SUM(f.productStandardCost * f.OrderQuantity) / 
    1000000,2),' M'
    ) AS ProductionCost
FROM FactInternetSales_All f

GROUP BY YEAR(STR_TO_DATE(f.OrderDateKey, '%Y%m%d'))
ORDER BY Year;

# Q.12 Build KPI's
SELECT
    COUNT(DISTINCT f.CustomerKey) AS TotalCustomers,
    COUNT(f.SalesOrderNumber) AS TotalOrders,
    CONCAT(
        ROUND(SUM(
            (f.UnitPrice * f.OrderQuantity)
            - (f.UnitPrice * f.OrderQuantity * f.UnitPriceDiscountPct)
        ) / 1000000, 2), ' M'
    ) AS TotalSales,
    CONCAT(
        ROUND(SUM(
            (f.UnitPrice * f.OrderQuantity)
            - (f.UnitPrice * f.OrderQuantity * f.UnitPriceDiscountPct)
            - (COALESCE(f.productStandardCost, 0) * f.OrderQuantity)
        ) / 1000000, 2), ' M'
    ) AS TotalProfit
FROM FactInternetSales_All f;



#calculation of total productioncost
SELECT
    concat(cast(SUM((f.productstandardcost * f.OrderQuantity)
            ) / 1000000
        AS DECIMAL(12,2)),' M'
    ) AS Totalproductioncost
FROM FactInternetSales_All f;
```

## 🛠️ Key SQL Operations & Executed Queries

* **Relational Joins & Product Lookups:** Merged fact tables with dimension tables (`DimProduct`) using `ProductKey` to extract specific attributes like `ProductName`, `StandardCost` (`ProductionCost`), and unit pricing.
* **Calculated Financial Metrics:** Engineered SQL formulas to compute derived metrics dynamically, including:
  * **Sales Amount:** Calculated via order quantities, unit prices, and discount percentages.
  * **Total Profit:** Computed by subtracting total standard production costs from net sales revenue and formatted into millions (`'M'`).
* **Time-Series & Date Transformations:** Utilized string-to-date conversion functions (`STR_TO_DATE`) and calendar extractions to group data by `Year`, `MonthFullName`, and calendar `Quarter`.
* **Aggregations & Grouping:**
  * **Total Sales Summary:** Aggregated overall macro sales figures reaching **29.36 M**.
  * **Year-Wise Sales Trend:** Grouped annual performance showing key progression milestones (e.g., **16.35 M** in peak years).
  * **Month-Wise Sales Trend:** Evaluated monthly sales velocity, tracking seasonal peaks like December at **3.21 M**.
  * **Quarter-Wise Performance:** Summarized quarterly revenue contributions (e.g., Q4 at **9.11 M**, Q3 at **7.64 M**, Q2 at **7.09 M**, and Q1 at **5.52 M**).
<img width="931" height="256" alt="profit" src="https://github.com/user-attachments/assets/83ae8cd4-577b-4aeb-8b2c-3c0324792359" />
<img width="373" height="277" alt="productname,sales" src="https://github.com/user-attachments/assets/c0bcd45d-462e-4037-8402-1d045a9a210e" />
<img width="363" height="291" alt="product name,productioncost" src="https://github.com/user-attachments/assets/2f83aff4-8c46-4957-ba71-20f6d9c61692" />
<img width="466" height="275" alt="product name" src="https://github.com/user-attachments/assets/fb73158d-2e30-40f3-98a1-4cbf32bf3b6a" />
<img width="1920" height="1080" alt="Screenshot (87)" src="https://github.com/user-attachments/assets/a74c95b3-93cf-46ec-afe5-3f009300542c" />
<img width="1920" height="1080" alt="Screenshot (86)" src="https://github.com/user-attachments/assets/69aa8002-abaa-4f1e-859b-4548cea52f7d" />
