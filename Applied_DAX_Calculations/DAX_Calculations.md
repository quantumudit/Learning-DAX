# Applied DAX Calculations

Although there are lots of functions available in DAX but, in most of the cases, we don't need all of them very frequently.

In fact, there are some particular functions and calculations in DAX that we are going to need in most cases.

## Aggregation Functions

The most important aggregation functions used for calculating the key measures are as follows :

- `SUM()`

- `AVERAGE()`

- `MAX()`

- `MIN()`



```dax
Total Sales = SUM( Sales[Total Revenue] )
```







Similarly, we can use `COUNTROWS()` function to calculate the "*Total Transactions*" :

```dax
Total Transactions = COUNTROWS( Sales )
```
