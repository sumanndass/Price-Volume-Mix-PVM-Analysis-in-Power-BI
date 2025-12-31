# 📊 Price–Volume–Mix (PVM) Analysis in Power BI
## Revenue Change Explained | Growth Quality | Strategic Risk & Opportunity
Mix Analysis explains why revenue changed between two periods by splitting the change into three clear drivers: 1️⃣ Price Effect, 2️⃣ Volume Effect, 3️⃣ Mix Effect.

## 🔍 What This Project Solves
- Most revenue dashboards answer **“what changed”**.
- Executives actually ask **“why did it change, is it good, and where should we act?”**
  <enbr>
- This Power BI model answers:
  - Why did revenue grow or decline?
  - Was growth driven by **real demand** or **pricing actions**?
  - Did **product mix create value or silently dilute it**?
  - Which **region–customer–product** actually moved the needle?
  - Where is growth **strategically risky vs sustainable**?
- This is achieved through a fully decomposed Price–Volume–Mix framework, implemented in DAX and visualized across 3 analytical layers.

## 🧱 Data Model Overview
- Fact Table
  - [FactSales](https://github.com/sumanndass/Price-Volume-Mix-PVM-Analysis-in-Power-BI/blob/main/Mix_Analysis_5_Year_Sample_Data_3000_Rows.xlsx) (cleaned Excel data)
    - Date
    - Region
    - Customer_Segment
    - Product
    - Quantity
    - Actual_Price
- Date Dimension
  ```dax
  DimDate = 
  ADDCOLUMNS(
      CALENDARAUTO(),
      "Year", YEAR([Date]),
      "Month", FORMAT([Date], "MMM"),
      "MonthNo", MONTH([Date]),
      "YearMonth", FORMAT([Date], "YYYY-MM")
  )
  ```
  - 📌 Enables:
    - YoY comparisons
    - Trend analysis
    - Strategic period slicing

## 📐 Base Measures (Foundation Layer)
- These are the economic primitives of the model.
  - Total Quantity
    ```dax
    _Total Quantity = SUM(FactSales[Quantity])
    ```
  - Total Revenue
    ```dax
    _Total Revenue = SUMX(FactSales, FactSales[Quantity] * FactSales[Actual_Price])
    ```
  - Average Price
    ```dax
    _Avg Price = DIVIDE([_Total Revenue], [_Total Quantity])
    ```
- 📌 Why this matters:
  - PVM analysis cannot work without separating price and quantity
  - Average price acts as the bridge between value and volume

## ⏳ Time Intelligence Measures
- These create the baseline comparison (Last Year).
  ```dax
  _Revenue LY =
  CALCULATE([_Total Revenue], SAMEPERIODLASTYEAR(DimDate[Date]))
  
  _Quantity LY =
  CALCULATE([_Total Quantity], SAMEPERIODLASTYEAR(DimDate[Date]))
  
  _Avg Price LY =
  CALCULATE([_Avg Price], SAMEPERIODLASTYEAR(DimDate[Date]))
  ```
- 📌 Interpretation:
  - “What would revenue look like if nothing changed except time?”

## 🔬 Core PVM Decomposition (The Heart of the Model)
- Revenue change is split into three mutually exclusive drivers.
- 1️⃣ **Price Effect**
  - Impact of price change, assuming last year’s volume
    ```dax
    _Price Effect = 
    VAR pricediff = [_Avg Price] - [_Avg Price LY]
    RETURN
        [_Quantity LY] * pricediff
    ```
  - 🧠 Business meaning:
    - Positive → Successful pricing power
    - Negative → Discounts, competitive pressure, or price erosion
- 2️⃣ **Volume Effect**
  - Impact of quantity change, assuming last year’s price
    ```dax
    _Volume Effect =
    VAR voldiff = [_Total Quantity] - [_Quantity LY]
    RETURN
        voldiff * [_Avg Price LY]
    ```
  - 🧠 Business meaning:
    - Positive → Real demand growth
    - Negative → Market slowdown, churn, or lost share
- 3️⃣ **Mix Effect**
  - Hidden impact caused by selling a different product mix
    ```dax
    _Mix Effect =
    VAR pricediff = [_Avg Price] - [_Avg Price LY]
    VAR voldiff = [_Total Quantity] - [_Quantity LY]
    RETURN
        IF([_Total Revenue] && [_Revenue LY], voldiff * pricediff)
    ```
  - 🧠 Business meaning:
    - Positive → Shift toward premium / high-margin products
    - Negative → Growth coming from economy or low-value products
  - 📌 **This is where most dashboards fail** — mix is invisible unless explicitly modeled.
- 🔁 **Revenue Change Validation**
  ```dax
  _Revenue Changes =
  IF([_Total Revenue] && [_Revenue LY],
     [_Total Revenue] - [_Revenue LY]
  )
  
  _PVM Check =
  [_Price Effect] + [_Volume Effect] + [_Mix Effect]
  ```
  - ✔ Ensures mathematical integrity of decomposition.
## 🧮 Waterfall Logic (Page 1 Visual Engine)
- **PMV Steps Table**
  | StepOrder | StepName      |
  | --------- | ------------- |
  | 1         | Revenue LY    |
  | 2         | Price Effect  |
  | 3         | Volume Effect |
  | 4         | Mix Effect    |
