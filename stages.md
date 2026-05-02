# Snowflake Stages, Storage Integrations & File Formats — EXPERT LEVEL MASTER GUIDE

---

## 🔷 STRUCTURED DELIVERY PLAN

1. Conceptual Foundation (Stages as ingestion abstraction)
2. Types of Stages (Internal vs External vs Table vs User)
3. Storage Integrations (IAM + Trust Model Deep Dive)
4. File Formats (Full parsing engine understanding)
5. Stage Creation & Querying (Advanced usage patterns)
6. COPY INTO Internals (Execution model + options)
7. Snowpipe Internals (Event-driven ingestion)
8. Performance Engineering (File sizing, parallelism math)
9. Metadata & Observability (Auditing + debugging)
10. Edge Cases, Failure Scenarios & Interview Traps

---

# 🔷 1. CONCEPTUAL FOUNDATION (REAL MODEL)

A stage is not just a pointer — it is a **data access abstraction layer** that allows Snowflake to read external or internal files and convert them into micro-partitions. During ingestion, Snowflake does not directly insert rows; it scans files in parallel, parses them using file formats, and creates compressed columnar micro-partitions. The performance of ingestion depends on file structure, size, and distribution. Stages are therefore tightly coupled with compute efficiency, not just storage access. In real pipelines, stages act as the boundary between raw data (data lake) and structured warehouse layers.

---

# 🔷 2. TYPES OF STAGES (DETAILED)

## 🔶 Internal Stage

Stores files inside Snowflake-managed storage.

### Properties:
- Data physically stored in Snowflake
- Supports PUT/GET commands
- Encrypted automatically
- Best for small-scale ingestion or testing

### Example:
```sql
CREATE STAGE int_stage;
PUT file://data.csv @int_stage;
```

---

## 🔶 External Stage

Points to cloud storage (S3, ADLS, GCS).

### Properties:
- No data duplication
- Requires authentication (integration or keys)
- Scales to TB/PB ingestion
- Industry standard for DE pipelines

### Example:
```sql
CREATE STAGE ext_stage
URL='s3://bucket/path/'
STORAGE_INTEGRATION = s3_int;
```

---

## 🔶 Table Stage

```sql
@%table_name
```
- Scoped to a single table
- Useful for quick ingestion

---

## 🔶 User Stage

```sql
@~
```
- Personal staging
- Not for production pipelines

---

# 🔷 3. STORAGE INTEGRATIONS (DEEP INTERNALS)

Storage integrations use IAM roles instead of credentials. Snowflake assumes the role via a trust relationship. The IAM role trusts Snowflake’s generated IAM user, enabling secure access without exposing keys.

### Flow:
1. Create integration in Snowflake
2. Get IAM user ARN using DESC INTEGRATION
3. Configure trust policy in AWS
4. Grant access to S3 bucket

### Example:
```sql
CREATE STORAGE INTEGRATION s3_int
TYPE = EXTERNAL_STAGE
STORAGE_PROVIDER = S3
ENABLED = TRUE
STORAGE_AWS_ROLE_ARN = 'arn:aws:iam::123:role/snowflake-role'
STORAGE_ALLOWED_LOCATIONS = ('s3://bucket/');
```

---

# 🔷 4. FILE FORMATS (FULL ENGINE)

File formats define parsing rules. Snowflake reads raw bytes and applies transformations before ingestion.

## CSV

```sql
CREATE FILE FORMAT csv_fmt
TYPE = CSV
FIELD_DELIMITER = '|'
FIELD_OPTIONALLY_ENCLOSED_BY = '"'
SKIP_HEADER = 1
NULL_IF = ('NULL','')
TRIM_SPACE = TRUE;
```

## JSON

```sql
CREATE FILE FORMAT json_fmt
TYPE = JSON
STRIP_OUTER_ARRAY = TRUE
IGNORE_UTF8_ERRORS = TRUE;
```

## PARQUET

```sql
CREATE FILE FORMAT pq_fmt
TYPE = PARQUET;
```

### Key Insight:
Parquet = columnar → best for performance

---

# 🔷 5. STAGE QUERYING (ADVANCED)

You can query stage directly:

```sql
SELECT t.$1, t.$2
FROM @stage/file.csv (FILE_FORMAT => csv_fmt) t;
```

### Insight:
- Enables schema-on-read
- Useful for debugging before loading

---

# 🔷 6. COPY INTO INTERNALS

COPY INTO is not just a load — it is a distributed ingestion engine.

### Internal Behavior:
- Each file processed independently
- Parallel threads = number of files
- Micro-partitions created per file chunk
- Atomic per file

### Example:
```sql
COPY INTO orders
FROM @stage
FILE_FORMAT = (FORMAT_NAME = csv_fmt)
ON_ERROR = 'CONTINUE'
FORCE = TRUE;
```

### Advanced Options:
- FORCE → reload files
- VALIDATION_MODE → debug
- MATCH_BY_COLUMN_NAME → schema evolution

---

# 🔷 7. SNOWPIPE INTERNALS

Snowpipe enables continuous ingestion.

### Flow:
S3 → Event (SQS) → Snowflake → Micro-batch load

### Example:
```sql
CREATE PIPE pipe1
AUTO_INGEST = TRUE
AS COPY INTO orders FROM @stage;
```

### Key Points:
- Near real-time
- Micro-batch ingestion
- Serverless compute

---

# 🔷 8. PERFORMANCE ENGINEERING

### Critical Rules:
- File size: 100MB–1GB
- Too small → overhead
- Too large → poor parallelism
- Parallelism = number of files

### Formula:
Throughput ∝ Number of files processed in parallel

---

# 🔷 9. METADATA & DEBUGGING

Snowflake exposes metadata columns:

```sql
SELECT METADATA$FILENAME, METADATA$FILE_ROW_NUMBER
FROM @stage;
```

### Use Cases:
- Data lineage
- Debugging bad records
- Audit tracking

---

# 🔷 10. EDGE CASES & INTERVIEW TRAPS

❌ Stage stores data → Only internal stage does  
❌ COPY INTO is simple → It’s distributed system  
❌ File format optional → Causes silent corruption  
❌ Snowpipe is real-time → It’s near real-time  

---

# 🔥 FINAL MENTAL MODEL

Stage → Access Layer  
File Format → Parsing Engine  
COPY INTO → Distributed Loader  
Snowpipe → Streaming Engine  

---

# END
