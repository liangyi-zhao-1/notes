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
