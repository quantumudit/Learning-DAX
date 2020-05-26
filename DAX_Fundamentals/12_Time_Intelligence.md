# Time Intelligence

Time intelligence is a set of operations used for computations over date and time.

Common time intelligence operations are :

-   Year To Date (YTD)
-   Quater To Date (QTD)
-   Month To Date (MTD)
-   Running Total (3MMT, 6MMT, 12MMT etc.)
-   Same Period Last Year (SPLY)
-   Past Period (PP)
-   Working days computations
-   Fiscal year computations
-   Calculation over weeks

and so on.

**_Time Intelligence Functions_**

The time intelligence functions are table functions.

1.  `DATESBETWEEN()`
2.  `DATESYTD()`
3.  `TOTALYTD()`
4.  `DATEADD()`
5.  `SAMEPERIODLASTYEAR()`
6.  `PARALLELPERIOD()`
7.  `PREVIOUSYEAR()`/ `PREVIOUSQUATER()` / `PREVIOUSMONTH()` / `PREVIOUSDAY()`
8.  `DATESINPERIOD()`

## `DATESBETWEEN()`

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

The above DAX formula will calculate the sales amount cumulatively starting from _January-2007_.

So, against the row context of _March-2007_, the sales amount would be the summation of the sales amount of _January, February & March-2007_.

But, the limitation of this formula is that it doesn't have any lower bound and will remain correct as long as we are in the _Year- 2007_.

Therefore, if we look at the "_YTD Sales_" for the month of _January-2008_, then the amount would be the summation of "Sales Amount" starting from _January-2007 to January-2008_ instead of only the "_Sales Amount_" of _January-2008_.

Therefore, we need to write a more dynamic "_Sales YTD_" formula that will calculate correctly for each row context.

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

The above formula will correctly calculate the "Sales YTD" without any error.

Still, we can modify the same formula, as follows -:

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

In DAX, we have customized time intelligence functions that help us in calculating YTD, so that, we don't have to write so much for calculating "_Sales YTD_"

#### Calculating Running Total (Cumulative Sum)

We can calculate the cumulative sum or, running total in 2 ways, i.e.,

1.  Using Time Intelligence
2.  Without Using Time Intelliigence

Without using the time intelligence functions, we can calculate the running total as follows -:

```dax
Cumulative Sales Amount =

VAR LastVisibleDate = MAX('Date Table'[Date])

RETURN
CALCULATE(
    [Sales Amount],
    'Date Table'[Date] <= LastVisibleDate
)
```

We can calculate the same thing using a time intelligence function that we have already discussed, i.e., `DATESBETWEEN()`, as follows -:

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

## `DATESYTD()`

For calculating the "Sales YTD", we can simply use the `DATESYTD()` operator, as follows -:

```dax
Sales_YTD (TI) =
CALCULATE(
    [Sales Amount],
    DATESYTD('Date Table'[Date])
)
```

The above formula will correctly calculate the "Sales YTD" for each row context correctly

In the `DATESYTD` function, we can also specify the month from which we want to calculate "_YTD_".

So, for example, if we want to calculate "_Sales YTD_" for a financial year where the year starts with _1st July_ then, the formula would be -:

```dax
Sales_YTD (Fiscal Year) =
CALCULATE(
    [Sales Amount],
    DATESYTD('Date Table'[Date], "06-30")
)
```

In the 2nd argument of `DATESYTD` we provided the `[YearEndDate]` which is _30th June_, in case of a fiscal year.

## `TOTALYTD()`

Instead of using `CALCULATE()` and then `DATESYTD()`, we can use `TOTALYTD()` function to calculate "_Sales YTD_" as follows -:

```dax
Sales YTD =
TOTALYTD(
    [Sales Amount],
    'Date Table'[Date]
)
```

Similarly, for calculating the "_Sales YTD_" for a fiscal year -:

```dax
Sales YTD (Fiscal Year) =
TOTALYTD(
    [Sales Amount],
    'Date Table'[Date],"06-30"
)
```

As we are more familiar with using `CALCULATE()`, hence, its always recommended to use the `CALCULATE()` & `DATESYTD()` functions to calculate YTD instad of using the `TOTALYTD()` function.

Moreover, we can't add additional filters with `TOTALYTD()` whereas, we can do so with `CALCULATE()` & `DATESYTD()` function.

## `DATEADD()`

This function shifts the date back and forth over time using parameter to define period.

The syntax of `DATEADD()` is :

```dax
DATEADD( << Date >>, << -i >>, << Time Period >> )
```

In the above syntax :

-   `<< Date >>` is the _Date_ column.
-   `<< -i >>` is the number period to shift backwards.
-   `<< Time Period >>` can be `YEAR`, `QUATER`, `MONTH` or, `DAY`.

We can use `DATEADD()` to calculate the "_Same Period Last Year Sales Amount (Sales SPLY)_" as follows -:

```dax
Sales SPLY =
CALCULATE(
    [Sales Amount],
    DATEADD('Date Table'[Date],-1,YEAR)
)
```

In the above formula, we retrived the previous year sales and as our data starts with _Year-2007_, therefore, we will have no result for _2007_.

Similarly, We want to see a monthly comparision then, we can use the below DAX formula which is nothing but, "_Same Period Last Month Sales (Sales SPLM)_" -:

```dax
Sales SPLM =
CALCULATE(
    [Sales Amount],
    DATEADD('Date Table'[Date],-1,MONTH)
)
```

In the above formula, the "_Sales SPLM_" for "_January-2007_" will remain blank and for "_February-2007_" we will see the sales amount of "_January-2007_" and so on.

#### CALCULATING GROWTH %

* * *

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

## `SAMEPERIODLASTYEAR()`

The `SAMEPERIODLASTYEAR()` function can be used instead of `DATEADD()` to calculate "_Sales SPLY_", as follows -:

```dax
Sales SPLY =
CALCULATE(
    [Sales Amount],
    SAMEPERIODLASTYEAR('Date Table'[Date])
)
```

We don't have to specify additional information like `-1` and `YEAR` that we have used to calculate "_Sales SPLY_" in case of `DATEADD()`.

## `PARALLELPERIOD()`

We can observe that the functions `DATEADD()` and `SAMEPERIODLASTYEAR()` calculates the result to the granular level.

So, if we drill down from year to quaters or, year to month or, year to day then, we can see the calculation happening.

Suppose, we are using `DATEADD()` or, `SAMEPERIODLASTYEAR()` to calculate "_Sales SPLY_" then we can see the _2007 Sales Amount_ against the _2008_ row and _2008 Sales Amount_ against _2009_ row and so on.</br>
         Moreover, if we drill down from _Year to Month_ then, we can see the "_January-2007 Sales Amount_" against the "_January-2008_" row, "_February-2007 Sales Amount_"  against "_February-2008_" row and so on.

Let's say, we don't want this granularity level information, rather we want to see only the sales amount of last year irrespective of any row information available, then, we can use `PARALLELPERIOD()` function instead of `DATEADD()`.

The syntax of `PRALLELPERIOD()` is same as as that of `DATEADD()` and we can calculate the "_Sales SPLY_" as follows -:

```dax
Sales SPLY (Parl) =
CALCULATE(
    [Sales Amount],
    PARALLELPERIOD('Date Table'[Date], -1, YEAR)
)
```

-   Using `PARALLELPERIOD()` function, we can calculate "_Sales YTD Over Past Year (Sales YTDOPY)_", as follows -:

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

We can also calculate "Sales SPLM" using the `PARALLELPERIOD()` which will only work at month level.

## `PREVIOUSYEAR()`

Just like we have `SAMEPERIODLASTYEAR()` as the simplified version to calculate "_Sales SPLY_" instead of using `DATEADD()`; similarly, we have `PREVIOUSYEAR()` as the simplified version for calculating "_Sales SPLY_" instead of `PARALLELPERIOD()` which will only work at year level and will show the year level information regardless of granularity selected.

The DAX formula for calculating "_Sales SPLY_" using the `PREVIOUSYEAR()` function is as follows -:

```dax
Sales SPLY (Parl2) =
CALCULATE(
    [Sales Amount],
    PREVIOUSYEAR('Date Table'[Date])
)
```

We have other time intelligence functions to support `PARALLELPERIOD()`, like -:

1.  `PREVIOUSMONTH()`
2.  `PREVIOUSQUATER()`
3.  `PREVIOUSDAY()`

The `PREVIOUSYEAR()` function come with an additional argument option where we can specify the `[YearEndDate]` for handling fiscal year calculations.

## `DATESINPERIOD()`

This function is used to calculate the moving annual total, such as, 12MMT, 3MMT, 6MMT etc.

We can use the following DAX formula to calculate _12MMT_ -:

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

To calculate the _6MMT_ -:

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

To calculate the _3MMT_ -:

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

Using the `DATESINPERIOD()` fucntion, we can also calculate the moving total by `YEAR`,`QUATER`, `MONTH` and `DAYS` as well.

## MIXING TIME INTELLIGENCE FUNCTIONS

* * *

To calculate "_Sales Past Year YTD_", we can mix the the two time intelligence functions as follows :

```DAX
Sales PYTD =
CALCULATE ( [Sales Amount], DATESYTD ( SAMEPERIODLASTYEAR ( 'Date'[Date] ) ) )
```

or, if we have "_Sales YTD_" as one of our measures then,

```DAX
Sales PYTD2 =
CALCULATE(
    [Sales YTD],
    SAMEPERIODLASTYEAR('Date'[Date])
)
```

or, we can store the tables executed by time intelligence functions in variables and call them as follows, to get the same result :

```DAX
Sales PYTD =
VAR Dates =
    VALUES ( 'Date'[Date] )
VAR PY =
    SAMEPERIODLASTYEAR ( Dates )
VAR PYTD =
    DATESYTD ( PY )
RETURN
    CALCULATE ( [Sales Amount], PYTD )
```

But, we can't use the following formula for calculating "_Sales PYTD_"

```DAX
Sales PYTD (Wrong) =
VAR SalesYTD =
    CALCULATE ( [Sales Amount], DATESYTD ( 'Date'[Date] ) )
RETURN
    CALCULATE ( SalesYTD, SAMEPERIODLASTYEAR ( 'Date'[Date] ) )
```

The problem with above formula is :

-   `VAR` returns a constant number/table that can't be manipulated by the `CALCULATE()` function, i.e., `VAR` calculates the results beyond the scope of `CALCULATE()` function present in the `RETURN` statement.
-   It is safe to use `VAR` when the functions inside `RETURN` doesn't depend upon the filter context of defined variables.

## Semi-Additive Measures

##### Additive Measures :

Additive measures are those that summed up over any dimension.

**_For example :_**<br>
The "_Sales Amount_" is an additive measure that gets summed up over any dimension ; it maybe over time or, category etc. and the grand total is also the sum of all individual numbers.

So, the `SUM()` or, `SUMX()` functions are actually the additive measures.

##### Non-additive Measures :

Non-additive measures are those that doesn't summed up over the dimension.

**_For example :_** The `DISTINCTCOUNT()` function gives dimension count and we can't sum-up the counts.

Similarly, The "_Date_" dimension is non-additive.

##### Semi-additive Measures :

Semi-additive measures are sometimes additives and sometimes not.

For example : The "_Current Account Balance_" is non-additive for a single customer whereas, its additive for multiple customers.

The current account balance of a customer for a month is nothing but, the balance on the last date of the month.

Similarly, the current account balance for a quater is the account balance on the last date of the last month of the quater and so on for semester and year level as well.

If we use time-intelligence function, then, we see the current account balance at the end of a month is the sum of the account balance present on each date of that month and similarly, for quater, the current account balance becomes the sum of all the three moths present in that month.

To tackle such scenarios, we actually have to modify the way we represent the "_Account Balance_".

So, our "_Account Balance_" formula would be :

```DAX
Account Balance =
CALCULATE (
    SUM ( 'Balances'[Balance] ),
    LASTNONBLANK ( 'Date'[Date], CALCULATE ( COUNTROWS ( Balances ) ) )
)
```

The `LASTNONBLANK()` function returns the last date of the "_Balances_" table when the "_Account Balance_" is non-zero.

The above formula is correct for an individual customer but, if we have many customers then, the grand total of all the customers might come wrong depending upon the difference in last dates for individual customers when the "_Account Balance_" is non-zero.

To resolve this issue, we have to modify our formula as follows :

```DAX
Account Balance =
SUMX (
    VALUES ( 'Balances'[Name] ),
    CALCULATE (
        SUM ( 'Balances'[Balance] ),
        LASTNONBLANK ( 'Date'[Date], CALCULATE ( COUNTROWS ( Balances ) ) )
    )
)
```

`SUMX()` computes the last balance customer by customer and sum them up for the grand total.
