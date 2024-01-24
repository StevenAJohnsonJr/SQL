# Creating and Altering Database Tables

This chapter will show how to use the `CREATE TABLE` statement to create the table, the attributes, and the relationship constraints. We will also touch on the ALTER TABLE statement to change an existing table.

## Tools

1. PostgreSQL - Installed from the Getting Started Chapter.

2. DBeaver - Installed from the Getting Started Chapter.

3. Reference for [DBeaver Documentation](https://dbeaver.com/docs/wiki/)

## Goals

After this chapter, doing the exercises, and discussing with your instructor, you should understand the following concepts.

1. Creating a table in your database
2. Altering an existing table
3. Adding relationship constraints

<br>

## Carnival's Next Steps
Monique did a great job with the initial database. But we noticed that she didnt normalize her data enough. When we take a look at her VehicleType what do you notice?

## Practice: VehicleType Normalization

Review Carnival's the ERD and identify what you need to do to normalize the VehicleTypes Table.

1. What tables still need to be created?
1. What are the relationships can you identify?
1. Which should hold the primary key of the other as a foreign key?

<br>

## How to create a table

We are all set with our PostgreSQL database now created. We will begin to practice our SQL skills. 

To initially create a database we use what is called a Data Definition command, known as `CREATE` statement. The diagram below shows other SQL commands and their categorization within the SQL language.

<img src="./images/sql-commands.png">



### The `CREATE TABLE` Statement
The `CREATE TABLE` statement is used for creating a new table. Here is a simple example statement. "*CREATE TABLE* is followed by the  name of the table, then opening/closing pararenthesis which contain all the fields (attributes) of the table.

``` 
CREATE TABLE my_first_table (
    first_column TEXT,
    second_column INT
); 
```

### Before we can move on, let's discuss Data Types.
What is a data type? In relational databases, data types tell us what the data requirements are for a field. From the example above you can see we are giving a data type to each field on the table. The *first_column* requires data to be stored as a data type of *TEXT* and the *second_column* must be stored as an *INT*. 

### Short list of Data Types
| Data Types  | Example usage |
| ------------- | ------------- |
| Integer  | int(2)  |
| Character varying  | varchar(50)  |
| Boolean  | bool  |
| Numeric  | decimal(2)  |s
| Timestamp with time zone | timestampz  |

<br>

## Adding Foreign Key Constraint to new table

A foreign key constraint allows one table to be linked to another table. FOREIGN KEYs are a field (or collection of fields) from one table that references the PRIMARY KEY in another table. The table which hold the FOREIGN KEY is called the child table, and the table containing the candidate key is called the parent table.

To add a foreign key contraint when creating a table the following must be added to the CREATE TABLE statement for every field that need a contraint.


``` 

  FOREIGN KEY (column_name) REFERENCES relatedTableName(related_table_column_name)

```
or simply append this statement on the same line as a field definition

``` 

 column_name INT (column_name) REFERENCES relatedTableName(related_table_column_name)

```

Sometimes we will need to change the structure of an already existing table. We use an ALTER TABLE statement to change things like field names, field data types, adding foreign key contraints etc.


## Adding Foreign Key Constraint to existing table

To add a Foreign Key constraint, we need to use an ALTER TABLE statement.

``` 
  ALTER TABLE child_table 
  ADD CONSTRAINT constraint_name 
  FOREIGN KEY (fk_columns) 
  REFERENCES parent_table (parent_key_columns);

```
For our normalizing example with vehicleTypes and VehicleBodyTypes, the goal is to link our new table with the old table.  First we need to add a new column to the vehicleTypes table that will hold our vehicle_bodytype_id.

```
    ALTER TABLE vehicletypes  
    ADD column vehicle_bodytype_id int 
```

Then we need to add a Foreign Key to the VehicleTypes table.

```
  ALTER TABLE vehicletypes 
  ADD CONSTRAINT vehicletypes_vehiclebodytypes 
  FOREIGN KEY (vehicle_bodytype_id ) 
  REFERENCES vehiclebodytypes (vehicle_bodytype_id );
```

<b>Stop and think:</b> 
- Why did we add a column to the vehicleTypes table? What data will we put in that column?
- What data will need to go into the VehicleBodyTypes table?
- What data will need to go into the vehicle_bodytype_id column in the vehicletypes table?  

