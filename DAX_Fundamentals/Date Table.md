# Building A Date Table

- The auto generated date table by Power BI can sometimes take us into undesirable result. Therefore, it is always advised to create a date table and use it in calculations.
- Follow the following path  and untick the `Auto date/time` to turn-off the auto date & time table functionality of Power BI -:

> File > Opetions & Settings > Options > Date load

- The most commonly used date functions available in DAX are follows -:

## CALANDERAUTO()
---

* Automatically creates a date table based on the database content.
* The first argument is the last month of the fiscal year.

Syntax -:

````dax
Date = CALENDERAUTO (6)
````
## CALENDER()
---

* Returns a table with a single column named "Date" containing continuous set of dates in the given range (Inclusive).

Syntax -:

````dax
Date = 
CALANDER(
    DATE (2005,1,1),
    DATE (2015, 12, 31)
)
````
Or,

````dax
Date = 
CALANDER (
    MIN (Sales[OrderDate]),
    MAX (Sales[OrderDate])
)
````
## Creating A Date Table
---

Let's first create a date table using the simple `CALENDERAUTO()` function.

````dax
Date_Table_1 = CALENDERAUTO()
````
This will create a date table by taking the range from the earliest date to the latest date available in the dataset.

We mayn't require such a long date table for our calculations.

We can use filter and other modifiers for making our desired date table as follows -:

````dax
Date_Table_2 =

VAR FirstSalesDate = MIN(Sales[Order Date])
VAR YearFirstOrder = YEAR(FirstSalesDate) 

RETURN
    FILTER(
        CALENDARAUTO(), 
        YEAR( [Date] ) >= YearFirstOrder
        )
````
The above DAX formula will create a date table ranging from the earliest `Order Date` Present in the "Sales" Table to the latest date present in the dataset.

But, this date table will only have one column and thus, we need to extract various attributes like "Year", "Month Name", "Weekday Name" etc.

To do this, we have to create a date table with the following DAX code -:

```dax
Date Table = 

VAR FirstSalesDate = MIN( Sales[Order Date])
VAR YearFirstOrder = YEAR( FirstSalesDate)
VAR Dates =
    FILTER(
        CALENDARAUTO(), 
        YEAR([Date]) >= YearFirstOrder
    )
RETURN
ADDCOLUMNS(
    Dates,
    "Year", YEAR( [Date] ),
    "Month", FORMAT( [Date], "mmm"),
    "Month Number", MONTH( [Date]),
    "Quater", FORMAT( [Date], "\QQ"),
    "Day of Week", FORMAT( [Date], "ddd"),
    "Weekday Number", WEEKDAY( [Date])
)
```
After building this date table we need to mark it as a date table from the "Mark as Date Table" option present in the "Modelling" tab of Power BI.

Then, we can easily slice and dice the data using this date table by creating a relationship between `Date` column present in any fact table and `Date` column of the created date table.

## Special Case Scenario
---

* Let's assume there is a case where a fact table has two date columns,viz, `Order Date` and `Delivery Date`.

* We have already created a date table and created a relationship between `Date[Date]` and `Sales[Order Date]` and we know that we can't create multiple active relationship between two tables.

* So, although we can slice the `Sales[Order Date]` column using our date table but, we can't slice the `Sales[Delivery Date]`.

* Now, we have two options to resolve this issue.
    1. To create another "Date" Table and link it with the `Sales[Delivery Date]` column.
    2. To use the `USERELATIONSHIP()` function.

* Usually, it is always preferred to choose the 2nd option, i.e., to use the `USERELATIONSHIP()` function rather than creating another "Date" table.

* Example of `USERELATIONSHIP()` -:

```dax
//Calculating Delivery Amount

Delivered Amount = 
CALCULATE(
    [Sales Amount], 
    USERELATIONSHIP(Sales[Delivery Date],'Date Table'[Date])
)
```


