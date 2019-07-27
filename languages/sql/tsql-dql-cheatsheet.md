[SQL](/languages/sql)
# T-SQL DQL Cheatsheet

You can find here both super obvious and less used functions - it's quite subjective. I tried to note down those that I would use less often - those that are more prone to be forgotten.

#### Table of contents
- [LIKE operators](/languages/sql/tsql-dql-cheatsheet?id=like-operators)

#### `LIKE` operators

| Operator | Usage |
--- | ---
| `%` | Most common - any 0..n characters |
| `_` | Any, exactly one character |
| `[]` | Exactly one character that matches phrase in brackets, ie. [a-c] will match one char of value 'a', 'b' or 'c' | 
| `^` | NOT operator, useable with any of the operators above |

Note that use of wildcard operator on the beginning of the string predicate is non-SARGable. SARGability needs separate note but in a nutshell - RDBMS would not be able to perform seek on column with leading wildcard in a query (ie. `%non-sargable`, `_non-sargable` etc.).

Examples:
```sql
    SELECT name
    FROM employees
    -- Starts with 'J' and and 0..n chars afterwads 
    WHERE name LIKE 'J%'
    OR
    -- Starts with 'J' and exactly one char afterwards
    name LIKE 'J_'
    OR
    -- Starts with 'J', 'K' or 'N' and 0..n chars afterwards
    name LIKE '[JKN]%'
    OR
    -- Starts with any char that is not from range 'a' to 'd' and one char afterwards
    name LIKE '^[a-d]_'
    -- And so on with usage inside string, on the end of it etc....
```

