# RELATIONAL FUNCTIONS

There are two types of relational functions available in DAX -:

1. `RELATED()`
2. `RELATEDTABLE()`

## RELATED()
---

- `RELATED()` gives access to the values present in "One-side" to the "Many-side" of a relationship.
- For example -: If several products comes under one sub-category and there is a many-to-one relationship between the "Product" table and "Product Sub-category" table then, using `RELATED()` , we can create a sub-category column in the "Products" table -:

```dax
Sub-Category = RELATED('Product Subcategory'[Subcategory])
```
- Similarly, several "Sub-category" may come under a single "Category" and hence, there exist another many-to-one relationship between the "Product Sub-Category" table and "Product Category" table.

- So, we can see that the "Product" table is linked with "Product Category" table through the "Product Sub-Category" table and thus, we can also access the "Category Name" for each "Product" in the "Products" Table also using a single `RELATED()` function -:

```dax
Category = RELATED('Product Category'[Category])
```

