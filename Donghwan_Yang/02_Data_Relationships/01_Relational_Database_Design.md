# Relational Database Design

Designing a relational database involves planning how data will be structured, stored, and related within tables to ensure consistency, reduce redundancy, and support efficient querying.

---

## 1. What is Relational Database Design?
Relational database design is the process of organizing data into tables (relations) based on principles of **data normalization** and **relationship modeling**. It helps:

- Prevent data anomalies
- Ensure data integrity
- Reduce duplication (redundancy)
- Improve scalability and maintainability

---

## 2. Key Concepts in Relational Design

### **Entities and Attributes**
- **Entity**: A real-world object (e.g., customer, product, order).
- **Attribute**: A property of the entity (e.g., name, price, date).

### **Relationships**
- Describes how two entities relate to each other (e.g., one-to-many, many-to-many).
- Implemented through **foreign keys** in relational databases.

### **Primary Keys and Foreign Keys**
- **Primary Key**: A unique identifier for each record.
- **Foreign Key**: A reference to the primary key of another table.

---

## 3. Normalization to Handle Duplication
Normalization is the process of organizing tables to reduce data duplication and improve data integrity.

### **Common Normal Forms**

#### **1st Normal Form (1NF)**
- Ensure each field contains only atomic values (no multiple values).
- Example:
  - ❌ `skills = "Python, SQL"`
  - ✅ Separate into multiple rows or a new table.

#### **2nd Normal Form (2NF)**
- Must be in 1NF.
- Remove partial dependencies (i.e., all non-key attributes must depend on the whole primary key).

#### **3rd Normal Form (3NF)**
- Must be in 2NF.
- Remove transitive dependencies (i.e., non-key attributes should not depend on other non-key attributes).

### **Why Normalize?**
- Avoid duplication of data (e.g., customer name repeated in every order row).
- Easier updates and consistency.
- Reduced storage waste.
- **Easier column value modification**: When a column (e.g., product price) exists only in one table, updates can be made in one place without affecting multiple rows or causing inconsistencies.

---

## 4. Designing with Duplication in Mind

### **Step-by-Step Process**

1. **Identify Entities**
   - E.g., Customers, Orders, Products

2. **Define Attributes for Each Entity**
   - Customers: name, email, phone
   - Products: name, price
   - Orders: order_date, total_amount

3. **Determine Relationships**
   - One customer can place many orders (1:N)
   - An order can contain multiple products (M:N)

4. **Create Separate Tables to Avoid Duplication**
   - Store customer info once in a `customers` table
   - Use `customer_id` in `orders` table as a foreign key
   - Use a junction table (`order_items`) for many-to-many between `orders` and `products`

5. **Apply Normalization Rules**
   - Eliminate repeating groups and redundant fields
   - Ensure foreign key relationships are used to reference shared data

---

## 5. Example: From Flat to Normalized Design

### ❌ Flat Table (Redundant and Unnormalized)
| order_id | customer_name | product_name | product_price |
|----------|----------------|---------------|----------------|
| 1        | Alice          | Keyboard      | 50             |
| 1        | Alice          | Mouse         | 20             |
| 2        | Bob            | Monitor       | 200            |

### ✅ Normalized Structure
- `customers(id, name)`
- `orders(id, customer_id, order_date)`
- `products(id, name, price)`
- `order_items(order_id, product_id, quantity)`

This structure avoids duplicating customer and product information and allows for easier updates.

---

## 6. Summary
- **Relational database design** ensures your data is structured efficiently.
- **Normalization** is key to handling duplicate data and maintaining integrity.
- Use **primary and foreign keys** to link data across tables.
- Always evaluate relationships and eliminate redundancy through **well-designed schemas**.
- **Dividing tables improves modifiability**: When data is normalized, changes (e.g., updating a product's price) only need to be made once, reducing the risk of inconsistencies and errors.

Proper design leads to better performance, scalability, and ease of maintenance.
