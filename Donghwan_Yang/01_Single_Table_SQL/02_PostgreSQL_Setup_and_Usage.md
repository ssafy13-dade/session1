# Using SQL with PostgreSQL

1. **pgAdmin (Browser/Desktop)** - A graphical user interface (GUI) for managing PostgreSQL databases.
2. **psql (Command Line)** - A command-line interface (CLI) for executing SQL queries directly.

Both methods communicate with the PostgreSQL database server using SQL, allowing users to perform operations such as data retrieval, insertion, updating, and deletion.

## Components Explained

- **User / DBA**: The individual interacting with the database.
- **pgAdmin**: A GUI-based tool that simplifies database management.
- **psql**: A CLI-based tool for executing SQL commands.
- **Database Server (PostgreSQL)**: The central system storing and managing data.
- **SQL (Structured Query Language)**: The language used for database operations.

---

## Understanding `sudo`

The `sudo` (superuser do) command in Linux-based systems allows a permitted user to execute a command as the superuser (root) or another user, as specified by the security policy.

### Key Features:

- Grants temporary elevated privileges.
- Ensures security by requiring user authentication.
- Logs command execution for auditing purposes.

### Common Usage Examples:

```sh
sudo apt update   # Updates package lists (Debian-based systems)
sudo systemctl restart postgresql   # Restarts PostgreSQL service
sudo useradd newuser   # Creates a new user
```

By using `sudo`, users can perform administrative tasks without logging in as the root user, improving security and system integrity.

---

## Why Run PostgreSQL in a Linux Environment?

Running PostgreSQL on Linux is preferred due to its stability, security, and better performance optimization compared to other operating systems. Linux provides:

- **Efficient Resource Management** - PostgreSQL runs efficiently with Linux’s memory and process management.
- **Security and Stability** - Linux’s permission system helps secure database access.
- **Automation and Scripting** - Easy automation with shell scripts and cron jobs.
- **Better Performance Tuning** - Many PostgreSQL optimizations work best on Linux-based systems.

## Setting Up PostgreSQL in Linux

### 1. Installing PostgreSQL

First, install PostgreSQL using the package manager:

```sh
sudo apt update
sudo apt install postgresql postgresql-contrib  # Debian-based systems
```

For Red Hat-based systems:

```sh
sudo yum install postgresql-server postgresql-contrib
```

### 2. Starting the PostgreSQL Service

Enable and start the PostgreSQL service:

```sh
sudo systemctl enable postgresql
sudo systemctl start postgresql
```

To check the status:

```sh
sudo systemctl status postgresql
```

### 3. Creating a New PostgreSQL User and Database

PostgreSQL runs under the `postgres` user by default. Switch to this user:

```sh
sudo -i -u postgres
```

Create a new user:

```sh
createuser --interactive --pwprompt
```

It will prompt for:

- Username
- Password
- Role permissions (Superuser, Database Creation, Role Creation)

Create a new database:

```sh
createdb mydatabase
```

### 4. Connecting to PostgreSQL

To connect to the database using `psql`:

```sh
psql -U myuser -d mydatabase
```

Alternatively, if using the default `postgres` user:

```sh
psql
```

### 5. Basic Database Commands

Once inside `psql`, you can run basic SQL commands:

```sql
\l  -- List all databases
\c mydatabase  -- Connect to a database
\dt  -- Show tables in the current database
\q  -- Exit psql
```

### 6. Creating a Table in PostgreSQL

Once connected to your database, you can create a new table using the `CREATE TABLE` statement:

```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    age INTEGER,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

- `id SERIAL PRIMARY KEY` - Creates a unique identifier for each row.
- `name VARCHAR(100) NOT NULL` - Ensures that the name cannot be empty.
- `email VARCHAR(255) UNIQUE NOT NULL` - Stores unique email addresses.
- `age INTEGER` - Stores an optional integer value.
- `created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP` - Automatically records the time of insertion.

### 7. Inserting Data into the Table

To insert records into the `users` table:

```sql
INSERT INTO users (name, email, age) VALUES ('Alice', 'alice@example.com', 25);
INSERT INTO users (name, email, age) VALUES ('Bob', 'bob@example.com', 30);
```

### 8. Querying Data from the Table

Retrieve all users:

```sql
SELECT * FROM users;
```

Retrieve a specific user by email:

```sql
SELECT * FROM users WHERE email = 'alice@example.com';
```

---

## CRUD Operations in PostgreSQL

### 1. Create (INSERT)

Adding new records into a table:

```sql
INSERT INTO users (name, email, age) VALUES ('Charlie', 'charlie@example.com', 28);
```

### 2. Read (SELECT)

Retrieve all data:

```sql
SELECT * FROM users;
```

Retrieve specific columns:

```sql
SELECT name, email FROM users;
```

Retrieve records using `LIKE` (pattern matching):

```sql
SELECT * FROM users WHERE email LIKE '%example.com';
```

- `%` is a wildcard that matches any sequence of characters.

### 3. Update (UPDATE)

Updating a user's age:

```sql
UPDATE users SET age = 29 WHERE email = 'charlie@example.com';
```

### 4. Delete (DELETE)

Deleting a user from the table:

```sql
DELETE FROM users WHERE email = 'bob@example.com';
```

Delete all records (be cautious):

```sql
DELETE FROM users;
```

### Using `*` in SQL

- `*` is a wildcard used to select all columns in a query.
- Example:

```sql
SELECT * FROM users;
```

- Returns all columns from the `users` table.

---

This setup ensures a properly configured PostgreSQL environment for development and database management, along with a strong understanding of CRUD operations.

