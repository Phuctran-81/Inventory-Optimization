# Reorder Point Setup Model
## I. OVERALL
## 1. Executive Summary
**Objective:** Engineered an automated Inventory Reorder Point (ROP) system for 500 unique store-item combinations to solve the "Overstock vs. Stock-out" paradox. Developed a **Seasonally-Adjusted Engine** that maintains a **99% Service Level** while reducing excess inventory by **2.8% compared** to traditional rolling-average models.
## 2. Key Project Insights
- **High-Precision Seasonality:** Identified the differencerate in sales between weekdays and the weekly average, which traditional model miss, allowing for more accurate inventory calculations.
- **Automated Promotion Detection:** Developed a statistical outlier detection model (Basline 30day + 2 standard deviation) to flag missing promotion dates and calculate a **Promotion Lift Factor**, reducing shortage risks during high-traffic events.
- **Strategic Capital Allocation:** Implemented ABC Revenue Classification to priortize Class A items (80% of revenue), ensuring 90% daily availability for top-performers while minimizing the "carrying cost" of slow-moving stock.
- **Validated Reliability:** Conducted a rigorous 12-month backtest across 26,474 data points to prove model efficiency.
-----------------------------
## II. ANALYTICAL FRAMEWORK: A 6-STAGE DEEP DIVE
## 1. Ask
The goal of this project is to build a  model establishing inventory reorder point. The model will balance between ensuring the product availability and minizing carrying cost.
## 2. Prepare
### Data Sourcing:
The raw dataset was originally sourced from Kaggle with **913,000 rows** (Store Item Demand Forecasting Challenge). For this analysis, the data has been ingested into a SQL Server environment. This setup allows for the high-performance processing required to handle the 5-year history of daily sales across 10 stores and 50 items.
### Data Storage
A tiered strategy in MS SQL Server was used to ensure integrity:
- Bronze Layer: Contains immutable raw CSV imports as a “Single Source of Truth”.
- Silver Layer: Contains processed datasets (remove duplicate, error data, add new column ….)
- Gold Layer: Contains cleaned, validated dataset for staged analysis.
  
## 3. Process (Data Engineering & Cleaning)
<img width="810" height="431" alt="image" src="https://github.com/user-attachments/assets/ad1d22a8-d48d-4379-99fe-c259cbf1e582" />

### A. Bronze to Silver
#### Data integrity and consistency verification
Data integrity Verification: Used `COUNT(DISTINCT item)` grouped by store to ensure that no store lost product.
Data consistency Verification:
- Verifying data duplication by checking the composite key of `date` + `store` + `item` has zero duplicates, ensuring that the dataset will be accurate and not double-counted.
- Checking data errors to ensure no null values and no zero sales. Removed one row have zero sales.
#### Data enrichment
New drived columns were created to unlock deeper insight:
- Temporal Feature: calculating new column such as week_calendar to enable calculate the weekly inventory, extracting weekday_name and weekday_number to enable calculate the seasonality of weekday sales, creating new column dwh_create_date to manage data daily updated.
- Primary Key: create primary_key to manage data.
### B. Silver to Gold
Create two new table gold.dim_seasonality_index and gold.dim_abc_category to optimize the performance when updating new data.
#### a. gold.dim_seasonality_index table *([CLick to jump SQL Script](https://github.com/Phuctran-81/Reorder-Point-Setup-Model/blob/main/scripts/gold/proc_load_gold_dim_seasonality_index.sql))*
- Calculating the average of the ratio between weekday sales and average week sales to get seasonality index of each weekday.
- Performed ROW_NUMBER () to create primary key for this table.
#### b. gold.dim_category *([Click to jump to SQL Script](https://github.com/Phuctran-81/Reorder-Point-Setup-Model/blob/main/scripts/gold/proc_load_gold_dim_category.sql))*
- Performed Window function rolling-sum the percentage of revenue of each item at each store to cagorize the item.
- Performed ROW_NUMBER () to create primary key for this table.
#### c. Gold.fact_train *([Click to jump to SQL Script](https://github.com/Phuctran-81/Reorder-Point-Setup-Model/blob/main/scripts/gold/proc_load_gold_fact_train.sql))*
- Performed Window_function to calculate the 30 day average sales of each item at each store and the 30 day standard deviation of sales of each item at each store.
- Flagging the promotion day by detecting days where sales > 30 day average sales + 2 * 30 day standard deviation.
- Performed JOIN function to get foreign key from gold.dim_category and gold.dim_seasonality_index table.
- Finally, filter out 30 first days of each item at each store (15.000 rows), that rows miss data to calculate the complete 30 day average sales and 30 day standard deviation.
<img width="1115" height="704" alt="image" src="https://github.com/user-attachments/assets/627239a8-78df-4026-a791-2c7ec1ccb8b0" />


## 4. ANALYZE:
To ensure the ROP was responsive to real-word volatility, the analysis focused on isolating  distinct demand signals:
- Systemic Seasonality: Calculated unique Seasonality Indices for 500 store-item combinations.
  + Identified a predictable 16.17% demand surge on Sundays, which was previously causing stock-outs in traditional models.
- ABC Segmentation:
  + Applied Pareto analysis to 5 years of revenue data, categorizing 500 store-item combinations into tiered priority to optimize capital allocation.
- Heuristic Event Detection:
  + Implemented an outlier detection algorithm (Z > 2.0) to reconstruct missing promotion metadata.
  + Quatified a Promotion Lift Factor to automatically scale reorder points during high-traffic events, reducing shortage rate to 3.92%.
- Performance Validation (Backtesting):
  + Finding: The seasonality-adjusted model maintained a 99% service level while achieving a 2.82% reduction in excess inventory compared to the baseline method.
## 5. SHARE:
#### a. Seasonality-Adjusted ROP model
- The backtest is implemented in total 26,474 weeks (across 50 items in 10 stores), there are **26,275 overstock weeks** with the **average excess rate at 21.77%** and **199 outstock weeks** with the **average shortage rate at 2.07%**.
<img width="1607" height="751" alt="image" src="https://github.com/user-attachments/assets/4f88cf6c-e9ac-443e-bb55-9706c2f677e2" />
The seasonality-adjusted ROP model
- The Seasonality-Adjusted ROP model satisfies the **service level of 99%** which is similar to Traditional ROP model while simultaneously **reducing weekly excess inventory by 2.82%** compared to the traditional ROP method.
<img width="1602" height="745" alt="image" src="https://github.com/user-attachments/assets/2965561c-cbf9-4674-8ea1-ebaf283f0c37" />
The traditional ROP model

#### b. Promotion ROP model
- The backtest is implemented in 7,321 promotion date (across 50 items in 10 stores), there are 6,799 overstock dates with the average excess rate at 14.91% and 522 outstock dates with the average shortage rate at 3.92%.
<img width="1604" height="743" alt="image" src="https://github.com/user-attachments/assets/aa3f1532-742a-4db7-bea9-8473b42380b9" />
- After backtesting Promotion ROP i decide to plus 10% Promotion ROP to cover abnormal demand surge dates. The results show that the number of outstock dates decrease significantly from 522 outstock date of old model to 48 outstock dates. The shortage rate also drop to 2.93%, however the excess rate higher than olde model approximately 10%.
<img width="1601" height="745" alt="image" src="https://github.com/user-attachments/assets/d2f04fa8-faa3-44dd-a2ba-3016ff8772b2" />



## 4. ACT:
1. Full Deployment: Transition 500 store-item profiles to the Seasonality-Adjusted ROP to immediately capture the 2.8% capital efficiency gain.
2. Adaptive Promotion Buffers: Implement a 10% tactical safety buffer on top of the calculated Left% to mitigate the 3.92% shortage risk identified during the backtest.
#### a. Adopting the Seasonality-Adjusted ROP model for inventory planning.
By applying category-specific safety stock levels, we can prioritize availability for high-performance items while minimizing carrying costs for slower-moving stock.
#### b. Adopting store-item specific lift factor
By applying store-item specific lift factor for calculating ROP and adding 10% tactical safety buffer, this can help mitigate the risk of abnormal demand surges, specifically addressing the current 3.92% shortage rate observed during previous promotions.
# III. Dataset and Tools used: 
- Datasets:
  + [train](https://github.com/Phuctran-81/Reorder-Point-Setup-Model/releases/tag/V1.0.0) is dataset used to build the model.
  + [test](https://github.com/Phuctran-81/Reorder-Point-Setup-Model/releases/tag/V1.0.0) is dataset contained the final year data and ROP using for testing the success level.
- Database: SQL Server Management Studio 21.
- Data visualization: Access the dashboard at [Tableau Public].
