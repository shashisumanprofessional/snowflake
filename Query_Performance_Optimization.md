Below is a **clear, practical, Snowflake-focused explanation** of **Query Performance Optimization**, written in a way that’s useful for **real projects and interviews**.

---

## 1️⃣ Query Profiling using **Query History**

**Query History** helps you analyze **past queries** to understand:

* Execution time
* Warehouses used
* Data scanned
* Errors or failures

### How to access

* **UI**: Snowsight → *Activity* → *Query History*
* **SQL**:

```sql
SELECT *
FROM SNOWFLAKE.ACCOUNT_USAGE.QUERY_HISTORY
ORDER BY START_TIME DESC;
```

### Why it matters

* Identify slow or expensive queries
* Detect repeated full table scans
* Track credit usage per query

---

## 2️⃣ Understanding **Query Profile**

**Query Profile** shows the **execution plan** of a query.

### Key components to focus on

* **Execution Tree** – operators executed (Scan, Join, Aggregate)
* **% Time Spent** – bottlenecks
* **Bytes Scanned** – indicates pruning effectiveness
* **Partitions scanned vs total**

### Red flags 🚩

* High scan time
* Large bytes scanned
* No partition pruning
* Broadcast joins on large tables

---

## 3️⃣ Avoiding **Full Table Scans**

Full table scans are expensive and slow.

### Common causes

* Missing `WHERE` clause
* Filters on non-selective columns
* Functions applied to filter columns

❌ Bad:

```sql
SELECT * FROM sales WHERE YEAR(order_date) = 2024;
```

✅ Good:

```sql
SELECT * FROM sales 
WHERE order_date BETWEEN '2024-01-01' AND '2024-12-31';
```

---

## 4️⃣ Micro-partition Pruning Techniques

Snowflake stores data in **micro-partitions** (~50–500 MB).

### How pruning works

* Snowflake uses **metadata** (min/max values)
* Partitions outside filter range are skipped

### Best practices

* Filter on **natural data ranges** (dates, IDs)
* Avoid wrapping columns in functions
* Use selective predicates early

---

## 5️⃣ Effective Use of **Clustering Keys**

Clustering improves pruning for **large tables** with frequent range queries.

### When to use clustering

* Table size > **1 TB**
* Queries often filter on same column(s)
* Poor partition pruning observed

### Example

```sql
ALTER TABLE sales 
CLUSTER BY (order_date, region);
```

### Monitor clustering

```sql
SELECT SYSTEM$CLUSTERING_INFORMATION('sales');
```

⚠️ Over-clustering increases cost.

---

## 6️⃣ Warehouse Sizing Strategies

### Scale **UP** vs Scale **OUT**

| Strategy  | Use When                |
| --------- | ----------------------- |
| Scale UP  | Single query is slow    |
| Scale OUT | Many concurrent queries |

### Best practices

* Start **SMALL**
* Use **auto-suspend** & **auto-resume**
* Separate warehouses:

  * ETL
  * Reporting
  * Ad-hoc analytics

---

## 7️⃣ Result Caching Behavior

Snowflake caches query results for **24 hours**.

### Cache is reused if:

* Same query text
* Same underlying data
* Same role
* No DML on tables

### Cache benefits

* Instant query results
* **Zero credits consumed**

### Disable cache (for testing)

```sql
ALTER SESSION SET USE_CACHED_RESULT = FALSE;
```

---

## 8️⃣ Performance Optimization Checklist ✅

✔ Use selective filters
✔ Check Query Profile regularly
✔ Avoid unnecessary `SELECT *`
✔ Leverage result cache
✔ Use clustering only when needed
✔ Right-size warehouses

---

## 9️⃣ Interview-ready summary

> “Snowflake query performance is optimized by analyzing Query History and Query Profile, avoiding full table scans through effective filtering and micro-partition pruning, using clustering keys selectively, choosing the right warehouse size, and leveraging result caching.”



