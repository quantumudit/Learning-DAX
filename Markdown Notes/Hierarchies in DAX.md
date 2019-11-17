# HIERARCHIES
---
Hierarchies are pre-defined exploration path, like -:

- Year > Month > Day
- Category > Sub-category > Product

Hierarchies make browsing a model easier.

## Natural Hirarchies
---
For natural hierarchies, there exists a 1:M relationship between the parent and children

## Unnatural Hierarchies
---
The data is shuffled in case of unnatural hierarchies.

## Hierarchies Across Tables
---

- All columns in a hierarchy need to belong to the same table. 
- Therefore, if we have multiple tables then, we can use the `RELATED()` function to get the columns in right place and we may hide those tables, thereafter.

## FILTER() & CROSSFILTER()
---

