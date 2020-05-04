# Evaluation Context

### RANKING (USING THE CONCEPTS OF EVALUATION CONTEXT)
---

We can create a calculated column that ranks the products based on their price as follows :

```DAX
Rank =
VAR CurrentPrice = 'Product'[Price]
RETURN
    COUNTROWS ( FILTER ( 'Product', 'Product'[Price] > CurrentPrice ) + 1 )
```
- In the above DAX expression, `FILTER()`creates an inner row context.
- To use an outer row context inside the inner row context we have to use variables.

But, the ranking result we get from the above DAX expression is not perfect.

***For example :***</br>
If 4 products have same price then, it will rank all of them as 1 but, for the 2nd highest value, the expression will give the rank as 5 as its current row context has 4 products greater than its price.

But, we expect the rank to be 2 and not 5.

We can write the same code even without using VAR as follows (This is not the best practice at all) :

```DAX
Rank =
COUNTROWS (
    FILTER ( 'Product', 'Product'[Price] > EARLIER ( 'Product'[Price] ) ) + 1
)
```
- `EARLIER()` doesn't mean "*Previous row*" rather it generates the outer row context just like the `VAR`. So, it works perfectly even if the "*Price*" column is not sorted.

***Fixing the ranking issue :***

In the previous formula, we are actually counting the number of products that has price greater than the current row context but,
all we need is to count the price of the products and do the ranks accordingly.

So, a correct formula would be :

```DAX
Rank =
VAR CurrentPrice = 'Product'[Price]
RETURN
    COUNTROWS (
        FILTER ( ALL ( 'Product'[Price] ), 'Product'[Price] > CurrentPrice ) + 1
    )
```

### Evaluation Context and Relationships :
---

If we are in the one-side of the related tables and want to access the values from the many-side table then :

`RELATED(Table[Column])` : Opens a new row context on the target table *(One-side of the related tables)* by following the relationship.

If we are in the many-side of the related tables and want to access the values from the one-side table then :

`RELATEDTABLE(Table)` : Filters the parameter table *(Many-side of the related tables)* by following the relationship and returns only rows
related to the current one.

>***Notes :***</br> While filter context automatically moves through the relationships, row context needs functions like `RELATED()` to move between related tables.

### Filters & Relationship
---

By default, filter context always propagates from the one-side to the many-side of the relationships *(Typically, from dimension tables to the fact table)*

But, there is a option of cross-filtering, where developers can choose to alter the direction of filter propagation as well.
