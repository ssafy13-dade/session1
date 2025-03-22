# Database Keys in Relational Databases

Database keys are essential components of relational database design. They are used to uniquely identify records and establish meaningful relationships between tables.

---

## 1. Primary Key

### **Definition**
A **Primary Key** is a column (or set of columns) that uniquely identifies each row in a table. It must contain **unique** values and cannot contain **NULL**.

### **Characteristics**
- Each table can have **only one** primary key.
- Ensures entity integrity.
- Automatically creates a **unique index**.

### **Example**
```sql
CREATE TABLE users (
    user_id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL
);
```

Here, `user_id` is the primary key. It uniquely identifies each user.

---

## 2. Logical Key (Candidate Key)

### **Definition**
A **Logical Key**, often called a **Candidate Key**, is a column (or combination of columns) that can **uniquely identify** rows in a table.

Every table may have multiple candidate keys, but **only one** is chosen as the primary key. The others are considered **alternate keys**.

### **Example**
```sql
CREATE TABLE students (
    student_id SERIAL PRIMARY KEY,
    national_id VARCHAR(20) UNIQUE,
    email VARCHAR(255) UNIQUE
);
```

- `national_id` and `email` are logical keys (candidate keys).
- `student_id` is chosen as the primary key.

Logical keys are useful when multiple columns are capable of uniquely identifying a record.

---

## 3. Foreign Key

### **Definition**
A **Foreign Key** is a column (or set of columns) that creates a relationship between two tables. It refers to the **primary key** in another table.

### **Purpose**
- Maintains **referential integrity**.
- Prevents invalid data from being inserted (e.g., an order cannot reference a non-existent customer).

### **Example**
```sql
CREATE TABLE orders (
    order_id SERIAL PRIMARY KEY,
    user_id INT REFERENCES users(user_id),
    order_date DATE NOT NULL
);
```

Here, `user_id` in `orders` is a foreign key that references `user_id` in the `users` table.

---

## Summary
| Key Type     | Description                                      | Allows NULLs | Must Be Unique | Used For                  |
|--------------|--------------------------------------------------|--------------|----------------|---------------------------|
| Primary Key  | Uniquely identifies each record in a table       | ❌           | ✅              | Entity integrity           |
| Logical Key  | Candidate key that could serve as a primary key  | ✅ (unless chosen) | ✅       | Alternate identification  |
| Foreign Key  | Creates a link to a primary key in another table | ✅           | ❌              | Referential integrity      |

Understanding and correctly implementing database keys ensures data accuracy, integrity, and optimal relational design.
