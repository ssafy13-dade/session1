# PostgreSQL Data Types

PostgreSQL provides various data types to store different kinds of data efficiently. Below are some commonly used types categorized into Binary, Numeric, Text, and Date types.

## 1. Binary Types
Binary data types store raw binary data (byte sequences) and are useful for storing images, files, or other binary objects.

### **BYTEA** (Binary Data)
- Used for storing variable-length binary data.
- Data is stored in a format that allows binary input and output.
- Example:
  
  ```sql
  CREATE TABLE files (
      id SERIAL PRIMARY KEY,
      filename TEXT NOT NULL,
      data BYTEA NOT NULL
  );
  
  INSERT INTO files (filename, data) VALUES ('image.png', '\xDEADBEEF');
  ```

## 2. Numeric Types
PostgreSQL supports a wide range of numeric types, including integers, floating points, and precise numeric values.

### **Integer Types**
| Type      | Storage Size | Range |
|-----------|--------------|----------------------------------|
| SMALLINT  | 2 bytes      | -32,768 to 32,767              |
| INTEGER   | 4 bytes      | -2,147,483,648 to 2,147,483,647 |
| BIGINT    | 8 bytes      | -9,223,372,036,854,775,808 to 9,223,372,036,854,775,807 |

Example:
```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    age SMALLINT
);
```

### **Floating-Point Types**
| Type      | Storage Size | Precision |
|-----------|--------------|-----------------|
| REAL      | 4 bytes      | 6 decimal digits |
| DOUBLE PRECISION | 8 bytes | 15 decimal digits |

Example:
```sql
CREATE TABLE measurements (
    id SERIAL PRIMARY KEY,
    temperature REAL,
    pressure DOUBLE PRECISION
);
```

### **Numeric (Exact Precision)**
- `NUMERIC(precision, scale)` or `DECIMAL(precision, scale)`
- Used for precise calculations, especially in financial applications.
- Example:
  
  ```sql
  CREATE TABLE transactions (
      id SERIAL PRIMARY KEY,
      amount NUMERIC(10, 2) -- Up to 10 digits, 2 after the decimal
  );
  ```

## 3. Text Types
Text types store character data, including short and long strings.

### **CHAR(n) (Fixed-Length String)**
- Stores a fixed number of characters (n). If the string is shorter, it is padded with spaces.
- Example:
  ```sql
  CREATE TABLE employees (
      id SERIAL PRIMARY KEY,
      department CHAR(10)
  );
  ```

### **VARCHAR(n) (Variable-Length String)**
- Stores a string of up to `n` characters. Unlike `CHAR`, it does not pad with spaces.
- Example:
  ```sql
  CREATE TABLE customers (
      id SERIAL PRIMARY KEY,
      name VARCHAR(50)
  );
  ```

### **TEXT (Unlimited Length String)**
- Stores an unlimited-length string.
- Example:
  ```sql
  CREATE TABLE articles (
      id SERIAL PRIMARY KEY,
      content TEXT
  );
  ```

## 4. Date & Time Types
PostgreSQL offers powerful date and time data types for handling temporal data.

### **DATE** (Stores Date Only)
- Stores only the date (year, month, day) without time.
- Format: `YYYY-MM-DD`
- Example:
  
  ```sql
  CREATE TABLE events (
      id SERIAL PRIMARY KEY,
      event_date DATE
  );
  ```

### **TIME** (Stores Time Only)
- Stores only the time of day without a date.
- Format: `HH:MI:SS`
- Example:
  
  ```sql
  CREATE TABLE schedules (
      id SERIAL PRIMARY KEY,
      start_time TIME
  );
  ```

### **TIMESTAMP** (Stores Date & Time)
- Stores both date and time.
- Format: `YYYY-MM-DD HH:MI:SS`
- Example:
  
  ```sql
  CREATE TABLE logs (
      id SERIAL PRIMARY KEY,
      created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
  );
  ```

### **INTERVAL** (Stores Time Interval)
- Represents a period of time (e.g., days, months, years).
- Example:
  
  ```sql
  CREATE TABLE subscriptions (
      id SERIAL PRIMARY KEY,
      duration INTERVAL
  );
  ```

## Summary
PostgreSQL provides a variety of data types to store and manipulate different kinds of data efficiently:
- **Binary Types**: `BYTEA` for storing raw binary data.
- **Numeric Types**: `INTEGER`, `REAL`, `NUMERIC` for storing numbers.
- **Text Types**: `CHAR`, `VARCHAR`, `TEXT` for handling character data.
- **Date & Time Types**: `DATE`, `TIME`, `TIMESTAMP`, `INTERVAL` for handling temporal data.

These types help ensure efficient data storage and retrieval while maintaining data integrity.
