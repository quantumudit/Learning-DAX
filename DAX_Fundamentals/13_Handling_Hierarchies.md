# HIERARCHIES

Hierarchies are pre-defined exploration path, like -:

- Year > Semester > Quater > Month > Day
- Category > Sub-category > Product

Hierarchies make browsing a model easier.

<u>**_Types of Hierarchy_**</u>

- **Natural Hirarchies**

For natural hierarchies, there exists a _1:M relationship_ between the parent and children

- **Unnatural Hierarchies**

The data is shuffled in case of unnatural hierarchies.

<u>**_Properties of Hierarchy_**</u>

- All columns in a hierarchy need to belong to the same table.
- Therefore, if we have multiple tables then, we can use the `RELATED()` function to get the columns in right place and we may hide those tables, thereafter.

## Filtering & Cross-filtering

We know that the filters from the _One-Side_ of the table automatically propagated to the _Many-Side_ of the table automatically.

To verify whether the a table is getting filtered by a certain coulmn from the _One-Side_ of the table or, not; we can use a function named `ISFILTERED()`

### `ISFILTERED()`

---

This function returns a boolean value as the output. Therefore, if the filtering is happening then, it will return `TRUE` otherwise `FALSE`.

If we have a slicer of `Brands` in the report then,to verify whether the `Product[Brand]` is filtering the `Sales` Table or, not ; we have to write the following DAX expression :

```DAX
Check = ISFILTERED ( Product[Brand] )
```

### `ISCROSSFILTERED()`

---

When the `Product[Brand]` is filtering the `Sales` table, then, the other columns in the "_Products_" table (_One-Side_) along withe the `Sales` table gets cross-filtered.

Therfore, the result of the below expressions will be returned as `TRUE`

```DAX
Check = ISCROSSFILTERED ( Product[Colors] ) //Returns TRUE
```

and

```DAX
Check = ISCROSSFILTERED ( Sales[Net Price] ) // Returns TRUE
```

**_When we have a slicer from the Many-Side of the table_**

Let's say we have the `Sales[Quantity]` slicer present in the report then, the result of the following functions would be as follows :

```DAX
Check = ISFILTERED ( Sales[Quantity] ) // Returns TRUE
```

and any other column from the "_Sales_" table gets cross-filtered, thus, for `Sales[Net price]` column :

```DAX
Check = ISCROSSFILTERED ( Sales[Net Price] ) // Returns TRUE
```

But, as the slicer belongs to the _Many-Side_ of the relationship, therefore none of the tables in the _One-Side_ gets cross-filtered, therefore, the result of the following DAX expression will be `FALSE`:

```DAX
Check = ISCROSSFILTERED ( Product[Colors] ) // Returns FALSE
```

## Percentage Over Parent

If we have a hierarchy as :

**_Category > Sub-Category > Products_**

then, the percentage over the parent refers to :

- Percentage of _Products_ over its belonging _Sub-Category_
- Percentage of _Sub-Categories_ over its belonging _Category_
- Percentage of _Categories_ over the _Grand Total_

To perform the above calculation with a single measure, we are going to use `ISINSCOPE()` as a key function.

### `ISINSCOPE()`

---

`ISINSCOPE()` is similar to `ISFILTERED()` but, it also checks that the column is currently used as a group-by column or, not.

This function is very much useful to avoid detecting the slicers as filters.

To calculate the "_Percentage over the parent_" calculation, we have to write the following DAX expression :

```DAX
Percentage to Parent =
VAR CurrentSales = [Sales Amount]
VAR SubCatSates =
    CALCULATE(
        [Sales Amount],
        ALLSELECTED('Product'[Product Name])
    )
VAR CatSales =
    CALCULATE(
        [Sales Amount],
        ALLSELECTED('Product'[Subcategory])
    )
VAR TotalSales =
    CALCULATE(
        [Sales Amount],
        ALLSELECTED('Product')
    )

VAR RatioToParent =
    IF(
        ISINSCOPE('Product'[Product Name]),
        DIVIDE(CurrentSales,SubCatSates),
        IF(
            ISINSCOPE('Product'[Subcategory]),
            DIVIDE(CurrentSales,CatSales),
            IF(
                ISINSCOPE('Product'[Category]),
                DIVIDE(CurrentSales,TotalSales)
            )
        )
    )

RETURN
RatioToParent
```

## Parent-Child Hierarchy

In some scenarios, we can also have ragged/unbalanced/parent-child hierarchies as follows :

<img src = "../Images/Ragged_Hierarchy.png" width = "550">

In such hierarchies, data is associated with both leaves and nodes.

While _SSAS(Tabular)_ has features to handle such parent-child hierarchies but, neither _Power BI_ nor _Power Pivot_ has such luxury.

Therefore, we need to perform some work-around to deal with this kind of problem.

As a desired solution to such problem, we expect

- The numbers in _child-node_ must add-up to give the number of their respective _parent-node_.
- The _Leaves_ shouldn't expand further.

**_Solution_**

The parent-child hierarchy shown in the above image can be represented in a table format as follows :

| PersonKey | Name      | ParentKey |
| --------- | --------- | --------- |
| 1         | Bill      |           |
| 6         | Annabel   |           |
| 7         | Catherine | 6         |
| 8         | Harry     | 6         |
| 9         | Michale   | 6         |
| 2         | Brad      | 1         |
| 3         | Julie     | 1         |
| 4         | Chris     | 2         |
| 5         | Vincent   | 2         |

### `PATH()`

The `PATH()` function performs the parent-child navigation recursively and we can create a calculated column for getting the full path of each node as follows :

```DAX
FullPath =
PATH ( Persons[PersonKey], Persons[ParentKey] )
```

After execution, the modified table will be as follows :

| PersonKey | Name      | ParentKey | FullPath |
| --------- | --------- | --------- | -------- |
| 1         | Bill      |           | 1        |
| 6         | Annabel   |           | 6        |
| 7         | Catherine | 6         | 6/7      |
| 8         | Harry     | 6         | 6/8      |
| 9         | Michale   | 6         | 6/9      |
| 2         | Brad      | 1         | 1/2      |
| 3         | Julie     | 1         | 1/3      |
| 4         | Chris     | 2         | 1/2/4    |
| 5         | Vincent   | 2         | 1/2/5    |

### `PATHITEM()`

Now, we need to extract the various levels of hierarchy as the calculated columns as follows :

**For Level-1**

```DAX
Level-1 =
LOOKUPVALUE(
    Persons[Name],Persons[PersonKey],PATHITEM(Persons[FullPath],1,INTEGER)
)
```

**For Level-2**

```DAX
Level-2 =
IF(
    PATHLENGTH(Persons[FullPath]) >= 2,
    LOOKUPVALUE(
        Persons[Name],Persons[PersonKey],PATHITEM(Persons[FullPath],2,INTEGER)
    ),
    Persons[Level-1]
)
```

**For Level-3**

```DAX
Level-3 =
IF(
    PATHLENGTH(Persons[FullPath]) >= 3,
    LOOKUPVALUE(
        Persons[Name],Persons[PersonKey],PATHITEM(Persons[FullPath],3,INTEGER)
    ),
    Persons[Level-2]
)
```

### `PATHLENGTH()`

The `PATHLENGTH()` simply gives the number of parents a node has,i.e., it is used to calculate the node depth.

The formula to calculate the node depth is as follows :

```DAX
NodeDepth = PATHLENGTH(Persons[FullPath])
```

After adding all these columns, the table looks as follows :

| PersonKey | Name      | ParentKey | FullPath | Level-1 | Level-2   | Level-3   | NodeDepth |
| --------- | --------- | --------- | -------- | ------- | --------- | --------- | --------- |
| 1         | Bill      |           | 1        | Bill    | Bill      | Bill      | 1         |
| 6         | Annabel   |           | 6        | Annabel | Annabel   | Annabel   | 1         |
| 7         | Catherine | 6         | 6/7      | Annabel | Catherine | Catherine | 2         |
| 8         | Harry     | 6         | 6/8      | Annabel | Harry     | Harry     | 2         |
| 9         | Michale   | 6         | 6/9      | Annabel | Michale   | Michale   | 2         |
| 2         | Brad      | 1         | 1/2      | Bill    | Brad      | Brad      | 2         |
| 3         | Julie     | 1         | 1/3      | Bill    | Julie     | Julie     | 2         |
| 4         | Chris     | 2         | 1/2/4    | Bill    | Brad      | Chris     | 3         |
| 5         | Vincent   | 2         | 1/2/5    | Bill    | Brad      | Vincent   | 3         |

To show the "_Persons_" and their "_Sales Amount_" in a proper hierarchical manner in the UI, we need to calculate the "_Sales Amount_" as follows :

```DAX
Sales Amt =
IF (
    MAX ( Persons[NodeDepth] ) < [BrowswDepth],
    BLANK (),
    SUM ( Sales[Amount] )
)
```

Where, `[BrowseDepth]` is defined as :

```DAX
BrowswDepth =
ISINSCOPE ( Persons[Level-1] ) + ISINSCOPE ( Persons[Level-2] )
    + ISINSCOPE ( Persons[Level-3] )
```

The `ISINSCOPE()` returns a boolean values (0/1) and by summing those values, we exactly get the browsed node depth in the UI.

But, there is still a problem associated,i.e., as soon as we add some other measure apart from "_Sales Amount_" that doesn't take this "_Browse Depth_" and "_Maximum Node Depth_" calculation into consideration, then, we will lose the desired hierarchy format.

Another problem associated with this kind of hierarchy format is; we are not able to see the individual "_Sales Amount_" for the top level parent (in this case, they are _Annabel_ and _Brad_).

If we want to see that, then, we have to pay the cost of slightly breaking the hierarchical structure,i.e., the top level parent nodes will appear twice (One showing their individual amount and the other showing the total amount of the hierarchy stream).

The DAX function to show such result is :
