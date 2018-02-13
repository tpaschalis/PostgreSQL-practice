# Appendix : Setting it up..!
First of all, I created a new user `paschalis` using 
```sql
CREATE USER paschalis with password '123456' valid until 'infinity';
ALTER USER paschalis SET statement_timeout = 750;
```

The [Getting Started](https://pgexercises.com/gettingstarted.html) page gives a detailed overview of the `tables` that exist in the database, namely the `members`, `facilities`, and `bookings` tables.
The first thing I did was examine the database dump using `vim clubdata.sql`. I saw that it did not specify any users, so I added my newly created user to the `.sql` dump

```sql
GRANT CONNECT ON DATABASE exercises TO paschalis;
\c exercises
CREATE SCHEMA cd;
GRANT USAGE ON SCHEMA cd TO paschalis;
REVOKE ALL PRIVILEGES ON SCHEMA PUBLIC FROM paschalis;
```


I then re-created the database from the `.sql` dump, using the superuser `postgres`

```sql
$ psql -U postgres -f clubdata.sql -x
Password for user postgres: 
CREATE DATABASE
GRANT
You are now connected to database "exercises" as user "postgres".
CREATE SCHEMA
GRANT
REVOKE
SET
SET
SET
SET
SET
SET
SET
CREATE TABLE
CREATE TABLE
CREATE TABLE
COPY 4044
COPY 9
COPY 31
ALTER TABLE
ALTER TABLE
ALTER TABLE
ALTER TABLE
ALTER TABLE
ALTER TABLE
CREATE INDEX
CREATE INDEX
CREATE INDEX
CREATE INDEX
CREATE INDEX
CREATE INDEX
CREATE INDEX
ALTER DATABASE
GRANT
ANALYZE
```



So, in the initial `psql` "shell", I check for the existence of the initial user, the database, and examine the schema.
```
postgres-# \du

                                    List of roles
  Role name  |                         Attributes                         | Member of 
-------------+------------------------------------------------------------+-----------
 ........... | .......................................................... | ..........
 paschalis   | Password valid until infinity                              | {}
 ........... | .......................................................... | ..........



postgres-# \l
                                  List of databases
   Name    |  Owner   | Encoding |   Collate   |    Ctype    |   Access privileges   
-----------+----------+----------+-------------+-------------+-----------------------
 ......... | ........ | ........ | ........... | ........... | ......................
 exercises | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =Tc/postgres         +
           |          |          |             |             | postgres=CTc/postgres+
           |          |          |             |             | paschalis=c/postgres
 ......... | ........ | ........ | ........... | ........... | ......................

postgres=# \c exercises
You are now connected to database "exercises" as user "postgres".

exercises=# SELECT table_schema,table_name
FROM information_schema.tables
ORDER BY table_schema,table_name;


    table_schema    |              table_name               
--------------------+---------------------------------------
 cd                 | bookings
 cd                 | facilities
 cd                 | members
 information_schema | administrable_role_authorizations
 information_schema | applicable_roles
 information_schema | attributes
 .................. | ....................................
 .................. | ....................................
 .................. | ....................................
 pg_catalog         | pg_user
 pg_catalog         | pg_user_mapping
 pg_catalog         | pg_user_mappings
 pg_catalog         | pg_views
(191 rows)
```

To remove the restrictions on transactions and grant write privileges, you should, and so that other users can access and modify the table, one can

```sql
 grant all privileges on table cd.members to paschalis;
ERROR:  cannot execute GRANT in a read-only transaction

 begin;
BEGIN
 set transaction read write;
SET
 alter database exercises set default_transaction_read_only = off;
ALTER DATABASE
 commit;
COMMIT

  GRANT ALL PRIVILEGES ON TABLE cd.members TO paschalis;
GRANT
  GRANT ALL PRIVILEGES ON TABLE cd.facilities TO paschalis;
GRANT
  GRANT ALL PRIVILEGES ON TABLE cd.bookings TO paschalis;
GRANT
```


The database has been recreated, and it all seems to be working okay! I can now just `psql -U paschalis -d exercises -x` from my shell to go on exploring the database!



