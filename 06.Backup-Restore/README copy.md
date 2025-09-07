## Backup adn Restore: 


### Backup Types:
Oracle supports multiple backup types, depending on mode, scope, and tool used.

#### 1. Based on Backup Mode:

- **Cold Backup** (Consistent (সামঞ্জস্যপূর্ণ) Backup):
    - Database is shut down required before backup.
    - Backup files (datafiles, control files, redo logs) are consistent.
    - Incomplete recovery
    - Backup is consistent
    - Offline Backup (consistent)
    - You can only recover till the point where you had taken the backup
    - Restore is simple (just copy back files).
    - Downside → database downtime.
    - Backup Process::
        - Shut down the database: `SHUTDOWN IMMEDIATE;`.
        - Copy all data files, control files, and redo logs using OS utilities (e.g., cp, tar).
    - Restore Process:
        - Copy files back to their original locations.
        - Start the database: `STARTUP`;.
        - In ARCHIVELOG mode, archived logs can be applied for point-in-time recovery; in NOARCHIVELOG mode, changes after the backup are lost.

- **Hot Backup** (Inconsistent (অসামঞ্জস্যপূর্ণ) Backup):
    - Database remains open & running.
    - Backup is usually done with `RMAN` or by putting tablespaces in backup mode (legacy).
    - Needs archived redo logs for recovery.
    - Online Backup (inconsistent)
    - Common in production (no downtime).


#### 2. Based on Backup Method:

- **Physical Backup**:
    - Copy actual database files: datafiles, control files, redo logs, archived logs.
    - Usually done with `RMAN`.
    - Backing up the following types of files:
	    - Server parameter file (SPFILE)
	    - Control files
	    - Data files
        - Redo logs
	    - Archive logs
    - Used for disaster recovery.

- **Logical Backup**:
    - Extracts schemas, tables or rows, procedure into dump files.
    - Tools: Data Pump utility (`expdp`/`impdp`) or legacy `exp`/`imp`.
    - Performed using: `expdp user/pass DUMPFILE=backup.dmp SCHEMAS=schema_name`.
    - Useful for migrations, partial restores.
    - Suppliment to the Physical backups
    - Not good for disaster recovery.
    - Restore Process:
        - Uses: `impdp user/pass DUMPFILE=backup.dmp SCHEMAS=schema_name`.
        - 

#### 3. Based on RMAN Backup Type:
- **Full Backup**:
    - Copies all data blocks in the datafiles.
    - Includes all data files, control files, and optionally archived redo logs.
    - Performed using `RMAN` with the command: `BACKUP DATABASE;`.
    - Can be executed while the database is online (in ARCHIVELOG mode) or offline.
    - Independent of incremental levels.
    - Example: `RMAN> BACKUP DATABASE;`
    - Restore Process:
        - Restores all data files to their original or new locations using: `RESTORE DATABASE;`.
        - Recovery applies redo logs or incremental backups to bring the database to a consistent state.

- **Incremental Backup**:
    - Backups or copies only changed blocks since last backup.
    - Levels:
        - **Level 0** : Same as full backup, baseline. Incremental backup includes all used data file blocks like a Full backup.
        - **Level 1 Differential** : Backup all blocks after the latest Level 0 or Level 1 backup. This is the default type of the Incremental backup. Backs up blocks changed since the most recent Level 0 or Level 1 backup (default). 
        - **Level 1 Cumulative** : Backup all blocks changed since the last Level 0 backup, reducing restore time but requiring more space.
    - Uses `RMAN` command: `BACKUP INCREMENTAL LEVEL [0|1] [CUMULATIVE] DATABASE;`.
    - Restore Process:
        - Restores the Level 0 backup first (if needed), then applies Level 1 incremental backups using: `RESTORE DATABASE;`, `RECOVER DATABASE;`.
        - RMAN automatically selects incremental backups over archived logs for faster recovery.



#### 4. Based on Backup Scope:
- Whole Database Backup → All files of the DB.
- Tablespace Backup → Specific tablespaces only.
- Datafile Backup → Individual datafiles.
- Control File Backup → For recovering database structure.
- Archived Redo Log Backup → Needed for point-in-time recovery.


### Backup Formats:
- **Image Copies**: Bit-for-bit duplicates of files, restorable without processing.
- **Backup Sets**: Proprietary format with compression, suitable for tape.



### Restore and Recovery Overview:
After backup, you can restore and recover in different ways:

1. **Restore**: Copies backup files (data files, control files, etc.) to their original or new locations.

2. **Recovery**: Uses redo logs + archived logs to apply changes after restore, bringing DB to a consistent state.
    - Complete Recovery: Applies all redo to the current point-in-time.
    - Incomplete Recovery: Recovers to a specific time or SCN, requiring a RESETLOGS operation.

3. **Block Media Recovery**: Repairs specific corrupt blocks without restoring entire files, using `RECOVER BLOCK`


### Types of Restore/Recovery:

#### Complete Recovery:
- Restores the database to a fully consistent state by applying all available redo logs or incremental backups after restoring data files. 
- Used in ARCHIVELOG mode to recover all committed transactions up to the point of failure.
- Restores database to the most recent state (before failure).
- Requires all archived redo logs + online redo logs after the backup.
- Process:
    - Restore: Use `RMAN` to restore data files from a full or incremental backup: `RESTORE DATABASE;`.
    - Recover: Apply archived redo logs or incremental backups to roll forward changes: `RECOVER DATABASE;`.

#### Incomplete Recovery (Point-in-Time Recovery – PITR):
- Recovers the database to a specific point in time, System Change Number (SCN), or log sequence number, rather than the current state.
- Restores database to a previous time, SCN, or log sequence.
- Types:
    - Time-based: Recover until specific timestamp `'2025-09-03:14:00:00'`.
    - SCN-based: Recover until specific SCN `123456`.
    - Cancel-based: Apply logs until manually stopped.
- Useful when user errors occur (e.g., accidental `DROP TABLE`).
- Process:
    - Restore: Restore data files from a backup prior to the desired point: `RESTORE DATABASE UNTIL TIME '2025-09-03:12:00:00';`.
    - Recover: Apply archived redo logs up to the specified time, SCN, or log sequence: `RECOVER DATABASE UNTIL TIME '2025-09-03:12:00:00';`.


#### Tablespace Point-in-Time Recovery (TSPITR):
- Restores and recovers specific tablespaces instead of the entire database, useful when only certain data files are damaged or lost.
- Restore a single tablespace to a past point without affecting others.
- Process:
    - Restore: Restore the affected tablespace’s data files: `RESTORE TABLESPACE users;`.
    - Recover: Apply redo logs to the tablespace: `RECOVER TABLESPACE users;`.

#### Datafile Recovery:
- Restores and recovers individual data files, typically when a single file is corrupted or lost.
- Process:
    - Restore: Restore the specific data file: `RESTORE DATAFILE 'path/to/datafile.dbf';`.
    - Recover: Apply redo logs to the data file: `RECOVER DATAFILE 'path/to/datafile.dbf';`.


#### Block Media Recovery
- Recovers specific corrupted data blocks without restoring entire data files, minimizing downtime and resource usage.
- Process:
    - Identify corrupted blocks using tools like DBVERIFY, ANALYZE, or error messages in alert logs.
    - Recover blocks using: RECOVER DATAFILE 'path/to/datafile.dbf' BLOCK 123;.
    - RMAN uses block-level backups or full backups with archived redo logs to reconstruct blocks.
    - Example: `RMAN> recover datafile 4 block 123;`


#### Logical Recovery (Data Pump Import):
- Recovers specific database objects (e.g., tables, schemas) from logical backups created with Data Pump Export (`expdp`). 
- Process:
    - Import objects from a dump file: `impdp user/pass DUMPFILE=backup.dmp SCHEMAS=schema_name TABLE_EXISTS_ACTION=REPLACE`.
    - Optionally remap schemas or tablespaces during import.
    - Example: `impdp system/pass DUMPFILE=backup.dmp SCHEMAS=hr REMAP_SCHEMA=hr:hr_new`



#### Control File Recovery: 
-  Restores and recovers the database control file, which contains metadata about the database structure.
- Process:
    - Restore from a backup: `RESTORE CONTROLFILE FROM 'path/to/backup/controlfile';`.
    - Mount the database: `ALTER DATABASE MOUNT;`.
    - Recover the database if needed, then open: `RECOVER DATABASE;`, `ALTER DATABASE OPEN;`.



### Oracle Backup vs Restore/Recovery:

| **Category**   | **Backup Types**                                                                                                                          | **Restore/Recovery Types**                                                                                                                                                                             |
| -------------- | ----------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Mode**       | - **Cold (Consistent)**: DB shut down, consistent backup.<br>- **Hot (Inconsistent)**: DB open, needs archived redo logs.                 | - **Complete Recovery**: Bring DB to latest state.<br>- **Incomplete Recovery (PITR)**: Restore to past point (time, SCN, or log sequence).                                                            |
| **Method**     | - **Physical**: Datafiles, control files, redo logs (RMAN or OS copy).<br>- **Logical**: Data Pump (expdp/impdp), exports schemas/tables. | - **Logical Restore**: Import schemas/tables with Data Pump.<br>- **Physical Restore**: Replace datafiles, control files, redo logs.                                                                   |
| **RMAN Types** | - **Full Backup**: All blocks.<br>- **Incremental (Level 0/1 Differential, Cumulative)**.<br>- **Image Copy**: Exact copy of files.       | - **Block Media Recovery**: Restore only damaged blocks.<br>- **Tablespace PITR (TSPITR)**: Recover a single tablespace.                                                                               |
| **Scope**      | - **Whole Database**.<br>- **Tablespace Backup**.<br>- **Datafile Backup**.<br>- **Control File Backup**.<br>- **Archived Log Backup**.   | - **Database Restore**: Whole DB back.<br>- **Tablespace Restore**: One or more tablespaces.<br>- **Datafile Restore**: Individual file restore.<br>- **Control File Restore**: Recreate DB structure. |



### Flow Diagram:

```
          ┌───────────────┐
          │   BACKUP      │
          └──────┬────────┘
                 │
 ┌───────────────┼─────────────────┐
 │               │                 │
 ▼               ▼                 ▼
Cold/Hot     Physical/Logical   RMAN Types
 Backup          Backup        (Full, Incremental)
                 │
                 ▼
         Whole DB / TS / File
                 │
                 ▼
          ┌───────────────┐
          │   RESTORE     │
          └──────┬────────┘
                 │
 ┌───────────────┼─────────────────┐
 │               │                 │
 ▼               ▼                 ▼
Complete     Incomplete         Specialized
Recovery     Recovery (PITR)    (TSPITR, Block)

```










