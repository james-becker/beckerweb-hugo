---
title: "Displaying and Killing Open Connections via Postgres Queries"
date: 2019-09-06T15:27:25+07:00
draft: false
---

## Displaying and Killing Open Connections via Postgres Queries

The `pg_stat_activity` table can display all current connections to all databases. This is useful if you need to kill connections to perform some database-level activity, such as renaming an imported database.

```sql
select pid as process_id, 
       usename as username, 
       datname as database_name, 
       client_addr as client_address, 
       application_name,
       backend_start,
       state,
       state_change
from pg_stat_activity;
```

This query lists the `pids` of the connection processes you may want to kill.You can kill them, but before you can perform your database-level operation, you will need to prevent new client reconnections. To do this, the `pg_database` table needs to be updated, but this can only be done as a superuser. Instead, run:

```sql
ALTER database "my-db"
  with is_template true
       allow_connections false;
```

To kill all processes of open connections to a target database, you can now SELECT them and run pg_terminate_backend() via the following query:

```sql
SELECT pg_terminate_backend(pg_stat_activity.pid)
FROM pg_stat_activity
WHERE pg_stat_activity.datname = 'my-db'
  AND pid <> pg_backend_pid();
```

And then perform your database operations:

```sql
ALTER DATABASE "my-db" RENAME TO "my-db-bak";
ALTER DATABASE "my-db-new" RENAME TO "my-db";
```

Remember to re-allow database connections after the changes are complete, with:

```sql
ALTER database "my-db"
  with is_template true
       allow_connections true;
```

