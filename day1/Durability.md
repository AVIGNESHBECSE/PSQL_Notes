This property ensures that once transaction successfully committed, the data
presists forever even in the event of power failure, crash and such.

To achieve this, PostgreSQL uses Write-Ahead Log (WAL) or transaction logs.

# Write-Ahead Log [WAL]

It is a sequential and append only file where all changes in database are recorded
in this file.

1. **Before Modification**, PostgreSQL first writes description of changes to WAL
before any data page changes in database's shared memory. This description is called
WAL Record.

2. **WAL Record** : Type of operation [insert/delete/update], Transaction ID [XID],
location of data being stored / changed along with new / old data.

3. **Sequential Writing**, as said earlier, append only. It's faster on secondary
storage device on write operation. Also it uses `fsync()` system call for this to
store physical device than in cache.

4. **Transaction Committed** : PostgreSQL ensures all WAL record are written
in physical storage.



# Recovery and failure point

When system reboots or crashes, the content on in-memory data are erased and
thus PostgreSQL replays the WAL record to effectively redoing the changes made 
by committed transactions that were not yet reflected in the data files on disk

# Checkpointing

While WAL record seems effective, it causes issue on time consumption when large
record is replayed.

To reduce replay time and WAL record limit, periodically PostgreSQL performs
**checkpoint** 

Here's how it works : 

1. During checkpoint, PostgreSQL writes all modified data pages from buffer pool
to data files on disk.

2. A special checkpoint is written to WAL Record. Including all changes that
has been flushed to the data files.

3. During recovery, PostgreSQL replays WAL record from last checkpoint.

 
Now further detailed log regarding WAL records, checkpoint and such will be shared on day2 [log.md](../day2/log.md)

> Move to next [day2](../day2) now.

