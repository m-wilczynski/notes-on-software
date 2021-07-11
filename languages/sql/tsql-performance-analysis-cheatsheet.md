[SQL](/languages/sql)
# T-SQL Performance Analysis Cheatsheet

Cheatsheet for analysing performance on SQL Server.

**Contents**
- [Lock analysis](/languages/sql/tsql-performance-analysis-cheatsheet?id=lock-analysis)
- [Inspecting query cache](/languages/sql/tsql-performance-analysis-cheatsheet?id=inspecting-query-cache)
- [Analyzing top longest running queries](/languages/sql/tsql-performance-analysis-cheatsheet?id=analyzing-top-longest-running-queries)


### Lock analysis

```sql
SELECT 
     request_session_id AS session_id, 
    ,DB_NAME(resource_database_id) AS [database],
    ,request_mode AS mode, resource_type as [type],  
    ,resource_associated_entity_id AS entity,
    ,resource_description,  request_status AS status
FROM sys.dm_tran_locks;
```

### Inspecting query cache

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


### Analyzing top longest running queries

```sql
WITH query_stats AS
(
    SELECT 
        stats.*,
        SUBSTRING
        (
            sql_executions.text,
            (stats.statement_start_offset/2) + 1,
            ((
                    CASE statement_end_offset
                    WHEN -1 THEN DATALENGTH(sql_executions.text)
                    ELSE stats.statement_end_offset 
                    END - stats.statement_start_offset
            )/2) + 1
        ) AS statement_text
    FROM sys.dm_exec_query_stats AS stats
    CROSS APPLY sys.dm_exec_sql_text(stats.sql_handle) as sql_executions
)
SELECT TOP 15
    query_hash                                          AS "Query Hash",
    SUM(execution_count)                                AS "Execution Count",       --Total executions of this query; if count is low, don't bother analyzing or even filter it out
    SUM(total_elapsed_time) / SUM(execution_count)      AS "Avg Elapsed Time",      --Elapsed time (all the operations required for query including waiting on blocking)
    SUM(total_worker_time) / SUM(execution_count)       AS "Avg CPU Time",          --Actual query time time (if much lower than elapsed, then troubleshoot blocking issues)
    SUM(total_logical_reads) / SUM(execution_count)     AS "Avg Logical Reads",     --Reads of data pages from buffer cache, ie. memory (if really high, perhaps an index is missing?)
    SUM(total_physical_reads) / SUM(execution_count)    AS "Avg Physical Reads",    --Reads of data pages from disk (if really high, it seems you have low cache hit ratio) 
    IIF(
        SUM(total_logical_reads) = 0, 0,
        (1.0*SUM(total_logical_reads) - SUM(total_physical_reads))
        / SUM(total_logical_reads) * 100
    )                                                   AS "Cache Hit Ratio (%)",   --Cache hit ratio (if low, perhaps invest in more RAM?)
    MIN(statement_text)                                 AS "Statement Text"
FROM query_stats
GROUP BY query_hash
ORDER BY 3 DESC;  

```

**Loosely based on:** https://docs.microsoft.com/en-us/sql/relational-databases/system-dynamic-management-views/sys-dm-exec-query-stats-transact-sql


