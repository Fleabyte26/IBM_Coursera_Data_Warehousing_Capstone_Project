# IBM_Coursera_Data_Warehousing_Capstone_Project
Help to navigate this lab

IBM Data Warehouse Engineer Capstone — Full Step-by-Step Tutorial with Insights
Purpose: This tutorial guides students through all 16 steps of the IBM Data Warehouse Engineer Capstone. It includes fixes, tips, screenshots, and insights about common lab confusion so learners don’t get lost.

Step 0 — General Notes and Observations
Lab sessions are temporary: Every new session creates a fresh environment; saved files may not persist.


Files often appear in .theia/ — always move CSVs and scripts to project root:


mv .theia/<filename> ./
Instructor instructions are inconsistent:


Lab steps, capstone tasks, and screenshots often reference different names for the same table or file.


Students may blindly follow instructions without verifying database names or table structures. Always check:


SHOW DATABASES;
SHOW TABLES;
\d <table_name>  -- PostgreSQL
Hosts and passwords differ:


MySQL host: localhost


PostgreSQL host: postgres (Docker container)


Replace placeholders in setup scripts and ETL.sh.


Key insight: Students often fail not because SQL is hard, but because instructions jump between labs and capstone tasks, use inconsistent table names, or omit critical setup details.



Step 1 — OLTP MySQL Setup
Download CSV: oltpdata.csv


Download script: setupmysqldb.sh → add MySQL password


Run:


bash setupmysqldb.sh
Verify table:


USE sales;
SELECT * FROM sales_data WHERE rowid IN (1301, 2605);
Screenshot: importdata.jpg


Insight: The lab instructions sometimes refer to a sales_db database, but the environment uses sales. Always check SHOW DATABASES;.

Step 2 — Dimension Tables (ERD)
ERD tool → create:


softcartDimDate


softcartDimCategory


softcartDimItem


softcartDimCountry


Screenshot:


softcartDimDate.jpg


dimtables.jpg


Insight: Field names may differ from instructions. Verify types in the ERD tool.

Step 3 — Fact Table and Relationships
Create softcartFactSales table


Add relationships: one-to-one, one-to-many


Screenshots:


softcartFactSales.jpg


softcartRelationships.jpg


Insight: Relationships are often skipped or misreferenced in instructions. Draw all tables before proceeding.

Step 4 — Load OLTP Data
Load data using SQL if phpMyAdmin import fails.


Verify:


SELECT * FROM softcartFactSales LIMIT 10;

Step 5 — Reporting Queries
Task 1: Grouping Sets → groupingsets.png


Task 2: Cube Query → cube.png


Task 3: MQT → total_sales_per_country.png


Insight: Instructions sometimes describe a “totalsales” column that is missing. Use price * quantity if needed.

Step 6 — Looker Studio
Task 1: Import ecommerce.csv → dataimport.jpg


Task 2: Show first 10 rows → first10rows.jpg


Task 3: Create data source → datasource.jpg


Insight: CSV files may have different columns than instructions; adjust fields accordingly.

Step 7 — Dashboards
Task 4: Line chart → linechart.jpg


Task 5: Pie chart → piechart.jpg


Task 6: Bar chart → barchart.jpg


Insight: If your CSV lacks totalsales, calculate using price * quantity. Lab instructions do not always match provided data.

Step 8 — ETL Setup
Move CSVs to project root:


mv .theia/sales_newdata.csv ./sales_newdata.csv
Download and edit setup scripts:


setupmysqldb.sh → MySQL password


setuppostgresqldb.sh → host = postgres, Postgres password


Start MySQL and PostgreSQL servers in IDE.



Step 9 — ETL Task 1
Download ETL.sh


wget <ETL.sh URL>
Update passwords in ETL.sh


Run ETL.sh:


bash ETL.sh
Screenshot: extract_load_data.png


Insight: ETL.sh often contains placeholders (<replace ...>). Replace all with real SQL and ensure hostnames and passwords match your environment.

Step 10 — ETL Task 2 (DimDate)
INSERT INTO DimDate (dateid, year, month, day)
SELECT DISTINCT
      dateid,
      dateid / 10000 AS year,
      (dateid / 100) % 100 AS month,
      dateid % 100 AS day
FROM sales_data;
Screenshot: DimDate.png


Handles integer dateid (common SN Labs issue).



Step 11 — ETL Task 3 (FactSales)
INSERT INTO FactSales (id, dateid, itemid, categoryid, countryid, quantity, total)
SELECT ROW_NUMBER() OVER () AS id,
      dateid,
      itemid,
      categoryid,
      countryid,
      quantity,
      quantity * price AS total
FROM sales_data;
Screenshot: FactSales.png


Insight: Some labs fail due to mismatched column names; use ROW_NUMBER() to generate id.

Step 12 — ETL Task 4 (Export CSV)
\COPY DimDate TO '/home/project/DimDate.csv' CSV HEADER;
\COPY FactSales TO '/home/project/FactSales.csv' CSV HEADER;
Verify files exist:


ls -l /home/project/DimDate.csv /home/project/FactSales.csv

Step 13 — Cron Job
Automate ETL.sh every 24 hours:


crontab -e
# Add line:
0 0 * * * /home/project/ETL.sh
Screenshot: schedule_job.png



Step 14 — Troubleshooting Tips
Always start MySQL/PostgreSQL servers first.


Move CSVs out of .theia/.


Replace all placeholders in scripts.


Check column names in tables: \d FactSales / \d DimDate.


DimDate.dateid is integer → use arithmetic to extract year/month/day.


FactSales id may not exist → use ROW_NUMBER().


Instructions often jump between lab and capstone tasks — verify table names.


If totals missing → compute price * quantity.



Step 15 — Screenshots for Submission
importdata.jpg


softcartDimDate.jpg


dimtables.jpg


softcartFactSales.jpg


softcartRelationships.jpg


groupingsets.png


cube.png


total_sales_per_country.png


dataimport.jpg


first10rows.jpg


datasource.jpg


linechart.jpg


piechart.jpg


barchart.jpg


extract_load_data.png


DimDate.png


FactSales.png


schedule_job.png



Step 16 — Key Insights on Lab Organization
Lab instructions are poorly organized and reference steps inconsistently.


Capstone tasks sometimes rename tables or columns from the original lab.


Students should verify table names, column names, and CSV content before running SQL or ETL scripts.


Many errors are environment setup issues (MySQL/PostgreSQL servers not running, files in .theia/, passwords not exported).


The tutorial’s approach: treat every lab step as an investigation, check outputs, and update scripts accordingly.





