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
In the below image we can see that the two tables are connected through "ProductID" in a one-to-many relationship but, in the one-side of the relationship we don't have any corresponding "Product Name" for the "ProductID - 4" & "ProductID - 5".

![Image_1](https://user-images.githubusercontent.com/54057814/73245271-d96d2300-41d1-11ea-8562-438f6e6d740c.png)

Therefore, an additional blank row has been created in the one-side of the relationship that stores all the values of the missing "ProductID".

So, if we show a table with "Product Name" (Taken from 1-side of the related tables) and "Values" (Taken from many-side of the related tables) then, we will see an additional blank row which stores the aggregated values of missing "ProductID" (in the one-side of relationship).

## `COUNTROWS()` With Table Functions
----
- Now, if we use `DISTINCT()` then, it will ignore the "blank" row created in the "1-side" of the table and will give a result of 3 in the 'Total' row, 1 against each "Product Name" and "Blank" against the created blank row.
- If we use `ALL()` then, it will take the blank row created and will show 4 against each row element and in the `total` row as well.

Now, let's say, we want to execute the output as `ALL()` but, using the logic of `DISTINCT()`,i.e., if we want the "Total" row as well as all the corresponding rows to show "3" then, we can use the `ALLNOBLANKROW()` function.

Similarly, if we want to execute the output as `DISTINCT()` but, using the logic of `ALL()`,i.e., we want to show 1 against each row element (including the blank row) and 4 in the `Total` row then, we can use the `VALUES()` function.

## ALLNOBLANKROW()
---
- Ignores the blank rows present in the visual and also the filters applied (Just like `ALL()`).

```dax

Counting = 
COUNTROWS(
        ALLNOBLANKROW (Product[Product Key])
        )
```

## VALUES()
---

- Returns the unique values of a column including the additional blank rows (visible with the current filter context).

```dax

Counting = 
COUNTROWS(
        VALUES (Product[Product Key])
        )
```
The below image shows the different results produced by using `ALL()`, `VALUES()`, `DISTINCT()` & `ALLNOBLANKROW()` within `COUNTROWS()`.

![Image_2](https://user-images.githubusercontent.com/54057814/73245415-2c46da80-41d2-11ea-9d04-ccdfa28cebc1.png)

> Note :
- If we have some data in the many-side of the relationship but, not on the one-side then, the relationship is called as an "Invalid Relationship". 
- An additional blank row gets added on the one-side of the relationship to store the values of the many-side of the relationship that don't have a match on the one-side.

- It is always suggested to use `VALUES()` instead of `DISTINCT()` and `ALL()` instead of `ALLNOBLANKROWS()`unless we don't have a specifc reason.

 ## ALLSELECTED()
 ---
 The basic difference between `ALL()` and `ALLSELECTED()` is how the two functions behave with the current filter context.
 
 - `ALL()` ignores all the filter context whereas, `ALLSELECTED()` returns the values considering the current filter context.
 - For Example :
 
 If we have 5 products and their sum is 100 then, with `ALL()` function, even if we take a "product" slicer and select only 3 products, still, the grand total will show 100.
 But, in case of "ALLSELECTED()" the sum in the grand total will be the aggregate of the selected 3 products only.
 
![Image_3](https://user-images.githubusercontent.com/54057814/73245448-3e287d80-41d2-11ea-9ef5-dc5c1f4c1c38.png)
 
 ## SELECTEDVALUE()
 ---
 
 - `SELECTEDVALUE()` is a function that retrieves the value of a column when a single value is visible.
 
 - We can use `SELECTEDVALUE()` instead of `VALUES()`, if we want to show the value selected in a slicer because, it handles multiple selections/no selections much efficiently.
 - For example :
 
 ```dax
 SELECTEDVALUE(
 	'Product Category'[Category],
	"Multiple Values"
	)
	
// is equivalent of writing -:

IF(
	HASONEVALUE('Product Category'[Category]),
	VALUES('Product Category'[Category],
	"Multiple Values"
	)
 ```
 
 ## ISEMPTY()
 ---
 
 - This function checks wheather a table is empty or, not and returns a boolean expression.
 - `ISEMPTY()` is faster and hence we can use it with `VALUES()` instead of using `COUNTROWS()` with `VALUES()` togather , as follows -:
 
 ```dax
 ISEMPTY(VALUES('Products'[Unit Price])
 
 // is equivalent and faster than
 
 COUNTROWS(VALUES('Products'[Unit Price])) = 0
 
 ```
 
 ## TABLE VARIABLES
 ---
 - A variable can contain either a scalar value or, a table and they are very useful in splitting complex DAX.
 - We can use a variable to store a table for furture execution as follows 
 
 ```dax
 VAR
 	SalesGreaterThan10 = FILTER(Sales, Sales[Quantity]>10)
 RETURN
 	SUMX(
		FILTER(
			SalesGreaterThan10,
			RELATED(Product[Color]) = "Red"
			),
			Sales[Amount]
		)
```
