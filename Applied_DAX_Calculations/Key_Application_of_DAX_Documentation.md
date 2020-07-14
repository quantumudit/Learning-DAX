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
