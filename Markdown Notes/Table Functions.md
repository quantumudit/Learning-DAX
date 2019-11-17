# Table Functions
---

- Some of the basic table functions used in DAX -:
	1. `FILTER()`
	2. `ALL()`
	3. `VALUES()`
	4. `DISTINCT()`
	5. `RELATEDTABLE()`

## FILTER()
---

- `FILTER()` takes a table as input and also returns a table as output.
- `FILTER()` can be iterated by using X-Aggregation function, like, `SUMX()`.
- For example -:

```dax
Big Sales Amount = 
SUMX(
    FILTER(
        Sales,
        Sales[Quantity] > 1
    ),
    Sales[Quantity] * Sales[Net Price]
)
```
## ALL()
---
- `ALL()` ignores any filter applied to a table and returns the whole table as output.
- For example, the result of the following "measure" will be same for each row element because `ALL()` ignores the filters -:
```dax
All Sales = 
SUMX(
    ALL(Sales),
    Sales[Quantity] * Sales[Net Price]
)
```
By using `ALL()`, we can calculate the share of each row element, as follows -:

```dax
Pct Sales = DIVIDE([Sales Amount],[All Sales])
```
- `ALL()` can also take a single column as input and returns a column with all the distinct values available in the input column.
- So, for example -: If in the "Products" table we have various products with different colors and we wanted a table with a list of all unique colors available then, we can use the following DAX expression to do so -:

```dax
Colors Table = ALL('Product'[Color])
```
- If we use more than one column inside the `ALL()` then, as the output, we get all the unique existing combinations available between them. This is similar to the output of "Remove duplicate" option in excel with multiple columns selected.
- For example -:
```dax
Colors Table = ALL('Product'[Color],'Product'[Category])
```
## ALLEXCEPT()
---
- `ALLEXCEPT()` returns a table with all existing distinct combination between the columns of a table exculding the column mentioned in the function.
- For example -: The following DAX expression will return all the existing unique combination between the columns of "Orders" table expect the `Orders[City]`.
```dax
Orders Combination = ALLEXCEPT(Orders, Orders[City])
```
## Mixing Filters
---
Let's say, we want to calculate the share of each row element "Orders" table upon the total "Sales Amount" where the channel is "Internet".

For doing so, we will be needing a measure, that will provide the "Sales Amount" for `Orders[channel] = "Internet"` irrespective of the row element present.

The following DAX expression will return the desired result -:
```dax
Internet Sales Amount = 
SUMX(
    FILTER(
        ALL(Orders),
        Orders[Channel] = "Internet"
        ),
    Orders[Quantity] * Orders[Net Price]
)
```
## DISTINCT()
---
- The output of `DISTINCT()`is same as that of `ALL()` but, the main difference is -:
    
`ALL()` Removes all the filters and thus, the output with `ALL()` always provides the same result irrespective of the row element whereas, `DISTINCT()` takes the row element into consideration and provides the output accordingly.

Let's understand the following with an example -:

```dax
# ALL Colors = 
COUNTROWS(
    ALL('Product'[Color])
)
```
With the above DAX expression, we will get the number of distinct colors available in each row irrespective of the row element present. But, if we use `DISTINCT()` instead of `ALL()` then, we will get the number of distinct colors available by the row element.

The following will be the required DAX expression -:

```dax
# Distinct Colors = 
COUNTROWS(
    DISTINCT('Product'[Color])
)
```

## DISTINCTCOUNT()
---
- Instead of using `COUNTROWS()` and then `DISTINCT()`, in order to count the number of unique elements present in the coloumn, we can directly use the `DISTINCTCOUNT()` function available in DAX.

The required DAX code will be -:

```dax
# Distinct Colors = DISTINCTCOUNT('Product'[Color])
```


     



