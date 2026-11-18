<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
</head>
<body>

<h1>End-to-End Retail Data Engineering Pipeline in Microsoft Fabric</h1>

<p>An end-to-end data engineering and analytics project built on <strong>Microsoft Fabric</strong> implementing the <strong>Medallion Architecture (Bronze → Silver → Gold)</strong>. The pipeline ingests multi-format retail data from Azure Data Lake Storage (ADLS Gen2), cleans and standardizes messy records using PySpark, aggregates product-level KPIs into Delta tables, and delivers executive insights via Power BI.</p>

<hr>

<h2>🏗️ Architecture & Data Flow</h2>

<pre>
[ADLS Gen2: Source]             [Microsoft Fabric Pipeline]             [OneLake Lakehouse (Delta)]             [Analytics & BI]
 ├── orders_data.csv      ───▶   Copy Activity (orders_data)     ───▶   Bronze Layer (Raw Parquet/Delta)
 ├── returns_data.xlsx    ───▶   Copy Activity (retrns_data)     ───▶   Silver Layer (PySpark Cleansing)  ───▶   SQL Analytics Endpoint
 └── inventory_data.json  ───▶   Copy Activity (inventory_data)  ───▶   Gold Layer (Product KPIs)         ───▶   Power BI Dashboard (.pbix)
</pre>

<hr>

<h2>🛠️ Tech Stack</h2>

<ul>
    <li><strong>Cloud & Lakehouse:</strong> Microsoft Fabric, OneLake, Delta Lake</li>
    <li><strong>Data Orchestration:</strong> Fabric Data Pipelines (Copy Data activities with sequential dependencies)</li>
    <li><strong>Data Processing:</strong> Apache Spark / PySpark (<code>synapse_pyspark</code>), Spark SQL</li>
    <li><strong>Data Storage Formats:</strong> CSV, Excel (<code>.xlsx</code>), JSON, Snappy-compressed Parquet, Delta Tables</li>
    <li><strong>Analytics & Visualization:</strong> T-SQL (Fabric SQL Analytics Endpoint), Power BI Desktop</li>
</ul>

<hr>

<h2>📂 Repository Structure</h2>

<ul>
    <li><code>pipelines/Pipeline_Retail_Project_2.json</code> — Exported ARM/Fabric pipeline template orchestrating multi-source ingestion into the Bronze Lakehouse folder.</li>
    <li><code>pipelines/manifest_2.json</code> — Pipeline metadata, visual layout SVG, and linked service dependencies.</li>
    <li><code>notebooks/</code> — PySpark notebook handling Bronze table registration, Silver data cleansing, and Gold KPI aggregations.</li>
    <li><code>sql/SQL query 1_2.sql</code> — SQL queries executed against the Fabric SQL Analytics Endpoint to validate and query Gold KPIs.</li>
    <li><code>powerbi/</code> — Power BI (<code>.pbix</code>) report connected to <code>gold_product_kpis</code>.</li>
</ul>

<hr>

<h2>🔄 Pipeline & Medallion Layer Breakdown</h2>

<h3>1. Ingestion Layer (Fabric Data Pipeline)</h3>
<p>The <code>Pipeline_Retail_Project</code> orchestrates three sequential <strong>Copy Data</strong> activities from Azure Blob Storage (<code>source</code> container) into the <code>Files/Bronze</code> directory of the <code>medallion</code> Lakehouse as Snappy-compressed Parquet files:</p>
<ol>
    <li><code>orders_data</code>: Ingests <code>orders_data.csv</code> (comma-delimited text).</li>
    <li><code>retrns_data</code>: Triggered upon success of <code>orders_data</code>; extracts <code>Sheet1</code> from <code>returns_data.xlsx</code>.</li>
    <li><code>inventory_data</code>: Triggered upon success of <code>retrns_data</code>; ingests <code>inventory_data.json</code>.</li>
</ol>

<h3>2. Bronze Layer (Raw Ingestion)</h3>
<ul>
    <li>Reads raw Parquet files from OneLake (<code>Files/Bronze/</code>) and persists them as unmanaged/managed Delta tables (<code>bronze_orders</code>, <code>bronze_returns</code>, <code>bronze_inventory</code>).</li>
    <li>Promotes the embedded header row in <code>bronze_returns</code> (originating from Excel ingestion) using RDD index filtering.</li>
</ul>

<h3>3. Silver Layer (Data Cleansing & Standardization)</h3>
<p>Applies schema enforcement, regex parsing, and deduplication across all three datasets:</p>
<ul>
    <li><strong><code>silver_orders</code>:</strong> Standardizes column names; converts written quantities (<code>"one"</code>, <code>"two"</code>, <code>"three"</code>) to integers; parses 6 inconsistent date formats (<code>yyyy/MM/dd</code>, <code>dd-MM-yyyy</code>, <code>MM/dd/yyyy</code>, etc.); strips currency symbols; validates email syntax via regex; cleans special characters from shipping addresses; and deduplicates on <code>order_id</code>.</li>
    <li><strong><code>silver_returns</code>:</strong> Extracts numeric <code>order_id</code> values from float-formatted Excel strings (e.g., <code>1001.0</code>); normalizes multi-currency strings (<code>INR</code>, <code>USD</code>, <code>Rs.</code>, <code>₹</code>, <code>$</code>); standardizes <code>refund_status</code> and <code>pickup_address</code>; and deduplicates on <code>return_id</code>.</li>
    <li><strong><code>silver_inventory</code>:</strong> Converts text stock counts (<code>"twenty"</code>, <code>"twenty five"</code>, <code>"25 units"</code>) to integers; standardizes boolean availability flags (<code>YES</code>/<code>Y</code>/<code>True</code> → <code>true</code>); cleans warehouse codes and cost prices; and deduplicates on <code>product_id</code>.</li>
</ul>

<h3>4. Gold Layer (<code>gold_product_kpis</code>)</h3>
<p>Creates a unified <code>product_key</code> across Orders, Returns, and Inventory to compute business-ready metrics:</p>
<ul>
    <li><strong>Sales Metrics:</strong> <code>Total_Orders</code>, <code>Unique_Customers</code>, <code>Total_Quantity_Sold</code>, <code>Total_Revenue</code>, <code>Avg_Order_Value</code>.</li>
    <li><strong>Returns Metrics:</strong> Links returns primarily via <code>order_id</code> (with fallback to <code>product_key</code>) to calculate <code>Total_Returns</code>, <code>Returned_Orders</code>, and <code>Return_Rate_Pct</code>.</li>
    <li><strong>Inventory Metrics:</strong> <code>Total_Stock</code>, <code>Inventory_Records</code>, and <code>Available_Inventory_Records</code>.</li>
</ul>

<hr>

<h2>📊 Dashboard Preview</h2>

<blockquote>
    <p><em>https://github.com/saikrishna64/Retail360-End-to-End-Medallion-Architecture-Executive-Analytics-Suite/blob/main/Screenshot%202026-11-18%20235251.png</em></p>
</blockquote>

<hr>

<h2>🚀 How to Reproduce</h2>

<ol>
    <li><strong>Provision Storage & Lakehouse:</strong> Create a Lakehouse named <code>medallion</code> in a Microsoft Fabric workspace and upload the raw source files (<code>orders_data.csv</code>, <code>returns_data.xlsx</code>, <code>inventory_data.json</code>) to an ADLS Gen2 container.</li>
    <li><strong>Import Pipeline:</strong> Import <code>Pipeline_Retail_Project_2.json</code> into Fabric Data Factory and rebind the ADLS Gen2 and Lakehouse connection parameters.</li>
    <li><strong>Run ETL Notebook:</strong> Attach the <code>medallion</code> Lakehouse to the PySpark notebook and execute all cells to populate <code>bronze_*</code>, <code>silver_*</code>, and <code>gold_product_kpis</code> Delta tables.</li>
    <li><strong>Explore & Visualize:</strong> Query <code>dbo.gold_product_kpis</code> via the SQL Analytics Endpoint or open the <code>.pbix</code> file in Power BI Desktop and update the data source connection to your Fabric Lakehouse/SQL endpoint.</li>
</ol>

</body>
</html>
