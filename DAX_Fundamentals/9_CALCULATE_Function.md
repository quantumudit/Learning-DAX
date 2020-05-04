# The `CALCULATE()` Function

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


#### `CALCULATE()` With `ALL()`
---

`ALL()` is a filter modifier that removes any existing filters from the filter context.

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

Let's say, we are showing the *Product colors* in the rows and taken the *Category* as the slicer then, upon slicing the colors by any product category we expect the total share always to be 100% with the above formula but, this doesn't happen because `ALL()` removes all the filter context.

Now to achieve the result, we have to use `ALLSELECTED()` instead of just `ALL()`.

So, the share formula would be :

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
 
