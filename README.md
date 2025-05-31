# 🗄️Microsoft SQL Queries

This repository provides a comprehensive guide on connecting Power BI to a SQL Server database using custom SQL queries. It includes example queries, setup instructions, and best practices for efficient data loading and transformation.

Once the server and database are specified, the provided SQL commands facilitate seamless data retrieval and integration.

## ⚙️ Inventory Transactions
This SQL command retrieves the transactional data from the `PO_TRANSACTION` table for the past 91 days, counting back from today while filtering on records where `ACTION_DATE` is set to "I".
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
