# SQL notes
## When to use composite index
1. need to see the distribution of the data
2. need to look the index whether it worth

## when the index is useful
- if the main search param is the index
  
## Use UNION ALL when query has OR

## When query with optional params
1. Separate each param to one UNION ALL
2. Use OPTION(RECOMPILE)


## Date
### Add Date
```SQL
-- MySQL
DATE_ADD('2025-10-11', INTERVAL 1 DAY)
DATE_SUB('2025-11-12', INTERVAL 1 DAY)
-- MS SQL
DATEADD(DAY, -1, '2025-11-12')
```

## Rounding
```SQL
ROUND(123.45, -2) --100.00
```
