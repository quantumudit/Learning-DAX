# Coditional Functions

The conditional functions that we have in DAX are -:

1. `AND()`
2. `OR()`
3. `NOT()`
4. `IF()`

- All the conditional functions executes a boolean logic (TRUE/FALSE) and further executions happen based on that.

- The `AND()` / `OR()` can be expressed as follows -:

> AND (A , B) = A && B

> OR (A , B) = A || B

- `AND()` and `OR()` operators can only take two arguments but using their respective operators, we can concatenate multiple arguments.

- Let's create a calculated column in "Products" table named "Color Type" using conditional fuctions -:

```dax
Color Type = 
IF(
    OR('Product'[Color] = "Black" , 'Product'[Color] = "Pink"),
    "Trendy", "Boring"
)
```
let's say, we want to add "White" color along with "Black" and "Pink" as "Trendy" then we have to pass another `OR()` fuction that will take inner `OR()` as the first argument then `'Product'[Color] = "White"` as the second argument, as follows -:

```dax
Color Type = 
IF(
    OR(
        OR('Product'[Color] = "Black" , 'Product'[Color] = "Pink"), 
        'Product'[Color] = "White"
    ),
    "Trendy", "Boring"
)
```
But, instead of writing such complex code, we can just use the OR-operator, as follows -:

```dax
Color Type = 
IF('Product'[Color] = "Black" 
    || 'Product'[Color] = "Pink" 
    || 'Product'[Color] = "White",
    "Trendy", "Boring"
)
```
This code is relatively simple than the first one but, there is more advanced and readable way to write the same thing.

## The `IN` Operator
---
- The `IN` operator checks if the value belongs to a certain category or, not.
- The arguments inside the `IN` operators are separaed with comma sepators that acts like a OR-Operator.
- Using the `IN` operator, we can specify multiple OR-conditions in a more readable way.
- Let's say, we want to define "Black", "Pink", "White", "Purple" and "Red" as "Trendy" then, we can write the DAX expression as follows -:

```dax
Color Type = 
IF('Product'[Color] IN {"Black", "White", "Pink", "Purple", "Red"},
    "Trendy", "Boring"
)
```
Here, we can see that, we don't have to write `'Product'[Color]` multiple times as we have to do in case of `OR()` / `||`.

## The `SWITCH()` Function
---

- `SWITCH()` fuction is used to replace the complexity associated with Nested `IF()` clauses.
- `SWITH()` takes multiple arguments as input.
- The first argument in `SWITCH()` is the expression to be evaluated and the following arguments are just the conditions and their respective outputs applied to the expression.
- Let's say we want to categorize the colors into 4 categories, i.e., "Trendy", "Fashionable", "Simple" & "Boring" then, we can do so using `SWITCH()` as follows -:

```dax
Color Category = 
SWITCH('Product'[Color],
        "Black", "Trendy",
        "Pink", "Fashionable",
        "White", "Simple",
        "Boring"
)
```
### `SWITH()` with CASE WHEN
---
