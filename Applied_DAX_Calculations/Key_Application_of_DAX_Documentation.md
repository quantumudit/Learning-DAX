# Applied DAX Calculations

Although there are lots of functions available in DAX but, in most of the cases, we don't need all of them very frequently.

In fact, there are some particular functions and calculations in DAX that we are going to need in most cases.

## Aggregation Functions

The most important aggregation functions used for calculating the key measures are as follows :

- `SUM()`

- `AVERAGE()`

- `MAX()`

- `MIN()`

By using the `SUM()` function, we can calculate "_Total Sales_" as follows :

```dax
Total Sales = SUM( Sales[Total Revenue] )
```

Additionally, we can also make use of other aggregation functions for the following calculations :

For calculating "_Total Quantity Sold_" :

```dax
Total Quantity Sold = SUM( Sales[Order Quantity] )
```

For calculating "_Average Sales_" :

```dax
Average Sales = AVERAGE( Sales[Total Revenue] )
```

For calculating "_Maximum Sales_" :

```dax
Maximum Sales = MAX( Sales[Total Revenue] )
```

For calculating "_Minimum Sales_" :

```dax
Minimum Sales = MIN( Sales[Total Revenue] )
```

## Count Functions

The count functions are the aggregation functions that helps in counting entities.

Common count functions that are often used are :

- `COUNTROWS()`
- `DISTINCTCOUNT()`
- `COUNTBLANK()`

we can use `COUNTROWS()` function to calculate the "_Total Transactions_" :

```dax
Total Transactions = COUNTROWS( Sales )
```

Alternatively, we can use functions like `COUNT()` or, `COUNTA()` to get the same result as that of `COUNTROWS()`.

Similarly, we can count the number of blank rows present (if any) in the "_Total Revenue_" column, by using the `COUNTBLANK()` function:

```dax
Total Blanks = COUNTBLANK(Sales[Total Revenue])
```

We can count the number of distinct products present in the data model by using the `DISTINCTCOUNT()` function :

```dax
Total Products Bought = DISTINCTCOUNT(Sales[_ProductID])
```

## Iteration Functions

Iteration functions are capable of executing complex calculation by iterating over each row.

Some of the most important iteration functions that are used very often are :

- `SUMX()`
- `AVERAGEX()`
- `MINX()`
- `MAXX()`

By using the `SUMX()` function, we can calculate "_Total Cost_" as follows :

```dax
Total Cost =
SUMX(
    Sales,
    Sales[Order Quantity] * Sales[Total Unit Cost]
)
```

Similarly, we can calculate the "_Average Cost_" as follows :

```dax
Average Cost =
AVERAGEX(
    Sales,
    Sales[Order Quantity] * Sales[Total Unit Cost]
)
```

For calculating "_Maximum Cost_" :

```dax
Maximum Cost =
MAXX(
    Sales,
    Sales[Order Quantity] * Sales[Total Unit Cost]
)
```

For calculating "_Minimum Cost_" :

```dax
Minimum Cost =
MINX(
    Sales,
    Sales[Order Quantity] * Sales[Total Unit Cost]
)
```

## Measure Branching

By the virtue of measure branching methodology, we can create a measure by utilizing existing calculated DAX measures.

_For example :_ We can calculate "_Total Profit_" by using "_Total Sales_" and "_Total Cost_" measures, as follows :

```dax
Total Profit = [Total Sales] - [Total Cost]
```

Similarly, we can calculate "_Profit Margin_" as follows :

```dax
Profit Margin =
DIVIDE(
    [Total Profit], [Total Sales], 0
)
```

## Combining DAX Functions

Along with measure branching, we can combine DAX functions together to get a certain result.

**For Example:**

We can calculate "_Sales Segment_",i.e., the "_Total Sales_" when the "_Profit Margin_" is greater than 35% :

```dax
Sales Segment =
CALCULATE (
    [Total Sales],
    FILTER (
        Sales,
        [Profit Margin] > 0.35
    )
)
```

The above formula iterates over the "_Sales_" table for checks whether the "_Profit Margin_" for each row item is greater than 35% or, not and then, returns a virtual table with row elements having "_Profit Margin_" greater than 35% and the "_Total Sales_" gets calculated only for those rows.

We can achieve the same result by using `SUMX()` function too :

```dax
Sales Segment =
SUMX (
    FILTER (
        Sales,
        [Profit Margin] > 0.35
    ),
    [Total Sales]
)
```

In the similar pattern, we can also calculate "_Cost Segment_" :

```dax
Cost Segment =
SUMX (
    FILTER (
        Sales,
        [Profit Margin] > 0.35
    ),
    [Total Cost]
)
```

## Error Handling

Some of the important DAX function that are used to handle possible errors and blank values are :

- `BLANK()`
- `ISBLANK()`
- `IFERROR()`

**For Example :**

Suppose, we have to calculate the "_YoY Sales Difference_",i.e., the difference between current year sales and past year sales then, we might caught into a situation where, we would not be having data for the last year sales.

In those situation the result of "_YoY Sales Difference_" would be ambiguous thus, we can use error handling DAX function to avoid such situation as follows :

```dax
YoY Sales Difference =
IF (
    ISBLANK ( [Sales LY] ),
    BLANK (),
    [Total Sales] - [Sales LY]
)
```

Where "_Sales LY_" is :

```dax
Sales Last Year =
CALCULATE (
    [Total Sales],
    SAMEPERIODLASTYEAR ( Dates[Date] )
)
```

Similarly, we can use `IFERROR()` in case, some values are returning error/NaN as output.

## Logical Function

The two most popular logical functions used in DAX are :

- `IF()`
- `SWITCH()`

We can use simple `IF()` function to execute only positive "_YoY Sales Difference_" :

```dax
+Ve YoY Sales Difference =
IF (
    [YoY Sales Difference] >= 0,
    [YoY Sales Difference],
    BLANK ()
)
```

similarly,

```dax
-Ve YoY Sales Difference =
IF (
    [YoY Sales Difference] < 0,
    [YoY Sales Difference],
    BLANK ()
)
```

To write multiple conditions, we can use `SWITCH()` statement instead of using nested `IF()` functions.

So, we can categorized the "_YoY Sales Difference_" by using a `SWITCH()` function as follows :

```dax
YoY Sales Difference Type =
SWITCH (
    TRUE (),
    [YoY Sales Difference] < 0, "-Ve Value",
    [YoY Sales Difference] = 0, "Zero Value",
    [YoY Sales Difference] > 0, "+Ve Value"
)
```

We can use the logical functions with iterating functions, to find out some deep insights too.

**For Example :**

We can find out the "_Total Cost_" only when the "_Order Quantity_" is greater than 3 as follows :

```dax
Total Costs High Quantity =
SUMX (
    Sales,
    IF (
        Sales[Order Quantity] <= 3,
        0,
        Sales[Order Quantity] * Sales[Total Unit Cost]
    )
)
```

## Divide Function

The `DIVIDE()` function prevents the division errors and thats why it is recommended to use instead of the `/` operator.

We can calculate "_YoY Sales Growth_" using the `DIVIDE()` function as follows :

```dax
YoY Sales Growth =
DIVIDE (
    [YoY Sales Difference],
    [Total Sales],
    0
)
```

## Formatting Function

## Table Functions

Table functions in DAX is used to create virtual tables that can be used within a measure or, for summarizing values.

- Table functions can be used in any function where we can place a filter or, table.
- Table functions allows us to dictate what the underlying table looks like before a calculation takes place.
- The only way to really understand this is through many examples and testing results as we try new inputs to the formula.

Some of the important table functions used in DAX are :

- `FILTER()`
- `VALUES()`
- `DISTINCT()`
- `ALL()`
- `ALLSELECTED()`
- `ALLEXCEPT()`
- `SELECTEDVALUE()`
- `HASONEVALUE()`

We can use the **DAX Studio** to visualize the virtual tables generated through DAX using the `EVALUATE` syntax :

### `FILTER()`

The following DAX expression returns a table where the "_Order Quantity_" is greater than 10 and "_Channel_" is "Export":

```dax
EVALUATE
FILTER (
    Sales,
    Sales[Order Quantity] > 10
        && Sales[Channel] = "Export"
)
```

Now, we can put up this virtual table within a `CALCULATE()` function to calculate the "_Total Sales_" only for the row items where "_Order Quantity_" is greater than 10 and "_Channel_" is "Export", as follows :

```dax
Export Sales High Quantity =
CALCULATE (
    [Total Sales],
    FILTER (
        Sales,
        Sales[Order Quantity] > 10
            && Sales[Channel] = "Export"
    )
)
```

### `VALUES()`

The `VALUES()` function provides a one column unique item list of the input column.

To get a list of unique "_Channel_" we have in the "_Sales_" table :

```dax
EVALUATE
VALUES ( Sales[Channel] )
```

We can use the `VALUES()` function to calculate the average sales for warehouses, as follows :

```dax
Average Warehouse Sales =
AVERAGEX (
    VALUES ( Sales[Warehouse] ),
    [Total Sales]
)
```

Similarly, we can calculate the "_Average Monthly Sales_" as follows :

```dax
Average Monthly Sales =
AVERAGEX (
    VALUES ( Dates[Month-Year] ),
    [Total Sales]
)
```

### `DISTINCT()`

The `DISTINCT()` function is very similar to the `VALUES()` function but, the minute difference prevails when we have blank values or, there is a relationship mismatch.

Blank values wouldn't appear when we use the column with `DISTINCT()`.

The behavior of `DISTINCT()` & `VALUES()` functions in case of a relationship mismatch can be understood with the following example :

Let's say, if we have some customers information in the "_Sales_" table (Facts table) which are not present in the "_Customers_" table (Dimension table) then, `VALUES([Customer Name])` will provide the "_Customer Name_" along with a blank row that represents the missing customers in the "_Customers_" table.

In such scenario, if we show the "_Sales Amount_" by Customers as a visual in Power BI then, a blank row will appear that aggregates "_Sales Amount_" for all the missing customers.

But, if we use `DISTINCT()` then, we will not see any blank rows.

Therefore, its always preferred to use `VALUES()` over `DISTINCT()` to avoid data loss and troubleshoot relationship mismatch issues.

### `SELECTEDVALUE()`

If based on the context the virtual table generated through `VALUES()` function just has one row (a single value) then, the table function `VALUES()` could be used a measure.

But, in some cases, we can't just use `VALUES()` alone as a measure.

**For Example :**

If we are showing a matrix visual of "Sales Amount" by "Customer Names" in a matrix visual along with the grand total row then, if we place the following DAX function then, it will throw an error, only because of the existence of grand total in the visual :

```dax
Cust. Nm. = VALUES(Customers[Customer Name])
```

To resolve this problem, we have to use `HASONEVALUE()` function with `IF()` clause to ensure a one row virtual table as the output context for `VALUES()` function to work properly.

So, the correct formula would be:

```dax
Cust. Nm. =
IF (
    HASONEVALUE ( Customers[Customer Names] ),
    VALUES ( Customers[Customer Names] ),
    BLANK ()
)
```

But, there is a simpler way to achieve the same result with `SELECTEDVALUE()` function.

So, we can actually achieve the same desired output, with the following DAX function :

```dax
Cust. Nm. =
SELECTEDVALUE ( Customers[Customer Names], BLANK () )
```

### `ALL()`

The `ALL()` function removes filter from the current context. So, it is mostly used to calculate shares or, part to whole calculations.

We can calculate "_% Total Product Sales_" using the `ALL()` function as follows :

```dax
% Total Product Sales =
DIVIDE (
    [Total Sales],
    CALCULATE ( [Total Sales], ALL ( Products[Product Name] ) ),
    0
)
```

### `ALLEXCEPT()`

The `ALLEXCEPT()` function removes the filter from the current context except from the specified attribute/column.

### `ALLSELECTED()`

## Context Manipulating Function

The most important DAX functions that manipulates the current context in DAX are :

1. `ISFILTERED()`
2. `ISCROSSFILTERED()`
3. `RELATED()`
4. `EARLIER()`

### `ISFILTERED()`

The `ISFILTERED()` function returns `True` if the attribute gets directly filtered by the current filter context.

**For Example :**

If we have a visual showing the "_Total Sales_" by "_Customer Name_" then, `ISFILTERED()` will return `True` for the "_Customers_" table and "_Customer Name_" column only.

Also, at the grand total level, the filter context over the "_Customer Name_" collapses and thus, `ISFILTERD()` returns `False` at the Total level.

### `ISCROSSFILTERED()`

The `ISCROSSFILTERED()` function returns `True` if the attribute gets filtered indirectly (via relationships) by the current filter context.

**For Example :**

In the above stated scenario, `ISCROSSFILTERED()` will return `True` as the "_Sales_" table, "_Customer ID_" column of both "_Sales_" and "_Customer_" table gets filtered indirectly through the relationship.

However, the attributes from the other dimension table will return `False` as they neither gets filtered directly or, indirectly by the current filter context.

Similarly, just like the `ISFILTERED()` function, `ISCROSSFILTERED()` will also return `False` at the grand total level.

### `RELATED()`

The `RELATED()` function allows the user to move from one table to the other via relationship.

So, if we need a particular attribute from the dimension table to the fact table, then, we can use the `RELATED()` function, as follows :

**For Example :**

If we want the "_Product Names_" in the "_Sales_" table, then, we have to write the following DAX expression :

```dax
Prod. Nm. = RELATED ( 'Products'[Product Name] )
```

### `EARLIER()`

## The `CALCULATE()` Function

The `CALCULATE()` function allows to override or, add the current context before the evaluating the expression.

### Advantages of using `CALCULATE()`

- `CALCULATE()` allows to create a variety of different filters in the calculations that can't be created in any other way.

- Use of additional DAX functions like `FILTER()`, `ALL()`, `ALLSELECTED()`, `VALUES()`, `DISTINCT()` etc. with `CALCULATE()`, modifies the context and filtering on the table prior to the evaluation of the expression and thus, results in faster execution of DAX queries.

### Manipulating Relationships with `CALCULATE()`

The two important DAX functions that are used with `CALCULATE()` function to manipulate the properties of existing relationships (both active & inactive) are:

- `USERELATIONSHIP()`
- `CROSSFILTER()`

#### `USERELATIONSHIP()`

## Time Intelligence Functions

Some of the most used time intelligence functions used in DAX are :

A date table / calender table is essential for working with time intelligence function.

There are 2 ways to create date table / calender table within Power BI, i.e.,

- Using Power Query
- Using DAX

The following DAX code, creates a simple "Dates" table for our model :

```dax
Dates =
VAR FirstOrderDate =
    MIN ( Sales[OrderDate] )
VAR FirstOrderYear =
    YEAR ( FirstOrderDate )
VAR DateRange =
    FILTER ( CALENDARAUTO (), YEAR ( [Date] ) >= FirstOrderYear )
VAR FYStartMonth = 4
RETURN
    ADDCOLUMNS (
        DateRange,
        "Year", YEAR ( [Date] ),
        "QuarterNumber", QUARTER ( [Date] ),
        "Quarter", FORMAT ( [Date], "\QQ" ),
        "MonthNumber", MONTH ( [Date] ),
        "Full Month Name", FORMAT ( [Date], "mmmm" ),
        "Month", FORMAT ( [Date], "mmm" ),
        "Quarter-YearSort", CONVERT ( FORMAT ( [Date], "yyyy0Q" ), INTEGER ),
        "Quarter-Year", FORMAT ( [Date], "\QQ yyyy" ),
        "Month-YearSort", IF (
            MONTH ( [Date] ) <= 9,
            CONVERT ( FORMAT ( [Date], "yyyy0m" ), INTEGER ),
            CONVERT ( FORMAT ( [Date], "yyyym" ), INTEGER )
        ),
        "Month-Year", FORMAT ( [Date], "mmm yyyy" ),
        "FY", IF (
            MONTH ( [Date] ) >= FYStartMonth,
            "FY"
                & RIGHT ( YEAR ( [Date] ) + 1, 2 ),
            "FY" & RIGHT ( YEAR ( [Date] ), 2 )
        )
    )
```

### `DATEADD()`

The `DATEADD()` is one of the most versatile time intelligence function available.

By using this single function, we can perform a lot of time intelligence calculations, as follows :

- Calculating the last year sales, i.e., "_Sales LY_" :

```dax
Sales LY =
CALCULATE ( [Total Sales], DATEADD ( Dates[Date], -1, YEAR ) )
```

- Calculating the last month sales, i.e., "_Sales LM_" :

```dax
Sales LM =
CALCULATE ( [Total Sales], DATEADD ( Dates[Date], -1, MONTH ) )
```

- Calculating the next month sales, i.e., "_Sales NM_" :

```dax
Sales NM =
CALCULATE ( [Total Sales], DATEADD ( Dates[Date], 1, MONTH ) )
```

- Calculating day-on-day growth, i.e., "_DoD Growth_" :

```dax
DoD Growth =
DIVIDE (
    [Total Sales],
    CALCULATE ( [Total Sales], DATEADD ( Dates[Date], -1, DAY ) ),
    0
) - 1
```

- Calculating day-on-day growth, i.e., "_MoM Growth_" :

```dax
MoM Growth =
DIVIDE (
    [Total Sales],
    CALCULATE ( [Total Sales], DATEADD ( Dates[Date], -1, MONTH ) ),
    0
) - 1
```

### `SAMEPERIODLASTYEAR()`

The `SAMEPERIODLASTYEAR()` enables to calculate the value of a measure one year before.

So, we can calculate the "_Sales LY_" in a single line using the `SAMEPERIODLASTYEAR()` function, as follows :

```dax
Sales SPLY =
CALCULATE(
    [Total Sales],
    SAMEPERIODLASTYEAR(Dates[Date])
)
```

We can calculate the value of a measure 2 year before by using the `SAMEPERIODLASTYEAR()` function two times as follows :

```dax
Sales 2yr Ago =
CALCULATE (
    CALCULATE ( [Total Sales], SAMEPERIODLASTYEAR ( Dates[Date] ) ),
    SAMEPERIODLASTYEAR ( dates[Date] )
)
```

But, for such scenarios, the `DATEADD()` function works best, as follows :

```dax
Sales LY =
CALCULATE ( [Total Sales], DATEADD ( Dates[Date], -2, YEAR ) )
```

### Time Aggregations

Aggregations over time can be done with the following DAX functions :

- `DATESYTD()` : For year-to-date calculation
- `DATESQTD()` : For quarter-to-date calculation
- `DATESMTD()` : For month-to-date calculation

#### `DATESYTD()`

The following DAX expression calculates, "_Sales YTD_" :

```dax
Sales YTD =
CALCULATE ( [Total Sales], DATESYTD ( Dates[Date] ) )
```

Similarly, we can calculate "_Sales QTD_" and "_Sales MTD_" as follows :

For "_Sales QTD_"

```dax
Sales QTD =
CALCULATE ( [Total Sales], DATESQTD ( Dates[Date] ) )
```

For "_Sales MTD_"

```dax
Sales MTD =
CALCULATE ( [Total Sales], DATESMTD ( Dates[Date] ) )
```

We can also perform the same time aggregations, without using the `CALCULATE()` function as follows :

For "_Sales YTD_" :

```dax
Sales YTD (Direct) = TOTALYTD([Total Sales],Dates[Date])
```

### Time Comparisons

There are some specific time intelligence functions in DAX that calculates measures with a pre-defined context.

These functions are as follows :

- `PREVIOUSDAY()`
- `PREVIOUSMONTH()`
- `PREVIOUSQUARTER()`
- `PREVIOUSYEAR()`
- `PARALLELPERIOD()`

#### `PREVIOUSMONTH()`

```dax
Sales Previous Month =
CALCULATE ( [Total Sales], PREVIOUSMONTH ( Dates[Date] ) )
```

The above measure, calculates at the month context, i.e., if we have individual dates in the rows then, we will be seeing the same number for each date of the full month.

#### `PARALLELPERIOD()`

By using `PARALLELPERIOD()`, we can dynamically perform time intelligence calculation with a pre-defined context.

`PARALLELPERIOD()` omits the need of using the `PREVIOUSMONTH()`, `PREVIOUSQUARTER()` and `PREVIOUSYEAR()`, as follows :

- To calculate previous month sales at month context :

```dax
Sales Previous Month (Dynamic) =
CALCULATE ( [Total Sales], PARALLELPERIOD ( Dates[Date], -1, MONTH ) )
```

### Information Based Time Intelligence

Some of the information based time intelligence functions are as follows:

-

#### `OPENINGBALANCEMONTH()`

With `OPENINGBALANCEMONTH()` formula, we get the number present in the last date of the previous month as the opening balance for the current month.

This could be useful in financial analysis scenario, inventory management or, stocks calculations, in particular.

- We can calculate `OPENINGBALANCEMONTH()` for the "_Total Sales_", as follows:

```dax
Opening Month Sales Balance =
OPENINGBALANCEMONTH ( [Total Sales], Dates[Date] )
```

#### `STARTOFMONTH()`

The `STARTOFMONTH()` works opposite to the `OPENINGBALANCEMONTH()` formula, i.e., it retrieves the value present in the first date of the current month.

- We can calculate the `STARTOFMONTH()` for the "_Total Sales_" as follows:

```dax
Start of Month Balance =
CALCULATE ( [Total Sales], STARTOFMONTH ( Dates[Date] ) )
```

#### `ENDOFYEAR()`
