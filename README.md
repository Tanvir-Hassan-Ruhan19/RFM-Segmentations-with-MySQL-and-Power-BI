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
## CODE 
```sql
SELECT * FROM sample_sales_data LIMIT 5;
```

## Output
| OrderNo | QuantityOrdered | PriceEach | OrderLineNo | Sales   | OrderDate | Status  | Quarter | Month | Year | ProductLine | MSR | ProductCode | CustomerName             | Phone       | AddressLine1                  | AddressLine2 | City          | State | PostalCode | Country | Territory | ContactLastName | ContactFirstName | DealSize |
| ------- | --------------- | --------- | ----------- | ------- | --------- | ------- | ------- | ----- | ---- | ----------- | --- | ----------- | ------------------------ | ----------- | ----------------------------- | ------------ | ------------- | ----- | ---------- | ------- | --------- | --------------- | ---------------- | -------- |
| 10107   | 30              | 95.7      | 2           | 2871    | 24/2/03   | Shipped | 1       | 2     | 2003 | Motorcycles | 95  | S10\_1678   | Land of Toys Inc.        | 2125557818  | 897 Long Airport Avenue       |              | NYC           | NY    | 10022      | USA     | NA        | Yu              | Kwai             | Small    |
| 10121   | 34              | 81.35     | 5           | 2765.9  | 7/5/03    | Shipped | 2       | 5     | 2003 | Motorcycles | 95  | S10\_1678   | Reims Collectables       | 26.47.1555  | 59 rue de l'Abbaye            |              | Reims         |       | 51100      | France  | EMEA      | Henriot         | Paul             | Small    |
| 10134   | 41              | 94.74     | 2           | 3884.34 | 1/7/03    | Shipped | 3       | 7     | 2003 | Motorcycles | 95  | S10\_1678   | Lyon Souveniers          | +33 1 46... | 27 rue du Colonel Pierre Avia |              | Paris         |       | 75508      | France  | EMEA      | Da Cunha        | Daniel           | Medium   |
| 10145   | 45              | 83.26     | 6           | 3746.7  | 25/8/03   | Shipped | 3       | 8     | 2003 | Motorcycles | 95  | S10\_1678   | Toys4GrownUps.com        | 6265557265  | 78934 Hillside Dr.            |              | Pasadena      | CA    | 90003      | USA     | NA        | Young           | Julie            | Medium   |
| 10159   | 49              | 100       | 14          | 5205.27 | 10/10/03  | Shipped | 4       | 10    | 2003 | Motorcycles | 95  | S10\_1678   | Corporate Gift Ideas Co. | 6505551386  | 7734 Strong St.               |              | San Francisco | CA    |            | USA     | NA        | Brown           | Julie            | Medium   |
## 2.Customer-Level Metrics Query
## Code
``SELECT 
    CUSTOMERNAME,
    ROUND(SUM(SALES),0) AS CLV,
    COUNT(DISTINCT ORDERNUMBER) AS FREQUENCY,
    SUM(QUANTITYORDERED) AS TOTAL_QTY_ORDERED,
    MAX(STR_TO_DATE(ORDERDATE,'%d/%m/%y')) AS CUSTOMER_LAST_TRASACTION_DATE,
    DATEDIFF(
        (SELECT MAX(STR_TO_DATE(ORDERDATE, '%d/%m/%y')) FROM SAMPLE_SALES_DATA),
        MAX(STR_TO_DATE(ORDERDATE, '%d/%m/%y'))
    ) AS CUSTOMER_RECENCY
FROM SAMPLE_SALES_DATA
GROUP BY CUSTOMERNAME
LIMIT 5;``

## Output:

| CustomerName                 | CLV    | Frequency | Total\_Qty\_Ordered | Last\_Transaction\_Date | Recency |
| ---------------------------- | ------ | --------- | ------------------- | ----------------------- | ------- |
| Alpha Cognac                 | 140977 | 3         | 1374                | 2005-03-28              | 64      |
| Amica Models & Co.           | 188235 | 2         | 1686                | 2004-09-09              | 264     |
| Anna's Decorations, Ltd      | 307992 | 4         | 2938                | 2005-03-09              | 83      |
| Atelier graphique            | 48360  | 3         | 540                 | 2004-11-25              | 187     |
| Australian Collectables, Ltd | 129183 | 3         | 1410                | 2005-05-09              | 22      |


## 3.Creating View For Segmentation 
## CODE:

CREATE VIEW RFM_SEGMENTATION_DATA AS
WITH CLV AS 
(
    SELECT
        CUSTOMERNAME,
        MAX(STR_TO_DATE(ORDERDATE, '%d/%m/%y')) AS CUSTOMER_LAST_TRANSACTION_DATE,
        DATEDIFF(
            (SELECT MAX(STR_TO_DATE(ORDERDATE, '%d/%m/%y')) FROM SAMPLE_SALES_DATA),
            MAX(STR_TO_DATE(ORDERDATE, '%d/%m/%y'))
        ) AS RECENCY_VALUE,
        COUNT(DISTINCT ORDERNUMBER) AS FREQUENCY_VALUE,
        SUM(QUANTITYORDERED) AS TOTAL_QTY_ORDERED,
        ROUND(SUM(SALES),0) AS MONETARY_VALUE
    FROM SAMPLE_SALES_DATA
    GROUP BY CUSTOMERNAME
),

RFM_SCORE AS
(
    SELECT 
        C.*,
        NTILE(5) OVER(ORDER BY RECENCY_VALUE DESC) AS R_SCORE,
        NTILE(5) OVER(ORDER BY FREQUENCY_VALUE ASC) AS F_SCORE,
        NTILE(5) OVER(ORDER BY MONETARY_VALUE ASC) AS M_SCORE
    FROM CLV AS C
),

RFM_COMBINATION AS
(
    SELECT
        R.*,
        R_SCORE + F_SCORE + M_SCORE AS TOTAL_RFM_SCORE,
        CONCAT_WS('', R_SCORE, F_SCORE, M_SCORE) AS RFM_COMBINATION
    FROM RFM_SCORE AS R
)

SELECT
    RC.*,
    CASE
        WHEN RFM_COMBINATION IN (455, 515, 542, 544, 552, 553, 452, 545, 554, 555) 
            THEN "Champions"
        WHEN RFM_COMBINATION IN (344, 345, 353, 354, 355, 443, 451, 342, 351, 352, 441, 442, 444, 445, 453, 454, 541, 543, 515, 551) 
            THEN 'Loyal Customers'
        WHEN RFM_COMBINATION IN (513, 413, 511, 411, 512, 341, 412, 343, 514) 
            THEN 'Potential Loyalists'
        WHEN RFM_COMBINATION IN (414, 415, 214, 211, 212, 213, 241, 251, 312, 314, 311, 313, 315, 243, 245, 252, 253, 255, 242, 244, 254) 
            THEN 'Promising Customers'
        WHEN RFM_COMBINATION IN (141, 142, 143, 144, 151, 152, 155, 145, 153, 154, 215) 
            THEN 'Needs Attention'
        WHEN RFM_COMBINATION IN (113, 111, 112, 114, 115) 
            THEN 'About to Sleep'
        ELSE "Other"
    END AS CUSTOMER_SEGMENT
FROM RFM_COMBINATION RC;


## 4.Final Segmentation
## CODE:

``SELECT * 
FROM RFM_SEGMENTATION_DATA
LIMIT 5;``

| CustomerName            | Last\_Transaction\_Date | Recency | Frequency | Total\_Qty\_Ordered | MonetaryValue | R\_Score | F\_Score | M\_Score | Total\_RFM\_Score | RFM\_Combination | Segment   |
| ----------------------- | ----------------------- | ------- | --------- | ------------------- | ------------- | -------- | -------- | -------- | ----------------- | ---------------- | --------- |
| Boards & Toys Co.       | 2005-02-08              | 112     | 2         | 204                 | 18259         | 4        | 2        | 1        | 7                 | 421              | Champions |
| Atelier graphique       | 2004-11-25              | 187     | 3         | 540                 | 48360         | 3        | 3        | 1        | 7                 | 331              | Champions |
| Auto-Moto Classics Inc. | 2004-12-03              | 179     | 3         | 574                 | 52959         | 3        | 3        | 1        | 7                 | 331              | Champions |
| Microscale Inc.         | 2004-11-03              | 209     | 2         | 762                 | 66290         | 2        | 1        | 1        | 4                 | 211              | Champions |
| Royale Belge            | 2005-01-10              | 141     | 4         | 556                 | 66880         | 4        | 5        | 1        | 10                | 451              | Champions |


## 5.Final RFM

## CODE:
``SELECT
	CUSTOMER_SEGMENT,
    SUM(MONETARY_VALUE) AS TOTAL_SPENDING,
    ROUND(AVG(MONETARY_VALUE),0) AS AVERAGE_SPENDING,
    SUM(FREQUENCY_VALUE) AS TOTAL_ORDER,
    SUM(TOTAL_QTY_ORDERED) AS TOTAL_QTY_ORDERED
FROM RFM_SEGMENTATION_DATA
GROUP BY CUSTOMER_SEGMENT;``

## Output:

| Customer Segment    | Total Spending | Average Spending | Total Orders | Total Qty Ordered |
| ------------------- | -------------- | ---------------- | ------------ | ----------------- |
| Champions           | 367,337        | 61,223           | 17           | 3,968             |
| About to Sleep      | 716,736        | 89,592           | 15           | 7,360             |
| Loyal Customers     | 577,609        | 115,522          | 13           | 5,722             |
| Potential Loyalists | 3,139,858      | 149,517          | 54           | 31,252            |
| Promising Customers | 3,329,075      | 184,949          | 50           | 32,178            |
| Needs Attention     | 3,473,776      | 231,585          | 47           | 33,536            |
| At Risk             | 2,312,250      | 289,031          | 31           | 22,820            |
| Lost/Inactive       | 6,148,617      | 558,965          | 80           | 61,298            |



## 🔑 Key Insights

- **Champions** and **Loyal Customers** are high-value segments, representing the most profitable and engaged customers.  
- **Needs Attention** and **About to Sleep** indicate customers that require re-engagement campaigns to prevent churn.  
- Segmenting customers by **RFM** enables businesses to:
  - Optimize marketing campaigns  
  - Improve customer retention strategies  
  - Focus on high-value customers for long-term growth
