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
- [x] Joins.
- [x] Updates.
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
```sql
select * from cd.facilities;
```


Question :    
You want to print out a list of all of the facilities and their cost to members. How would you retrieve a list of only facility names and costs?   
```sql
select name, membercost from cd.facilities;`
```

Question :    
How can you produce a list of facilities that charge a fee to members?   
```sql
select name, membercost from cd.facilities where membercost > 0;
```

Question :   
How can you produce a list of facilities *that charge a fee* to members, and *that fee* is less than 1/50th of the monthly maintenance cost? Return the facid, facility name, member cost, and monthly maintenance of the facilities in question.    
```sql 
select facid, name, membercost, monthlymaintenance from cd.facilities 
where membercost >0 and membercost*50 < monthlymaintenance;
```

Question :   
How can you produce a list of all facilities with the word 'Tennis' in their name?   
```sql
select * from cd.facilities where name like '%Tennis%';
```    
Reminder, `_` matches single characters, and `%` matches any strings.

Question :    
How can you retrieve the details of facilities with ID 1 and 5? Try to do it without using the OR operator.    
```sql
select * from cd.facilities where facid in (1, 5);
```

Question :  
How can you produce a list of facilities, with each labelled as 'cheap' or 'expensive' depending on if their monthly maintenance cost is more than $100? Return the name and monthly maintenance of the facilities in question.   
```sql
select name, facid,
case
   when (monthlymaintenance > 100) then 'expensive'
   when (monthlymaintenance < 100) then 'cheap'
end as cost
from cd.facilities;

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

or

```sql
select name, facid
	case when (monthlymaintenance > 100) then
		'expensive'
	else
		'cheap'
	end as cost
	from cd.facilities;
```

The `AS` operator is *very* useful in labeling columns or expressions, not only for display purposes, but also for easy reference.


Question :   
How can you produce a list of members who joined after the start of September 2012? Return the memid, surname, firstname, and joindate of the members in question.   
```sql
select memid, surname, firstname, joindate from cd.members where joindate >= '2012-09-01';
```


Question :   
How can you produce an ordered list of the first 10 surnames in the members table? The list must not contain duplicates.   
```sql
select distinct surname from cd.members order by surname desc limit 10;
```

Question :   
You, for some reason, want a combined list of all surnames and all facility names. Yes, this is a contrived example :-). Produce that list!    
```sql
select surname from cd.members union select name from cd.facilities;
```

UNION combines the results of two queries into a single table. Both results must have the same number of columns and compatible data types. UNION removes duplicate rows, unlike UNION ALL.

Question :   
You'd like to get the signup date of your last member. How can you retrieve this information?    
```sql
select max(joindate) as latest_join from cd.members;
```

Question :   
You'd like to get the first and last name of the last member(s) who signed up - not just the date. How can you do that?    
```sql
select firstname, surname, joindate from cd.members 
where joindate = (select max(joindate) from cd.members);
```

I found out that, returning the max(joindate) `AS latest` and using this reference to filter didn't work. One other approach would be 
```sql
select firstname, surname, joindate from cd.members
order by joindate desc
limit 1;
```




## Chapter 2 - Joins and Subqueries
Next up is a way to answer to more structurally complex questions, such as

* Retrieve the start times of members' bookings
* Work out the start times of bookings for tennis courts
* Produce a list of all members who have recommended another member
* Produce a list of all members, along with their recommender
* Produce a list of all members who have used a tennis court
* Produce a list of costly bookings
* Produce a list of all members, along with their recommender, using no joins.
* Produce a list of costly bookings, using a subquery


Question :    
How can you produce a list of the start times for bookings by members named 'David Farrell'?   
```sql
select bookings.starttime
from cd.members inner join cd.bookings
on members.memid = bookings.memid
where members.firstname = 'David' and members.surname = 'Farrell';
```


Question :   
How can you produce a list of the start times for bookings for tennis courts, for the date '2012-09-21'? Return a list of start time and facility name pairings, ordered by the time.   
```sql
select bookings.starttime, facilities.name
from cd.facilities inner join cd.bookings
on facilities.facid = bookings.facid
where facilities.name like 'Tennis C%' and
bookings.starttime >= '2012-09-21' and bookings.starttime < '2012-09-22'
order by bookings.starttime;
```


Question :   
How can you output a list of all members who have recommended another member? Ensure that there are no duplicates in the list, and that results are ordered by (surname, firstname).   
```sql
select recommended.firstname, recommended.surname
from cd.members inner join cd.members recommended
on recommended.memid = members.recommendedby
order by surname, firstname;
```

Question :   
How can you output a list of all members, including the individual who recommended them (if any)? Ensure that results are ordered by (surname, firstname).    
```sql
select members.firstname, members.surname, recommender.firstname, recommender.surname
from cd.members left outer join cd.members recommender
on recommender.memid = members.recommendedby
order by members.surname, members.firstname;
```


Question :   
How can you produce a list of all members who have used a tennis court? Include in your output the name of the court, and the name of the member formatted as a single column. Ensure no duplicate data, and order by the member name.   
```sql
select distinct members.firstname, members.surname, facilities.name
from cd.members
inner join cd.bookings on members.memid = bookings.memid
inner join cd.facilities on bookings.facid = facilities.facid
where bookings.facid in (0,1)
order by members.firstname, members.surname;
```


Question :   
How can you produce a list of bookings on the day of 2012-09-14 which will cost the member (or guest) more than $30? Remember that guests have different costs to members (the listed costs are per half-hour 'slot'), and the guest user is always ID 0. Include in your output the name of the facility, the name of the member formatted as a single column, and the cost. Order by descending cost, and do not use any subqueries.    
```sql
select members.firstname, members.surname, facilities.name,
case 
	when members.memid = 0 then bookings.slots * facilities.guestcost
	else bookings.slots * facilities.membercost
end as cost
from cd.members 
	inner join cd.bookings on members.memid = bookings.memid
	inner join cd.facilities on bookings.facid = facilities.facid
where 
bookings.starttime >= '2012-09-14' and bookings.starttime < '2012-09-15' and (
(members.memid = 0 and bookings.slots*facilities.guestcost >= 30 ) or 
(members.memid != 0 and bookings.slots*facilities.membercost >= 30) )
order by cost desc;
```


Question :   
How can you output a list of all members, including the individual who recommended them (if any), without using any joins? Ensure that there are no duplicates in the list, and that each firstname + surname pairing is formatted as a column and ordered.   
```sql
select 
distinct members.firstname, members.surname, 
(select recommender.firstname || ' ' || recommender.surname from cd.members recommender where recommender.memid = members.recommendedby)
from cd.members 
order by members.firstname, members.surname;
```
Reminder, subquery must return *one* column, that's why I concantenated the firstname/surname strings.



Question :   
The Produce a list of costly bookings exercise contained some messy logic: we had to calculate the booking cost in both the WHERE clause and the CASE statement. Try to simplify this calculation using subqueries. For reference, the question was:   

How can you produce a list of bookings on the day of 2012-09-14 which will cost the member (or guest) more than $30? Remember that guests have different costs to members (the listed costs are per half-hour 'slot'), and the guest user is always ID 0. Include in your output the name of the facility, the name of the member formatted as a single column, and the cost. Order by descending cost.   
```sql

```


## Chapter 3 - Modifying data
Okay, reading data from a database can range from pretty straightforward, to a total nightmare.   
But when working with a real-world database, you deal with with inserting, updating, and deleting information. Let's work through questions as

* Insert some data into a table
* Insert multiple rows of data into a table
* Insert calculated data into a table
* Update some existing data
* Update multiple rows and columns at the same time
* Update a row based on the contents of another row
* Delete all bookings
* Delete a member from the cd.members table
* Delete based on a subquery



Question :   
The club is adding a new facility - a spa. We need to add it into the facilities table. Use the following values:   
`facid: 9, Name: 'Spa', membercost: 20, guestcost: 30, initialoutlay: 100000, monthlymaintenance: 800.`
```sql
insert into cd.facilities (facid, name, membercost, guestcost, initialoutlay, monthlymaintenance) 
values (9, 'Spa', 20, 30, 100000, 800);
```


Question :   
In the previous exercise, you learned how to add a facility. Now you're going to add multiple facilities in one command. Use the following values:   
`facid: 9, Name: 'Spa', membercost: 20, guestcost: 30, initialoutlay: 100000, monthlymaintenance: 800.`   
`facid: 10, Name: 'Squash Court 2', membercost: 3.5, guestcost: 17.5, initialoutlay: 5000, monthlymaintenance: 80.`   

```sql
insert into cd.facilities
(facid, name, membercost, guestcost, initialoutlay, monthlymaintenance)
values 
(11, 'Spa 2', 20, 30, 100000, 800),
(12, 'Squash Court 2', 3.5, 17.5, 5000, 80);
INSERT 0 2
```


Question :    
Let's try adding the spa to the facilities table again. This time, though, we want to automatically generate the value for the next facid, rather than specifying it as a constant. Use the following values for everything else:    
`Name: 'Spa', membercost: 20, guestcost: 30, initialoutlay: 100000, monthlymaintenance: 800.`    

```sql
insert into cd.facilities
(facid, name, membercost, guestcost, initialoutlay, monthlymaintenance)
values
( (select max(facid) from cd.facilities) + 1, 'Spa', 20, 30, 100000, 800);
```

Question :   
We made a mistake when entering the data for the second tennis court. The initial outlay was 10000 rather than 8000: you need to alter the data to fix the error.    
```sql
update cd.facilities
set 
initialoutlay = 10000
where name = 'Tennis Court 2';
```

Question :   
We want to increase the price of the tennis courts for both members and guests. Update the costs to be 6 for members, and 30 for guests.    
```sql
update cd.facilities
set
(membercost = 6, guestcost = 30)
where name like 'Tennis C%';
```
Alternatively I could use `facid in(0,1)`

Question :   
We want to alter the price of the second tennis court so that it costs 10% more than the first one. Try to do this without using constant values for the prices, so that we can reuse the statement if we want to.    
```sql
update cd.facilities
set 
membercost = (select membercost from cd.facilities where facid = 0)*1.1,
guestcost = (select guestcost from cd.facilities where facid = 0)*1,1
where facilities.facid = 1;
```

Question :    
As part of a clearout of our database, we want to delete all bookings from the cd.bookings table. How can we accomplish this?    
```sql
delete from cd.bookings;
```

Question :   
We want to remove member 37, who has never made a booking, from our database. How can we achieve that?    
```sql
delete from cd.members where memid = 37;
DELETE 1
```

Question :   
In our previous exercises, we deleted a specific member who had never made a booking. How can we make that more general, to delete all members who have never made a booking?   
```sql
delete from cd.members where memid not in (select memid from cd.bookings);
DELETE 0
```


## Chapter 4 - Aggregates
Aggregates allow to produce more complex queries, answering a whole different level of questions to help with decision making.   
The questions I'll be going through are

* Count the number of facilities
* Count the number of expensive facilities
* Count the number of recommendations each member makes.
* List the total slots booked per facility
* List the total slots booked per facility in a given month
* List the total slots booked per facility per month
* Find the count of members who have made at least one booking
* List facilities with more than 1000 slots booked
* Find the total revenue of each facility
* Find facilities with a total revenue less than 1000
* Output the facility id that has the highest number of slots booked
* List the total slots booked per facility per month, part 2
* List the total hours booked per named facility
* List each member's first booking after September 1st 2012
* Produce a list of member names, with each row containing the total member count
* Produce a numbered list of members
* Output the facility id that has the highest number of slots booked, again
* Rank members by (rounded) hours used
* Find the top three revenue generating facilities
* Classify facilities by value
* Calculate the payback time for each facility
* Calculate a rolling average of total revenue



Question :   
For our first foray into aggregates, we're going to stick to something simple. We want to know how many facilities exist - simply produce a total count.   
```sql
select
count(facid)
from cd.facilities; 
```


Question :   
Produce a count of the number of facilities that have a cost to guests of 10 or more.    
```sql
select
count (*)   
from cd.facilities
where guestcost >= 10;
```
One can also use `count(*)` so as to just count the rows, not based on the presence of a specific value.


Question :   
Produce a count of the number of recommendations each member has made. Order by member ID.    
```sql
select recommendedby, count(memid)
from cd.members
where recommendedby > 0 
group by recommendedby
order by recommendedby;
```

Question :   
Produce a list of the total number of slots booked per facility. For now, just produce an output table consisting of facility id and slots, sorted by facility id.    
```sql
select facid, sum(slots)
from cd.bookings
group by facid
sort by facid;
```

Question :   
Produce a list of the total number of slots booked per facility in the month of September 2012. Produce an output table consisting of facility id and slots, sorted by the number of slots.    
```sql
select facid, sum(slots)
from cd.bookings
where starttime >= '2012-09-01' and starttime < '2012-10-01'
group by facid
order by sum(slots);
```

Question :   
Produce a list of the total number of slots booked per facility per month in the year of 2012. Produce an output table consisting of facility id and slots, sorted by the id and month.    
```sql
select facid, sum(slots), extract(month from starttime) as mth
from cd.bookings
where extract(year from starttime) = 2012
group by mth, facid
order by facid, mth;
```

Question :   
Find the total number of members who have made at least one booking.    
```sql
select count(distinct memid)
from cd.bookings;
```


Question :   
Produce a list of facilities with more than 1000 slots booked. Produce an output table consisting of facility id and hours, sorted by facility id.    
```sql
select facid, sum(slots)
from cd.bookings
group by facid
having sum(slots) > 1000
order by facid;
```
When you encounter `ERROR:  aggregate functions are not allowed in WHERE`, the funtion `HAVING` might be useful.



Question :   
Produce a list of facilities along with their total revenue. The output table should consist of facility name and revenue, sorted by revenue. Remember that there's a different cost for guests and members!    
```sql
select facilities.name, sum(slots*case when memid = 0 then facilities.guestcost else facilities.membercost end) as TotalMoney
from cd.bookings
inner join cd.facilities on bookings.facid = facilities.facid
group by facilities.name
order by TotalMoney;
```

Question :   
Produce a list of facilities with a total revenue less than 1000. Produce an output table consisting of facility name and revenue, sorted by revenue. Remember that there's a different cost for guests and members!    
```sql
select name, totalrevenue from 
(
select facilities.name, 
sum(case when memid = 0 then slots * facilities.guestcost else slots * membercost end) as totalrevenue
from cd.bookings
inner join cd.facilities 
on cd.bookings.facid = cd.facilities.facid
group by facilities.name
)
as selected_facilities where totalrevenue <= 1000
order by totalrevenue;
```
One would try to just use the directly previous statement, plus the `HAVING` function! Well, this doesn't work, as the `HAVING` does not support column names.

Question :   
 Output the facility id that has the highest number of slots booked. For bonus points, try a version without a LIMIT clause. This version will probably look messy!   

```sql
select facid, sum(slots)
from cd.bookings group by facid
order by sum(slots) desc
limit 2;
```
I couldn't get `TOP` to work, but I didn't look more into it.

Question :   

Produce a list of the total number of slots booked per facility per month in the year of 2012. In this version, include output rows containing totals for all months per facility, and a total for all months for all facilities. The output table should consist of facility id, month and slots, sorted by the id and month. When calculating the aggregated values for all months and all facids, return null values in the month and facid columns.    

```sql
select facid, extract(month from starttime) as month, sum(slots)
from cd.bookings
where starttime >= '2012-01-01'
and starttime <= '2013-01-01'
group by rollup(facid, month)
order by facid, month;
```

Question :   
Produce a list of the total number of hours booked per facility, remembering that a slot lasts half an hour. The output table should consist of the facility id, name, and hours booked, sorted by facility id. Try formatting the hours to two decimal places.    

```sql
```



## Chapter 5 - Dates/Timestamps

Working with real-world data, you will almost always deal with some kind of date and time data. Such data is also valuable for debugging purposes, and might be life-saving in disaster scenarios.

To familiarize myself, I've answered questions such as  



* Produce a timestamp for 1 a.m. on the 31st of August 2012
* Subtract timestamps from each other
* Generate a list of all the dates in October 2012
* Get the day of the month from a timestamp
* Work out the number of seconds between timestamps
* Work out the number of days in each month of 2012
* Work out the number of days remaining in the month
* Work out the end time of bookings
* Return a count of bookings for each month
* Work out the utilisation percentage for each facility by month


Question :   
Produce a timestamp for 1 a.m. on the 31st of August 2012.    

```sql
select '2012-08-31 01:00:00'::timestamp;
```
I lie the approach of casting a formatted string to a timestamp this way.   
One could also use something simpler like `select timestamp '2012-08-31 01:00:00';`






