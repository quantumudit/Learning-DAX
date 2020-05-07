# ITERATORS


### `MINX()` & `MAXX()`

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
