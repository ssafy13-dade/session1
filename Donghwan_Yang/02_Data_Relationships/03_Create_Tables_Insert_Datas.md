# Creating Tables and Inserting Data in PostgreSQL

This guide explains how to define table structures and insert data into tables using SQL in PostgreSQL. These are foundational steps in working with relational databases.

---

## 1. Creating Tables

### **Basic Syntax**
```sql
CREATE TABLE table_name (
    column1 data_type constraints,
    column2 data_type constraints,
    ...
);
```

### **Example: Users Table**
```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    age INTEGER,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### **Explanation**
- `SERIAL PRIMARY KEY`: Automatically generates a unique ID.
- `VARCHAR(n)`: Variable-length string.
- `UNIQUE`: Ensures no duplicate values.
- `NOT NULL`: Field must have a value.
- `DEFAULT CURRENT_TIMESTAMP`: Automatically sets current time.

---

## 2. Inserting Data into Tables

### **Basic Syntax**
```sql
INSERT INTO table_name (column1, column2, ...) 
VALUES (value1, value2, ...);
```

### **Example**
```sql
INSERT INTO users (name, email, age) 
VALUES ('Alice', 'alice@example.com', 28);

INSERT INTO users (name, email, age) 
VALUES ('Bob', 'bob@example.com', 34);
```

- You can omit the `id` and `created_at` fields because they are handled automatically.

### **Inserting Multiple Rows**
```sql
INSERT INTO users (name, email, age) VALUES
('Charlie', 'charlie@example.com', 22),
('Diana', 'diana@example.com', 30);
```

---

## 3. Viewing Inserted Data

### **View All Records**
```sql
SELECT * FROM users;
```

### **Filter Specific Data**
```sql
SELECT * FROM users WHERE age > 25;
```

---

## Summary
- Use `CREATE TABLE` to define the structure of a table.
- Use `INSERT INTO` to add data.
- PostgreSQL supports automatic ID generation with `SERIAL`.
- You can insert one or multiple rows at a time.
- Use `SELECT` to verify inserted data.

These operations form the core of managing data in PostgreSQL databases.
