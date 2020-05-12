## `EVALUATE` Clause

`EVALUATE` is the statement used to write DAX queries. It can be thought as the `SELECT *` (of *SQL*) for DAX queries.

The syntax of `EVALUATE` is as follows -:

```dax
DEFINE
    MEASURE <table_name>[col_Name] = <expression>
EVALUATE
    <table_name>
ORDER BY
    <expression> [ASC|DESC],
    <expression> [ASC|DESC],
    .
    .
    .
START AT <Value1>, <Value2>, ...
```

- `DEFINE` allow us to create local measures (Very similar to the concept of "*Query Scoped Calculated Members*" in *MDX*).
- `ORDER BY` allow us to sort the output table by columns in ascending (`ASC`) or, descending (`DESC`) order.

Let's say, we want to query a full table with `EVALUATE`, then, we may use the following expression -:

```dax
EVALUATE
'Product'

ORDER BY
'Product'[Category],
'Product'[Subcategory],
'Product'[Product Name]
```
By default, `ORDER BY` Clause sorts everything in ascending order and to change the sorting type, we can write the following DAX code :

```DAX
EVALUATE
'Product'

ORDER BY
'Product'[Category] DESC,
'Product'[Subcategory] DESC,
'Product'[Product Name] ASC
```
> ***Note :***</br> We can also sort the `'Product'[Product Name]` without `ASC` as the default sort order is ascending.

- The `START AT` clause is no more used in DAX.

## `CALCULATETABLE()`

`CALCULATETABLE()` is identical to the `CALCULATE()` function in terms of features and way of calculation but unlike `CALCULATE()`, it evaluates a table expression and returns a table as output.

>***Note :***</br>`CALCULATE()` returns a scalar value or, a single number/string as the output.


***Using `FILTER()` to filter the table :***

To filter only the *red color products*, we need to write the following DAX code :

```DAX
EVALUATE
FILTER ( 'Product', 'Product'[Color] = "Red" )
```

`FILTER()` traverses the whole *Product* table and checks for `'Product'[Color] = "Red"` for each row item.

This process is an inefficient way and could affect the performance when the table has millions of rows.

We can use `CALCULATETABLE()` instead :

***Using `CALCULATETABLE()` to filter the table :***

The syntax of `CALCULATETABLE()` is exactly same as that of `FILTER()` but, in terms of performance it is much more efficient.

So, to filter only the *red color products* much efficiently, we can also write the following DAX code :

```dax
EVALUATE
CALCULATETABLE ( 'Product', 'Product'[Color] = "Red" )
```
Internally, `CALCULATETABLE()` prepares the filter context prior to the actual calculation rather than invoking the filter expression each time to check for the condition in each row of the table, as follows :

```dax
EVALUATE
CALCULATETABLE (
    'Product',
    FILTER ( ALL ( 'Product'[Color] ), 'Product'[Color] = "Red" )
)
```
So, let's say, if we have 10 distinct product colors, then, the `FILTER()` created by `CALCULATETABLE()` internally iterates only for 16 times, whereas, the former filteration of table with just the `FILTER()` statement iterates over all the rows of the *Product* table which can be in the order of thousands or, millions.

>***Note :***</br>
*The key reason for performance improvement by `CALCULATETABLE()` is tnvoking a smaller table to iterate over and thus preparing the filter context prior to the calculation*

Similarly, if we want to see the *Products* table where, "*Product Color*" can be "*Red, Black or, Blue*" then :

```DAX
EVALUATE
CALCULATETABLE (
    'Product',
    'Product'[Color] IN { "Red", "Black", "Blue" }
    )
```

***Where to use `FILTER()`***

Althoug, it seems as if `CALCULATETABLE()` replaces the `FILTER()` but, there are certain situation where we can't use `CALCULATETABLE()`.

Suppose, if we want to see the "*Product*" table only for the products where the *Sales Amount > 1000000* then, we can't use `CALCULATETABLE()`

Such scenarios requires iteration over the whole table and thus, `CALCULATETABLE()` just doesn't work here.

So, the DAX expression to execute this statement is :

```DAX
EVALUATE
FILTER('Product',[Sales Amount]>1000000)
```
>***Note :***</br>




## `SELECTCOLUMNS()`

The `SELECTCOLUMNS()` functions is used select and show some specific columns from a table and from its related table as well.

Each expression of this function is executed in a row context.

*For example :*</br>

```DAX
EVALUATE
SELECTCOLUMNS (
    'Product',
    "Category",RELATED('Product Category'[Category]),
    "Color", 'Product'[Color],
    "Name", 'Product'[Product Name],
    "Sales Amount", [Sales Amount]
)
```
## `ADDCOLUMNS()`

The `ADDCOLUMNS()` functions returns an existing table and also gives the flexibility to add more columns to it.

*For example :*</br>
If we want to create a separate table with *Sales Amount* column added to the *Products* table then, we have to write the following DAX expression :

```DAX
EVALUATE
ADDCOLUMNS (
    'Product',
    "Sales Amount", [Sales Amount]
)
```

### Using `ADDCOLUMNS()` & `SELECTCOLUMNS()` Together

```DAX
EVALUATE
ADDCOLUMNS (
    SELECTCOLUMNS (
        'Product',
        "Color", 'Product'[Color],
        "Product Amount", [Sales Amount]
    ),
    "Product Color Amount", [Sales Amount]
)
```












## UNION()
---

- `UNION()` helps us to combine two or, more tables having the same number of columns.
- Duplicate outcomes can be seen with the use of `UNION()` which can be eliminated using `DISTINCT()` function.
- Let's illustrate this using an example -:

Let's create two tables named "SunMon" and "MonTue" from the "Dates" table (Hence, related with each other and with the "Sales" Table) and another table named "MyMonTue" which is not related to any of the existing tables.

```dax
EVALUATE

VAR SunMon =
	CALCULATETABLE(
		VALUES('Date'[Day of Week]),
		'Date'[Day of Week Number] IN {1,2}
	)

VAR MonTue =
	CALCULATETABLE(
		VALUES('Date'[Day of Week]),
		'Date'[Day of Week Number] IN {2,3}
	)

VAR MyMonTue = {"Monday", "Tuesday"}

RETURN
	ADDCOLUMNS(
		SunMon,
		"Sales", [Sales Amount]
		)
```
The Return statement will execute two row elements, i.e, "Sunday" and "Monday" with their respective "Sales Amount".

Similarly, if we want the "Sales Amount" for "Monday" and "Tuesday" then, we can write the following DAX code as the `RETURN` statement

```dax
RETURN
	ADDCOLUMNS(
		MonTue,
		"Sales", [Sales Amount]
		)
```
For the following `RETURN` statement, the "Sales Amount" number will be executed incorrectly as "MyMonTue" table don't have any relationship with the "Sales Table" -:

```dax
RETURN
	ADDCOLUMNS(
		MyMonTue,
		"Sales", [Sales Amount]
		)
```

To combine the results of two tables in a single table, we can use `UNION()` in the `RETURN` statement -:

```dax
RETURN
	ADDCOLUMNS(
		UNION(SunMon, MonTue),
		"Sales", [Sales Amount]
		)
```

The result of this will consist a total of 4 row items and their respective "Sales Amount" with "Monday" repeated once.

To remove the repeatation, we can use  `DISTINCT()` before `UNION()` in the `RETURN` argument -:

```dax
RETURN
	ADDCOLUMNS(
		DISTINCT(UNION(SunMon, MonTue)),
		"Sales", [Sales Amount]
		)
```
In case, we use "MyMonTue" table in `UNION()` with "MonTue" table then, we will get the same 4 outcomes but, the "Sales Amount" against each row element will be same and incorrect.

This is because, the former table don't have any relationship with any of the xisting tables.

## INTERSECT()
---

- The order of the arguments provided inside `INTERSETCT()` plays a key role because, it executes the element(s) of the first argument common with the rest of the arguments.

```dax
RETURN
	ADDCOLUMNS(
		INTERSECT(SunMon, MyMonTue),
		"Sales", [Sales Amount]
		)
```
For example, if we intersect "SunMon" and "MyMonTue" table keeping the "SunMon" table first within the `INTERSECT()` argument then, eventhough, "MyMonTue" table don't have any relationship with "Sales Table" but, the result will come perfectly.
This is because, `INTERSECT()` first takes out "Monday" from the "SunMon" table (As this is the common element between the two intersecting tables) as the result and then, shows the "Sales Amount" against it.

```dax
RETURN
	ADDCOLUMNS(
		INTERSECT(MyMonTue, SunMon),
		"Sales", [Sales Amount]
		)
```
But, if we use, "MyMonTue" table as the first argument then, the result will be incorrect because, the "Monday" will be filtered out by `INTERSECT()` will be from the "MyMonTue" table that don't have any relationship with the "Sales Table".

## EXCEPT()
---

- Just like the `INTERSECT()`, the order of the arguments is important in case of `EXCEPT()` as well.
- `EXCEPT()` filters out the element(s), which is present in the first table but, not in the tables followed.
- The execution process is similar to that of `INTERSECT()`.

```dax
RETURN
	ADDCOLUMNS(
		EXCEPT(MonTue, SunMon),
		"Sales", [Sales Amount]
		)
```
In the result of we can see only "Tuesday" ("Tuesday" is present only in the first table)and its corresponding "Sales Amount".

## GROUPBY()
---
- In some of the cases, where we can't use `SUMMARIZE()` or, `SUMMARIZECOLUMNS()` then, we can use `GROUPBY()` function to aggregate the numbers.
- For example, let's create a table using DAX where we will define "Class" of the products based on their "Sales Amount"

```dax
EVALUATE

VAR ProdAndSales =
	ADDCOLUMNS(
		ALL('Product'[Product Name]),
		"Sales", [Sales Amount]
	)

VAR ProdAndClass =
	ADDCOLUMNS(
		ProdAndSales,
		"Class",
		SWITCH(
			TRUE(),
			[Sales] <= 100000, "Low",
			[Sales] <= 700000, "Medium",
			"High"
			)
	)

RETURN ProdAndClass
ORDER BY [Sales] DESC
```
The `RETURN` statement will execute a table with 3 columns, i.e, "Product Name", "Sales" and "Class".

Now, in order to see the "Sales Amount" by class, we can use the following `RETURN` statement -:

```dax
RETURN
	GROUPBY(
		ProdAndClass,
		[Class],
		"Total", SUMX(CURRENTGROUP(),[Sales])
		)
```

The above code will execute a table with 3 row elements,i.e., "Low", "Medium" and "High" and their corresponding "Sales Amount".
