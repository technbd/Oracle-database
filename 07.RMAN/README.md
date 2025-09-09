## RMAN:

RMAN (Recovery Manager) is Oracle’s built-in utility for backup, restore, and recovery of Oracle databases. RMAN is a powerful tool provided by Oracle for backing up, restoring, and recovering Oracle databases. It is integrated into the Oracle Database software and is designed to simplify and automate database backup and recovery operations.

- RMAN utility comes with the Oracle binaries
- No need installation and no license required 
- It defaults connect to database env variables defined
- RMAN utility can be used only when your database is at least MOUNT stage



### Key Features:
- Integrated with Oracle Database, no separate installation, ships with Oracle. Integrates with Oracle Data Guard, RAC, and other Oracle features.
- **Block-level Backup** and **backup only used blocks**, making backups smaller and faster than raw file copies.
- Provides block-level corruption detection and repair.
- Automates backup and recovery tasks.
- Supports full, incremental, and cumulative backups.
- Recovery Options
    - Point-in-time recovery (PITR).
    - Tablespace or datafile recovery.
    - Block media recovery (only restore corrupted blocks).
- Native support for compressed and encrypted backups.
- Supports backup to disk, tape, or cloud storage (e.g., Oracle Cloud, AWS S3).



### Key Components of RMAN:
- **RMAN Client**: The command-line interface or GUI used to issue backup and recovery commands.
- **Target Database**: The Oracle database being backed up or restored.
- **Recovery Catalog** (optional): A separate database schema that stores metadata about RMAN operations, enhancing management for multiple databases. A recovery catalog provides centralized management for multiple databases and longer retention of backup history.
- **Control File Repository**: By default, RMAN stores backup metadata in the target DB’s control file.
- **Image Copies**: Exact byte-for-byte OS-level copies of datafiles.
- **Backup Pieces & Sets**: Files created during backups, containing database data, control files, or archived redo logs. Backup output files (binary, not plain copies).
- **Channels**: Connections RMAN uses to perform I/O operations (can be parallelized).



### RMAN Backup Types:
- **Full Backup**: Backs up all data blocks in the database.
- **Incremental Backup**:
    - Level 0 : Same as full backup, baseline. Incremental backup includes all used data file blocks like a Full backup.
    - Level 1 Differential : Backup all blocks after the latest Level 0 or Level 1 backup. This is the default type of the Incremental backup. Backs up blocks changed since the most recent Level 0 or Level 1 backup (default).
    - Level 1 Cumulative : Backup all blocks changed since the last Level 0 backup, reducing restore time but requiring more space.



### Backup Terminology:
- **Image Copies** (Dulicate): An image copy is an exact copy of a single data file, archived redo log file, or control file. Backups consume more space, file to file backup, its called snapshot backup. 
- **Backup Sets** (Merged together): Datafile backup sets are typically smaller than datafile **image copies** and take less time to write. Backups consume less space. Used to RMAN. 



### NORESETLOGS vs RESETLOGS:

#### NORESETLOGS:
- When all changes (up to the current point) have been applied via redo logs, and the database is fully consistent (e.g., recovery using all archived logs since the last backup, ending with the current redo log).
- Opens the database continuing with the existing redo log sequence.
- Opens the database without resetting the redo log sequence.
- Recovery using the current (non-backup) control file.
- Restored all datafiles and applied all redo logs (up to the last available SCN).
- Oracle defaults to NORESETLOGS for complete recoveries.
- Used after a complete recovery, when all datafiles, controlfile, and redo logs are consistent.
- Example:
```
STARTUP MOUNT;
RECOVER DATABASE;                       --> Applies all available redo (RMAN Command)
ALTER DATABASE OPEN NORESETLOGS;
```


#### RESETLOGS:
- When using recover control file or a recreated control file from a backup. The backup control file lacks the latest SCN, so RESETLOGS synchronizes it.
- Opens the database after resetting the redo log sequence back to 1.
- Previous redo logs cannot be applied beyond this point.
- Archives cannot be applied across the reset point.
- For incomplete recoveries or backup control files, RESETLOGS is mandatory.
- Example:
```
STARTUP MOUNT;
RECOVER DATABASE UNTIL TIME '2025-09-08:12:00:00';
RECOVER DATABASE UNTIL SEQUENCE 2500;                  --> Incomplete recovery if sequence 2501 is missing (RMAN Command)
ALTER DATABASE OPEN RESETLOGS;
```






### RTO: Recovery Time Object:

The maximum acceptable time it should take to restore service after a failure. The amount of time a system, application, or database can be down before it causes significant harm to the business.

#### In RMAN Context:
- RTO determines how quickly you need to restore and recover the Oracle database after a failure.
- RMAN supports faster recovery through features like:
    - Parallel Recovery: Uses multiple channels to speed up restore and recovery.
    - Incremental Backups: Reduces recovery time by applying only changed blocks.
    - Data Recovery Advisor: Automates diagnosis and repair, minimizing downtime.
- Example: If your RTO is 2 hours, you configure RMAN to prioritize fast restore operations, such as using high-parallelism disk channels or restoring from a local disk rather than tape.
- Command example to optimize for RTO:
```
CONFIGURE DEVICE TYPE DISK PARALLELISM 4;
RESTORE DATABASE;
RECOVER DATABASE;
```



### RPO: Recovery Point Object:
The maximum acceptable amount of data (time-wise) that can be lost due to a failure. The amount of data (measured in time) a business can afford to lose between the last backup and the point of failure.

#### In RMAN Context:
- RPO determines how frequently you need to backup to minimize data loss.
- Back up archived redo logs frequently.
- RMAN helps achieve low RPO through:
    - Frequent Incremental Backups: Captures changes since the last backup.
    - Archived Redo Log Backups: Archivelog backup frequency. Enables point-in-time recovery (PITR) to reduce data loss.
    - Real-Time Apply: In Data Guard setups, RMAN can back up logs applied in near real-time.
- Example: If your RPO is 15 minutes, you schedule frequent archived log backups and ensure redo logs are backed up regularly.
- Command example to support low RPO:
```
BACKUP INCREMENTAL LEVEL 1 DATABASE PLUS ARCHIVELOG;
```



### RMAN Backup Behavior:
- If no backup directory is mentioned → RMAN stores backups in the autobackup location. 
- If no mention spfile backup, rman spfile backup to autobackup directory.
- Stored in the autobackup destination (FRA by default).
- Autobackup destination = FRA (default) or `CONFIGURE` … path.
- If **FRA** is not configured → backups to `$ORACLE_HOME/dbs` (not recommended).
- Every backup or structural change, RMAN automatically backsup Control file + SPFILE. 
- If `CONFIGURE CONTROLFILE AUTOBACKUP ON` → RMAN automatically backs up:
    - Control file
    - SPFILE
- RMAN backs up archived logs after database files by default (`PLUS ARCHIVELOG`).
- RMAN automatically checks blocks during backup.
- If FRA is full, Oracle starts deleting obsolete/backup-redundant files first.
- If space still insufficient → database operations may hang until space is freed.
- When RMAN starts a backup, **it first forces a log switch** to generate a new **archived redo log**. At the same time, a new **SCN** (System Change Number) is generated, which serves as a consistent checkpoint for the backup.




### RMAN Backup files:
- Datafiles
- Archived redo logs
- Control files
- Server Parameter file (SPFILE)



### License:
This project is licensed under the **MIT** License.



### Author:
- **Name:** Technbd   
- **Topic:** Oracle RMAN Backup & Recovery Notes  




Oracle Recovery Manager (RMAN) is the primary tool for backup, restore, and recovery in Oracle databases. It provides a reliable, automated, and efficient mechanism to protect data, ensuring business continuity.

