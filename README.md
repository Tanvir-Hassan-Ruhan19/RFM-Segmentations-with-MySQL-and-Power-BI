# 📊 RFM Segmentation Project

## 🧠 Project Overview
This project demonstrates an **end-to-end RFM (Recency, Frequency, Monetary) Segmentation** workflow using MySQL and Power BI. It classifies customers into meaningful segments for actionable insights.  

**Key Steps:**
- ✅ Created a dedicated MySQL database (`RFM_SALES`)  
- ✅ Inserted sample customer data into `SAMPLE_SALES_DATA` table  
- ✅ Calculated customer-level metrics: CLV, Frequency, Total Quantity Ordered, Recency  
- ✅ Performed RFM Segmentation and created a view `RFM_SEGMENTATION_DATA`  
- ✅ Aggregated segment-level metrics for business analysis  

---

## 📂 Sample Data

```sql
SELECT * FROM sample_sales_data LIMIT 5;
| OrderNo | QuantityOrdered | PriceEach | OrderLineNo | Sales   | OrderDate | Status  | Quarter | Month | Year | ProductLine | MSR | ProductCode | CustomerName             | Phone       | AddressLine1                  | AddressLine2 | City          | State | PostalCode | Country | Territory | ContactLastName | ContactFirstName | DealSize |
| ------- | --------------- | --------- | ----------- | ------- | --------- | ------- | ------- | ----- | ---- | ----------- | --- | ----------- | ------------------------ | ----------- | ----------------------------- | ------------ | ------------- | ----- | ---------- | ------- | --------- | --------------- | ---------------- | -------- |
| 10107   | 30              | 95.7      | 2           | 2871    | 24/2/03   | Shipped | 1       | 2     | 2003 | Motorcycles | 95  | S10\_1678   | Land of Toys Inc.        | 2125557818  | 897 Long Airport Avenue       |              | NYC           | NY    | 10022      | USA     | NA        | Yu              | Kwai             | Small    |
| 10121   | 34              | 81.35     | 5           | 2765.9  | 7/5/03    | Shipped | 2       | 5     | 2003 | Motorcycles | 95  | S10\_1678   | Reims Collectables       | 26.47.1555  | 59 rue de l'Abbaye            |              | Reims         |       | 51100      | France  | EMEA      | Henriot         | Paul             | Small    |
| 10134   | 41              | 94.74     | 2           | 3884.34 | 1/7/03    | Shipped | 3       | 7     | 2003 | Motorcycles | 95  | S10\_1678   | Lyon Souveniers          | +33 1 46... | 27 rue du Colonel Pierre Avia |              | Paris         |       | 75508      | France  | EMEA      | Da Cunha        | Daniel           | Medium   |
| 10145   | 45              | 83.26     | 6           | 3746.7  | 25/8/03   | Shipped | 3       | 8     | 2003 | Motorcycles | 95  | S10\_1678   | Toys4GrownUps.com        | 6265557265  | 78934 Hillside Dr.            |              | Pasadena      | CA    | 90003      | USA     | NA        | Young           | Julie            | Medium   |
| 10159   | 49              | 100       | 14          | 5205.27 | 10/10/03  | Shipped | 4       | 10    | 2003 | Motorcycles | 95  | S10\_1678   | Corporate Gift Ideas Co. | 6505551386  | 7734 Strong St.               |              | San Francisco | CA    |            | USA     | NA        | Brown           | Julie            | Medium   |

