# The `CALCULATE()` Function & Context Transition


The basic `CALCULATE()` function looks as follows :

```dax
Formula =
CALCULATE ( [Sales Amount], Sales[Net Price] > 100 )
```

is equivalent of writing

```dax
Formula =
CALCULATE (
    [Sales Amount],
    FILTER ( ALL ( Sales[Net Price] ), Sales[Net Price] > 100 )
)
```

##### Properties of `CALCULATE()`

- `CALCULATE()` is fast and flexible.
- Multiple filter condition in the same `CALCULATE()` are intersected *(And-ing)* together to form a single filter context.
- When multiple `CALCULATE()` statements are nested within each other then the filter condition of inner-most `CALCULATE()`, overwrites other filter context.

***For Example :***

```dax
Measure =
CALCULATE (
    CALCULATE ( [Sales Amount], Product[Color] IN { "Red", "Blue" } ),
    Product[Color] IN { "Yellow", "Blue" }
)
```
For the above measure, the result will be the sales amount for "*Red*" or, "*Blue*" color products only.

### `CALCULATE()` Operators
---




The various operators used with the `CALCULATE()` are :

- `ALL()` : Remove all previously existing filters **(REMOVERFILTERS)**
- `KEEPFILTERS()`: Adds new filters keeping the previous filters **(ADDFILTER)**

Let's understand these operators with suitable examples .

#### `CALCULATE()` With `ALL()`
---

- `ALL()` is a table function that also acts as a filter modifier to remove any existing filters from the filter context.
- `ALL()` must be understood as `REMOVEFILTERS()`


##### Inefficient Way To Calculate Share With `CALCULATE()` and `ALL()`

If we have "*Brands*" as rows and want to show the share of each brand then, we can write the following DAX function

```DAX
Brand Share =
DIVIDE (
    [Sales Amount],
    CALCULATE ( [Sales Amount], ALL ( 'Product'[Brand] ) )
)
```

Similarly, if we have "*Product Colors*" as rows then, the share would be :

```DAX
Color Share =
DIVIDE (
    [Sales Amount],
    CALCULATE ( [Sales Amount], ALL ( 'Product'[Color] ) )
)
```

But, if we want to get the share for any column from the "*Product*" table into the rows then, the DAX would be :

```DAX
Product Share =
DIVIDE ( [Sales Amount], CALCULATE ( [Sales Amount], ALL ( 'Product' ) ) )
```

Similarly, if we want to get the shares for any column of "*Product*" or, "*Customer*" table taken as the rows then, we have to write the DAX as follows :

```DAX
Product/Color Share =
DIVIDE (
    [Sales Amount],
    CALCULATE ( [Sales Amount], ALL ( 'Product' ), ALL ( 'Customer' ) )
)
```
##### Best Practice

Rather than taking the *dimension tables* into the `ALL()` statement, we can directly take the *fact table* into `ALL()`.

This is because of the fact that we ultimately seek the results form the *fact table* and all the *dimension table* eventually filters the *fact table* only.

So, in order to find share irrespective of any column from any dimension table taken into the rows, the DAX will be :

```DAX
DIVIDE ( [Sales Amount], CALCULATE ( [Sales Amount], ALL ( Sales ) ) )
```

#### `CALCULATE()` With `ALLSELECTED()`
---
- Sometimes, we may want to retrieve the *selection total* or, *visual total* rather than, removing the filters completely.
- `ALLSELECTED()` allows us to get the *visual total*

Let's say, we are showing the *Product Colors* in the rows and taken the *Category* as the slicer then, upon slicing the colors by any *Product Category* we may expect the *Total* to be 100% ,i.e., Share upon the selected category total but, with the above formula this doesn't happens as `ALL()` removes all the filter context.

Now to achieve the result, we have to use `ALLSELECTED()` instead of just `ALL()`.

So, the share formula should be written as follows :

```DAX
DIVIDE ( [Sales Amount], CALCULATE ( [Sales Amount], ALLSELECTED ( Sales ) ) )
```
#### `CALCULATE()` With `KEEPFILTERS()`
---

Some properties of the `KEEPFILTERS()` function is :

- It can only be used in the predicate part of the `CALCULATE()`.
- It keeps the previous filter, so that the new filter doesn't override the previous one.
- It doesn't remove the existing filter rather combine the new filter to produce desired results.

***For Example :***

If there is a formula like :

```dax
Red-blue sales =
CALCULATE (
    [Sales Amount],
    KEEPFILTERS ( Product[Color] IN { "Red", "Blue" } )
)
```
 and then we have slicer for "*Product[Color]*" in the same page with only "*Red*" selected along with many other colors, then,
 the result of the above created measure would be sales for only *red color* products instead of sales for products with either *red or, blue
 color*.

 If we have nested `CALCULATE()` functions then, the filter context of inner-most `CALCULATE()` function overwrites all other filters given in outer `CALCULATE()` statements.

 We can use `KEEPFILTERS()` function to avoid this overwriting and produce a result that intersects all the filter contexts, as follows :

 ```DAX
 Black Color Sales =
CALCULATE (
    CALCULATE (
        [Sales Amount],
        KEEPFILTERS ( 'Product'[Color] IN { "Red", "Black" } )
    ),
    'Product'[Color] IN { "Yellow", "Black" }
)
```

- Without the `KEEPFILTERS()`, we would have got the "*Sales Amount*" for the "*Red*" and "*Black*" color products.

- With the `KEEPFILTERS()`, now the filters in outer and inner `CALCULATE()` statements will intersect and we will get "*Sales Amount*" only for the "*Black*" color products.

> ***Notes :***</br>
Multiple filters in `CALCULATE()`: **Intersects** the filters</br>
Nested `CALCULATE()` statements : **Overwrites** the filters

### `CALCULATE()` Best Practices With `ALL()` & `KEEPFILTERS()`
---

- ***`CALCULATE()` doesn't support multiple column address in its predicate part.***

so, if we write something as follows :

```DAX
BigSales =
CALCULATE
(
	[Sales Amount],
	Sales[Quantity] * Sales[Net Price] > 1000
)
```
 then, we will get an error.

 In order to get the results for the above formula, we have to write our DAX code as follows :

 ```DAX
BigSales =
CALCULATE (
    [Sales Amount],
    FILTER ( Sales, Sales[Quantity] * Sales[Net Price] > 1000 )
)
```
But, the above solutions costs a lot of memory because, the `FILTER()` has to scan through all the rows of the table to execute the rows that satisfies the given condition.

So, there is even a better way to execute the same results with less memory usage:

We only need two columns from the "*Sales*" table ,i.e., `Sales[Quantity]` and `Sales[Net Price]` therefore, instead of scanning the whole table, we can utilize the power of `ALL()` statement to lessen the memory usage as follows :

```DAX
BigSales =
CALCULATE (
    [Sales Amount],
    FILTER (
        ALL ( Sales[Quantity], Sales[Net Price] ),
        Sales[Quantity] * Sales[Net Price] > 1000
    )
)
```
But, as soon as we add a slicer for either `Sales[Quantity]` or, `Sales[Net Price]` then, we will get wrong results for the above formula because `ALL()` removes all the filters .

So, for example, if we want to see the "*Sales Amount*" where the `Sales[Quantity] = 3` then, the above formula will give the wrong result.

So, a much correct formula would be :

```DAX
BigSales =
CALCULATE (
    [Sales Amount],
    KEEPFILTERS (
        FILTER (
            ALL ( Sales[Quantity], Sales[Net Price] ),
            Sales[Quantity] * Sales[Net Price] > 1000
        )
    )
)
```
 or,

```DAX
BigSales =
CALCULATE (
    [Sales Amount],
    FILTER (
        ALLSELECTED ( Sales[Quantity], Sales[Net Price] ),
        Sales[Quantity] * Sales[Net Price] > 1000
    )
)
```

- ***Boolean filters in `CALCULATE()` cannot use aggregators or, measures***

Let's say if we want to see the "*Sales Amount*" where the "*Sales Quantity*" is less than 0.01 times of total "*Total Sales Quantity*" then, writing a measure as follows, would be wrong :

```DAX
ConditionalSales =
CALCULATE ( [Sales Amount], Sales[Quantity] < SUM ( Sales[Quantity] ) / 100 )
```
However, we can use variables to obtain our desired result, as follows :

```DAX
-- Best Practice

ConditionalSales =
VAR TotalQty =
    SUM ( Sales[Quantity] )
RETURN
    CALCULATE ( [Sales Amount], Sales[Quantity] < TotalQty / 100 )
```

### `CALCULATE()` With `USERELATIONSHIP()`
---

- `USERELATIONSHIP()` used to temporarily activate an existing inactive relationship for the sake of calculation.
- It can only be used in the predicate part of the `CALCULATE()` function.

***For Example :*** </br>


In the above image; an inactive *one-to-many* relationship exists between the `Date[Date]` and `Sales[Delivery Date]` and therefore, to filter the "*Dates*" based on "*Delivery Date*"just for a specific calculation, we have to use `USERELATIONSHIP()` as follows :

```DAX
Delivery Amount =
CALCULATE (
    [Sales Amount],
    USERELATIONSHIP ( Sales[Delivery Date], 'Date'[Date] )
)
```

### `CALCULATE()` With `CROSSFILTER()`
---

- `CROSSFILTER()` used to temporarily change the direction of an existing active relationship for the sake of calculation.
- It can only be used in the predicate part of the `CALCULATE()` function.

The syntax of `CROSSFILTER()` is as follows :

```DAX
CROSSFILTER(«LeftColumnName»,«RightColumnName»,«CrossFilterType»)
```

In the above syntax, the first two arguments, i.e.,  `«LeftColumnName»` and `«RightColumnName»` takes up the two columns in relationship and the third argument, i.e., `«CrossFilterType»` can take 3 type of values as follows :

- `Both` : Makes the relationship bi-directional.
- `None` : Makes the relationship inactive.
- `OneWay`: Makes the relationship uni-directional.

***For Example :****

Consider the below data model :



We can see there is no relationship between the "*Customer*" table and the "*Date*" table and therefore, if we want to see the "*Number of Customers*" by "*Month*" then, using the following DAX expression will give us a wrong result :

```DAX
# Customers = COUNTROWS( Customer )
```

The result of the above DAX expression will always be the "*Total number of Customers*" irrespective of the row context.

Therefore, to get the correct result, we need to use the `CROSSFILTER()` as follows :

```DAX
# Customers =
CALCULATE (
    COUNTROWS ( Customer ),
    CROSSFILTER ( Sales[CustomerKey], Customer[CustomerKey], BOTH )
)
```
The above formula, temporarily makes the relationship between the "*Customer*" and "*Sales*" table bi-directional and thus, the "*Date*" filters can now propagate to the "*Customer*" table through the "*Sales*" table to give us the correct result for "*Number of Customers*" by "*Month*".


### Context Transition
