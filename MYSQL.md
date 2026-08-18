# SQL Data Analysis & Database Engineering - Adventure Works

## 📊 Overview
The MySQL database component powers the backend data engineering and metric aggregation for the Adventure Works portfolio project. Using **MySQL Workbench**, structured tables (`dimcustomer`, `dimdate`, `dimproduct`, `factinternetsales`) and optimized SQL queries were executed to perform relational lookups, compute financial metrics, and format aggregations for downstream reporting.

---

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
