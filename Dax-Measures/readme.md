# DAX Measures — Shield Insurance Analysis

Documentation of the DAX measures powering the Shield Insurance dashboard, grouped by what they're used for.

----

## Core Measures

```dax
Total_Revenue = SUM(fact_premiums[final_premium_amt(INR)])
```

```dax
Total_customers = COUNT(dim_customer[customer_code])
```

```dax
Policy_Count = COUNT(dim_policies[policy_id])
```

```dax
Total_Policies = DISTINCTCOUNT(fact_premiums[policy_id])
```

```dax
Total Premium Rows = COUNTROWS(fact_premiums)
```

```dax
Total Expected Settlement = SUM(fact_premiums[Total Expected Settlement (Corrected)])
```

```dax
Expected Settlement % = AVERAGE(dim_customer[Settlement Rate])
```

----

## Daily Averages

```dax
Daily_Revenue = AVERAGEX(VALUES(dim_date[date]),[Total_Revenue])
```

```dax
Daily_Customers = AVERAGEX(VALUES(dim_date[date]), [Total_customers])
```

----

## Previous Month Comparisons

```dax
Revenue PM = 
CALCULATE(
    [Total_Revenue],
    PREVIOUSMONTH('dim_date'[date])
)
```

```dax
Total_Revenue_PM = 
IF(
    HASONEVALUE(dim_date[mmm_yy]),
    CALCULATE([Total_Revenue], PREVIOUSMONTH(dim_date[date])),
    BLANK()
)
```

```dax
Total_customers_PM = 
IF(
    HASONEVALUE(dim_date[mmm_yy]),
    CALCULATE([Total_customers], PREVIOUSMONTH(dim_date[date])),
    BLANK()
)
```

```dax
Daily_Revenue_PM = 
IF(
    HASONEVALUE(dim_date[mmm_yy]),
    CALCULATE([Daily_Revenue], PREVIOUSMONTH(dim_date[date])),
    BLANK()
)
```

```dax
Daily_Customers_PM = 
IF(
    HASONEVALUE(dim_date[mmm_yy]),
    CALCULATE([Daily_Customers], PREVIOUSMONTH(dim_date[date])),
    BLANK()
)
```

```dax
Policies PM = 
CALCULATE(
    [Total Premium Rows],
    PREVIOUSMONTH('dim_date'[date])
)
```

```dax
Previous Month Policy Count = 
CALCULATE(
    [Policy_Count],
    DATEADD(dim_date[date], -1, MONTH)
)
```

----

## Month-over-Month Growth %

```dax
Revenue MoM % = 
DIVIDE(
    [Total_Revenue] - [Revenue PM],
    [Revenue PM],
    0
)
```

```dax
Total_Revenue_MoM% = 
IF(
    HASONEVALUE(dim_date[mmm_yy]),
    DIVIDE([Total_Revenue] - [Total_Revenue_PM], [Total_Revenue_PM], 0),
    BLANK()
)
```

```dax
Total_customers_MoM% = 
IF(
    HASONEVALUE(dim_date[mmm_yy]),
    DIVIDE([Total_customers] - [Total_customers_PM], [Total_customers_PM], 0),
    BLANK()
)
```

```dax
Daily_Revenue_MoM% = 
IF(
    HASONEVALUE(dim_date[mmm_yy]),
    DIVIDE([Daily_Revenue] - [Daily_Revenue_PM], [Daily_Revenue_PM], 0),
    BLANK()
)
```

```dax
Daily_Customers_MoM% = 
IF(
    HASONEVALUE(dim_date[mmm_yy]),
    DIVIDE([Daily_Customers] - [Daily_Customers_PM], [Daily_Customers_PM], 0),
    BLANK()
)
```

```dax
Policy Count MoM % = 
DIVIDE(
    [Total Premium Rows] - [Policies PM],
    [Policies PM],
    0
)
```

----

## Conditional Formatting (MoM Indicator Colors)

```dax
Revenue_MoM_Color = 
IF(
    ISBLANK([Total_Revenue_MoM%]), "#808080",
    IF([Total_Revenue_MoM%] >= 0, "#1E7145", "#C62828")
)
```

```dax
Customers_MoM_Color = 
IF(
    ISBLANK([Total_customers_MoM%]), "#808080",
    IF([Total_customers_MoM%] >= 0, "#1E7145", "#C62828")
)
```

```dax
DailyRevenue_MoM_Color = 
IF(
    ISBLANK([Daily_Revenue_MoM%]), "#808080",
    IF([Daily_Revenue_MoM%] >= 0, "#1E7145", "#C62828")
)
```

```dax
DailyCustomers_MoM_Color = 
IF(
    ISBLANK([Daily_Customers_MoM%]), "#808080",
    IF([Daily_Customers_MoM%] >= 0, "#1E7145", "#C62828")
)
```

----

## Sales Mode Analysis

```dax
Digital_Revenue % = 
DIVIDE(
    CALCULATE([Total_Revenue], KEEPFILTERS(fact_premiums[sales_mode] IN {"Online-App", "Online-Website"})),
    CALCULATE([Total_Revenue], ALL(fact_premiums[sales_mode]))
)
```

```dax
Offline_Revenue % = 
DIVIDE(
    CALCULATE([Total_Revenue], KEEPFILTERS(fact_premiums[sales_mode] IN {"Offline-Agent", "Offline-Direct"})),
    CALCULATE([Total_Revenue], ALL(fact_premiums[sales_mode]))
)
```

```dax
Sales_Mode_Revenue_Share % = 
DIVIDE(
    [Total_Revenue],
    CALCULATE([Total_Revenue], ALL(fact_premiums[sales_mode]))
)
```

----

## Age Group / Policy Analysis

```dax
Age = DATEDIFF(dim_customer[dob], TODAY(), YEAR)
```

```dax
Age Group = 
SWITCH(
    TRUE(),
    [Age] <= 24, "18-24",
    [Age] <= 30, "25-30",
    [Age] <= 40, "31-40",
    [Age] <= 50, "41-50",
    [Age] <= 65, "51-65",
    "65+"
)
```

```dax
Is Preferred Policy = 
VAR CurrentCellValue = [Total_customers]
VAR MaxForAgeGroup =
    MAXX(
        ALLSELECTED(dim_policies[policy_id]),
        CALCULATE([Total_customers])
    )
RETURN
IF( NOT ISBLANK(CurrentCellValue) && CurrentCellValue = MaxForAgeGroup, 1, 0 )
```

----

## Page Toggle Logic

```dax
Test Selected = SELECTEDVALUE('Selected_Table'[SelectorValue], "NOTHING SELECTED")
```
