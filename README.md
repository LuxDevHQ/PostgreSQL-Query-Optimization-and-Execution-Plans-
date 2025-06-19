# PostgreSQL Query Optimization and Execution Plans (Beginner Level)

## Objective

To introduce students to the basics of query optimization in PostgreSQL and understand how the database engine interprets and executes SQL queries. This will help them write efficient queries and understand how to debug slow-performing SQL code.

## 1. Introduction to Query Optimization

### What is Query Optimization?

Query optimization is the process of enhancing the performance of a query by improving its execution plan. PostgreSQL does this automatically using its internal query planner.

### Why is it Important?

- Reduces query execution time
- Minimizes resource usage (CPU, memory, I/O)
- Improves user experience in applications

## 2. PostgreSQL Query Planner and Executor

### Query Lifecycle:

1. **Parsing** – SQL query is parsed into a tree
2. **Rewriting** – Query is rewritten based on rules
3. **Planning/Optimization** – Multiple execution strategies are evaluated; the best one is chosen
4. **Execution** – The plan is executed

## 3. EXPLAIN and EXPLAIN ANALYZE

### EXPLAIN

Shows the execution plan that PostgreSQL thinks is most efficient.

```sql
EXPLAIN SELECT * FROM employees WHERE department = 'HR';
```

### EXPLAIN ANALYZE

Actually runs the query and shows the real execution time and cost.

```sql
EXPLAIN ANALYZE SELECT * FROM employees WHERE department = 'HR';
```

### Understanding EXPLAIN Output

The output shows:
- **Cost**: Estimated startup and total cost
- **Rows**: Estimated number of rows returned
- **Width**: Average width of rows in bytes
- **Node Types**: Different operations (Seq Scan, Index Scan, etc.)

### Example Output:
```
Seq Scan on employees  (cost=0.00..15.00 rows=5 width=100)
  Filter: (department = 'HR'::text)
```

## 4. Common Query Operations

### Sequential Scan (Seq Scan)
- Reads every row in the table
- Used when no suitable index exists
- Expensive for large tables

### Index Scan
- Uses an index to find rows quickly
- Much faster than sequential scan for selective queries
- Requires appropriate indexes

### Nested Loop Join
- For each row in the outer table, searches the inner table
- Good for small datasets
- Can be slow for large joins

### Hash Join
- Builds a hash table from one relation
- Probes the hash table with the other relation
- Efficient for larger datasets

## 5. Basic Optimization Techniques

### Creating Indexes
```sql
-- Create an index on frequently queried columns
CREATE INDEX idx_employee_department ON employees(department);
```

### Using WHERE Clauses Effectively
```sql
-- Good: Uses selective filtering
SELECT * FROM employees WHERE employee_id = 12345;

-- Less efficient: Non-selective filtering
SELECT * FROM employees WHERE salary > 1000;
```

### Limiting Results
```sql
-- Use LIMIT to reduce data transfer
SELECT * FROM employees ORDER BY hire_date DESC LIMIT 10;
```

## 6. Analyzing Query Performance

### Using EXPLAIN (ANALYZE, BUFFERS)
```sql
EXPLAIN (ANALYZE, BUFFERS) 
SELECT e.name, d.department_name 
FROM employees e 
JOIN departments d ON e.department_id = d.id 
WHERE e.salary > 50000;
```

### Key Metrics to Watch:
- **Execution Time**: Total time to run the query
- **Planning Time**: Time spent creating the execution plan
- **Shared Hit**: Data found in PostgreSQL's cache
- **Shared Read**: Data read from disk

## 7. Common Performance Issues

### Missing Indexes
```sql
-- Problem: Sequential scan on large table
SELECT * FROM orders WHERE customer_id = 123;

-- Solution: Create appropriate index
CREATE INDEX idx_orders_customer_id ON orders(customer_id);
```

### Inefficient JOINs
```sql
-- Less efficient: Cartesian product risk
SELECT * FROM employees, departments 
WHERE employees.name LIKE '%John%';

-- Better: Proper JOIN with conditions
SELECT e.name, d.department_name 
FROM employees e 
JOIN departments d ON e.department_id = d.id 
WHERE e.name LIKE '%John%';
```

## 8. Best Practices for Beginners

### Query Writing Tips:
- Use specific column names instead of `SELECT *`
- Add appropriate WHERE clauses to filter data
- Use JOINs instead of subqueries when possible
- Consider using LIMIT for large result sets

### Index Strategy:
- Create indexes on frequently queried columns
- Consider composite indexes for multi-column queries
- Don't over-index (impacts INSERT/UPDATE performance)
- Monitor index usage with `pg_stat_user_indexes`

### Regular Maintenance:
```sql
-- Update table statistics
ANALYZE employees;

-- Rebuild indexes if needed
REINDEX INDEX idx_employee_department;
```

## 9. Practice Exercises

### Exercise 1: Basic EXPLAIN
```sql
-- Run EXPLAIN on this query and interpret the results
EXPLAIN SELECT name, salary FROM employees WHERE department = 'Engineering';
```

### Exercise 2: Compare Performance
```sql
-- Compare these two queries using EXPLAIN ANALYZE
-- Query 1:
SELECT * FROM employees WHERE salary BETWEEN 40000 AND 60000;

-- Query 2 (with index):
CREATE INDEX idx_salary ON employees(salary);
SELECT * FROM employees WHERE salary BETWEEN 40000 AND 60000;
```

### Exercise 3: JOIN Optimization
```sql
-- Analyze this JOIN query
EXPLAIN ANALYZE 
SELECT e.name, d.department_name, e.salary 
FROM employees e 
JOIN departments d ON e.department_id = d.id 
WHERE d.department_name = 'Sales';
```

## Summary

Understanding query optimization helps you:
- Write more efficient SQL queries
- Identify performance bottlenecks
- Make informed decisions about indexing
- Debug slow-running queries

Remember: Start simple, measure performance, and optimize based on actual usage patterns rather than assumptions.
