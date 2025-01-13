Now you will find log data on `/var/lib/postgres/data/log` and you'll find the .json file which we will be delving into.

# Selection of a user table records

```sql
SELECT * FROM users 
    WHERE email = 'test@mail.com';
```

The following output is stored on [q1.log](./q1.log)

Keys that we will be observing are `ps`, `message`, and `detail`

#### Technical Flow Summary

    1. Parsing : The SQL query is translated to structured parse tree

Raw SQL string is tokenized into keyword, symbols and etc.,
These tokens are converted to parse tree and syntax is checked at this process.
We can find this parsing phase from the first log itself **PARSER STATISTICS**

Note that ps [process state] is 'idle' at this point.

    2. Analysis : Name resolution, type checking and such semantic

Table name, column name, and such along with it's type for given privilage checking.
This can be observed on next log line by **PARSE ANALYSIS STATISTICS** 

Now the ps is 'SELECT' than idle since executing the query as parsing tree is completed.

    3. Rewrite : Examines for rules, views or triggers to transform query tree

In our example statement we haven't included any views or such so; let's skip this one
However keep in mind that, the query will be rewritten or transformed accordingly as per rule / view / trigger 
**REWRITER STATISTICS** on the next log line.


    4. Planning : Query tree transform into execution plan 

In this phase, multiple strategies are considered, such as sequential scans, index scans, join algorithms based on 
available index and such.
Planner chooses efficient execution path with factors like CPU , I/O and memory usage consideration.
There are 3 different general process involved in this planning.
**PLANNER STATISTICS** on the next log line.

    5. Execution : Chosen plan executed -- Data fetched, processed and returned

Executor processes the plan tree step by step. Interact with storage engine for RW 
Matching query condition found and returned back to the client.
**EXECUTOR STATISTICS** as found on the log line and final process.

#### Extra 

At the end of log, we can observe that it took duration of 5.714 milliseconds overall.

PARSE ANALYSIS STATISTICS [3.9 ms] and then PLANNER STATISTICS [0.6 ms] took the most of time as it had to resolve, analyze and come up with efficient path.

> Moving on next [walrecord.md](./walrecord.md)
