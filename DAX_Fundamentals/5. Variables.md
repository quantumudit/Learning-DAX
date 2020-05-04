# DAX Variables

- Defining variables in DAX calculations helps to avoid repeating of subexpressions in the code.
- Let's say we want to calculate the "Average Delivery Days" with some conditions, then, we might use the following DAX expression to do so -:
```dax
Average Delivery Days = 
AVERAGEX(
    Sales,
        IF(
            Sales[Delivery Date] - Sales[Order Date] > 3,
            Sales[Delivery Date] - Sales[Order Date]
        )
)
```
- Although, the above code is absolutly correct but, we can see a lot of repeatation of column names that makes the code look more complex.
- We can write the same expression using variables in a much readable way, as follows -:

```dax
Avg Delivery Days =
AVERAGEX(
    Sales,
    VAR DeltaDays = Sales[Delivery Date] - Sales[Order Date]
    RETURN
    IF(
        DeltaDays > 3,
        DeltaDays
    )
)
```
- We can define multiple variables but, we can only able to return a single result.
- The scope of the variable lies in between `VAR` and `RETURN` and thus, we can't define any varibale after `RETURN` statement.
- The variables are local and constant values that can't be changed during the calculation and can pnly be applied to the native DAX expression.
- We can creat an variable by referring to the variable created prior to that, for example -:

```dax
Avg Delivery Days = 
AVERAGEX(
    Sales,
    VAR OrderDate = Sales[Order Date]
    VAR DeliveryDate = Sales[Delivery Date]
    VAR DeltaDays = OrderDate - DeliveryDate
    RETURN
    IF(
        DeltaDays > 3,
        DeltaDays
    )
)
```

In the above example, we have created a variable to store "OrderDate" and other to store "DeliveryDate" by referring to those two variables, we have created another variable "DeltaDays"

- The variable names don't take spaces (" ") in-between,therefore, we can follow either "Camelcase" or, "Snakecase" format to give the name of the variable.
