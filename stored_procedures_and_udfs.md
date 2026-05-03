# Snowflake Stored Procedures & UDFs — Definitive Production Guide

## 1. Foundations

Snowflake programmability includes:
- UDFs (expression-level computation)
- Stored Procedures (workflow orchestration)

Execution layers:
- SQL Engine (set-based)
- Snowpark Runtime (Python/Java)
- Procedure Engine

---

## 2. Execution Internals

### UDF Execution
- Inlined into query plan (SQL UDF)
- Row-level execution for non-SQL UDFs
- Vectorized where possible

### Stored Procedure Execution
- Runs as independent execution unit
- Submits queries via engine
- Maintains session state

---

## 3. UDF Deep Dive

### Types
- SQL UDF (best performance)
- JavaScript UDF
- Python UDF (Snowpark)
- External Functions

---

### SQL UDF (Compiled into Plan)
```sql
CREATE FUNCTION f(x INT)
RETURNS INT
AS $$ x + 1 $$;
```

**Key Insight**
- No runtime overhead
- Fully optimized

---

### Python UDF (Snowpark Execution)

Execution flow:
1. Query triggers UDF
2. Data shipped to Python runtime
3. Processed row/batch-wise
4. Returned to engine

```sql
CREATE FUNCTION py_double(x INT)
RETURNS INT
LANGUAGE PYTHON
HANDLER='handler'
AS $$
def handler(x):
    return x*2
$$;
```

---

### External Functions

Architecture:
Snowflake → API Gateway → Lambda → External Service → Response

Used for:
- ML inference
- External validation

---

### UDF Limitations

- No DML
- No transactions
- Limited state

---

### UDF Production Patterns

1. Standardization layer
2. Masking logic
3. Derived metrics

---

## 4. Stored Procedures Deep Dive

### Types
- JavaScript
- SQL (Snowflake Scripting)
- Python (Snowpark)

---

### Execution Flow

1. CALL procedure
2. Procedure submits queries
3. Handles results
4. Returns output

---

### Example (Snowflake Scripting)

```sql
CREATE PROCEDURE etl_proc()
RETURNS STRING
LANGUAGE SQL
AS
$$
BEGIN
    INSERT INTO tgt SELECT * FROM src;
    RETURN 'DONE';
END;
$$;
```

---

### Transactions

- Default: autocommit
- Explicit control allowed

```sql
BEGIN;
INSERT ...
COMMIT;
```

---

### Error Handling

```sql
BEGIN
    TRY
        INSERT INTO t VALUES(1);
    CATCH
        RETURN 'FAILED';
END;
```

---

### Dynamic SQL

```sql
EXECUTE IMMEDIATE 'SELECT COUNT(*) FROM ' || :table_name;
```

---

## 5. UDF vs Stored Procedure

| Capability | UDF | Stored Procedure |
|-----------|-----|------------------|
| Inline execution | Yes | No |
| DML | No | Yes |
| Control flow | No | Yes |
| Use case | Transformation | Orchestration |

---

## 6. Security Model

- EXECUTE AS CALLER
- EXECUTE AS OWNER

Secure UDF:
- Prevents data leakage

---

## 7. Performance Engineering

### UDF
- Prefer SQL over Python
- Avoid row-by-row heavy logic

### Stored Procedure
- Batch queries
- Minimize round trips

---

## 8. Observability

Key tables:
- QUERY_HISTORY
- TASK_HISTORY
- ACCESS_HISTORY

---

## 9. Debugging

Techniques:
- Logging tables
- RESULT_SCAN
- Query profiling

---

## 10. Architecture Patterns

### Pattern: ETL Pipeline

Task → Stored Procedure → COPY INTO → Transform → Load

---

### Pattern: Data API

External Function + API Gateway

---

## 11. Misconceptions

- UDF ≠ mini stored procedure
- Python ≠ always faster
- Stored procedures ≠ replacement for SQL

---

## 12. Best Practices

- Use SQL-first approach
- Keep UDFs deterministic
- Use SPs only for orchestration
- Implement logging & retry

---

## 13. Advanced Topics

### Snowpark Integration
- DataFrame APIs
- Lazy evaluation

### Parallelism
- Controlled by warehouse size
- UDFs may reduce parallel efficiency

---

## 14. Production Case Study

Scenario:
Daily ingestion pipeline

Flow:
1. Task triggers SP
2. SP runs COPY INTO
3. Validates data
4. Logs errors
5. Returns status

---

## 15. Final Notes

Mastery requires:
- Understanding execution model
- Choosing right construct
- Avoiding anti-patterns

