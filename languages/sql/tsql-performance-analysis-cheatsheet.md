[SQL](/languages/sql)
# T-SQL Performance Analysis Cheatsheet

Cheatsheet for analysing performance on SQL Server.

**Contents**
- [Lock analysis](/languages/sql/tsql-performance-analysis-cheatsheet?id=lock-analysis)


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