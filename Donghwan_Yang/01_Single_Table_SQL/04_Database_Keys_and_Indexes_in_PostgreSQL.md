# Database Keys and Indexes in PostgreSQL

## 1. Database Keys in PostgreSQL
Database keys ensure data integrity and optimize queries. PostgreSQL supports various types of keys:

### **Primary Key**
- A unique identifier for each row in a table.
- Automatically creates a unique index on the column.
- Example:
  ```sql
  CREATE TABLE employees (
      id SERIAL PRIMARY KEY,
      name VARCHAR(100) NOT NULL
  );
  ```

### **Foreign Key**
- Establishes a relationship between two tables.
- Ensures referential integrity.
- Example:
  ```sql
  CREATE TABLE departments (
      id SERIAL PRIMARY KEY,
      name VARCHAR(50) NOT NULL
  );

  CREATE TABLE employees (
      id SERIAL PRIMARY KEY,
      name VARCHAR(100) NOT NULL,
      department_id INT REFERENCES departments(id) ON DELETE CASCADE
  );
  ```

### **Unique Key**
- Ensures uniqueness of values in a column (other than primary key).
- Allows multiple `NULL` values.
- Example:
  ```sql
  CREATE TABLE users (
      id SERIAL PRIMARY KEY,
      email VARCHAR(255) UNIQUE NOT NULL
  );
  ```

### **Composite Key**
- A key formed by multiple columns.
- Ensures uniqueness for the combined values.
- Example:
  ```sql
  CREATE TABLE orders (
      order_id INT,
      product_id INT,
      PRIMARY KEY (order_id, product_id)
  );
  ```

---

## 2. Indexes in PostgreSQL
Indexes improve query performance by speeding up data retrieval.

### **Types of Indexes**

#### **B-Tree Index (Default)**
- The most commonly used index in PostgreSQL.
- Optimized for queries using equality (`=`), inequality (`!=`), and range (`<`, `>`, `BETWEEN`) operators.
- Uses a self-balancing tree structure, ensuring logarithmic search time (`O(log n)`).
- Suitable for **most** queries, including sorting (`ORDER BY`).
- Example:
  ```sql
  CREATE INDEX idx_users_email ON users(email);
  ```

#### **Hash Index**
- Optimized for equality searches (`=`), but **not** for range queries.
- Faster than B-Tree for exact matches but does **not** support sorting or range queries.
- Example use case: indexing hashed passwords or unique identifiers.
- Example:
  ```sql
  CREATE INDEX idx_users_hash ON users USING hash(email);
  ```
- **Note**: Hash indexes were not WAL-logged before PostgreSQL 10, making them unsafe for replication and crash recovery.

#### **GIN Index (Generalized Inverted Index)**
- Used for full-text search and JSONB operations.
- Example:
  ```sql
  CREATE INDEX idx_documents ON articles USING GIN (to_tsvector('english', content));
  ```

#### **GiST Index (Generalized Search Tree)**
- Useful for geometric, full-text search, and custom indexing.
- Example:
  ```sql
  CREATE INDEX idx_locations ON places USING GIST (coordinates);
  ```

#### **Partial Index**
- Indexes only specific rows.
- Example:
  ```sql
  CREATE INDEX idx_active_users ON users(email) WHERE active = true;
  ```

#### **Expression Index**
- Indexes expressions instead of columns.
- Example:
  ```sql
  CREATE INDEX idx_lower_email ON users (LOWER(email));
  ```

---

## 3. SERIAL vs. AUTO_INCREMENT (Other RDBMS)
PostgreSQL does not use `AUTO_INCREMENT` like MySQL. Instead, it provides the `SERIAL` and `BIGSERIAL` data types.

### **Using `SERIAL` in PostgreSQL**
- Automatically creates a sequence for auto-incrementing values.
- Example:
  ```sql
  CREATE TABLE customers (
      id SERIAL PRIMARY KEY,
      name VARCHAR(100)
  );
  ```
- The above creates an implicit sequence named `customers_id_seq` and links it to the `id` column.
- You can manually access the sequence:
  ```sql
  SELECT nextval('customers_id_seq');
  ```

### **Using `AUTO_INCREMENT` in MySQL**
- MySQL and other RDBMS use `AUTO_INCREMENT`.
- Example:
  ```sql
  CREATE TABLE customers (
      id INT AUTO_INCREMENT PRIMARY KEY,
      name VARCHAR(100)
  );
  ```

### **Key Differences**
| Feature          | PostgreSQL (`SERIAL`)       | MySQL (`AUTO_INCREMENT`) |
|----------------|------------------------|----------------------|
| Default Mechanism | Uses an implicit sequence | Uses `AUTO_INCREMENT` |
| Custom Sequence | Yes, can manage manually | No, fully automatic |
| Reset Behavior | Can reset with `ALTER SEQUENCE` | Can reset with `ALTER TABLE` |
| Flexibility | More customizable via sequences | Less flexible |

---

## Summary
- **Keys**: Primary, Foreign, Unique, and Composite Keys ensure data integrity.
- **Indexes**: Various types like B-Tree, Hash, GIN, and GiST improve query performance.
- **SERIAL vs. AUTO_INCREMENT**: PostgreSQL uses `SERIAL` with sequences, while MySQL uses `AUTO_INCREMENT`.

Using the right keys and indexes helps optimize performance and maintain data integrity in PostgreSQL.
