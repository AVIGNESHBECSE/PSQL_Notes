WAL records are stored within `/var/lib/postgres/data/pg_wal` and you will find long numbers as a file name.

Using `pg_waldump` tool command, provided along with postgres, you can read the given wal record file.

I've executed following on my terminal 

```bash
pg_waldump 000000010000000000000005

rmgr: XLOG        len (rec/tot):    114/   114, tx:          0, lsn: 0/05000450, prev 0/05000418, desc: CHECKPOINT_SHUTDOWN redo 0/5000450; tli 1; prev tli 1; fpw true; wal_level replica; xid 0:748; oid 16421; multi 1; offset 0; oldest xid 730 in DB 1; oldest multi 1 in DB 1; oldest/newest commit timestamp xid: 0/0; oldest running xid 0; shutdown
rmgr: Standby     len (rec/tot):     50/    50, tx:          0, lsn: 0/050004C8, prev 0/05000450, desc: RUNNING_XACTS nextXid 748 latestCompletedXid 747 oldestRunningXid 748
pg_waldump: error: error in WAL record at 0/50004C8: invalid record length at 0/5000500: expected at least 24, got 0

```

These WAL records shows detailed checkpoint and standby status records; 

WAL records indicate the beginning and end of transactions, as seen in following `RUNNING_XACTS` records.
special WAL records such as `CHECKPOINT_SHUTDOWN` capture state of database at certain point of time.
These records are identified by the Log Sequence Number [`lsn`] 
`tx` indicates direct transaction or associated ones. Here it is zero since no direct transactions are associated.
`nextXid` is the next transaction ID to be assigned

We can observe that `latestCompletedXid` is aligned with `nextXid` on the following checkpoint, indicating the **consistency**

Standby server use this information to understand which transactions might be in process and ensure they apply WAL records in consistent manner.
This avoids conflicits, or data inconsistencies during replication.

```sql
SELECT * FROM pg_control_checkpoint();
```

This returns details about the last checkpoint, helping you correlate checkpoint WAL records with internal state.

```json
{
  "checkpoint_lsn": "0/5000558",
  "redo_lsn": "0/5000500",
  "redo_wal_file": "000000010000000000000005",
  "timeline_id": 1,
  "prev_timeline_id": 1,
  "full_page_writes": true,
  "next_xid": "0:748",
  "next_oid": 16421,
  "next_multixact_id": 1,
  "next_multi_offset": 0,
  "oldest_xid": 730,
  "oldest_xid_dbid": 1,
  "oldest_active_xid": 748,
  "oldest_multi_xid": 1,
  "oldest_multi_dbid": 1,
  "oldest_commit_ts_xid": 0,
  "newest_commit_ts_xid": 0,
  "checkpoint_time": "2025-01-13 13:51:51+05:30"
}
```

```sql
SELECT * FROM pg_current_wal_lsn();
```

```
 pg_current_wal_lsn 
--------------------
 0/50006B8
(1 row)
```

We can infer that checkpoint occurred at given time and find the transaction ids as well.

Initially, the dirty page [modifications] obtained from client requst are processed
and stored within the shared buffers.

WAL record is created and stored into WAL buffer.

These buffers are flushed to disk on certain conditions, where it meet : 
	i) Buffer is full
	ii) Checkpoint occurrence
	iii) Trasaction commits

At first, it stores the WAL record on WAL segment files, which are we witnessing
as "redo_wal_file" stored under `pg_wal`. 

Typically on recovery, the WAL record are replayed from last checkpoint from WAL
segment files.

The structure of WAL segment file name is 
	i) Timeline ID : 00000001 
	ii) Log ID : 00000000 [Logical log file number]
	iii) Segment ID: 0000005 [Sequential segment within log]

Once checkpoint is reached, all dirty pages are written to disk.

Overall, life cycle of WAL Segment is

	1) Creation : New WAL Segment files are created as needed.
	2) Recycling : Instead of disposing, old ones can be reused to conserve resource.
	3) Archiving : `archive_command` allows special copy of segment files for backup
	4) Removal : Based on `wal_keep_size` and usability, it is disposed.

So, do note that, WAL record are replayed and used when recovery or crashed occurs.
Until then, the dirty pages on shared buffer are used for inserting or updating 
the database [disk].

These dirty pages on shared buffer are deferred in terms of writing into disk,
to improve I/O operation and such.

> Moving on next [index.md](./index.md)
