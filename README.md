# Practicing on PostgreSQL

### What's this about?
I recently started practicing again on PostgreSQL, and databases in general. I want to get past the beginner/intermediate level where I can say that I have mastered some expert features of SQL. 







Learning about databases is ~100x easier when working on real data, especially *your* data, i.e. when you have a project, and a goal to achieve. That's how I started learning the basics, in order to satisfy the needs of my own projects. But now, wishing to "dive deeper" I thought about exploring some other, unknown dataset. Finally, I hope it will also serve as a a conversation starter in an interview.

Here's an account of me solving [pgexercises](https://pgexercises.com).

[PosgreSQL exercises](https://pgexercises.com/) aka `pgexercises`is a website by [Alisdair Owens](https://www.zaltys.net/). It provides a way to in-depth practice (Postgres) SQL with a real-like database that is complex enough to be used for both simple `SELECT * FROM` statements, as well as practice more advanced things such as recursion, dates/tims and aggregation.
> The dataset for these exercises is for a newly created country club, with a set of members, facilities such as tennis courts, and booking history for those facilities.

I set up PosgreSQL on my Ubuntu 14.04 machine using the official documentation from [here](https://www.postgresql.org/download/linux/ubuntu/), and downloaded the pgexercises database from [here](https://pgexercises.com/dbfiles/clubdata.sql).
The local setting-up of the database can be found on the bottom of this readme

- [x] Basics.
- [ ] Joins.
- [ ] Updates.
- [ ] Aggregates.
- [ ] Date.
- [ ] String.
- [ ] Recursive.

## Chapter 1 - Basics
Let's go and work through the "Basics" section!  
It covers the basic `SQL Statements`, `SELECT/WHERE`, `LIKE`, `IN`, `CASE`, `DISTINCT`, `ORDER BY`, `LIMIT`, `UNION`, `MIN/MAX`.    

* Retrieve everything from a table
* Retrieve specific columns from a table
* Control which rows are retrieved
* Control which rows are retrieved - part 2
* Basic string searches
* Matching against multiple possible values
* Classify results into buckets
* Working with dates
* Removing duplicates, and ordering results
* Combining results from multiple queries
* Simple aggregation
* More aggregation


Question :   
How can you retrieve all the information from the cd.facilities table?    
`exercises=# select * from cd.facilities;`


Question :    
You want to print out a list of all of the facilities and their cost to members. How would you retrieve a list of only facility names and costs?   
`exercises=# select name, membercost from cd.facilities;`

Question :    
How can you produce a list of facilities that charge a fee to members?   
`exercises=# select name, membercost from cd.facilities where membercost > 0;`

Question :   
How can you produce a list of facilities *that charge a fee* to members, and *that fee* is less than 1/50th of the monthly maintenance cost? Return the facid, facility name, member cost, and monthly maintenance of the facilities in question.    
``` 
exercises=# select facid, name, membercost, monthlymaintenance from cd.facilities 
exercises=# where membercost >0 and membercost*50 < monthlymaintenance;
```

Question :   
How can you produce a list of all facilities with the word 'Tennis' in their name?   
`exercises=# select * from cd.facilities where name like '%Tennis%';`    
Remember, `_` matches single characters, and `%` matches any strings.

Question :    
How can you retrieve the details of facilities with ID 1 and 5? Try to do it without using the OR operator.    
`exercises=# select * from cd.facilities where facid in (1, 5);`

Question :  
How can you produce a list of facilities, with each labelled as 'cheap' or 'expensive' depending on if their monthly maintenance cost is more than $100? Return the name and monthly maintenance of the facilities in question.   
```
exercises=# select name, facid,
exercises-# case
exercises-#    when (monthlymaintenance > 100) then 'expensive'
exercises-#    when (monthlymaintenance < 100) then 'cheap'
exercises-# end as cost
exercises-# from cd.facilities;

      name       | facid |   cost    
-----------------+-------+-----------
 Tennis Court 1  |     0 | expensive
 Tennis Court 2  |     1 | expensive
 Badminton Court |     2 | cheap
 Table Tennis    |     3 | cheap
 Massage Room 1  |     4 | expensive
 Massage Room 2  |     5 | expensive
 Squash Court    |     6 | cheap
 Snooker Table   |     7 | cheap
 Pool Table      |     8 | cheap
(9 rows)

```
The `AS` operator is *very* useful in labeling columns or expressions, not only for display purposes, but also for easy reference.


Question :   
How can you produce a list of members who joined after the start of September 2012? Return the memid, surname, firstname, and joindate of the members in question.   
`exercises=# select memid, surname, firstname, joindate from cd.members where joindate >= '2012-09-01';`


Question :   
How can you produce an ordered list of the first 10 surnames in the members table? The list must not contain duplicates.   
`exercises=# select distinct surname from cd.members order by surname desc limit 10;`

Question :   
You, for some reason, want a combined list of all surnames and all facility names. Yes, this is a contrived example :-). Produce that list!    
`exercises=# select surname from cd.members union select name from cd.facilities;`

UNION combines the results of two queries into a single table. Both results must have the same number of columns and compatible data types. UNION removes duplicate rows, unlike UNION ALL.

Question :   
You'd like to get the signup date of your last member. How can you retrieve this information?    
`exercises=# select max(joindate) as latest_join from cd.members;`

Question :   
You'd like to get the first and last name of the last member(s) who signed up - not just the date. How can you do that?    
```
exercises=# select firstname, surname, joindate from cd.members 
exercises=# where joindate = (select max(joindate) from cd.members);
```

I found out that, returning the max(joindate) `AS latest` and using this reference to filter didn't work. One other approach would be 
```
exercises=# select firstname, surname, joindate from cd.members
exercises=# order by joindate desc
exercises=# limit 1;
```





### Appendix : Setting it up..!
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


The database has been recreated, and it all seems to be working okay! I can now just `psql -U paschalis -d exercises -x` from my shell to go on exploring the database!



