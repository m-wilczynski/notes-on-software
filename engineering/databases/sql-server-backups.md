[Databases](/engineering/databases)
# SQL Server backups

### SQL Server data files

SQL Server has two (actually three but hey...) files behind the scenes: **.mdf** (Master Database File), that stores data and .**ldf** (Log Database File), that stores transactions, ie. changes to database. When change happens in database, it gets changed in-memory (in a buffer) and modified data is marked as dirty (dirty pages) and change is logged to transaction log in .ldf file on disk. Later on this change will be finally persisted to .mdf file asynchronously on so called `CHECKPOINT`.

### Backup types

SQL Server offers three types of backups: 
- full (.bak), 
- differential (.bak) 
- transaction log (.trn). 

The reason why three types of backups exist is to prevent data loss and to decrease recovery time when disaster occurs:
- full backup backups literally everything, that is currently in database (so it's costly to recover),
- differential backup consists of everything that changed since last full backup, so if full backup was on 1st day of month and you created differential each day, on 10th day of month you will have 10 backups, where 10th backup has all the 10 days (1d-10d) since full backup, 9th has 9 days (1d-9d) of changes sinces last backup and so on,
- transaction log backups are performed much more frequently (ie. per hour, per 15min - dependes od disaster recovery model in organization), since differential backups (not to mention full backups) take time.

The reason why we won't simply do full backup each X days + all of the transaction log backups is recovery time. In such model we would have to restore each transaction log in order, sequentially one after another after full backup restore. If we take transaction log backup per 15min, we would have to apply 96 .trn files after 1 .bak file to recover database from a backup day ago, instead of at most 16 .trn files, if we have differential backup every 4 hours.

### Recovery models

Full, differential and transaction log backups are used for different recovery models (which we could also call "backup strategies"):
- SIMPLE - consists of differential and full backups
- FULL - consists of transaction log, differential and full backups

Since we know recovery models and different backup types, a question may come to our mind: w*hy don't we give up on transaction log backups though (that's what SIMPLE recovery model actually is)?* 

First of all, we can't practically make frequent diferrential backups on larger/busy databases effectively. Also, if we recall how data is actually written from .ldf to .mdf file, we can think of a scenario, when we lose data, that was not yet written to .mdf file (but that could be somehow mitigated with recovery interval settings on server to issue frequent checkpoints at cost of busy I/O).