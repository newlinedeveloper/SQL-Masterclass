Great! First, let's create a **sample dataset** for the mock interview.  

---

### **Sample Dataset: E-Commerce System**
We'll use the following tables:  
1. **customers** – Stores customer details  
2. **orders** – Stores order details  
3. **products** – Stores product details  
4. **order_items** – Stores items in each order  

#### **SQL to Create & Populate Sample Tables**
```sql
-- Customers Table
CREATE TABLE customers (
    customer_id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(100) UNIQUE,
    city VARCHAR(50),
    signup_date DATE
);

INSERT INTO customers (name, email, city, signup_date) VALUES
('Alice Johnson', 'alice@example.com', 'New York', '2020-05-10'),
('Bob Smith', 'bob@example.com', 'Los Angeles', '2019-09-15'),
('Charlie Brown', 'charlie@example.com', 'Chicago', '2021-01-22');

-- Products Table
CREATE TABLE products (
    product_id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    category VARCHAR(50),
    price DECIMAL(10,2),
    stock_quantity INT
);

INSERT INTO products (name, category, price, stock_quantity) VALUES
('Laptop', 'Electronics', 1200.00, 10),
('Smartphone', 'Electronics', 800.00, 20),
('Tablet', 'Electronics', 400.00, 15),
('Headphones', 'Accessories', 150.00, 30),
('Smartwatch', 'Accessories', 250.00, 25);

-- Orders Table
CREATE TABLE orders (
    order_id SERIAL PRIMARY KEY,
    customer_id INT REFERENCES customers(customer_id),
    order_date TIMESTAMP DEFAULT NOW(),
    total_amount DECIMAL(10,2)
);

INSERT INTO orders (customer_id, order_date, total_amount) VALUES
(1, '2024-03-01 10:15:00', 2000.00),
(2, '2024-03-02 12:30:00', 800.00),
(3, '2024-03-05 14:00:00', 400.00);

-- Order Items Table
CREATE TABLE order_items (
    order_item_id SERIAL PRIMARY KEY,
    order_id INT REFERENCES orders(order_id),
    product_id INT REFERENCES products(product_id),
    quantity INT,
    subtotal DECIMAL(10,2)
);

INSERT INTO order_items (order_id, product_id, quantity, subtotal) VALUES
(1, 1, 1, 1200.00),
(1, 2, 1, 800.00),
(2, 2, 1, 800.00),
(3, 3, 1, 400.00);
```
---

## **🎤 Mock Interview for Senior SQL Developer**
I will ask questions based on different **SQL concepts**. Try to answer them, and I will provide feedback.

---

### **Round 1: SQL Queries & Optimization**
1️⃣ **Can you write a query to get the top 2 best-selling products?**  
💡 Expected Answer:
```sql
SELECT p.name, SUM(oi.quantity) AS total_sold
FROM order_items oi
JOIN products p ON oi.product_id = p.product_id
GROUP BY p.name
ORDER BY total_sold DESC
LIMIT 2;
```

2️⃣ **How would you optimize a query that frequently searches for customers by email?**  
💡 Expected Answer:  
- Create an **index** on the `email` column:
```sql
CREATE INDEX idx_customers_email ON customers(email);
```
- Use **EXPLAIN ANALYZE** to check performance.

---

### **Round 2: Advanced SQL Concepts**
3️⃣ **Write a query using a window function to find each customer's total spending and their ranking among all customers.**  
💡 Expected Answer:
```sql
SELECT c.name, SUM(o.total_amount) AS total_spent,
       RANK() OVER (ORDER BY SUM(o.total_amount) DESC) AS spending_rank
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id
GROUP BY c.name;
```

4️⃣ **What are the different types of partitioning in SQL? When would you use partitioning?**  
💡 Expected Answer:  
- **Types of Partitioning:**  
  - **Range Partitioning:** Splitting data based on ranges, e.g., years.  
  - **List Partitioning:** Groups specific values together.  
  - **Hash Partitioning:** Distributes data across partitions randomly.  
  - **Composite Partitioning:** Combination of the above.

- **When to use?** Large tables with billions of rows (e.g., time-series data like logs).

---

### **Round 3: Real-World Scenarios**
5️⃣ **How would you design a query to detect duplicate customers (same name and city but different email)?**  
💡 Expected Answer:
```sql
SELECT name, city, COUNT(*) 
FROM customers
GROUP BY name, city
HAVING COUNT(*) > 1;
```

6️⃣ **A customer support team wants to retrieve the last 5 orders of each customer. How would you do that?**  
💡 Expected Answer:
```sql
SELECT customer_id, order_id, order_date
FROM (
    SELECT customer_id, order_id, order_date,
           ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY order_date DESC) AS rn
    FROM orders
) sub
WHERE rn <= 5;
```

---

### **Round 4: Transactions & Performance Tuning**
7️⃣ **How do transactions work in SQL? What are isolation levels?**  
💡 Expected Answer:
- Transactions follow **ACID** properties (Atomicity, Consistency, Isolation, Durability).
- **Isolation Levels:**  
  1. **READ UNCOMMITTED** – Allows dirty reads.  
  2. **READ COMMITTED** – Prevents dirty reads.  
  3. **REPEATABLE READ** – Prevents non-repeatable reads.  
  4. **SERIALIZABLE** – Strictest, prevents phantom reads.  

---

### **Round 5: Stored Procedures & Triggers**
8️⃣ **Write a stored procedure that takes a product category as input and returns all products in that category.**  
💡 Expected Answer:
```sql
CREATE OR REPLACE FUNCTION get_products_by_category(category_name VARCHAR)
RETURNS TABLE(product_name VARCHAR, price DECIMAL) AS $$
BEGIN
    RETURN QUERY 
    SELECT name, price FROM products WHERE category = category_name;
END;
$$ LANGUAGE plpgsql;
```

9️⃣ **Write a trigger that automatically reduces stock when an order is placed.**  
💡 Expected Answer:
```sql
CREATE OR REPLACE FUNCTION reduce_stock() RETURNS TRIGGER AS $$
BEGIN
    UPDATE products 
    SET stock_quantity = stock_quantity - NEW.quantity
    WHERE product_id = NEW.product_id;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER order_stock_update
AFTER INSERT ON order_items
FOR EACH ROW EXECUTE FUNCTION reduce_stock();
```

---

### **Final Round: System Design Discussion**
🔟 **How would you design a scalable database schema for a ride-sharing app like Uber?**  
💡 Expected Answer:
- **Tables Needed:** `drivers`, `riders`, `rides`, `payments`, `reviews`.  
- **Indexes on frequently searched fields** like driver ID, rider ID.  
- **Partitioning rides data** (e.g., by date).  
- **Using caching (Redis) for hot data** (active rides).  

---
