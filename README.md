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
  ```dax
  _Waterfall Value =
  VAR Step = SELECTEDVALUE('PMV Steps'[StepName])
  RETURN
  SWITCH(
      Step,
      "Revenue LY", [_Revenue LY],
      "Price Effect", [_Price Effect],
      "Volume Effect", [_Volume Effect],
      "Mix Effect", [_Mix Effect]
  )
  ```
  - 📌 Enables a **true CFO-grade waterfall**.
## 🧠 Page 1 — Executive Revenue Narrative
- **Dynamic Title Measure**
  ```dax
  _Page 1 Title = 
  VAR revchng = [_Revenue Changes]
  VAR revchngtext = FORMAT(DIVIDE(revchng, 100000), "₹#,##0.00") & " Lakh"
  VAR ispositiveRev = IF(revchng>0, "grew", "declined")
  VAR pricetext = FORMAT(DIVIDE([_Price Effect], 100000), "₹#,##0.00") & " Lakh"
  VAR mixtext = FORMAT(ABS(DIVIDE([_Mix Effect], 100000)), "₹#,##0.00") & " Lakh"
  VAR PrimaryGrowthDriver =
      VAR Price = ABS([_Price Effect])
      VAR Volume = ABS([_Volume Effect])
      VAR Mix = ABS([_Mix Effect])
      RETURN
          SWITCH(
              TRUE(),
              Price >= Volume && Price >= Mix && [_Price Effect] < 0, "price decreases",
              Price >= Volume && Price >= Mix, "price increases",
              Volume >= Price && Volume >= Mix && [_Volume Effect] < 0, "lower demand",
              Volume >= Price && Volume >= Mix, "higher demand",
              "product mix changes"
          )
  VAR MixDirection =
      IF (
          [_Mix Effect] < 0,
          "diluted",
          "supported"
      )
  VAR MixCauseText =
      IF (
          [_Mix Effect] < 0,
          "due to higher Economy product share",
          "supported by premium product mix"
      )
  RETURN   
      "Revenue " & ispositiveRev & " " & revchngtext &
      ", driven primarily by " & PrimaryGrowthDriver &
      ", while product mix " & MixDirection & " " & mixtext & " " &
      MixCauseText & "."
  ```
  - This generates insights like:
    - “Revenue declined ₹2.49 Lakh, driven primarily by lower demand, while product mix diluted ₹0.03 Lakh due to higher Economy product share.”
  - 📌 Why this matters:
    - No analyst explanation required
    - Board-ready narrative
    - Automatically adapts to slicers
## 🧭 Page 2 — Who Created Value vs Who Diluted It
- **Drill Logic**
  - Uses ```SUMMARIZECOLUMNS + TOPN``` to identify:
    - Top Region
    - Top Customer Segment
    - Top Product
  - Based on **absolute revenue impact**, not just growth %.
    ```dax
    _Page 2 Title = 
    VAR selyear = SELECTEDVALUE(DimDate[Year])
    VAR isgrowth = IF([_Revenue Changes]<0, "declination", "growth")
    VAR selreg = 
        VAR treg =
            SUMMARIZECOLUMNS(
                DimDate[Year],
                FactSales[Region],
                "revchng", ABS([_Revenue Changes])
            )
        RETURN
            MAXX(
                TOPN(
                    1,
                    treg,
                    [revchng],
                    DESC
                ),
                FactSales[Region]
            )
    VAR selcus = 
        VAR tcus =
            SUMMARIZECOLUMNS(
                DimDate[Year],
                FactSales[Customer_Segment],
                "revchng", ABS([_Revenue Changes])
            )
        RETURN
            MAXX(
                TOPN(
                    1,
                    tcus,
                    [revchng],
                    DESC
                ),           
                FactSales[Customer_Segment]
            )
    VAR selprod = 
        VAR tprod =
            SUMMARIZECOLUMNS(
                DimDate[Year],
                FactSales[Product],
                "revchng", ABS([_Revenue Changes])
            )
        RETURN
            MAXX(
                TOPN(
                    1,
                    tprod,
                    [revchng],
                    DESC
                ),
                FactSales[Product]
            )
    VAR MixInsightText =
        IF (
            [_Mix Effect] < 0,
            "while product mix diluted overall revenue",
            "with product mix supporting revenue growth"
        )
    RETURN
        "In " & selyear & ", revenue " & isgrowth & " was concentrated in " & selreg & "-" & selcus & "-" & selprod & " products," & UNICHAR(10) & MixInsightText
    ```
    - 🧠 Insight example:
      - “In 2022, revenue growth was concentrated in East-SMB-Premium A products, with product mix supporting revenue growth.”
  - 📌 This answers:
    - Where to invest
    - Which segments are fragile
    - Who is driving real value
## 🧠 Page 3 — Strategic Risk & Opportunity Lens
