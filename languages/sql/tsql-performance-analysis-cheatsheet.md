[SQL](/languages/sql)
# T-SQL Performance Analysis Cheatsheet

Cheatsheet for analysing performance on SQL Server.

**Contents**
- [Lock analysis](/languages/sql/tsql-performance-analysis-cheatsheet?id=lock-analysis)
- [Inspecting query cache](/languages/sql/tsql-performance-analysis-cheatsheet?id=inspecting-query-cache)


#### Lock analysis

```sql
SELECT 
     request_session_id AS session_id, 
    ,DB_NAME(resource_database_id) AS [database],
    ,request_mode AS mode, resource_type as [type],  
    ,resource_associated_entity_id AS entity,
    ,resource_description,  request_status AS status
FROM sys.dm_tran_locks;
```

#### Inspecting query cache

```sql
SELECT 
     UseCounts
    ,Cacheobjtype
    ,Objtype
    ,TEXT
    ,query_plan
FROM sys.dm_exec_cached_plans 
CROSS APPLY sys.dm_exec_sql_text(plan_handle)
CROSS APPLY sys.dm_exec_query_plan(plan_handle)
```
**Source:** https://stackoverflow.com/a/7359705/2869055 by [Justin](https://stackoverflow.com/users/113141/justin) licensed under [CC-BY-SA 3.0](https://creativecommons.org/licenses/by-sa/3.0/)
