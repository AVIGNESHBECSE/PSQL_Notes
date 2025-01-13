Now you will find log data on `/var/lib/postgres/data/log` and you'll find the .json file which we will be delving into.

# Selection of a user table records

```sql
SELECT * FROM users 
    WHERE email = 'test@mail.com';
```

The following output is stored on [q1.log](./q1.log)


