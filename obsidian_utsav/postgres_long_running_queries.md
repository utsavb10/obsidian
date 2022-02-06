# Finding and killing long running queries on PostgreSQL

From time to time we need to investigate if there is any query running indefinitely on our PostgreSQL database.

These long running queries may interfere on the overall database performance and probably they are stuck on some background process.

In order to find them you can use the following query:

```
SELECT  
  pid,  
  now() - pg_stat_activity.query_start AS duration,  
  query,  
  state  
FROM pg_stat_activity  
WHERE (now() - pg_stat_activity.query_start) > interval '5 minutes';
```

The first returned column is the process id, the second is duration, following the query and state of this activity.

If state is idle you don’t need to worry about it, but active queries may be the reason behind low performances on your database.

> EDIT: I’ve added the pg_cancel_backend as first option to stop the query because it’s safer than pg_terminate_backend.

In order to cancel these long running queries you should execute:

SELECT pg_cancel_backend(__pid__);

The pid parameter is the value returned in the **pg_stat_activity** select.

It may take a few seconds to stop the query entirely using the **pg_cancel_backend** command.

If the you find the process is stuck you can kill it by running:

SELECT pg_terminate_backend(__pid__);

**Be careful with that!** As pointed by 

[Erwin Andreasen](https://medium.com/u/e9c11ffd8d45?source=post_page-----7c4f0449e86d-----------------------------------)

 in the comments bellow, pg_terminate_backend is the kill -9 in PostgreSQL. It will terminate the entire process which can lead to a full database restart in order to recover consistency.

Keep hacking!