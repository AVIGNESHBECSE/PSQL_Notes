Consistency in the context of database transactions means that transaction must
maintain the database's integrity constraints and rules.

So, transaction should leave the database in valid state and rules are followed.

# Constraint Test

```sql

BEGIN;

INSERT INTO child_table (parent_id) 
	VALUES (1337);

COMMIT;

```

In here we can observe that parent_table is empty and child_table insertion is
referring to empty value; thus, inconsistent.

Output : 

```
BEGIN
ERROR:  insert or update on table "child_table" violates foreign key constraint "child_table_parent_id_fkey"
DETAIL:  Key (parent_id)=(1337) is not present in table "parent_table".
ROLLBACK
```

# Invalid State Attempt

```sql

BEGIN;

INSERT INTO parent_table (value)
	VALUES(-1);

COMMIT;
```

Now we are trying to directly violate the schema constraints on parent_table
where value should be higher than 0

Output : 
```
BEGIN
ERROR:  new row for relation "parent_table" violates check constraint "parent_table_value_check"
DETAIL:  Failing row contains (1, -1).
ROLLBACK
```

> Let's go ahead with [Isolation.md](./Isolation.md), shall we?
