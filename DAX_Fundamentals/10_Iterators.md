# ITERATORS

Iterators are very userful when they are used with context transition to leverage techniques to create complex calculations in a very simple way.

Iterator functions iterates row by row over a table to compute an expression and aggregate its value.

Therefore, as a best practice iterators shouldn't be used while dealing with datasets having millions of rows.

## Working of Iterators

Let's say, we have a "*Date hierarchy*"" in the row section and want to see the maximum sales by *dates/months/years*.

Therefore, the maximum sales of *Janauary-2007* would be the sales of a date in *January-2007* where the sales is highest as compared to other dates.

Similarly, the maximum sales of year *2007* would be the sales of the month in the year *2007* where the sales is highest as compared to other months.

Now let's try to achieve the above result with DAX.

- ***Without Context Transition***

```DAX
Max Sales =
MAXX ( 'Date', SUMX ( Sales, Sales[Quantity] * Sales[Net Price] ) )
```

The above measure will only give correct result at "*Date*" level for the following reason :

At a month level,

- The current filter context for `SUMX()` is made up of two filters only, i.e.,

  1. Filter over the year (Let's say 2007)
  2. Filter over the month (Let's say January-2007)

Therefore, for a month level

- The `SUMX()` iterates over 31 days of *January* month of year *2007* by multiplying `Sales[Quantity]` and `Sales[Net Price]` and adding them togather.

But, the `MAXX()` has to iterate over all the dates

- Therefore, the `MAXX()` has only one value,i.e., "*Total Sales Amount for January-2007*" to iterate over all the 31-days and hence the result at month level is not *Maximum Sales Amount for January-2007*.

- To transfer the row context into the filter context or, to get the result from *many side (Sales Table)* of the relationship to *one side (Date Table)* of the relationship, we can use the function `RELATEDTABLE()`. So, a modified DAX expression to get the correct result would be :

```DAX
Max Sales 2 =
MAXX (
    'Date',
    SUMX ( RELATEDTABLE ( Sales ), Sales[Quantity] * Sales[Net Price] )
)
```
Now, we will get the desired result, because, at month level `MAXX()` iterates over each day and for each day `SUMX()` iterates and finally we get the correct result at month and year level as well.

***With Context Transition***

Rather than using `RELATEDTABLE()`, we can introduce the `CALCULATE()` function to perform context transition as follows :

```DAX
Max Sales =
MAXX (
    'Date',
    CALCULATE ( SUMX ( Sales, Sales[Quantity] * Sales[Net Price] ) )
)
```

Enclosing the `SUMX()` expression within `CALCULATE()` transforms the row context of "*Date*" table into the filter context for "*Sales*" table.

But, we can write the above code in more efficient way and even without using `SUMX()` as follows :

```DAX
Max Sales =
MAXX ( 'Date', [Sales Amount] )
```


## Iterator Functions

Some of the important iterators in DAX as follows :

- MAXX()
- MINX()
- AVERAGEX()
- SUMX()
- PRODUCTX()
- CONCATENATEX()
- VARX.P()
- VARX.S()
- STDEVX.P()
- STDEVX.S()
- MEDIANX()
- PERCENTILEX.EXC()
- PERCENTILEX.INC()
- GEOMEANX()

### `MINX()`, `MAXX()` & `AVERAGEX()`
---

To get the minimum and maximum values by iterating over a table, we can use the `MINX()` and `MAXX()` functions respectively.

***For example :***

- To get minimum sales amount per customer, the DAX will be :

```DAX
Min Sales Per Customer = MINX (Customer, [Sales Amount])
```
- To get maximum sales amount per customer, the DAX will be :

```DAX
Max Sales Per Customer = MAXX (Customer, [Sales Amount])
```
- To get average sales amount per customer, the DAX will be :

```DAX
Avg Sales Per Customer = AVERAGEX (Customer, [Sales Amount])
```

### `CONCATENATEX()`
---

If we want to see the *date* of the the month in which we have the maximum sales then, we can use the following DAX expression :

```DAX
Max Sales Date =
VAR Days = VALUES('Date'[Date]) -- returns the date in current row context
VAR MaxSales = MAXX(Days, [Sales Amount]) -- for month level, iterated for 30/31 times for over each date for maximum sales
VAR DaysWithMaxSale = FILTER(Days, [Sales Amount] = MaxSales) -- Returns a tabular output of the date with maximum sales
VAR Result =
CONCATENATEX(
   DaysWithMaxSale,
   FORMAT([Date],"dd-mm-yyyy"),
   ","
)

RETURN
IF(MaxSales <> BLANK(),Result,BLANK())
```

### `RANKX()`

The `RANKX()` function is useful to compute ranking.

This functions takes up two arguments as follows :
```dax
RANKX(<< Table >>, << Expression >>)
```

- `<< Table >>` is evaluated in the external filter context for iteration of the `<< Expression >>`, therefore, we need to enclose the `<< Table >>` with `ALL()` function.

- `<< Expression >>` is evaluated under the current row context of the `<< Table >>` during the iteration and then, evaluated in the external filter context for the rank calculation.

- Therefore, we must use a `[Measure]` or, `CALCULATE()` function in the `<< Expression >>` part to ensure the correct result.

Therefore, to rank the "*Product Categories*" by "*Sales Amount*", we have to write the following DAX expression :

```DAX
Ranking =
RANKX(
    ALL( 'Product'[Category] ),[Sales Amount]
    )
```

- If we want a dynamic ranking then, we should use `ALLSELECTED()` instead of `ALL()`.



### `ISINSCOPE()`
---

The `ISINSCOPE()` function is useful to control the calculation at different level of hierarchy.

Essentially, it tells about the level at which we want to execute the calculation.

During ranking of anything, we can see that the grand total is always ranked as `1` separately, which is not desired at all.

In such a scenario, we can utilize `ISINSCOPE()` to remove ranking at *sub-total* and *grand total*, as follows :

```DAX
Ranking =
IF(
    ISINSCOPE('Product'[Category]),
    RANKX(
        ALLSELECTED( 'Product'[Category] ),[Sales Amount]
    ),
    BLANK()
)
```
#### Handling Hierarchies With `ISINSCOPE()`
---

If we have "*Subcaregories*" nested within "*Categories*" in the rows and we want to see the ranking for both then, we can use the following DAX expression :

```DAX
Ranking =
IF (
    ISINSCOPE ( 'Product'[Subcategory] ),
    RANKX ( ALL ( 'Product'[Subcategory] ), [Sales Amount] ),
    IF (
        ISINSCOPE ( 'Product'[Category] ),
        RANKX ( ALL ( 'Product'[Category] ), [Sales Amount] )
    )
)
```

>***Notes :***</br>
We have to DAX in an orderly fashion for such scenario, i.e. the expression for the lowest level must come first followed by other levels in the hierarchy upto the highest level as the last expression.

In some cases, the above code does global ranking for the all the level but, if we want a local ranking, i.e., ranking of subcategories within a specific category then, we have to write the DAX espression as follows :

```DAX
Ranking =
IF (
    ISINSCOPE ( 'Product'[Subcategory] ),
    RANKX ( ALLEXCEPT ('Product','Product'[Subcategory] ), [Sales Amount] ),
    IF (
        ISINSCOPE ( 'Product'[Category] ),
        RANKX ( ALL ( 'Product'[Category] ), [Sales Amount] )
    )
)
```
To make the above ranking more dynamic, we can use `ALLSELECTED()` instead of `ALL()`.
