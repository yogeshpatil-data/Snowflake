# Snowflake Cost & Performance Optimization – Master Guide

## Structured Plan

1. Core Cost Model in Snowflake  
2. Compute Optimization (Warehouses)  
3. Storage Optimization (Tables & Retention)  
4. Query Performance Optimization  
5. Caching Mechanisms Deep Dive  
6. Clustering & Pruning Optimization  
7. Data Modeling Best Practices  
8. Workload Management & Concurrency  
9. Monitoring & Observability  
10. Common Anti-Patterns & Misconceptions  
11. Real Production Strategy  

---

# 1. Core Cost Model in Snowflake

Snowflake cost is primarily driven by two components: compute and storage. Compute cost depends on virtual warehouse usage, billed per second with a minimum charge. Storage cost depends on compressed data stored, including time travel and fail-safe retention. Understanding this separation is critical because optimizing one does not automatically optimize the other. Many beginners assume query cost is purely data size dependent, but in Snowflake, compute usage dominates. Therefore, efficient query execution and warehouse tuning directly impact cost.

---

# 2. Compute Optimization (Warehouses)

Virtual warehouses execute queries and are the main source of compute cost. Proper sizing is critical: small warehouses may lead to slow queries, while oversized warehouses waste money. Auto-suspend and auto-resume features help reduce idle costs by shutting down warehouses when not in use. Multi-cluster warehouses can handle concurrency but increase cost if over-provisioned. Always align warehouse size with workload type, such as ETL vs BI queries. Monitoring warehouse utilization is essential for optimization.

## Example

```sql
ALTER WAREHOUSE my_wh 
SET WAREHOUSE_SIZE = 'MEDIUM', 
AUTO_SUSPEND = 60, 
AUTO_RESUME = TRUE;
```

---

# 3. Storage Optimization

Storage cost includes active data, time travel data, and fail-safe retention. Permanent tables incur higher costs due to extended retention. Transient tables reduce cost by eliminating fail-safe storage. Reducing time travel retention for non-critical tables can significantly lower storage costs. Data compression is automatic, but unnecessary duplication of data increases storage usage. Regular cleanup of unused tables is important.

## Example

```sql
ALTER TABLE orders 
SET DATA_RETENTION_TIME_IN_DAYS = 1;
```

---

# 4. Query Performance Optimization

Efficient queries reduce compute usage and improve performance. Always filter data early using WHERE clauses to enable partition pruning. Avoid SELECT * and instead project only required columns. Joins should be optimized by ensuring smaller tables are broadcast when possible. Aggregations should be performed after filtering to reduce data size. Poor query design can lead to full table scans, increasing both cost and latency.

---

# 5. Caching Mechanisms

Snowflake uses three types of caching: result cache, metadata cache, and data cache. Result cache returns results instantly if the same query is executed again. Metadata cache speeds up partition pruning decisions. Data cache stores frequently accessed data in warehouse memory. Leveraging caching can significantly reduce compute usage. However, cache invalidation occurs when underlying data changes.

## Example

```sql
ALTER SESSION SET USE_CACHED_RESULT = TRUE

```


---

# 6. Clustering & Pruning Optimization

Clustering improves data organization within micro-partitions, enabling efficient pruning. Without clustering, data may be scattered, leading to more partitions being scanned. Clustering depth measures how well data is organized. Automatic clustering maintains data organization but incurs additional compute cost. Clustering should be applied only to large tables with frequent filtering on specific columns.

## Example

```sql
-- Create table with clustering
CREATE TABLE orders (
    order_id NUMBER,
    customer_id NUMBER,
    order_date DATE
)
CLUSTER BY (order_date, customer_id);

-- Add clustering to an existing table
ALTER TABLE orders CLUSTER BY (order_date);
```

---

# 7. Data Modeling Best Practices

Proper data modeling improves performance and reduces cost. Use denormalization where appropriate to reduce join complexity. Avoid excessive small tables, as they increase metadata overhead. Use appropriate table types such as transient tables for staging. Ensure data is loaded in a way that preserves natural clustering. Poor modeling leads to inefficient queries and higher costs.

---

# 8. Workload Management & Concurrency

Managing workloads effectively prevents resource contention. Separate warehouses for ETL, BI, and ad-hoc queries ensure isolation. Multi-cluster warehouses help handle concurrency but should be configured carefully. Resource monitors can prevent runaway costs by limiting usage. Proper workload management ensures consistent performance across users.

---

# 9. Monitoring & Observability

Monitoring is essential for optimization. Query history and warehouse usage views provide insights into performance bottlenecks. Query profiles help identify expensive operations such as scans and joins. Resource monitors track credit usage and can trigger alerts. Regular monitoring helps identify inefficiencies and optimize cost.

## Example
```sql
CREATE OR REPLACE RESOURCE MONITOR limiter
  WITH CREDIT_QUOTA = 5000
       NOTIFY_USERS = (JDOE, "Jane Smith", "John Doe")
  TRIGGERS ON 75 PERCENT DO NOTIFY
           ON 100 PERCENT DO SUSPEND
           ON 110 PERCENT DO SUSPEND_IMMEDIATE;
```

---

# 10. Common Anti-Patterns

- Using large warehouses unnecessarily  
- Running unfiltered queries (full table scans)  
- Overusing clustering  
- Ignoring caching benefits  
- Keeping high time travel retention unnecessarily  

---

# 11. Real Production Strategy

A typical production strategy involves using transient tables for staging, permanent tables for core data, and dynamic tables for transformations. Warehouses are sized based on workload and configured with auto-suspend. Queries are optimized using pruning and projection. Monitoring tools are used to track performance and cost. This layered approach ensures efficient and scalable data processing.

---

# Final Mental Model

- Compute drives cost → optimize queries and warehouses  
- Storage adds overhead → manage retention and table types  
- Performance = pruning + caching + query design  

---

# End of Guide
