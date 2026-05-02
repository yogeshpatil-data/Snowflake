# Snowflake Architecture – Master Guide

## Structured Plan

1. High-Level Architecture Overview  
2. Three Core Layers (Storage, Compute, Cloud Services)  
3. Micro-Partition Internals  
4. Query Lifecycle End-to-End  
5. Caching Mechanisms  
6. Query Optimizer Internals  
7. MVCC and Concurrency Model  
8. Clustering & Partitioning Concepts  
9. Performance & Cost Considerations  
10. Common Misconceptions & Interview Traps  

---

# 1. High-Level Architecture Overview

Snowflake follows a **multi-cluster shared-data architecture**, which fundamentally separates compute and storage. This is different from traditional systems where compute and storage are tightly coupled. The separation allows independent scaling, meaning storage can grow without impacting compute performance and vice versa. This design enables high concurrency, as multiple compute clusters can operate on the same underlying data simultaneously. It also allows cost optimization, as compute resources can be paused when not in use. This architecture is one of the most critical concepts to understand for interviews.

---

# 2. Storage Layer

The storage layer is responsible for persisting data in Snowflake. Data is stored in compressed, columnar format in cloud storage such as Amazon S3, Azure Blob Storage, or Google Cloud Storage. Internally, Snowflake organizes data into **micro-partitions**, which are immutable and automatically managed. Each micro-partition contains both data and metadata, enabling efficient pruning during query execution. Because storage is decoupled, all compute clusters access the same data without duplication. This layer is optimized for scalability and durability rather than direct query execution.

---

# 3. Compute Layer (Virtual Warehouses)

The compute layer consists of virtual warehouses, which are clusters of compute resources used to execute queries. Each warehouse operates independently, ensuring no resource contention between workloads. Warehouses can be scaled up (more power) or scaled out (more clusters) depending on workload requirements. They also support auto-suspend and auto-resume, helping control costs. Importantly, warehouses do not store data; they only process it. This separation ensures that compute failures do not affect stored data.

---

# 4. Cloud Services Layer

The cloud services layer acts as the brain of Snowflake. It manages query parsing, optimization, metadata storage, authentication, and transaction handling. It also maintains statistics about data, which are critical for query optimization. This layer ensures ACID compliance using MVCC and manages concurrency across multiple users. It also handles access control using role-based security. Without this layer, Snowflake would not be able to coordinate distributed query execution efficiently.

---

# 5. Micro-Partition Internals

Micro-partitions are the fundamental storage unit in Snowflake, typically around 16MB compressed. They store data in a columnar format and include metadata such as min/max values, distinct counts, and null counts. This metadata allows Snowflake to skip irrelevant partitions during query execution, a process known as partition pruning. Micro-partitions are immutable, meaning updates create new partitions rather than modifying existing ones. This immutability enables features like Time Travel and zero-copy cloning.

---

# 6. Query Lifecycle

When a query is submitted, it first goes to the cloud services layer for parsing and optimization. The optimizer generates an execution plan using metadata and statistics. The query is then assigned to a virtual warehouse, which executes it by scanning relevant micro-partitions. Only necessary data is retrieved due to pruning and projection pushdown. Results are then returned to the user and may be cached for future use. This entire lifecycle is designed to minimize data movement and maximize efficiency.

---

# 7. Caching Mechanisms

Snowflake uses multiple caching layers to improve performance. The result cache stores final query results and can return results instantly if the same query is executed again. The metadata cache stores partition information, reducing lookup time. The data cache stores frequently accessed data in the warehouse memory. These caching layers significantly reduce query execution time and compute cost. Understanding caching is crucial for debugging performance issues.

---

# 8. Query Optimizer

Snowflake uses a cost-based optimizer that determines the most efficient way to execute a query. It uses metadata from micro-partitions to minimize data scanning. The optimizer performs predicate pushdown, projection pushdown, join reordering, and partition pruning. It also decides between broadcast and distributed joins based on data size. However, the optimizer is not perfect and depends heavily on data distribution and clustering. Poor data organization can lead to suboptimal plans.

---

# 9. MVCC (Concurrency Model)

Snowflake uses Multi-Version Concurrency Control (MVCC) to handle concurrent operations. Instead of locking data, it creates new versions of data for updates. Each query sees a consistent snapshot of data as of its start time. This eliminates read-write conflicts and improves concurrency. MVCC also enables Time Travel by retaining historical versions of data. This approach is more efficient than traditional locking mechanisms used in relational databases.

---

# 10. Clustering & Partitioning

Snowflake does not support manual partitioning like traditional systems. Instead, it uses automatic micro-partitioning. Clustering is used to improve data locality within micro-partitions. A clustering key helps organize data to reduce overlap between partitions. This improves query performance by enabling better pruning. However, clustering requires maintenance and can increase costs due to automatic reclustering. It should only be used when necessary.

---

# 11. Performance & Cost Considerations

Performance in Snowflake depends on query design, clustering, and warehouse size. Cost is primarily driven by compute usage and storage retention. Using appropriate warehouse sizes and auto-suspend settings can reduce costs. Avoid unnecessary clustering and materialized views, as they incur additional compute costs. Proper data modeling and query optimization are essential for balancing performance and cost.

---

# 12. Common Misconceptions

- Snowflake does not use indexes; it relies on metadata pruning.  
- Virtual warehouses do not store data.  
- Clustering is not the same as partitioning.  
- Scaling compute does not affect storage performance.  
- Automatic clustering is not free.  

---

# Final Mental Model

- Storage = where data lives  
- Compute = where queries run  
- Cloud Services = brain controlling everything  

---

# End of Guide
