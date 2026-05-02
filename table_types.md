# Snowflake Table Types – Master Guide

## Overview
This document provides an in-depth, structured, and production-grade understanding of all Snowflake table types. It focuses on internal behavior, properties, implementation, use cases, and common misconceptions.

---

# 1. Permanent Tables

## Definition
Permanent tables are the default table type in Snowflake and are designed for storing critical, long-term production data. These tables support full data protection mechanisms including Time Travel and Fail-safe, ensuring recoverability and durability. They are tightly integrated with Snowflake’s micro-partition storage layer and support ACID transactions through MVCC.

## Properties
- Time Travel: 1–90 days
- Fail-safe: 7 days (non-configurable)
- Fully recoverable
- Highest storage cost due to retention

## Implementation
```sql
CREATE TABLE orders (
    id INT,
    amount NUMBER
);
```

## When to Use
- Core data warehouse tables
- Financial or audit data
- Business-critical datasets

## Misconceptions
- Not all tables should be permanent; they incur higher storage costs due to fail-safe.

---

# 2. Transient Tables

## Definition
Transient tables are similar to permanent tables but without fail-safe protection. They are designed for cost optimization where data recovery beyond Time Travel is not required.

## Properties
- Time Travel: 0–1 day
- No Fail-safe
- Lower storage cost

## Implementation
```sql
CREATE TRANSIENT TABLE staging_orders (
    id INT,
    amount NUMBER
);
```

## When to Use
- Staging layers
- Intermediate transformations
- Re-creatable data

## Misconceptions
- Not unsafe by default; suitable where recovery is not needed.

---

# 3. Temporary Tables

## Definition
Temporary tables are session-scoped tables that exist only for the duration of a user session. They are not visible outside the session and are automatically dropped when the session ends.

## Properties
- Lifetime: Session-only
- No Fail-safe
- No Time Travel (or minimal)

## Implementation
```sql
CREATE TEMP TABLE temp_orders (
    id INT,
    amount NUMBER
);
```

## When to Use
- Intermediate computations
- Debugging
- Temporary joins

## Misconceptions
- Not suitable for pipelines or shared workflows.

---

# 4. Dynamic Tables

## Definition
Dynamic tables are declarative pipeline tables that automatically refresh based on defined logic. They act as managed transformation layers with dependency tracking and incremental refresh capabilities.

## Properties
- Automatic refresh using TARGET_LAG
- DAG-based dependency tracking
- Incremental computation (when possible)

## Implementation
```sql
CREATE DYNAMIC TABLE daily_sales
TARGET_LAG = '1 hour'
AS
SELECT date, SUM(amount)
FROM orders
GROUP BY date;
```

## When to Use
- ETL/ELT pipelines
- Aggregation layers
- Data marts

## Misconceptions
- Not always incremental; may recompute fully.

---

# 5. Hybrid Tables

## Definition
Hybrid tables are designed for low-latency transactional workloads, combining row-based and columnar storage. They bring OLTP-like capabilities to Snowflake.

## Properties
- Supports primary keys and constraints
- Row-based access optimized
- Low latency

## Implementation
```sql
CREATE HYBRID TABLE users (
    id INT PRIMARY KEY,
    name STRING
);
```

## When to Use
- Real-time applications
- High-frequency updates
- Key-value access patterns

## Misconceptions
- Not suitable for large analytical workloads.

---

# 6. External Tables

## Definition
External tables allow querying data stored outside Snowflake in cloud storage like S3, ADLS, or GCS. Only metadata is stored in Snowflake.

## Properties
- Data remains external
- Lower storage cost
- Higher query latency

## Implementation
```sql
CREATE EXTERNAL TABLE ext_orders
LOCATION = @my_stage
FILE_FORMAT = (TYPE = PARQUET);
```

## When to Use
- Data lake integration
- Rarely accessed data
- Cost-sensitive storage

## Misconceptions
- Not as performant as internal tables.

---

# 7. Iceberg Tables

## Definition
Iceberg tables use the Apache Iceberg open table format, enabling interoperability across multiple data engines while maintaining Snowflake query capabilities.

## Properties
- Open format
- Supports ACID transactions
- Schema evolution

## Implementation
```sql
CREATE ICEBERG TABLE iceberg_orders (
    id INT,
    amount NUMBER
);
```

## When to Use
- Multi-engine architectures
- Data lakehouse setups

## Misconceptions
- Not as optimized as native Snowflake tables within Snowflake.

---

# Final Mental Model

- Permanent → reliability and durability
- Transient → cost optimization
- Temporary → session-level computation
- Dynamic → pipelines and transformations
- Hybrid → transactional workloads
- External → data lake integration
- Iceberg → open ecosystem interoperability

---

# End of Guide
