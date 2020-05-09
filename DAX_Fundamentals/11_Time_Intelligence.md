# Time Intelligence

## DATESBETWEEN()
---

Let's first try to calculate the "Sales Amount" between a certain range of dates using simple DAX code

```dax
Sales Amount = 
CALCULATE(
    [Sales Amount],
    'Date Table'[Date] >= DATE(2007,1,1) && 'Date Table'[Date] <= DATE(2007,5,15)
)
```
The above DAX code will calculate the sum of sales amount from January-2007 to May-2007 and will show the same result irrespective of any row context.

we can perform the same operation by using `DATESBETWEEN` operator more easily as follows -:

```dax
Sales Amount = 
CALCULATE(
    [Sales Amount],
    DATESBETWEEN(
        'Date Table'[Date],
        DATE(2007,1,1),
        DATE(2007,5,1)
    )
)
```
We can also calculate the "Sales YTD" using the `DATESBETWEEN` as follows -:

```dax
Sales YTD = 
VAR Last_Visible_Day = MAX('Date Table'[Date])
RETURN
CALCULATE(
    [Sales Amount],
    DATESBETWEEN(
        'Date Table'[Date],
        DATE(2007,1,1),
        Last_Visible_Day
    )
)
```
- The above DAX formula will calculate the sales amount cumulatively starting from January-2007.
- So, against the row context of March-2007, the sales amount would be the summation of the sales amount of January, February and March-2007.
-But, the limitation of this formula is that it doesn't have any lower bound and will remain correct as long as we are in the year 2007.
- This means, if we look at the "YTD Sales" for the month of January-2008, then the amount would be the summation of "Sales Amount" starting from January-2007 to January-2008 instead of only the "Sales Amount" of January-2008.
- Therefore, we need to write a more dynamic "Sales YTD" formula that will calculate correctly for each row context.

```dax
Sales YTD = 
VAR Last_Visible_Date = MAX('Date Table'[Date])
RETURN
CALCULATE(
    [Sales Amount],
    DATESBETWEEN(
        'Date Table'[Date],
        DATE(YEAR(Last_Visible_Date),1,1),
        Last_Visible_Date
    )
)
```
- The above formula will correctly calculate the "Sales YTD" without any error.

- Still, we can modify the same formula, as follows -:

```dax
Sales_YTD = 

VAR Last_Visisble_Date = MAX('Date Table'[Date])
VAR Custom_YTD = 
    DATESBETWEEN(
        'Date Table'[Date],
        DATE(YEAR(Last_Visisble_Date),1,1),
        Last_Visisble_Date
    )
RETURN
CALCULATE([Sales Amount], Custom_YTD)
```

In DAX, we have customized time intelligence functions that help us in calculating YTD, so that, we don't have to write so much for calculating "Sales YTD"

## DATESYTD()
---

- For calculating the "Sales YTD", we can simply use the `DATESYTD()` operator, as follows -:

```dax
Sales_YTD (TI) = 
CALCULATE(
    [Sales Amount],
    DATESYTD('Date Table'[Date])
)
```

The above formula will correctly calculate the "Sales YTD" for each row context correctly

- In the `DATESYTD` function, we can also specify the month from which we want to calculate "YTD".
- So, for example, if we want to calculate "Sales YTD" for a financial year where the year starts with 1st July then, the formula would be -:

```dax
Sales_YTD (Fiscal Year) = 
CALCULATE(
    [Sales Amount],
    DATESYTD('Date Table'[Date], "06-30")
)
```
- In the 2nd argument of `DATESYTD` we provided the `[YearEndDate]` which is 30th July, in case of a fiscal year.

## TOTALYTD()
---

- Instead of using `CALCULATE()` and then `DATESYTD()`, we can use `TOTALYTD()` function to calculate "Sales YTD" as follows -:

```dax
Sales YTD = 
TOTALYTD(
    [Sales Amount],
    'Date Table'[Date]
)
```
- Similarly, for calculating the "Sales YTD" for a fiscal year -:

```dax
Sales YTD (Fiscal Year) = 
TOTALYTD(
    [Sales Amount],
    'Date Table'[Date],"06-30"
)
```
- As we are more familiar with using `CALCULATE()`, hence, its always recommended to use the `CALCULATE()` & `DATESYTD()` functions to calculate YTD instad of using the `TOTALYTD()` function.

## DATEADD()
---
- This function shifts the date back and forth over time using parameter to define period.
- The time period can be `YEAR `, `QUATER`, `MONTH` or, `DAY`.
- We can use `DATEADD()` to calculate the "Same Period Last Year Sales Amount (Sales SPLY)" as follows -:

```dax
Sales SPLY = 
CALCULATE(
    [Sales Amount],
    DATEADD('Date Table'[Date],-1,YEAR)
)
```
- In the above formula, we retrived the previous year sales and as our data starts with year 2007, therefore, we will have no result for the entire year.

- For example -: We want to see a monthly comparision then, we can use the below DAX formula which is nothing but, "Same Period Last Month Sales (Sales SPLM)" -:

```dax
Sales SPLM = 
CALCULATE(
    [Sales Amount],
    DATEADD('Date Table'[Date],-1,MONTH)
)
```

- In the above formula, the row context for "January-2007" will remain blank and for "February-2007" we will see the sales amount of "January-2007"

### CALCULATING GROWTH %
---
We can use the following DAX formula to calculate the `Growth %` -:

```dax
Growth % = 
VAR SalesAmount = [Sales Amount]
VAR SalesSPLY = CALCULATE(
    [Sales Amount], 
    DATEADD('Date Table'[Date],-1, YEAR)
)
VAR RESULT = IF(
    SalesAmount <> 0 && SalesSPLY <> 0,
    DIVIDE(SalesAmount - SalesSPLY, SalesSPLY)
)
RETURN RESULT
```
## SAMEPERIODLASTYEAR()
---

- The `SAMEPERIODLASTYEAR()` function can be used instead of `DATEADD()` to calculate "Sales SPLY", as follows -:

```dax
Sales SPLY = 
CALCULATE(
    [Sales Amount],
    SAMEPERIODLASTYEAR('Date Table'[Date])
)
```
- We don't have to specify additional information like "-1" and "YEAR" that we have used to calculate "Sales SPLY" in case of `DATEADD()`.

## PARALLELPERIOD()
---

- We can observe that the functions `DATEADD()` and `SAMEPERIODLASTYEAR()` calculates the result to the granular level.

- So, if we drill down from year to quaters or, year to month or, year to day then, we can see the calculation happening.

- Suppose, we are using `DATEADD()` or, `SAMEPERIODLASTYEAR()` to calculate "Sales SPLY" then we can see the 2007 sales amount against the 2008 row and 2008 sales amount against 2009 row and so on. Moreover, if we drill down from year to month then, we can see the "January-2007" sales amount against the "January-2008" row, "February-2007" sales amount against "February-2008" row and so on.

- Let's assume, we don't want this granularity level information, rather we want to see only the sales amount of last year irrespective of any row information available, then, we can use `PARALLELPERIOD()` function instead of `DATEADD()`.

- The syntax of `PRALLELPERIOD()` is same as as that of `DATEADD()` and we can calculate the "Sales SPLY" as follows -:

```dax
Sales SPLY (Parl) = 
CALCULATE(
    [Sales Amount],
    PARALLELPERIOD('Date Table'[Date], -1, YEAR)
)
```
- Using `PARALLELPERIOD()` function, we can calculate "Sales YTD Over Past Year (Sales YTDOPY)", as follows -:

```dax
Sales YTDOPY = 
VAR SalesYTD = CALCULATE(
    [Sales Amount],
    DATESYTD('Date Table'[Date])
)
VAR SalesPY = CALCULATE(
    [Sales Amount],
    PARALLELPERIOD('Date Table'[Date], -1, YEAR)
)
RETURN
DIVIDE(SalesYTD, SalesPY)
```

- We can also calculate "Sales SPLM" using the `PARALLELPERIOD()` which will only work at month level.

## PREVIOUSYEAR()
---

- Just like we have `SAMEPERIODLASTYEAR()` as the simplified version to calculate "Sales SPLY" instead of using `DATEADD()` , similarly, we have `PREVIOUSYEAR()` as the simplified version for calculating "Sales SPLY" instead of `PARALLELPERIOD()` which will only work at year level and will show the year level information regardless of granularity selected.

- The DAX formula for calculating "Sales SPLY" using the `PREVIOUSYEAR()` function is as follows -:

```dax
Sales SPLY (Parl2) = 
CALCULATE(
    [Sales Amount],
    PREVIOUSYEAR('Date Table'[Date])
)
```

- We have other time intelligence functions to support `PARALLELPERIOD()`, like -:
 1. `PREVIOUSMONTH()`
 2. `PREVIOUSQUATER()`
 3. `PREVIOUSDAY()`

- The `PREVIOUSYEAR()` function come with an additional argument option where we can specify the `[YearEndDate]` for handling fiscal year calculations.

## DATESINPERIOD()
---
- This function is used to calculate the moving annual total, such as, 12MMT, 3MMT, 6MMT etc.

- We can use the following DAX formula to calculate 12MMT -:

```dax
Sales 12MMT = 
CALCULATE(
    [Sales Amount],
    DATESINPERIOD(
        'Date Table'[Date],
        MAX('Date Table'[Date]), -12, MONTH
    )
)
```
or, we can use the below DAX formula for the same operation -:

```dax
Sales 12MMT = 
CALCULATE(
    [Sales Amount],
    DATESINPERIOD(
        'Date Table'[Date],
        MAX('Date Table'[Date]), -1, YEAR
    )
)
```

- Similarly, we can calculate the 6MMT and 3MMT as follows -:

```dax
Sales 6MMT = 
CALCULATE(
    [Sales Amount],
    DATESINPERIOD(
    'Date Table'[Date],
    MAX('Date Table'[Date]), -6, MONTH
    )
)
```
```dax
Sales 3MMT = 
CALCULATE(
    [Sales Amount],
    DATESINPERIOD(
        'Date Table'[Date],
        MAX('Date Table'[Date]), -3, MONTH
    )
)
```

- Using the `DATESINPERIOD()` fucntion, we can also calculate the moving total by `YEAR`,`QUATER`, `MONTH` and `DAYS` as well.

## Calculating Running Total (Cumulative Sum)
---

- We can calculate the cumulative sum or, running total in 2 ways, i.e., 

1. Using Time Intelligence
2. Without Using Time Intelliigence

- Without using the time intelligence functions, we can calculate the running total as follows -:

```dax
Cumulative Sales Amount = 

VAR LastVisibleDate = MAX('Date Table'[Date])

RETURN
CALCULATE(
    [Sales Amount],
    'Date Table'[Date] <= LastVisibleDate
)
```
- We can calculate the same thing using a time intelligence function that we have already discussed, i.e., `DATESBETWEEN()`, as follows -:

```dax
Cumulative Sales Amount =

VAR Last_Visible_Day = MAX('Date Table'[Date])
RETURN
CALCULATE(
    [Sales Amount],
    DATESBETWEEN(
        'Date Table'[Date],
        DATE(2007,1,1),
        Last_Visible_Day
    )
)
```

## MIXING TIME INTELLIGENCE FUNCTIONS
---


