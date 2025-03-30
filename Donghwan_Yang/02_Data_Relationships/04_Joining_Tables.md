# Joining Tables in PostgreSQL

Joining tables is a core feature of relational databases, allowing you to retrieve and combine data across multiple related tables. This guide covers all common types of joins, including handling many-to-many relationships and cross joins.

---

## 1. Why Use Joins?
Joins allow you to:
- Access related data spread across multiple tables
- Avoid data duplication by keeping data normalized
- Build complex queries that reflect real-world relationships

---

## 2. Types of Joins

### **INNER JOIN**
Returns only the rows that have matching values in both tables.

```sql
SELECT orders.id, users.name, orders.order_date
FROM orders
INNER JOIN users ON orders.user_id = users.id;
```

### **LEFT JOIN (LEFT OUTER JOIN)**
Returns all rows from the left table, and matching rows from the right table. If no match, NULLs are returned.

```sql
SELECT users.name, orders.order_date
FROM users
LEFT JOIN orders ON users.id = orders.user_id;
```

### **RIGHT JOIN (RIGHT OUTER JOIN)**
Returns all rows from the right table, and matching rows from the left table.

```sql
SELECT users.name, orders.order_date
FROM orders
RIGHT JOIN users ON orders.user_id = users.id;
```

### **FULL JOIN (FULL OUTER JOIN)**
Returns rows when there is a match in one of the tables. Returns NULLs for non-matching rows from both sides.

```sql
SELECT users.name, orders.order_date
FROM users
FULL JOIN orders ON users.id = orders.user_id;
```

### **CROSS JOIN**
Returns the Cartesian product of the two tables (every row in A combined with every row in B).

```sql
SELECT a.name, b.product_name
FROM users a
CROSS JOIN products b;
```

> ⚠️ Be cautious: cross joins multiply row counts and can be very large.

---

## 3. Many-to-Many Relationships
A many-to-many relationship requires a **junction table** to break it into two one-to-many relationships.

### **Example Schema**
- `students(id, name)`
- `courses(id, title)`
- `enrollments(student_id, course_id)` → Junction table

### **Querying Many-to-Many with JOINs**
```sql
SELECT students.name, courses.title
FROM enrollments
JOIN students ON enrollments.student_id = students.id
JOIN courses ON enrollments.course_id = courses.id;
```

---

## 4. Summary of Join Types
| Join Type     | Returns                                                  |
|---------------|-----------------------------------------------------------|
| INNER JOIN    | Only matching rows from both tables                      |
| LEFT JOIN     | All rows from the left table, and matched rows from right|
| RIGHT JOIN    | All rows from the right table, and matched rows from left|
| FULL JOIN     | All rows when there's a match in either table            |
| CROSS JOIN    | Cartesian product of two tables                          |

---

## Final Thoughts
- Use joins to fetch related data efficiently.
- Always define foreign key relationships clearly.
- Normalize tables to take full advantage of joins.
- Be careful with CROSS JOINs due to their size impact.

By understanding and applying these types of joins, you can query complex, normalized relational data with confidence.
