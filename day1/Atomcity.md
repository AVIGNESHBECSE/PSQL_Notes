Atomicity guarantees that a transaction is treated as a single, indivisible unit of work

Let's put it to test by breaching constraints and other means

# Constraint Violation

```sql

BEGIN;

INSERT INTO users (email, password)
	VALUES ('test@mail.com', 'hashed_password');

INSERT INTO users (email, password)
	VALUES ('test@mail.com', 'hashed_another_password');

COMMIT;

```

Oh, also never expose plain password by any means; use hash and salting!

Okay, so we have violated the `Unique Constraint` and as you can observe email is occurring twice as same. First insertion is success but latter isn't.

Output would be,

```
BEGIN
INSERT 0 1
ERROR:  duplicate key value violates unique constraint "users_email_key"
DETAIL:  Key (email)=(test@mail.com) already exists.
ROLLBACK
```

Now if you try `SELECT * FROM users` you'll find 0 rows since it is rolled back.

Let's explore other ways to attempt the kind of failures as well.

# Data Type Error

```sql

BEGIN;

INSERT INTO users (email, password, is_active)
	VALUES ('test@mail.com', 'hashed_pass', 'really?');

COMMIT;

```

If you remember table schema, `is_active` is boolean. You can already guess the output.

```
BEGIN
ERROR:  invalid input syntax for type boolean: "really?"
LINE 2:         VALUES ('test@mail.com', 'hashed_pass', 'really?');
                                                        ^
ROLLBACK
```

# Timeout

```sql
SET statement_timeout = '100ms';

BEGIN;

INSERT INTO users (email, password) 
	VALUES ('test@mail.com', 'hashed_pass');

SELECT pg_sleep(1);

COMMIT;
```

We have simluted timeout by manually overriding default timeout value to 100 millisecond and then call PostgreSQL sleep command of 1 second.

```
SET
BEGIN
INSERT 0 1
ERROR:  canceling statement due to statement timeout
ROLLBACK
```

# Valid execution

```sql

BEGIN;

INSERT INTO users (email, password) 
	VALUES ('test@mail.com', 'hashed_pass');

COMMIT;

SELECT * FROM users;
```

This time, we have done a valid transaction and can be observed that row is inserted as from `SELECT` inference.

```
BEGIN
INSERT 0 1
COMMIT
 user_id |     email     |  password   | name | is_active |            created_at            
---------+---------------+-------------+------+-----------+----------------------------------
       6 | test@mail.com | hashed_pass |      | t         | 2025-01-10 12:19:19.021683+05:30
(1 row)
```

> Moving onto next `Consistency.md`

