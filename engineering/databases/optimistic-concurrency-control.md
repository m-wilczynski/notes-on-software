[Databases](/engineering/databases)
# Optimistic concurrency control

### Definition

**Optimistic concurrency control** (OCC) is a way of guaranteeing correct results  when concurrently modifying state in database/filesystem etc. without locking the resource like database record (which would be a **pessimistic concurrency control**). 

It has its advantages:
- no locking, 
- as a result -> no deadlocks and no need to manage them

but also introduce some challenges:
- managing versions in storage, 
- passing them to user and back, 
- introducing version mismatch strategies, ie. rollback or retry with user consent etc.). 

### Persistence level

There are two levels of actually implementing optimistic concurrency.

First deals only with persistence due to how we read and write data in RDBMS like SQL Server. Default behavior is to place locks on both reads and writes which hurts performance a lot. SQL engines like SQL Server  (with RCSI) solve it by allowing **"optimistic reads"**, ie. storing versions of rows in tempdb (in SQL Server case at least) and reading from it instead of actual table.

If we want to update row read under such circumstances, we can:
- enforce restrictive isolation level (which would be **"pessimistic writes"** strategy) 
- attempt to update with additional `WHERE` clause, that will point to an actual version and fail if no record is updated (which would be **"optimistic writes** strategy);

Optimistic write can be semi-automated with `SNAPSHOT` isolation in SQL Server if we want to be sure, that we are updating same version of row that we retrieved in same transaction.

### HTTP (or transport) level

This however does not solve the problem in web applications, where read and write are actually seperate HTTP calls! We might be already updating database state based on a data, that is stale before we even issued HTTP call to update it.

To prevent such scenarios we need to retrieve row version on HTTP read call and then update only if the same version is in database when HTTP write call is executed.

To store, retrieve and update row version, many ORMs implement optimistic version control out-of-the-box. 
For example `EF Core` supports it with timestamp-based **ConcurrencyToken** (either with data annotation `[Timestamp]` or with `IsRowVersion()` in fluent API). 

In typical HTTP web application, such version of record is sent to user using `ETag` HTTP header.

On modifying system state, general approach is to compare concurrency token on modifying state (`UPDATE` for example) in `WHERE` clause, ie. `UPDATE my_table SET xyz = 123 WHERE id = @id AND concurrency_token = @token`. If `affected records != expected records` (for single update: 0 != 1) concurrency control error should be raised. 

Optimistic concurrency can not only be used in classic `user -> web app -> db` scenario, but also help in dealing with distributed state in service-service (especially async) communication, when mismatched versions on requested operation should lead to message rejection or compensation (or both).