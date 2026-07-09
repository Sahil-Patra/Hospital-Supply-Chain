# Healthcare Supply Chain & Inventory Analytics

## Project Overview
This project simulates a real-world healthcare supply chain scenario focused on optimizing inventory levels for critical medical devices and clinical supplies (similar to brands managed by Teleflex, such as Arrow™, Rüsch™, and LMA™). 

In a healthcare environment, stockouts can directly disrupt patient care, while overstocking unnecessarily ties up hospital capital. This project uses **MySQL** to perform operational, financial, and logistical analysis on raw hospital database exports, and **Power BI** to deliver interactive executive dashboards.

## Business Problems Addressed
1. **Clinical Stockout Risk**: Identifying high-demand items that have fallen below required safety stock levels.
2. **Logistics & Delivery Exposure**: Determining if the remaining days of stock (Days of Cover) will last until the vendor's next scheduled delivery.
3. **Capital Allocation**: Valuing the total working capital tied up in the warehouse and monitoring storage capacity utilization.
4. **Procurement Bottlenecks**: Highlighting high-cost clinical items with exceptionally long vendor lead times.

## Tech Stack
* **Database Engine**: MySQL Server & MySQL Workbench
* **Data Visualization**: Power BI Desktop
* **Data Source**: [Hospital Supply Chain Dataset (Kaggle)](https://www.kaggle.com/datasets/vanpatangan/hospital-supply-chain)

---

## Data Architecture & Cleaning Workflow
The raw dataset consists of two relational tables imported into a MySQL schema named `hospital_supply_chain`:
* `inventory_data`: Daily stock ledger, containing standard inventory metrics, item details, and vendor mappings.
* `vendor_data`: Operational performance indicators of suppliers, including delivery dates and lead times.

### Data Cleaning (SQL View)
To ensure reliable analytical reporting and correct data typing in Power BI, a SQL view (`clean_inventory_view`) was created to handle date parsing, text trimming, and number casting:

```sql
CREATE OR REPLACE VIEW clean_inventory_view AS
SELECT 
    STR_TO_DATE(Date, '%Y-%m-%d') AS record_date,
    Item_ID AS item_id,
    TRIM(Item_Type) AS item_type,
    TRIM(Item_Name) AS item_name,
    CAST(Current_Stock AS SIGNED) AS current_stock,
    CAST(Min_Required AS SIGNED) AS safety_stock,
    CAST(Max_Capacity AS SIGNED) AS max_capacity,
    ROUND(Unit_Cost, 2) AS unit_cost,
    ROUND(Avg_Usage_Per_Day, 2) AS avg_usage_per_day,
    CAST(Restock_Lead_Time AS SIGNED) AS lead_time_days,
    TRIM(Vendor_ID) AS vendor_id
FROM inventory_data;
```

---

## SQL Analytics & Key Business Insights

### 1. Operational Stockout Alerts
**Business Context:** Identifies critical medical supplies currently below safe operational limits, calculating units short and the immediate financial cost to replenish them.

* **Key Result:** Out of `[Insert total number of items, e.g., 500]` tracked SKUs, `[Insert number of stockout items, e.g., 42]` are currently below safety stock levels.
* **Capital Risk:** This represents an immediate supply shortfall value of **`$[Insert total replenishment cost, e.g., 45,200]`**.
* *SQL Query Used:* [See query 1 in /sql_queries/operational_stockouts.sql]

### 2. Supply Risk Windows (Days of Cover vs. Lead Time)
**Business Context:** Calculates "Days of Cover" (how many days until current inventory reaches zero) and compares it to vendor delivery timelines to flag items at high risk of running out before the replacement order arrives.

* **Logistics Warning:** **`[Insert number of High Risk items, e.g., 12]`** items were flagged as **"HIGH RISK: Order Immediately"** because their remaining inventory will be depleted before the vendor's next delivery.
* *SQL Query Used:* [See query 2 in /sql_queries/lead_time_exposure.sql]

### 3. Inventory Valuation & Storage Utilization
**Business Context:** Assesses how much capital is tied up in the warehouse by category and evaluates capacity limits to identify overstocked areas.

* **Financial Summary:** Total capital tied up in the warehouse is **`$[Insert total inventory valuation, e.g., 1.2M]`**.
* **Storage Space:** The warehouse is operating at an overall capacity utilization of **`[Insert total storage utilization %, e.g., 68%]`**, with the **`[Insert most bloated category, e.g., Surgical Supplies]`** category holding the largest share of tied-up capital at **`$[Insert category value, e.g., $450,000]`**.
* *SQL Query Used:* [See query 3 in /sql_queries/inventory_valuation.sql]

### 4. High-Value Lead Time Bottlenecks
**Business Context:** Isolates high-cost medical instruments (unit cost > $50) with long vendor lead times, pinpointing supplier inefficiencies that could threaten major clinical procedures.

* **Supplier Alert:** The longest lead time among high-cost items is **`[Insert max lead time, e.g., 14]`** days, associated with **`[Insert item name, e.g., Implant Kits]`** supplied by **`[Insert Vendor Name, e.g., Global Med Corp]`**.
* *SQL Query Used:* [See query 4 in /sql_queries/high_cost_bottlenecks.sql]

---

## Power BI Dashboard
The SQL views were imported directly into Power BI Desktop to build an executive-level interactive report.

`[Insert a high-quality screenshot of your Power BI Dashboard here]`

### Dashboard Highlights:
* **Interactive Slicers:** Users can filter inventory health and valuation by Hospital Region, Item Type, or Vendor Name.
* **Risk Categorization Matrix:** A color-coded table dynamically highlights SKUs categorized as "HIGH RISK" to enable immediate ordering decisions.
* **Actionable Vendor Cards:** Displays direct information on vendor names and their `Next_Delivery_Date` to help inventory planners coordinate expedited shipments.

---

## How to Replicate This Project
1. **Download Raw Data**: Download the datasets from the [Kaggle Dataset Link](https://www.kaggle.com/datasets/vanpatangan/hospital-supply-chain).
2. **Database Setup**:
   * Open MySQL Workbench.
   * Create a schema named `hospital_supply_chain`.
   * Use the **Table Data Import Wizard** to import the unzipped CSVs into tables named `inventory_data` and `vendor_data`.
3. **Run SQL Scripts**: Run the SQL scripts located inside the `/sql_queries/` folder of this repository to clean the tables and generate your analytical reports.
4. **Connect Power BI**:
   * Open Power BI Desktop.
   * Get Data -> MySQL Database -> Server: `localhost` -> Database: `hospital_supply_chain`.
   * Select `clean_inventory_view` and `vendor_data`.
   * Create relationships on `vendor_id` and begin building your visuals.
