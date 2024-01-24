# Using Transactions in Procedures

We learned that transactions ensure that a set of operations can be a single atomic action that either succeeds or fails, therefore providing consistency.

Stored procedures are a series of SQL statements that are saved and can be invoked. We can optionally pass in parameters and have the stored procedure return data.

What happens when we put the two together?

## Resources to Browse Before Class

- [An Overview of the New Stored Procedures in PostgreSQL 11](https://severalnines.com/database-blog/overview-new-stored-procedures-postgresql-11#:~:text=Traditionally%2C%20PostgreSQL%20has%20provided%20all,or%20open%20a%20new%20one.)
- [PostgreSQL Transaction Management](https://www.postgresql.org/docs/11/plpgsql-transactions.html)
- [Transaction in stored procedure in SQL Server](http://techfunda.com/howto/192/transaction-in-stored-procedure)
- [An Overview of Stored Procedures in PostgreSQL](https://severalnines.com/database-blog/overview-new-stored-procedures-postgresql-11#:~:text=Traditionally%2C%20PostgreSQL%20has%20provided%20all,or%20open%20a%20new%20one.)
- [Video: Transaction within Stored Procedure](https://www.youtube.com/watch?v=KGpWMyb4ODA)
- [Video: Stored Procedures For Transactions](https://www.youtube.com/watch?v=njnbdnEnmlc)
- [Handling Transactions in Nested SQL Server Stored Procedures](https://www.mssqltips.com/sqlservertip/4897/handling-transactions-in-nested-sql-server-stored-procedures/)

## Why transactional stored procedures?

A complex stored procedure could have a series of statements that must all be executed successfully. If any statement fails, anything that happened previously must be undone, or rolled back. To handle this scenario, we can write complex stored procedures that contain transactions.

Examples of where transactional stored procedures would be used:

* Bank cash transfers _(debit/credit)_
* Inventory updates _(purchase confirmation, update inventory status, ship item)_
* Hiring an employee _(new employee record, new department/employee record, payroll record created)_

### Examples of Transactional Stored Procedures 

Let's look at an example where we are adding a new grade for a student and the recalculating their average.

```sql
create or replace procedure student_add_score(
   student_id int,
   score dec
)
language plpgsql    
as $$
begin
    -- Adding a new record into the grades table 
    insert into studentgrades(student_id, score) 
    values (student_id, score)
    
commit;

    -- Calculating and updating the students new average score
    update students 
    set avg_grade = (
        select avg(score) 
        from studentgrades
        where student_id = student_id
    ) 
    where student_id = student_id;
end;$$
```

Remember the transaction from the last chapter? We created a customer record and then created a sales record associated with that customer. Let's put that transaction into a stored procedure. This time, our new customer buys two vehicles and therefore we have two inserts for the Sales table.

```sql
CREATE OR REPLACE PROCEDURE new_customer_new_sale()
LANGUAGE plpgsql
AS $$
DECLARE 
  NewCustomerId integer;
  CurrentTS date;
BEGIN
	INSERT INTO customers(first_name,last_name,email,phone,street,city,state,zipcode,company_name)
		VALUES
		('BILL','Simlet','r.simlet@remves.com','615-876-1237','77 Miner Lane','San Jose','CA','95008','Remves') 
		RETURNING customer_id INTO NewCustomerId;

COMMIT;

	CurrentTS = CURRENT_DATE;

	INSERT INTO sales(sales_type_id,vehicle_id,employee_id,customer_id,dealership_id,price,deposit,purchase_date,pickup_date,invoice_number,payment_method)
		VALUES(1,1,1,NewCustomerId,1,24333.67,6500,CurrentTS,CurrentTS + interval '7 days',1273592747, 'solo');
		
COMMIT;
		
	INSERT INTO sales(sales_type_id,vehicle_id,employee_id,customer_id,dealership_id,price,deposit,purchase_date,pickup_date,invoice_number,payment_method)
		VALUES(1,1,1,NewCustomerId,1,24333.67,6500,CurrentTS,CurrentTS + interval '7 days',1273592747);

END;
$$;

CALL new_customer_new_sale();
```

When you run this, you will notice that you get an error. That's because the second insert is missing a value for `payment_method`. But because of the two commits before that statment, the new customer record and the first sales record still gets created.

How is this different from a stored procedure that doesn't contain a transaction? Try taking out the two `COMMIT` statements and see the change in behavior.
