[Databases](/engineering/databases)
# SQL Server index fragmentation

### SQL Server data organization

SQL Server organizes data into 8KB `pages` and then into 8-page `extents` (64KB total). How data is stored physically depends on a clustered key of particular table. To have a quick "lookup" using particular column (or columns) that do not represent actual, physical order of data, indexes has been introduced (using additional structures that also take space). Both data and indexes are being kept on pages called - respectively - data and index pages.

### Logical index fragmentation
 **Index fragmentation** is a situation that occurs when logical order of pages used by an index does not correspond to physical order of those pages. 
 
 ##### What causes it? 
 In general `INSERT`, `UPDATE` and `DELETE` operations cause it, since when data do not fit into 8KB page, the page gets split along with (in general) half it going to new page. 
 
 ##### But why would it even cause performance problems? 
 Well if pages are contiguous on disk, SQL Server can read-ahead up to 64 (512KB) of them at once (sequentional read); when they are "scattered" around, SQL Server needs to perform more I/O operations (random access read) leading to highers resource use. That situation is also called **logical fragmentation**. Even when SSDs were introduced, it somehow mitigated (but not fully resolved) fragmentation issue, since sequentional, read-ahead is still more performant than random access.
 
### Actions to take

 To avoid index fragmentation affecting perfromance of DB, we can schedule or execute ad-hoc two operations:
 - `ALTER INDEX IX_my_index ON mytable REBUILD` - rebuilds index from scratch; it's an offline operation that locks out table during the process (if not using `ONLINE` option on Enterprise Edition); it's multithreaded and uses lots of I/O; statistics are updated after it finishes. It's all or nothing operation,
 - `ALTER INDEX IX_my_index ON mytable REORGANIZE` - squishes rows together and organizes the pages for physical order to match logical one ; reorganizing index does not lock it out (it's an online operation); it is singlethreaded but has low disk footprint (only 8KB - single temp page)
 

### Internal index fragmentation

 Another type of fragmentation is **internal fragmentation** which stands for situation, when page density is low (and so we have too many pages when compared to amount of data we store). In general its cause is low `FILLFACTOR` setting  - option that controlls how much index page is filled with index data **on leaf level**, but only ***on index creation and rebuild (so not all the time!)***; if we have low page fill/density, SQL Server needs to jump around the pages to get data, since it doesn't get it from fetching single page (it did not fit on it).