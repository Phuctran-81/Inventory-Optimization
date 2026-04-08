# Reorder Point Setup Model
## I. Overall
### 1. Problem:
The goal of this project is to building a  model establishing inventory reorder point. The model will balance between ensuring the product availability and minizing carrying cost.
### 2. Action:
#### A. Buil the Reorder Point Setup Model
##### a. Categorizing the store-item
**Purpose:** The primary goal of the store-item classification is to prioritize management focus and capital allocation. By segmenting my store-item, I can apply different "Service Level" targets (Z-scores) to specific categories. This strategy approach ensures high availability for critical, high-value items (Class A) while preventing over-investment in slow-moving stock (Class C). \
**Method:** I categorized the store-item by sorting store-items by revenue in descending order to calculate the running percentage of total company revenue.
- Class A (High Value): The group of items generating 80% of revenue.
- Class B (Intermediate): store-Item generating the next 15% of revenue.
- Class C (Low Value): Items generating only the final 5% of revenue.

##### b. Seasonality-Adjusted ROP Model
Through the data exploratory analysis, I found that the sales is seasonal. Averagely, sales of monday is lower than the average week sales 20.84% while sales of saturdays and mondays are higher than the average week sales 12.46% and 18.40% respectively.\
From that result, i build a formula for establishing reorder point called Seasonality-Adjusted ROP model: \
`ROP = (Baseline 30d * Seasinality Index) + (Z * 1.25 * MAE) `
<img width="1273" height="82" alt="image" src="https://github.com/user-attachments/assets/4b2ad83f-14cd-4085-a37c-e99b11f8dc27" />


- **ROP:**
- **Baseline 30d:**
- **Seasonality Index:**
- **Z:**
- **MAE:**
##### c. Traditional ROP Model
Moreover, i also calculate the rop based on the formular called traditional ROP model: \
`ROP = Baseline 30d + Z * (Standard Deviation 30d)`
<img width="1172" height="68" alt="image" src="https://github.com/user-attachments/assets/e5ec7d58-0b31-441a-8b69-345be0aceecc" />




- **ROP:**
- **Baseline 30d:**
- **Z:**
- **Standard Deviation 30d:**
##### d. Promotion ROP model
**Promotion Lift Factor**
**Purpose:** To quantify the actual increase in sales volume during promotional events versus the standard 30-day baseline, ensuring future stock levels are optimized for high-traffic days. \
Lift factor is calculated for each specific item in each store by the formula:
<img width="1011" height="60" alt="image" src="https://github.com/user-attachments/assets/4fe546d6-0b15-4729-a40c-32f1d6d62d49" />

**Promotion ROP Model** \
After calculating Lift factor, i use the following formula for establishing Promotion ROP: 
<img width="1041" height="66" alt="image" src="https://github.com/user-attachments/assets/cb3b8354-ad4e-4982-b651-7260453f90ea" />





#### B. Success Level Backtesting
**Purpose:** Validate the reliability of the **Seasonality-Adjusted Reorder Point (ROP)** model against historical demand and compare its efficiency to the traditional ROP model. 

**Methodology:**
- **Backtest Duration:** the last 12-month data of dataset.
- **Scope:** 50 items across 10 stores (500 unique item-store).
- **Logic:** Simulated weekly ordering cucles by comparing forecasted ROP against actual weekly demand over the final year of the 5-year dataset.
### 3. Results:
#### a. Seasonality-Adjusted ROP model
- The backtest is implemented in total 26,474 weeks (across 50 items in 10 stores), there are **26,245 overstock weeks** with the **average excess rate at 20.39%** and **229 outstock weeks** with the **average shortage rate at 2.24%**. 
- The Seasonality-Adjusted ROP model satisfies the **service level of 99%** which is similar to Traditional ROP model while simultaneously **reducing weekly excess inventory by 13%** compared to the traditional ROP method.
#### b. Promotion ROP model
- The backtest is implemented in 7,321 promotion date (across 50 items in 10 stores), there are .... overstock date with the average excess rate at ....% and .... outstock date with the average shortage rate at ...%.
### 4. Recommendations:
#### a. Adopting the Seasonality-Adjusted ROP model for inventory planning.
By applying category-specific safety stock levels, we can prioritize availability for high-performance items while minimizing carrying costsfor slower-moving stock.
#### b. Adopting store-item specific lift factor
By applying store-item specific lift factor for calculating and adding 10% tactical safety buffer, this can help mitigate the risk of abnormal demand surges, specifically addressing the current 3.92% shortage rate during previous promotions.
