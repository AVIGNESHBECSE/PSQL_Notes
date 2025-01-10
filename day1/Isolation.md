Isolation determines how and when changes are made by one transaction become
visible to other concurrently running transactions.

We all know about what concurrency means since in this modern era CPU cores are
increasing whereas CPU Frequency speed reaching an bottleneck as it requires
more and more power.

By any mean we would need some kind of control over how data accessed in such
environment, otherwise there could be possibility of data racing and other
concurrent issue leading to inconsistency.

So, when many transaction running simultaneously, accessing and modifying the same
data; database system offer different degree of isolation level balancing between
concurrency performance and data consistency.

# Level 1 : Read Uncommitted

Lowest isolation level where it's certain to have data racing and such abnormalites

It's never used in production as this doesn't provide any kind of prevention.

> *Note* : PostgreSQL does NOT support this

```sql
-- Session 1
BEGIN;

INSERT INTO users (email, password) 
	VALUES ('test2@mail.com', 'hashed_pass');

COMMIT;
```

```sql
-- Session 2
BEGIN;

SET TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;

SELECT * FROM users;

COMMIT;
```

Through we can't execute but we can infer that both are running from different
session and different connection


Suppose Session 2 runs before Session 1 commit, then it would display the test2
email as well.

# Level 2 : Read Committed

Technically this prevents dirty read that we seen in level 1 but still has issue
such as phantom read and non-repeatable read.

*Phantom Read* is where execution of query returns a set of rows but in re-execution 
of the same query in the same trasaction returns different set of rows due to 
changes in different trasaction changes that matches current query condition and
is committed.

*Non-Repeatable Read* is trasaction reads same row twice within same transaction
but gets different values.

The main difference between these two are 

> Non-repeatable reads involve updates or deletes to rows that have already been read.
> Phantom reads involve inserts or deletes that change the result set of a query.

For execution, keep two terminals for different session.

Terminal **1**

```sql
BEGIN;

SELECT COUNT(*) FROM users WHERE password = 'hashed_pass';

```

This outputs `1` as we know, in previous SQL statement, we had created one row.


Terminal **2**

```sql
BEGIN;

INSERT INTO users (email, password) 
	VALUES ('test2@mail.com', 'hashed_pass');

COMMIT;
```

Now we have inserted into the table through our different trasaction.


Terminal **1**

```sql
SELECT COUNT(*) FROM users WHERE password = 'hashed_pass';

COMMIT;
```

This outputs `2`, quite different from previous statement in same transaction.

What we have encountered is *phantom read* and to simluate non-repeatable read, 
instead of insertion, we could update the existing value in 2nd terminal and
with first terminal selecting details of rows

In such case we will find difference as well, so now we know how important the
timing is and to avoid this issue we can move with next isolation leve.

Terminal **1**

```sql
BEGIN;

SELECT * FROM users;
```

Terminal **2**

```sql
BEGIN;

UPDATE users SET password = 'reset' WHERE email = 'test@mail.com';

COMMIT;
```

Terminal **1**

```sql
SELECT * FROM users;

COMMIT;
```

So, it is non-repeatable read. Output :

```
WARNING:  there is already a transaction in progress
BEGIN
 user_id |     email     |  password   | name | is_active |            created_at            
---------+---------------+-------------+------+-----------+----------------------------------
       6 | test@mail.com | hashed_pass |      | t         | 2025-01-10 12:19:19.021683+05:30
(1 row)


BEGIN
UPDATE 1
COMMIT


 user_id |     email     | password | name | is_active |            created_at            
---------+---------------+----------+------+-----------+----------------------------------
       6 | test@mail.com | reset    |      | t         | 2025-01-10 12:19:19.021683+05:30
(1 row)

WARNING:  there is no transaction in progress
COMMIT
```

Remember, insertion and update are different, so does phantom read and non-repeatable reads are.


# Level 3 : Repeatable Read

This isolation level prevents the non-repeatable read that we observed from above.

However, it doesn't prevent phantom read; let's explore why.

In this context, we are going to refer PostgreSQL internally so not bothering on
other SQL inner working.

In this isolation level, it ensures any data read cannot be changed in different
transaction until current one ends.

Only 'changed' we mentioned, not the newly added rows. Okay, so PostgreSQL internally
uses MVCC (Multiversion Concurrency Control) which keeps versions of data visible
to different transaction based on their snapshot.

Locks are applied to data to ensure this integrity.

#### Working Mechanism Inside PostgreSQL:

1. When a transaction begins with `Repeatable Read`, it takes snapshot of database.

2. This snapshot ensures transaction sees consistent state of database throughout
the lifecycle.

3. Any subsequent read will return the saved one from snapshot.

4. However, new inserted row will appear in subsequent queries.


Do the same non-repeatable observation that we done earlier in above but in
first terminal on `BEGIN` replace it with `BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ`

In that case, we notice output to be 1 throughout.

Now, Transactions are assigned unique identifiers (Transaction IDs or XIDs).

When a transaction reads data, it sees the version of each row that was current
according to its snapshot.

Snapshot has minimum XID [xmin], maximum XID [xmax] and list of XID [xip_list]

Row modified by XID which are greater than xmin are considered to be Future and not visible.
And XID which lesser than xmax are potentially visible as it is completed before
the snapshot.

XIDs are 32-bit integers, so they eventually wrap around

Each row in PostgreSQL table has hidden system column that stores column related: 
 

# Level 4 : Serializable

Ensures strongest isolation and outcome of concurrently executing transactions 
is equivalent to some serial (one-after-another) execution of those transactions.

PostgreSQL uses Serializable Snapshot Isolation [SSI] which is built on concept
of  MVCC, snapshots, and predicate lock. This lock prevents the phantom read.

SSI monitors transactions for potential conflicts that could lead to serialization anomalies

If SSI detects a situation where the concurrent execution of transactions might 
produce a result that is not equivalent to any serial execution, it will 
abort one or more of the transactions involved in the conflict.

Predicate locking acquires lock the condition in `WHERE` clauses to prevent from 
insertion or updating the rows that matches the condition.

Phantom read example in isolation 2 that we've seen earlier could be reused but
with `BEGIN TRANSACTION ISOLATION LEVEL SERIALIZABLE` as starting to both 
terminal session.

Terminal 2 is aborted as we re-run with given modification on SQL statement

However, as degree of isolation increased, we can find raise in likely-ness on
deadlocks and performance overhead.

> Last of ACID, `Durability.md`
