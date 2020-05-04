
## The Aggregators
---

- Aggregators work over a single column and can't work over an expression.

- Some of the aggregator functions are -:
    1. `SUM()`
    2. `AVERAGE()`
    3. `MAX()`
    4. `MIN()` 

- For example -: In order to calculate the "Sales Amount" we need to calculate the following DAX expression -:

```dax
Sales Amount = SUM(Sales[Quantity]) * SUM(Sales[Net Price])
```
Here we have to call `SUM()` twice because, aggregators can work upon a single column and not upon an expression ,i.e., the following DAX expression will throw an error as we enclosed an expression within `SUM()` -:

```dax
Sales Amnt = SUM(Sales[Quantity] * Sales[Net Price])

//This is not possible
```
To calculate expressions directly in DAX, we have "X-Aggregator" functions.

## The X-Aggregators
---

- The X-aggregation functions can work over expressions.
- Some of the important X-aggregators are as follows -:

    1. `SUMX()`
    2. `AVERAGEX()`
    3. `MINX()`
    4. `MAXX()`

So, using `SUMX()` we can calculate "Sales Amount" as follows -:

```dax
Sales Amnt = SUMX(Sales, Sales[Quantity] * Sales[Net Price])
```
- The X-Aggregators iterates over rows and therefore takes two arguments,i.e., "Table Name" and then the "Expression".

## The Count Aggregators
---
- The count aggregators are useful to count values and in DAX we have 4 such important functions -:

    1. `COUNT()`
    2. `COUNTA()`
    3. `COUNTROWS()`
    4. `DISTINCTCOUNT()`
    5. `COUNTBLANK()`

- We can calculate "Transactions" ,i.e., "No. of Sales" using `COUNTROWS()` as follows -:

```dax
Transaction = COUNTROWS(Sales)
```

- Similarly, we can calculate the number of unique customes, using the `DISTINCTCOUNT()` function -:

```dax
# Customers = DISTINCTCOUNT(Sales[CustomerKey])
```
Now, we can easily calculate "Sales By Transaction" and "Sales By Customers" using the following DAX code -:

```dax
Sales By Transaction = [Sales Amount]/[Transaction]
```
```dax
Sales By Customer = [Sales Amount]/[# Customers]
```
