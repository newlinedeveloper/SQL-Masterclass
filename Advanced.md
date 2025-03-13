### Advanced SQL 
---

## **1️⃣ Window Functions**  
Window functions allow you to perform calculations across a set of table rows related to the current row without collapsing rows into a single result.  

### **Types of Window Functions**  
1. **Ranking Functions**  
   - `RANK()`
   - `DENSE_RANK()`
   - `ROW_NUMBER()`
   - `NTILE(n)`

2. **Aggregate Functions Over a Window**  
   - `SUM()`
   - `AVG()`
   - `MIN()`
   - `MAX()`
   - `COUNT()`

3. **Offset Functions**  
   - `LEAD()`
   - `LAG()`
   - `FIRST_VALUE()`
   - `LAST_VALUE()`

### **Example Use Case: Find the Top 3 Products Per Month**  
```sql
SELECT 
    month, 
    product_name, 
    total_sales,
    RANK() OVER (PARTITION BY month ORDER BY total_sales DESC) AS rank
FROM sales_data
WHERE rank <= 3;
```
🔹 **Use Case:** Helps businesses identify top-selling products per month.

---

## **2️⃣ Common Table Expressions (CTEs)**  
CTEs are temporary result sets that improve query readability and maintainability.

### **Types of CTEs**  
1. **Simple CTE**  
2. **Recursive CTE** (for hierarchical structures)

### **Example Use Case: Employee Hierarchy**  
```sql
WITH RECURSIVE EmployeeHierarchy AS (
    SELECT id, name, manager_id, 1 AS level
    FROM employees
    WHERE manager_id IS NULL
    UNION ALL
    SELECT e.id, e.name, e.manager_id, eh.level + 1
    FROM employees e
    JOIN EmployeeHierarchy eh ON e.manager_id = eh.id
)
SELECT * FROM EmployeeHierarchy;
```
🔹 **Use Case:** Retrieves hierarchical data like organization charts.

---

## **3️⃣ Indexes & Performance Tuning**  
Indexes speed up query execution by reducing the amount of data scanned.

### **Types of Indexes**  
1. **B-Tree Index** (default, good for range queries)  
2. **Hash Index** (for exact matches)  
3. **GIN & GiST Indexes** (for full-text search)  
4. **Partial Indexes** (for specific conditions)  
5. **Covering Indexes** (indexes all needed columns)  

### **Example: Optimizing Search on Customer Name**  
```sql
CREATE INDEX idx_customer_name ON customers (name);
SELECT * FROM customers WHERE name = 'John Doe';
```
🔹 **Use Case:** Improves search speed in large datasets.

---

## **4️⃣ Partitioning**  
Partitioning divides a table into smaller, manageable pieces.

### **Types of Partitioning**  
1. **Range Partitioning** (based on value range)  
2. **List Partitioning** (based on predefined values)  
3. **Hash Partitioning** (based on a hash function)  
4. **Composite Partitioning** (combination of multiple types)  

### **Example Use Case: Partitioning Sales Data by Year**  
```sql
CREATE TABLE sales (
    id SERIAL PRIMARY KEY,
    year INT NOT NULL,
    amount DECIMAL(10,2) NOT NULL
) PARTITION BY RANGE (year);

CREATE TABLE sales_2023 PARTITION OF sales FOR VALUES FROM (2023) TO (2024);
```
🔹 **Use Case:** Improves query performance for large datasets.

---

## **5️⃣ Normalization & Denormalization**  
Normalization improves data integrity, while denormalization improves read performance.

### **Normalization Levels**  
1. **1NF (First Normal Form)** – Remove duplicate columns  
2. **2NF (Second Normal Form)** – Remove partial dependencies  
3. **3NF (Third Normal Form)** – Remove transitive dependencies  

### **Example Use Case: Breaking Customer and Orders into Normalized Tables**  
```sql
CREATE TABLE customers (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100)
);

CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    customer_id INT REFERENCES customers(id),
    order_date TIMESTAMP DEFAULT NOW()
);
```
🔹 **Use Case:** Reduces redundancy, ensuring data consistency.

---

## **6️⃣ JSON & NoSQL Features in SQL**  
Modern SQL databases allow semi-structured data storage.

### **Example Use Case: Storing and Querying JSON Data**  
```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    preferences JSONB
);

INSERT INTO users (name, preferences) 
VALUES ('Alice', '{"theme": "dark", "notifications": true}');

SELECT name, preferences->>'theme' FROM users WHERE name = 'Alice';
```
🔹 **Use Case:** Helps store dynamic user preferences.

---

## **7️⃣ Advanced Joins**  
Joins help combine data from multiple tables.

### **Types of Joins**  
1. **INNER JOIN** (matches in both tables)  
2. **LEFT JOIN** (all from left, matched from right)  
3. **RIGHT JOIN** (all from right, matched from left)  
4. **FULL JOIN** (all data from both tables)  
5. **SELF JOIN** (join the same table)  
6. **CROSS JOIN** (cartesian product)  
7. **LATERAL JOIN** (dependent subqueries)  

### **Example Use Case: Comparing Latest and Previous Orders**  
```sql
SELECT c.id, c.name, o1.order_date AS latest_order, o2.order_date AS previous_order
FROM customers c
JOIN orders o1 ON c.id = o1.customer_id
LEFT JOIN orders o2 ON c.id = o2.customer_id AND o2.order_date < o1.order_date
ORDER BY c.id, o1.order_date DESC;
```
🔹 **Use Case:** Helps analyze customer behavior.

---

## **8️⃣ Stored Procedures & Triggers**  
Stored procedures automate operations, and triggers react to table changes.

### **Example Use Case: Trigger for Low Stock Notification**  
```sql
CREATE OR REPLACE FUNCTION notify_low_stock() RETURNS TRIGGER AS $$
BEGIN
    IF NEW.stock < 10 THEN
        RAISE NOTICE 'Low stock warning for %', NEW.product_name;
    END IF;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER low_stock_trigger
AFTER UPDATE ON inventory
FOR EACH ROW EXECUTE FUNCTION notify_low_stock();
```
🔹 **Use Case:** Alerts when inventory is low.

---

## **9️⃣ Transactions & Isolation Levels**  
Transactions ensure atomicity, consistency, isolation, and durability (ACID).

### **Isolation Levels**  
1. **READ UNCOMMITTED** – Allows dirty reads  
2. **READ COMMITTED** – Prevents dirty reads  
3. **REPEATABLE READ** – Prevents non-repeatable reads  
4. **SERIALIZABLE** – Prevents phantom reads  

### **Example Use Case: Handling Airline Booking Transactions**  
```sql
BEGIN;
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;

UPDATE tickets SET status = 'Booked' WHERE id = 101 AND status = 'Available';

COMMIT;
```
🔹 **Use Case:** Prevents overbooking.

---

## **🔟 Materialized Views**  
Materialized views store query results for fast access.

### **Example Use Case: Precomputing Data for Dashboards**  
```sql
CREATE MATERIALIZED VIEW monthly_sales AS
SELECT 
    EXTRACT(MONTH FROM order_date) AS month,
    SUM(amount) AS total_sales
FROM orders
GROUP BY month;

REFRESH MATERIALIZED VIEW monthly_sales;
```
🔹 **Use Case:** Reduces computation load for analytics dashboards.

---
