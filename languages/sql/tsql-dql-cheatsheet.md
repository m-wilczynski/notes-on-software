[SQL](/languages/sql)
# T-SQL DQL Cheatsheet

You can find here both super obvious and less used functions - it's quite subjective. I tried to note down those that I would use less often - those that are more prone to be forgotten.

**Contents**
- [LIKE operators](/languages/sql/tsql-dql-cheatsheet?id=like-operators)
- [Window functions - rank](/languages/sql/tsql-dql-cheatsheet?id=window-functions-rank)

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
SELECT emp_name
FROM employees
-- Starts with 'J' and and 0..n chars afterwads 
WHERE emp_name LIKE 'J%'
OR
-- Starts with 'J' and exactly one char afterwards
emp_name LIKE 'J_'
OR
-- Starts with 'J', 'K' or 'N' and 0..n chars afterwards
emp_name LIKE '[JKN]%'
OR
-- Starts with any char that is not from range 'a' to 'd' and one char afterwards
emp_name LIKE '^[a-d]_'
-- And so on with usage inside string, on the end of it etc....
```

#### Window functions - rank

Rank is determined with `ORDER BY x` clause with aggregations defined by `PARTITION BY y`, ie: `MY_FUNC OVER (PARTITION BY y ORDER BY x)`

| Function | Usage |
| --- | --- |
| `ROW_NUMBER()` | Rank without ex aequo places in defined window |
| `RANK()` | Rank with ex aequo places in defined window, where next rank after shared places is additionally greater by `N-1` where `N = number of ex aequo places before it`, for example: 1, 2, 2, 2, 5  |
| `DENSE_RANK()` | Rank with ex aequo places in defined window without gaps, for example: 1, 2, 2, 2, 3 |
| `NTILE(how_many)` | Rank by distributing (almost) equally results into `how_many` baskets/groups in defined window  |

Examples:
```sql
SELECT
     emp_name
    ,emp_salary
    ,dept_id
    -- Employee's salary rank inside his/her department
    -- No ex aequo places
    ,ROW_NUMBER() OVER (PARTITION_BY dept_id ORDER BY emp_salary) emp_salary_rank1
    -- Ex aequo places with gaps
    ,RANK() OVER (PARTITION_BY dept_id ORDER BY emp_salary) emp_salary_rank2
    -- Ex aequo places without gaps
    ,DENSE_RANK() OVER (PARTITION_BY dept_id ORDER BY emp_salary) emp_salary_rank3,
    -- Is employee in group of better paid employees in his/her department (= 1) or not (= 2)
    ,NTILE(2) OVER (PARTITION_BY dept_id ORDER BY emp_salary) emp_salary_rank_group
FROM employees
```

#### References
- Adam Pelikant, *MS SQL Server, Zaawansowane metody programowania*
- Microsoft SQL Server docs - https://docs.microsoft.com/en-us/sql/t-sql