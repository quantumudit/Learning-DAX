## CALCULATED COLUMNS
---

Let's create the following calculated columns in the "Sales" Table -:

1. Line Cost
2. Line Amount
3. Line Margin
4. Margin (%)

```dax
Line Cost = Sales[Quantity] * Sales[Unit Cost]
```
```dax
Line Amount = Sales[Quantity] * Sales[Net Price]
````
```dax
Line Margin = Sales[Line Amount] - Sales[Line Cost]

// We can create a new calculated column by referring to one or, more calculated columns
```
```dax
Margin (%) = Sales[Line Margin] / Sales[Line Amount]
```
## MEASURES
---

- "Calculated columns" calculates row-by-row and thus, the engine knows the current row, implicitely.

- "Measures" do the calculations at report level and thus every cell in the report is taken as input for "Measures".

- Therefore, in a "Measure", we need to provide an aggregator function, such as, `SUM()` before the column name.

- For example -: If we need to calculate "Margin (%)" as a measure, we have to write the following DAX expression

```dax
Margin (%) = SUM(Sales[Line Margin])/SUM(Sales[Line Amount])
```

- The "Margin (%)" created through "calculated column" provides a wrong answer when we roll-up the items, therefore, it is always advised to use "Measure" much frequently and only use "calculated columns" when its highly necessary.

- Therefore, instead of creating columns for "Line Amount" , "Line Cost" etc. , we can create "Measures" as follows -:

```dax
Sales Amount = SUM(Sales[Quantity]) * SUM(Sales[Net Price])
```
```dax
Total Cost = SUM(Sales[Quantity])*SUM(Sales[Unit Cost])
```
```dax
Margin = [Sales Amount]-[Total Cost]
```
As we have created all the required parameters to calculate "Margin (%)", hence, we can safely delete all the calculated columns that we have created earlier and can rewrite the "Margin (%)" as follows -:

```dax
Margin (%) = [Margin]/[Sales Amount]
```

