# SQL (Structured Query Language)

### SQL is the language we use to issue commands to the database
SQL is a powerful language used to interact with relational databases. It allows users to perform various operations, including:

- **Create/Insert data** - Adding new records into a table.
- **Read/Select data** - Retrieving specific data from one or more tables.
- **Update data** - Modifying existing records in a table.
- **Delete data** - Removing records from a table.

## Terminology
Understanding basic database concepts is essential when working with SQL:

- **Database** - A structured collection of data that contains one or more tables.
- **Relation (or Table)** - A collection of rows (tuples) and columns (attributes) that store structured data.
- **Tuple (or Row)** - A single record in a table, representing an entity such as a person or an event.
- **Attribute (or Column/Field)** - A single piece of data that defines a property of an entity in a row.
- **Primary Key** - A unique identifier for each row in a table.
- **Foreign Key** - A field in one table that references the primary key of another table, establishing relationships between tables.

## Common Database Systems
There are several database management systems (DBMS) available, each with its strengths:

- **Major Database Management Systems in wide use:**
  - **PostgreSQL** - 100% Open-source, feature-rich, supports advanced SQL functions.
  - **Oracle** - Large, commercial, enterprise-scale, highly configurable.
  - **MySQL** - Fast, scalable, and widely used for web applications, with an open-source community version.
  - **SQL Server** - Developed by Microsoft, powerful enterprise-level solution, integrates with Windows ecosystems.
  
- **Smaller projects:**
  - **SQLite** - Lightweight, file-based, ideal for mobile and small-scale applications.
  - **HSQLDB** - Java-based, lightweight, used for testing and small applications.
  
## Basic SQL Commands
To interact with databases, SQL provides various commands categorized into different types:

### Data Definition Language (DDL) - Defines database structure
```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name VARCHAR(50),
    email VARCHAR(100) UNIQUE
);
```
- **CREATE** - Creates new databases, tables, or indexes.
- **ALTER** - Modifies existing database objects.
- **DROP** - Deletes databases, tables, or indexes.

### Data Manipulation Language (DML) - Manages data in tables
```sql
INSERT INTO users (name, email) VALUES ('Alice', 'alice@example.com');
UPDATE users SET name = 'Alice Smith' WHERE id = 1;
DELETE FROM users WHERE id = 1;
```
- **INSERT** - Adds new records to a table.
- **UPDATE** - Modifies existing records.
- **DELETE** - Removes records.

### Data Query Language (DQL) - Retrieves data
```sql
SELECT name, email FROM users WHERE id = 1;
```
- **SELECT** - Queries data from one or more tables.

### Data Control Language (DCL) - Controls access to data
```sql
GRANT SELECT ON users TO some_user;
REVOKE SELECT ON users FROM some_user;
```
- **GRANT** - Provides privileges to users.
- **REVOKE** - Removes privileges from users.

### Transaction Control Language (TCL) - Manages transactions
```sql
BEGIN;
UPDATE users SET name = 'Bob' WHERE id = 2;
COMMIT;
```
- **COMMIT** - Saves changes made during the transaction.
- **ROLLBACK** - Reverts changes made during the transaction.
- **SAVEPOINT** - Sets a checkpoint to roll back to a specific point.

## Why Learn SQL?
- SQL is essential for data analysis, database administration, and backend development.
- It allows efficient data retrieval and manipulation.
- SQL skills are highly valued in the industry, especially in data-driven roles.

By mastering SQL, you can efficiently interact with databases and gain a strong foundation for working with data in various applications.
