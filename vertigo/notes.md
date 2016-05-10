# Investigation on where to tackle the problem:
```
handle_client_startup() calls to
decide_startup_pool()
which reads username and dbname (and a few other things)
then it calls
set_pool(.., dbname, username, ...)
the first thing that that does is call 
find_database(dbname) which finds a matching database in the global list of databases
(and if fails)
register_auto_database(dbname) which creates a new database if one didn't yet exist
it calls parse_database, where I think most of the work is being done to actually determine 
the server side of the connection
```
# Implemented feature
If the dbname starts with "DYN&" the code will parse what follows and create 
a connection string config (what we specific on config.ini) on demand, dynamically. Example:
`DYN&dbname-vdb2&user-vusr2` is translated to `dbname=vdb2 user=vusr2`. The
keys should match what we can put in the `PgDatabase` struct.

# Issues
* Tested only with psql, not sure how it works with JDBC, etc.

# Testing notes
* Created local db `vert1`, owner `vusr1` with password `vpwd1`
* Created local db `vert2`, owner `vusr2` with password `vpwd2`
* In each one, created table MACHINES(name) and inserted values `machineDb1` and `machineDb2`, respectively.
* Executed: `echo "delete from MACHINES where name='test' ; insert into MACHINES values ('test') ; select * from MACHINES;" | psql -p 5433 dyn_dbname_vdb2_user_vusr2`
* Confirmed the result returns `machineDb2`.
* Confirmed that mixing usr/db values (e.g. `dyn_dbname_vdb2_user_vusr1`) denies the statements ("permission denied for relation machines").

# Problems we may have to deal with, or TODO

- Because of the name-value pair nature of the 'fake' connection string, we may
  get requests for the same end point with different ordering. We need to make
  sure all clients use a normalised format of this string, or we should
  normalise.
