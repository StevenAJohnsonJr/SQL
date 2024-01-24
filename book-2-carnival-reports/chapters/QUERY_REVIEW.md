# Query Fundamentals

## Videos to Watch First

* [SQL SELECT Tutorial |¦| SQL Tutorial |¦| SQL for Beginners](https://www.youtube.com/watch?v=YufocuHbYZo)

## Writing Queries for Carnival in DBeaver

## Selecting All Vehicles

Start off with a query to view all vehicles in your database.

```sql
SELECT * FROM Vehicles;
```

Using the askterisk (`*`) returns all columns in the query.  It is fine to use `*` in the intitial phases of development, but important to be specific about the columns you want to view as you build your query. The query will run much faster and there's a lower risk of accidently exposing data unitentionally.

You should also start getting into the habit immediately of using a table alias. Here is a query to view the engine size, floor price, and manufacturer price of all vehicles.

Note that the Vehicle table was aliased with the letter v, and all columns used that alias.

```sql
SELECT
    v.engine_type,
    v.floor_price,
    v.msr_price
FROM Vehicles v
```

## Practice: Dealers

Write a query that returns the business name, city, state, and website for each dealership. Use an alias for the Dealerships table.

## Practice: Customers

Write a query that returns the first name, last name, and email address of every customer. Use an alias for the Customers table.
