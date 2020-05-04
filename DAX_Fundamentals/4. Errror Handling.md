# Error Handling in DAX

The error handling functions in DAX are -:
1. `ISERROR()`
2. `IFEFFROR()`

## ISERROR()
---
- Takes a single expression as input and based on its evaluation, it returns either TRUE or, FLASE.
- If the expression leads to an error then, it returns `TRUE` otherwise, `FALSE`.
- The sytax of `ISERROR()` is -:
> ISERROR (Expression)
- Let's illustrate with an example -:
```dax
ISERROR(Sales[GrossMargin] / Sales[Amount])
```

## IFERROR()
---
- `IFERROR()` returns an alternative result provided by the user if an error occurs in the calculation.
- The syntax of `IFERROR()` is as follows -:
 > IFERROR (Expression, Alternate Result)

- Let's illustrate with an example -:

```dax
IFERROR(
    Sales[GrossMargin] / Sales[Amount],
    BLANK()
)
```
- `BLANK()` is a function in DAX that returns just a blank.

## DIVIDE()
---
- Instead of using the divide operator (/), we can use the `DIVIDE()` to handle the division errors that might occur in the calculation.
- The `DIVIDE()` function takes in 3 arguments as input and its syntax is -:
> DIVIDE (Numerator, Denominator, [Alternate Result])

- The default `[Alternate Result]` of `DIVIDE()` function is `BLANK()`, so if we don't provide the `[Alternate Result]` then also in case of division errors, `DIVIDE()` will return `BLANK()`.
- Let's illustrate this with an example -:

```dax
DIVIDE(
    Sales[GrossMargin],
    Sales[Amount],
    0
)
```
In the above example, we want to return "0" in case of any error.

