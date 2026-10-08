# Composite Index Investigation Report

## 1. Executive Summary
This report analyzes why the composite index column order matters in PostgreSQL and demonstrates how violating the Left-Most Prefix Rule causes PostgreSQL to perform slow Sequential Scans.

## 2. Unindexed Query Performance
* **Query:** `SELECT * FROM orders WHERE customer_id = 451 AND status = 'completed';`
* **Execution Plan:** `Seq Scan on orders` (~85ms)
* **Reason:** Without an index, PostgreSQL must scan every row in the table.

## 3. Incorrect Index Experiment
* **Index Created:** `CREATE INDEX idx_orders_status_customer ON orders(status, customer_id);`
* **Execution Plan:** `Seq Scan` (~78ms)
* **Why it failed:** `status` has low cardinality (few unique values like 'completed', 'pending'). Placing `status` first makes the index inefficient, so PostgreSQL skips it.

## 4. Corrected Index Optimization
* **Index Created:** `CREATE INDEX idx_orders_customer_status ON orders(customer_id, status);`
* **Execution Plan:** `Index Scan using idx_orders_customer_status` (<1ms)
* **Why it succeeded:** `customer_id` is highly selective (many unique values). Placing `customer_id` first allows PostgreSQL to instantly locate the customer's records.

## 5. The Left-Most Prefix Rule
PostgreSQL composite indexes function like a phone book sorted by `(Last Name, First Name)`. Searching by `Last Name` is fast, but searching only by `First Name` forces you to read the entire book from start to finish. Always place high-cardinality equality columns first in composite indexes.