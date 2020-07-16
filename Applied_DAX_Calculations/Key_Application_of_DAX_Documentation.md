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

The following DAX expression returns a table where the "_Order Quantity_" is greater than 10  and "_Channel_" is "Export":

```dax
EVALUATE
FILTER (
    Sales,
    Sales[Order Quantity] > 10
        && Sales[Channel] = "Export"
)
```

Now, we can put up this virtual table within a `CALCULATE()` function to calculate the "_Total Sales_" only for the row items where "_Order Quantity_" is greater than 10  and "_Channel_" is "Export", as follows :

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

## `SELECTEDVALUE()`



## `ALL()`

The `ALL()` function removes filter from the current context. So, it is mostly used to calculate shares or, part to whole calculations.

We can calculate "_% Total Product Sales_" using the `ALL()` function as follows :

```dax






























