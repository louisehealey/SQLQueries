# 🗄️Microsoft SQL Queries

This repository provides comprehensive queries that connect Power BI to a SQL Server database.

## 🏆 Benefits of Loading Data into Power BI via SQL  
- **Direct Querying** – Minimizes unnecessary data transfer, optimizing performance and data security.  
- **Enhanced Stability** – SQL efficiently handles large datasets, ensuring smooth operations.  
- **Reduced Complexity in Power BI Models** – Performing transformations in SQL decreases Power BI’s workload, resulting in simpler, faster-performing reports.  

## ⚙️ Inventory Transactions
This SQL command retrieves the transactional data from the `PO_TRANSACTION` table for the past 91 days, counting back from today while filtering on records where `ACTION_TYPE` is set to "I".
 ``` 
SELECT
   PO_TRANSACTION.PART_ID,
   PO_TRANSACTION.STORES_CODE,
   PO_TRANSACTION.ACTION_DATE,
   PO_TRANSACTION.ACTION_TYPE,
   PO_TRANSACTION.GL_YEAR,
   PO_TRANSACTION.QUANTITY
FROM PO_TRANSACTION
WHERE 
   PO_TRANSACTION.ACTION_TYPE = 'I'
    AND PO_TRANSACTION.ACTION_DATE >= DATEADD(DAY, -91, GETDATE())
 ``` 
## ⚙️ Stock Categorization
This SQL query retrieves stock-related data from the `STOCK_STATUS` table, filtering for specified Part-IDs. It aggregates Total On-Hand quantities and Safety Stock levels, grouping the results by store codes for better inventory analysis.
 ``` 
SELECT 
    STOCK_STATUS."PART_ID",
    SUM(CASE WHEN STOCK_STATUS."STORES_CODE" IN ('7F', '7Z') THEN       
    STOCK_STATUS."ON_HAND_QTY" ELSE 0 END) AS "QoH",
    SUM(CASE WHEN STOCK_STATUS."STORES_CODE" = '7F' THEN 
    STOCK_STATUS."SAFETY_STOCK" ELSE 0 END) AS "7F_SS",
    SUM(CASE WHEN STOCK_STATUS."STORES_CODE" = '7Z' THEN STOCK_STATUS."SAFETY_STOCK" ELSE 0 END) AS "7Z_SS"
FROM STOCK_STATUS
WHERE STOCK_STATUS."PART_ID" LIKE '900%' 
   OR STOCK_STATUS."PART_ID" LIKE '4%' 
   OR STOCK_STATUS."PART_ID" LIKE '1%'
GROUP BY STOCK_STATUS."PART_ID";

 ``` 
## ⚙️ Inventory and Jobs Overview by Part ID
This SQL query retrieves inventory and job-related data for all parts  with a European Union (EU) stores code. It provides insights into stock levels, part details, and outstanding balances.

By joining `STOCK_STATUS` and `JOB_MANAGER` on the `PART_MANAGER` table, this query optimizes data retrieval by reducing the number of tables and minimizing the amount of data loaded into the model.
 ``` 
SELECT 
    STOCK_STATUS."PART_ID", 
    PART_MANAGER."COMM_CODE", 
    PART_MANAGER."PART_DESC",
    STOCK_STATUS."ON_HAND_QTY",
    COALESCE(SUM(JOB_MANAGER."BALANCE_DUE"), 0) AS "BAL_DUE"
FROM 
    STOCK_STATUS
JOIN 
    PART_MANAGER
    ON STOCK_STATUS."PART_ID" = PART_MANAGER."PART_ID"
JOIN 
    JOB_MANAGER
    ON STOCK_STATUS."PART_ID" = JOB_MANAGER."PART_ID" 
    AND JOB_MANAGER."STORES_CODE" = 'EU'  
WHERE 
    STOCK_STATUS."STORES_CODE" = 'EU'
GROUP BY 
    STOCK_STATUS."PART_ID", 
    PART_MANAGER."COMM_CODE", 
    PART_MANAGER."PART_DESC",
    STOCK_STATUS."ON_HAND_QTY"

 ``` 
## ⚙️ Inventory Levels by Commodity  

This query retrieves the On-Hand Quantities for Parts with specific commodity code.  

```
SELECT  
  STOCK_STATUS."PART_ID",  
  STOCK_STATUS."ON_HAND_QTY",  
  PART_MANAGER."COMM_CODE"  
FROM  
  STOCK_STATUS  
JOIN  
  PART_MANAGER ON PART_MANAGER."PART_ID" = STOCK_STATUS."PART_ID"  
WHERE  
  PART_MANAGER."COMM_CODE" IN ('PCB', 'CABLE', 'OPTIC');

