[SQL](/languages/sql)
# T-SQL DDL Cheatsheet

Useful script for defining database structures and managing SQL Server DB from T-SQL in general.

**Contents**
- [Find multicolumn PK](/languages/sql/tsql-ddl-cheatsheet?id=find-multicolumn-pk)

#### Find multicolumn PK

```sql
WITH multicolumn_pk_per_table AS
(
    SELECT 
        COUNT(con_col.COLUMN_NAME) count_pk
        ,tab_con.TABLE_SCHEMA AS sch
        ,tab_con.TABLE_NAME AS tab
    FROM
        INFORMATION_SCHEMA.TABLE_CONSTRAINTS tab_con, 
        INFORMATION_SCHEMA.CONSTRAINT_COLUMN_USAGE con_col 
    WHERE 
        con_col.CONSTRAINT_NAME = tab_con.CONSTRAINT_NAME
        AND con_col.TABLE_NAME = tab_con.TABLE_NAME
        AND CONSTRAINT_TYPE = 'PRIMARY KEY'
    GROUP BY tab_con.TABLE_NAME, tab_con.TABLE_SCHEMA
)
SELECT *
FROM multicolumn_pk_per_table
WHERE count_pk > 1;
```
