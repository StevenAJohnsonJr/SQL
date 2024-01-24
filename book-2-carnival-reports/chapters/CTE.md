# Common Table Expressions

This chapter covers *Common Table Expressions* (CTE) and how to use it as an alternative to subqueries.

## Resources to Browse Before Class

- [Intro to Common Table Expressions](https://www.postgresqltutorial.com/postgresql-cte/)
- [SQL WITH Statements](https://www.youtube.com/watch?v=_SanZ41uTlw)
- [Subqueries and CTEs](https://www.youtube.com/watch?v=V3sa01sgcLU)
- [PosgresSQL WITH Clause](https://www.tutorialspoint.com/postgresql/postgresql_with_clause.htm)

## What is a CTE?

A Common Table Expression (CTE) is a temporary result set that you can reference within another SQL statement. Some people consider the use of CTEs to provide a more readable query in comparison to subqueries.
- It can be used within a SELECT, INSERT, UPDATE or DELETE.
- They result set is only accesible during runtime of the query.
- It is made up of using a WITH Statement and then a subsequent SQL statement.
- They can be used recursively (WITH RECURSIVE).


Basic Syntax WITH query is as follows where *name_for_summary_data* is the name given to the WITH clause result set.
```sql
WITH
   name_for_summary_data AS (SELECT Statement)
   SELECT columns
   FROM name_for_summary_data
   WHERE conditions <=> (
      SELECT column
      FROM name_for_summary_data)
   [ORDER BY columns]
```

## Employees

## Cars That Need Service

Here is an example of how you can use a CTE to break down what might be a very long and hard to read query into two parts to increase readability.

Let's imagine that the manager of service at a dealerships wants to see which cars haven't been in for a oil change recently. He wants to send a reminder to the owners of those vehicles.

### Setup

First, insert some oil change records into the database.

```sql
insert into oilchangelogs
	(date_occured, vehicle_id)
values
	('2020-01-09', 1),
	('2021-10-30', 2),
	('2019-02-20', 3),
	('2022-03-17', 4)
;
```

### First Solution

First, you can create a CTE to get cars that require an oil change service.

```sql
with vehicles_needing_service as
(
	select
		v.vehicle_id,
		v.year_of_car,
		v.miles_count,
		TO_CHAR(o.date_occured, 'YYYY-MM-DD') date_of_last_change
	from vehicles v

	join oilchangelogs o
		on v.vehicle_id = o.vehicle_id
	where o.date_occured < '2022-01-01'
)
```

Once that is defined, you can now query that set of results as part of a larger query to get all of the information needed.

```sql
select
	vs.vehicle_id,
	vs.miles_count,
	s.purchase_date,
	e.first_name || ' ' || e.last_name seller,
	c.first_name || ' ' || c.last_name purchaser,
	c.email
from vehicles_needing_service vs -- Use the CTE
join sales s
	on s.vehicle_id  = vs.vehicle_id
join employees e
	on s.employee_id = e.employee_id
join customers c
	on s.customer_id = c.customer_id
order by
	vs.vehicle_id,
	s.purchase_date desc
;
```

This produces results that are _kinda_ what you want. Note: If you are running this in DBeaver, you will need to highlight the whole query and then run it.

| id|miles_count|purchase_date|seller|purchaser|email|
|--|--|--|--|--|--|
|1 | 108 | 2014-04-30 |	Hervey Mossom |Hailey Atling |	hatlingnt@sakura.ne.jp|
|1 | 108 | 2012-01-01 |	Whitney Ozintsev |Kaycee Liepins |	kliepins6j@exblog.jp |
|1 | 108 | 2008-11-30 |	Matthieu Disley |Ned Duligal |	nduligal2a@google.ru|
|1 | 108 | 2006-12-15 |	Maurits Mawson |Dexter Brundell |	dbrundellpc@nba.com |
|1 | 108 | 2006-01-20 |	Shari Kitchingman |Darelle Maskill |	dmaskill2q@hugedomains.com |
|2 | 309 | 2018-03-05 |	Humbert Gomery |Dene Baistow |dbaistowjw@homestead.com |
|2 | 309 | 2016-06-23 |	Homer Affleck |Ursula Rickman |	urickman3d@eventbrite.com|
|2 | 309 | 2016-04-08 |	Glenn Skirrow |Nico Eric |	 nericn5@hibu.com|
|2 | 309 | 2013-08-27 |	Beale Merrett |Lidia Listone |	llistonerl@mail.ru|
|2 | 309 | 2008-12-02 |	Artemus Houdmont |Hulda Dennick |	hdennickns@de.vu |
|3 | 182 | 2018-01-08 |	Claudia Shields |Carolee Jouen |	cjouen5v@aboutads.info |
|3 | 182 | 2017-09-12 |	Pepillo Munroe |Vladimir Nice |	vnicek0@mtv.com|
|3 | 182 | 2012-08-15 |	Fons Patrickson |Darcey McNeilley |	dmcneilley2g@ft.com |
|3 | 182 | 2007-12-06 |	Markos Espinas |Rowland Torbeck |	rtorbeckf4@slashdot.org |
|3 | 182 | 2007-12-05 |	Madel Haughton |Emylee Kernoghan |	ekernoghanv@lulu.com |

### Second Solution

Since multiple people can purchase the same car over time, the service manager only wants the last purchaser. You can break this part of the SQL out into another CTE. You can use multiple CTEs as part of the query. You separate them with a comma.

```sql
with vehicles_needing_service as
(
	select
		v.vehicle_id,
		v.year_of_car,
		v.miles_count,
		TO_CHAR(o.date_occured, 'YYYY-MM-DD') date_of_last_change
	from vehicles v

	join oilchangelogs o
		on v.vehicle_id = o.vehicle_id
	where o.date_occured < '2022-01-01'
),
last_purchase as (
	select
		s.vehicle_id,
		max(s.purchase_date) purchased
	from sales s
	group by s.vehicle_id
)
```

Now you have another CTE that can be used to filter down the results to get the info about the last purchase only.

```sql
select
	vs.vehicle_id,       -- Get vehicle id from first CTE
	vs.miles_count,      -- Get miles from first CTE
	lp.purchased,        -- Get purchase date from second CTE
	e.first_name || ' ' || e.last_name seller,
	c.first_name || ' ' || c.last_name purchaser,
	c.email
from vehicles_needing_service vs
join last_purchase lp   -- Join the second CTE
	on lp.vehicle_id  = vs.vehicle_id
join sales s
	on lp.vehicle_id = s.vehicle_id
	and lp.purchased = s.purchase_date
join employees e
	on s.employee_id = e.employee_id
join customers c
	on s.customer_id = c.customer_id
;
```

| id|miles_count|purchase_date|seller|purchaser|email|
|--|--|--|--|--|--|
|1|	108|	2014-04-30|	Hervey Mossom|	Hailey Atling|	hatlingnt@sakura.ne.jp|
|2|	309|	2018-03-05|	Humbert Gomery|	Dene Baistow|	dbaistowjw@homestead.com|
|3|	182|	2018-01-08|	Claudia Shields|	Carolee Jouen|	cjouen5v@aboutads.info|

## Exploring Other Uses of CTEs

Here are some ideas to get you started.

### Top 5 Dealerships

* For the top 5 dealerships, which employees made the most sales?
     *Note: to get a list of the top 5 employees with the associated dealership, you will need to use a Windows function (next chapter).  There are other ways you can interpret this query to not return that strict of data.
* For the top 5 dealerships, which vehicle models were the most popular in sales?
* For the top 5 dealerships, were there more sales or leases?

### Used Cars

* For all used cars, which states sold the most? The least?
* For all used cars, which model is greatest in the inventory? Which make is greatest inventory?

## Practice

Talk with your teammates and think of another scenario where you can use a CTE to answer multiple business questions about employees, inventory, sales, deealerships or customers.
