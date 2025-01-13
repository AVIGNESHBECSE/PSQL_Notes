The following operations are executed under "acid" database

```sql
CREATE DATABASE acid;
```

And you can login through `psql acid`

The table referred throughout the session is 

```sql
CREATE TABLE users (
    user_id SERIAL PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL,
    password VARCHAR(255) NOT NULL, 
    name VARCHAR(100),
    is_active BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);
```

```sql
CREATE TABLE parent_table (
    id SERIAL PRIMARY KEY,
    value INT CHECK (value >= 0) 
);
```

```sql
CREATE TABLE child_table (
    id SERIAL PRIMARY KEY,
    parent_id INT REFERENCES parent_table(id)
);
```

Now, I've already have grip on basic DDL, DML, TCL, DQL and DCL so I assume you already know how to use with other SQL operations and clauses like WHERE, HAVING, and etc

Starting off with next basics, *transaction*

Transaction is a sequence of one or more SQL operations that are treated as a single unit of work.

So, if half of operation executed and error occurred in middle or anywhere at point of execution, it reverts the changes like the original state of the database.

If all of the operations completed then it is *committed* or else it is *rolled back* in SQL context.

Basic syntax : 
```sql	
[BEGIN / BEGIN TRANSACTION / START TRANSACTION];

 -- SQL Statements here [like update, create and etc]

[COMMIT / ROLLBACK] 

 -- Automatic rollback is called when any of SQL statement in above fails. 
 -- Also Checkpoint can be declared within statements with ability to rollback onto given checkpoint.
```
That's all for initial basics here, moving forward with ACID properties and related SQL with each .md file

Oh, and if you're wondering why I'm using caps for command despite SQL being case insensitive, well it makes the statements more readable so keep your code and statements tidy!

> Move to [Atomicity.md](./Atomicity.md)
