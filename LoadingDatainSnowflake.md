Below is a **clear, structured, interview-ready explanation** of **“Steps to Load Data into Snowflake”**, written at a **6-years experience level**. You can use this almost **verbatim** in interviews and then add examples from your project.

---

# 📥 Steps to Load Data into Snowflake (Detailed)

Loading data into Snowflake typically follows a **multi-step process**, whether the data comes from files, cloud storage, or streaming sources. The most common and recommended approach is **bulk loading using stages and `COPY INTO`**.

---

## **Step 1: Prepare the Target Table**

Before loading data, you must create a **destination table** that matches the structure of the incoming data.

### Key considerations:

* Correct **data types**
* Handling **nullable vs non-nullable columns**
* For semi-structured data, use `VARIANT`, `OBJECT`, or `ARRAY`
* Consider future **schema evolution**

📌 *In real projects*, we often load raw data into a **staging/raw table** first and then transform it into curated tables.

---

## **Step 2: Define a File Format**

Snowflake needs to understand how to read the incoming file.

### Common file formats:

* CSV
* JSON
* Parquet
* Avro
* ORC
* XML

### File format defines:

* Delimiter
* Header rows
* Compression
* Date/time formats
* Null handling
* Character encoding

📌 File formats can be:

* Defined once using `CREATE FILE FORMAT`
* Or inline within the `COPY INTO` command

---

## **Step 3: Stage the Data**

A **stage** is a location where Snowflake reads data files from.

### Types of stages:

1. **Internal Stage**

   * Managed by Snowflake
   * Requires `PUT` command to upload files
   * Used mostly for small/manual loads

2. **External Stage**

   * Cloud storage like **AWS S3, Azure Blob, GCS**
   * Preferred for production pipelines
   * Uses **storage integration** for secure access

3. **Table Stage**

   * Associated with a specific table
   * Simplifies load syntax

📌 *In enterprise environments*, external stages are most commonly used.

---

## **Step 4: Upload Data (If Using Internal Stage)**

This step applies **only to internal stages**.

* Use the `PUT` command to upload local files into Snowflake’s internal storage
* Supports compression and parallel uploads

📌 This step is **not required** for external stages.

---

## **Step 5: Load Data Using `COPY INTO`**

This is the **core data loading step**.

`COPY INTO`:

* Reads data from a stage
* Parses files using the file format
* Loads records into the target table

### Key `COPY INTO` options:

* `ON_ERROR` – controls behavior on errors
* `VALIDATION_MODE` – validate without loading
* `PURGE` – delete files after successful load
* `FORCE` – reload files even if loaded earlier
* `PATTERN` – load only specific files

📌 Snowflake automatically:

* Parallelizes loading
* Scales compute
* Tracks loaded files to avoid duplicates

---

## **Step 6: Validate the Load**

After loading, always verify data correctness.

### Validation methods:

* `VALIDATION_MODE = RETURN_ERRORS`
* `VALIDATION_MODE = RETURN_ALL_ERRORS`
* `COPY_HISTORY` / `LOAD_HISTORY`
* Querying rejected records
* Row count reconciliation

📌 This step is critical in **production pipelines**.

---

## **Step 7: Handle Errors and Rejected Records**

Snowflake allows granular error handling.

### Error strategies:

* Skip bad rows and load good data
* Log rejected rows for analysis
* Fix data issues and reload

Common errors include:

* Data type mismatches
* Extra or missing columns
* Invalid date formats

---

## **Step 8: Transform and Merge Data (Optional but Common)**

In real-world projects:

* Data is first loaded into **raw/staging tables**
* Then transformed into final tables using:

  * `INSERT`
  * `MERGE`
  * `CTAS`

📌 This supports:

* Deduplication
* Incremental loads
* Business logic enforcement

---

## **Step 9: Automate the Load (Production Setup)**

For ongoing ingestion, Snowflake provides:

### Automation options:

* **Snowpipe** – continuous, near real-time ingestion
* **Tasks** – scheduled batch loads
* **Streams + Tasks** – incremental processing
* External orchestrators (Airflow, ADF, etc.)

📌 Snowpipe is ideal for **event-driven ingestion**, while Tasks suit **batch pipelines**.

---

## **Step 10: Monitor and Optimize**

Finally, monitor and optimize loads.

### Monitoring tools:

* Query history
* Load history views
* Warehouse credit usage
* Snowflake UI dashboards

### Optimization strategies:

* Right-size warehouses
* Batch files efficiently
* Use clustering when needed
* Avoid unnecessary reloads

---

# 🔁 Summary Flow (Interview-Friendly)

> **Create table → Define file format → Create stage → Upload files (if internal) → COPY INTO → Validate → Handle errors → Transform → Automate → Monitor**

------------------------------------------------------------------------------------------------------------------------------------------------------






📦 Types of Stages in Snowflake and When to Use Each

A stage in Snowflake is a storage location used to load data into tables or unload data from tables. Stages act as an abstraction layer between Snowflake and data files.

Snowflake supports three main types of stages:

Internal Stages

External Stages

Table Stages

1️⃣ Internal Stages
🔹 What is it?

Internal stages are Snowflake-managed storage locations where data files are stored inside Snowflake.

Snowflake takes care of:

Storage

Security

Encryption

Access control

🔹 Types of Internal Stages
Type	Description
User Stage (@~)	Each user gets a personal stage
Named Internal Stage	Explicitly created stage
Table Stage (@%table_name)	Automatically created for each table
🔹 How data is loaded

Files are uploaded using the PUT command

Then loaded using COPY INTO

🔹 When to use Internal Stages

✅ Small to medium data loads
✅ Ad-hoc or manual data uploads
✅ Proof of concept (POC) or testing
✅ When cloud storage is not available

📌 Not recommended for production pipelines due to manual file uploads and limited scalability.

🔹 Example scenario

A data analyst needs to load a one-time CSV file received via email into Snowflake.

🔹 Pros

✔ Simple to use
✔ No cloud setup required
✔ Secure and encrypted

🔹 Cons

❌ Requires manual file upload
❌ Not ideal for automation
❌ Limited for large-scale ingestion

2️⃣ External Stages ⭐ (Most Common in Production)
🔹 What is it?

External stages reference cloud object storage outside Snowflake:

AWS S3

Azure Blob Storage

Google Cloud Storage (GCS)

Snowflake reads data directly from these locations.

🔹 Security & Access

Uses Storage Integration (recommended)

Or legacy credentials (not recommended)

Storage integration provides:

IAM-based access

Key rotation

Least privilege security

🔹 How data is loaded

Files land in cloud storage

Snowflake reads them via COPY INTO or Snowpipe

🔹 When to use External Stages

✅ Production data pipelines
✅ Large-volume data ingestion
✅ Automated and scheduled loads
✅ Integration with data lakes
✅ Snowpipe (auto-ingest)

📌 This is the industry standard approach.

🔹 Example scenario

Daily sales data is dropped into S3 by upstream systems and loaded into Snowflake every hour using Snowpipe.

🔹 Pros

✔ Highly scalable
✔ Fully automated
✔ Integrates with cloud ecosystems
✔ Supports continuous ingestion

🔹 Cons

❌ Requires cloud configuration
❌ Slightly more complex to set up

3️⃣ Table Stages
🔹 What is it?

A table stage is an internal stage automatically created for every table.

Syntax:

@%table_name


Files staged here are tightly coupled to that table.

🔹 How data is loaded

Files uploaded using PUT

Loaded using COPY INTO table_name

🔹 When to use Table Stages

✅ Quick loads specific to a table
✅ Temporary or intermediate data
✅ Simple staging use cases

📌 Not commonly used in enterprise pipelines.

🔹 Example scenario

A developer loads test data into a specific table during development.

🔹 Pros

✔ No need to create a stage explicitly
✔ Easy syntax

🔹 Cons

❌ Not reusable across tables
❌ Same limitations as internal stages



---------------------------------------------------------------------------------------------------------------------------------------------------------




Below is a **clear, detailed, interview-ready explanation** of the **difference between `PUT` and `COPY INTO`**, tailored for a **6-year experience Snowflake interview**, including **when/why to use each** and **internal vs external stage behavior**.

---

# 🔄 Difference Between `PUT` and `COPY INTO` in Snowflake

`PUT` and `COPY INTO` are **two different steps** in Snowflake’s data loading process, and they serve **very different purposes**.

---

## 🔹 High-Level Difference (One-Line Answer)

> **`PUT` uploads files into a Snowflake internal stage, while `COPY INTO` loads data from a stage into a Snowflake table.**

---

## 1️⃣ `PUT` Command

### 🔹 What is `PUT`?

`PUT` is a **client-side command** used to **upload local files** from your machine into a **Snowflake internal stage**.

It **does NOT load data into tables**.

---

### 🔹 When do you use `PUT`?

You use `PUT` **only when working with internal stages**.

Typical use cases:

* Ad-hoc or one-time data loads
* Testing or development
* Small datasets
* When cloud storage (S3/Azure/GCS) is not available

---

### 🔹 Where does `PUT` work?

✅ Internal stages
❌ External stages (not allowed)

---

### 🔹 Example

```sql
PUT file://local/path/data.csv @my_internal_stage;
```

This uploads the file to Snowflake-managed storage.

---

### 🔹 Key Characteristics of `PUT`

| Feature                    | Details |
| -------------------------- | ------- |
| Uploads files              | ✅ Yes   |
| Loads data into table      | ❌ No    |
| Works with internal stages | ✅ Yes   |
| Works with external stages | ❌ No    |
| Used in production         | Rare    |

---

## 2️⃣ `COPY INTO` Command

### 🔹 What is `COPY INTO`?

`COPY INTO` is a **server-side command** that **reads data from a stage** and **loads it into a Snowflake table**.

This is the **actual data ingestion step**.

---

### 🔹 When do you use `COPY INTO`?

You use `COPY INTO` **whenever you want to load data into a Snowflake table**, regardless of stage type.

Typical use cases:

* Batch data ingestion
* Scheduled loads
* Production pipelines
* Loading structured or semi-structured data

---

### 🔹 Where does `COPY INTO` work?

✅ Internal stages
✅ External stages

---

### 🔹 Example

```sql
COPY INTO sales
FROM @my_stage
FILE_FORMAT = (TYPE = CSV);
```

---

### 🔹 Key Characteristics of `COPY INTO`

| Feature                  | Details |
| ------------------------ | ------- |
| Uploads files            | ❌ No    |
| Loads data into table    | ✅ Yes   |
| Supports internal stages | ✅ Yes   |
| Supports external stages | ✅ Yes   |
| Production ready         | ✅ Yes   |

---

## 🔁 Internal Stage vs External Stage Usage

### 📌 Internal Stage Flow

```
Local File
   ↓ PUT
Internal Stage
   ↓ COPY INTO
Snowflake Table
```

* **Both `PUT` and `COPY INTO` are required**

---

### 📌 External Stage Flow

```
Cloud Storage (S3/Azure/GCS)
   ↓ COPY INTO (or Snowpipe)
Snowflake Table
```

* **`PUT` is NOT used**
* Files are already in cloud storage

---

## 🆚 Comparison Table

| Aspect                   | PUT          | COPY INTO             |
| ------------------------ | ------------ | --------------------- |
| Purpose                  | Upload files | Load data into tables |
| Runs on                  | Client side  | Snowflake server      |
| Used with internal stage | ✅ Yes        | ✅ Yes                 |
| Used with external stage | ❌ No         | ✅ Yes                 |
| Loads data into table    | ❌ No         | ✅ Yes                 |
| Automation friendly      | ❌            | ✅                     |

---

## 🎯 Interview-Ready Summary Answer

> `PUT` is used to upload local files into a Snowflake internal stage and is required only for internal stages. It does not load data into tables.
> `COPY INTO` is used to load data from a stage into Snowflake tables and works with both internal and external stages.
> In production systems, `COPY INTO` is heavily used, while `PUT` is typically avoided in favor of cloud storage and external stages.

--------------------------------------------------------------------------------------------------------------------------------------------








Below is a **step-by-step, interview-ready explanation** of **how to define external stages in Snowflake**, written for a **6-year experience level**, with **AWS / Azure / GCP examples**, security best practices, and common interview follow-ups.

---

# 🌐 How to Define External Stages in Snowflake

An **external stage** in Snowflake points to **cloud object storage** (AWS S3, Azure Blob, or Google Cloud Storage) where data files already exist. Snowflake reads data **directly from these locations** without copying it into Snowflake storage.

---

## 🔹 High-Level Steps to Define an External Stage

1. **Create a Storage Integration** (recommended & secure)
2. **Grant required permissions**
3. **Create a File Format** (optional but best practice)
4. **Create the External Stage**
5. **Validate access**
6. **Use COPY INTO or Snowpipe**

---

## 1️⃣ Create a Storage Integration (Best Practice)

A **storage integration** allows Snowflake to securely access cloud storage using IAM roles or service principals.

### Why this is important (interview point):

* No hard-coded credentials
* Supports key rotation
* Follows least-privilege security

---

### ✅ AWS S3 Example

```sql
CREATE STORAGE INTEGRATION s3_int
  TYPE = EXTERNAL_STAGE
  STORAGE_PROVIDER = S3
  ENABLED = TRUE
  STORAGE_AWS_ROLE_ARN = 'arn:aws:iam::123456789012:role/snowflake_role'
  STORAGE_ALLOWED_LOCATIONS = ('s3://my-bucket/data/');
```

After creation:

```sql
DESC STORAGE INTEGRATION s3_int;
```

Use the **AWS IAM user ARN & external ID** shown here to configure trust in AWS.

---

### ✅ Azure Blob Example

```sql
CREATE STORAGE INTEGRATION azure_int
  TYPE = EXTERNAL_STAGE
  STORAGE_PROVIDER = AZURE
  ENABLED = TRUE
  AZURE_TENANT_ID = '<tenant_id>'
  STORAGE_ALLOWED_LOCATIONS = ('azure://mycontainer.blob.core.windows.net/data/');
```

---

### ✅ GCP Example

```sql
CREATE STORAGE INTEGRATION gcs_int
  TYPE = EXTERNAL_STAGE
  STORAGE_PROVIDER = GCS
  ENABLED = TRUE
  STORAGE_ALLOWED_LOCATIONS = ('gcs://my-bucket/data/');
```

---

## 2️⃣ Create a File Format (Optional but Recommended)

A file format tells Snowflake **how to read the files**.

```sql
CREATE FILE FORMAT csv_ff
  TYPE = CSV
  FIELD_DELIMITER = ','
  SKIP_HEADER = 1
  NULL_IF = ('NULL', 'null');
```

📌 This makes stage definitions cleaner and reusable.

---

## 3️⃣ Create the External Stage

Now link:

* Cloud location
* Storage integration
* File format

---

### ✅ AWS S3 External Stage

```sql
CREATE STAGE s3_ext_stage
  URL = 's3://my-bucket/data/'
  STORAGE_INTEGRATION = s3_int
  FILE_FORMAT = csv_ff;
```

---

### ✅ Azure Blob External Stage

```sql
CREATE STAGE azure_ext_stage
  URL = 'azure://mycontainer.blob.core.windows.net/data/'
  STORAGE_INTEGRATION = azure_int
  FILE_FORMAT = csv_ff;
```

---

### ✅ GCP External Stage

```sql
CREATE STAGE gcs_ext_stage
  URL = 'gcs://my-bucket/data/'
  STORAGE_INTEGRATION = gcs_int
  FILE_FORMAT = csv_ff;
```

---

## 4️⃣ Validate the External Stage

Always test access before loading data.

```sql
LIST @s3_ext_stage;
```

If files are listed successfully, the stage is correctly configured.

---

## 5️⃣ Load Data Using the External Stage

```sql
COPY INTO sales
FROM @s3_ext_stage
FILE_FORMAT = (TYPE = CSV)
ON_ERROR = 'CONTINUE';
```

📌 Snowflake automatically:

* Reads files in parallel
* Tracks loaded files
* Prevents duplicate loads

---

## 6️⃣ (Optional) Use External Stage with Snowpipe

External stages are required for **Snowpipe auto-ingest**.

Typical flow:

```
Cloud Storage → Event Notification → Snowpipe → Snowflake Table
```

---

## 🔐 Alternative (Not Recommended): Using Credentials Directly

You *can* define an external stage using credentials, but **this is discouraged**.

```sql
CREATE STAGE s3_ext_stage
  URL = 's3://my-bucket/data/'
  CREDENTIALS = (AWS_KEY_ID='xxx' AWS_SECRET_KEY='yyy');
```

🚫 Interview note: **Always say storage integration is preferred.**

---

## 🆚 Interview Comparison: Storage Integration vs Credentials

| Feature            | Storage Integration | Credentials |
| ------------------ | ------------------- | ----------- |
| Security           | ⭐⭐⭐⭐⭐               | ⭐⭐          |
| Key rotation       | Automatic           | Manual      |
| Best practice      | ✅ Yes               | ❌ No        |
| Used in production | ✅                   | Rare        |

---

## 🎯 Interview-Ready Summary Answer

> To define an external stage in Snowflake, we first create a storage integration to securely connect Snowflake with cloud storage like S3, Azure Blob, or GCS. Then we create a file format if needed and define the external stage using the storage integration and cloud URL. External stages are commonly used in production pipelines and are required for Snowpipe-based ingestion.

------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------





Here’s a **comprehensive, interview-ready explanation** of **how Snowflake handles semi-structured data like JSON** and how to **load it into a VARIANT column**, tailored for a **6-year experienced data engineer**.

---

# 🌐 Handling Semi-Structured Data in Snowflake

Snowflake provides **native support for semi-structured data**, including:

* **JSON**
* **AVRO**
* **ORC**
* **Parquet**
* **XML**

---

## 1️⃣ Key Features

1. **VARIANT Data Type**

   * Snowflake uses the `VARIANT` column type to store **any semi-structured data**.
   * Supports nested objects, arrays, and mixed types.

2. **Native Parsing**

   * Snowflake can **ingest JSON directly** without needing schema definition.
   * You can query nested fields using:

     * Dot notation: `data.field1`
     * Bracket notation: `data['field1']`
   * Flatten arrays or nested structures using `FLATTEN` function.

3. **Schema-on-Read**

   * Unlike traditional relational tables, you don’t need to define every key in advance.
   * Flexible for evolving JSON structures.

---

## 2️⃣ Loading JSON into a VARIANT Column

Snowflake provides **two main approaches**:

### **A. Loading from staged JSON files (bulk load)**

1. **Create table with VARIANT column**

```sql
CREATE TABLE raw_json_data (
    id INT AUTOINCREMENT,
    data VARIANT
);
```

2. **Create file format**

```sql
CREATE FILE FORMAT json_ff
    TYPE = JSON;
```

3. **Create stage (internal or external)**

```sql
CREATE STAGE json_stage FILE_FORMAT = json_ff;
```

4. **Upload JSON files to stage**

```sql
PUT file://local/path/file1.json @json_stage;
```

5. **Load JSON into VARIANT column**

```sql
COPY INTO raw_json_data (data)
FROM @json_stage
FILE_FORMAT = (TYPE = JSON)
ON_ERROR = 'CONTINUE';
```

✅ After this, each JSON object in the file is stored as **one row in the `data` VARIANT column**.

---

### **B. Loading JSON directly from a string or query (inline)**

```sql
INSERT INTO raw_json_data (data)
VALUES (PARSE_JSON('{"name":"John","age":30,"address":{"city":"NY"}}'));
```

* `PARSE_JSON()` converts a JSON string into **VARIANT type**
* Useful for small inserts or API-based ingestion

---

## 3️⃣ Querying JSON in VARIANT Column

1. **Access top-level keys**

```sql
SELECT data:name AS name, data:age AS age
FROM raw_json_data;
```

2. **Access nested objects**

```sql
SELECT data:address.city AS city
FROM raw_json_data;
```

3. **Flatten arrays**

```sql
SELECT f.value AS item
FROM raw_json_data,
LATERAL FLATTEN(input => data:items) f;
```

---

## 4️⃣ Loading JSON with COPY INTO from S3 (External Stage)

```sql
CREATE STAGE s3_json_stage
  URL = 's3://mybucket/jsonfiles/'
  STORAGE_INTEGRATION = s3_int
  FILE_FORMAT = (TYPE = JSON);

COPY INTO raw_json_data (data)
FROM @s3_json_stage;
```

* Each JSON file object becomes a row in the table
* Works with nested arrays and objects

---

## 5️⃣ Handling Semi-Structured Data Best Practices

* **Use VARIANT columns** for raw data ingestion
* Store **raw JSON first**, then transform to structured tables if needed
* Use `FLATTEN` and `LATERAL` joins to normalize data
* For large-scale pipelines, combine **external stage + COPY INTO + VARIANT** for high performance
* Use **Streams + Tasks** for incremental JSON ingestion

---

## 🆚 Interview-Ready Summary

> Snowflake handles semi-structured data using the `VARIANT` data type, which can store JSON, Avro, Parquet, XML, etc. You can load JSON files directly into a VARIANT column using `COPY INTO` from a stage or insert JSON strings using `PARSE_JSON()`. Nested objects can be queried with dot/bracket notation, and arrays can be flattened with the `FLATTEN` function. This allows schema-on-read ingestion, flexible pipelines, and easy transformation to structured tables later.

------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------











Below is a **clear, step-by-step example** showing **how to use `FLATTEN` with `LATERAL`** in Snowflake, exactly the way interviewers expect you to explain it — with **JSON data, queries, and real-world logic**.

---

# 🔍 Example: Using FLATTEN and LATERAL in Snowflake

## 1️⃣ Sample JSON Data (Stored in VARIANT)

Assume we have JSON like this coming from an API:

```json
{
  "order_id": 101,
  "customer": {
    "name": "Alice",
    "country": "US"
  },
  "items": [
    { "item_id": "P1", "product": "Laptop", "price": 1200 },
    { "item_id": "P2", "product": "Mouse",  "price": 25 }
  ]
}
```

---

## 2️⃣ Table with VARIANT Column

```sql
CREATE TABLE orders_raw (
    data VARIANT
);
```

Load the JSON into this table (via `COPY INTO` or `INSERT`).

---

## 3️⃣ Why FLATTEN Is Needed

* JSON arrays (like `items`) cannot be queried as normal rows
* `FLATTEN` **explodes array elements into multiple rows**
* `LATERAL` allows `FLATTEN` to access each row’s JSON data

---

## 4️⃣ Basic FLATTEN Example (Array → Rows)

```sql
SELECT
    f.value:item_id::STRING AS item_id,
    f.value:product::STRING AS product,
    f.value:price::NUMBER  AS price
FROM orders_raw o,
     LATERAL FLATTEN(input => o.data:items) f;
```

### 🔎 What happens?

| order | item_id | product | price |
| ----- | ------- | ------- | ----- |
| 101   | P1      | Laptop  | 1200  |
| 101   | P2      | Mouse   | 25    |

Each element in the `items` array becomes **one row**.

---

## 5️⃣ Including Parent Fields (Very Common Interview Pattern)

```sql
SELECT
    o.data:order_id::INT             AS order_id,
    o.data:customer.name::STRING    AS customer_name,
    o.data:customer.country::STRING AS country,
    f.value:item_id::STRING          AS item_id,
    f.value:product::STRING          AS product,
    f.value:price::NUMBER            AS price
FROM orders_raw o,
     LATERAL FLATTEN(input => o.data:items) f;
```

This is how you **denormalize JSON into relational format**.

---

## 6️⃣ Using Explicit LATERAL JOIN Syntax

Equivalent and more readable:

```sql
SELECT
    o.data:order_id::INT AS order_id,
    f.value:product::STRING AS product
FROM orders_raw o
JOIN LATERAL FLATTEN(input => o.data:items) f;
```

---

## 7️⃣ FLATTEN with Nested Arrays (Advanced Example)

JSON with nested arrays:

```json
{
  "user_id": 1,
  "sessions": [
    {
      "session_id": "S1",
      "events": ["login", "search"]
    },
    {
      "session_id": "S2",
      "events": ["logout"]
    }
  ]
}
```

### Query:

```sql
SELECT
    o.data:user_id::INT AS user_id,
    s.value:session_id::STRING AS session_id,
    e.value::STRING AS event
FROM sessions_raw o,
     LATERAL FLATTEN(input => o.data:sessions) s,
     LATERAL FLATTEN(input => s.value:events) e;
```

---

## 8️⃣ Important FLATTEN Columns (Interview Favorite)

`FLATTEN` returns multiple metadata columns:

| Column  | Meaning              |
| ------- | -------------------- |
| `VALUE` | Actual array element |
| `INDEX` | Position in array    |
| `PATH`  | JSON path            |
| `THIS`  | Full original object |

Example:

```sql
SELECT
    f.index,
    f.value
FROM orders_raw o,
     LATERAL FLATTEN(input => o.data:items) f;
```

---

## 9️⃣ When to Use FLATTEN + LATERAL (Say This in Interview)

> I use `FLATTEN` with `LATERAL` when dealing with JSON arrays in a VARIANT column.
> `FLATTEN` explodes array elements into rows, and `LATERAL` allows the function to reference columns from the outer query row.

---

## 10️⃣ Real-World Use Case (Senior-Level Answer)

* Load raw JSON into a VARIANT column
* Use `FLATTEN` + `LATERAL` to normalize arrays
* Insert into structured fact/dimension tables
* Use Streams + Tasks for incremental processing

---

## 🎯 Interview-Ready One-Line Summary

> `FLATTEN` is used to convert JSON arrays into multiple rows, and `LATERAL` allows the flatten operation to access each row’s VARIANT data in Snowflake.

------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------







Below is a **structured, interview-ready explanation** of **how to detect corrupt data and how to validate whether a Snowflake load was successful**, written at a **senior (≈6 years) experience level** with **clear steps, SQL examples, and real-world practices**.

---

# ✅ How to Validate Data Quality and Load Success in Snowflake

Validation happens at **three levels**:

1. **Before loading** (file-level validation)
2. **During loading** (COPY validation & error handling)
3. **After loading** (data reconciliation & monitoring)

---

## 1️⃣ Pre-Load Validation (Detect Corrupt Data Before Load)

### 🔹 Step 1: Validate File Structure (Dry Run)

Use `VALIDATION_MODE` to check files **without loading data**.

```sql
COPY INTO sales
FROM @s3_stage
FILE_FORMAT = (TYPE = CSV)
VALIDATION_MODE = 'RETURN_ERRORS';
```

✔ Detects:

* Wrong delimiter
* Missing columns
* Invalid data types
* Malformed rows

---

### 🔹 Step 2: Check File Presence & Size

```sql
LIST @s3_stage;
```

✔ Ensures:

* Files exist
* File sizes are reasonable
* Correct folder path

---

## 2️⃣ During Load Validation (COPY INTO Controls)

### 🔹 Step 3: Use `ON_ERROR` Strategically

| Option            | Behavior                      |
| ----------------- | ----------------------------- |
| `ABORT_STATEMENT` | Stop load on first error      |
| `CONTINUE`        | Skip bad rows, load good ones |
| `SKIP_FILE`       | Skip entire bad file          |
| `SKIP_FILE_n`     | Skip file after n errors      |

```sql
COPY INTO sales
FROM @s3_stage
ON_ERROR = 'CONTINUE';
```

✔ Allows partial loads with error tolerance

---

### 🔹 Step 4: Capture Rejected Records

```sql
COPY INTO sales
FROM @s3_stage
VALIDATION_MODE = 'RETURN_ALL_ERRORS';
```

✔ Returns detailed error messages per row

---

## 3️⃣ Post-Load Validation (Confirm Load Success)

### 🔹 Step 5: Check COPY History (Most Important)

```sql
SELECT *
FROM TABLE(
  INFORMATION_SCHEMA.COPY_HISTORY(
    TABLE_NAME => 'SALES',
    START_TIME => DATEADD('hour', -2, CURRENT_TIMESTAMP())
  )
);
```

✔ Shows:

* Files loaded
* Rows loaded
* Rows rejected
* Error counts
* Load status

📌 **Interview tip:** This is your primary proof of load success.

---

### 🔹 Step 6: Reconcile Row Counts

Compare:

* Source file rows
* Target table rows

```sql
SELECT COUNT(*) FROM sales;
```

✔ Confirms completeness

---

### 🔹 Step 7: Check for NULLs & Invalid Values

```sql
SELECT COUNT(*)
FROM sales
WHERE amount IS NULL
   OR amount < 0;
```

✔ Detects corrupt or unexpected data

---

## 4️⃣ Validating Semi-Structured (JSON) Data

### 🔹 Step 8: Validate JSON Structure

```sql
SELECT
  data,
  IS_OBJECT(data) AS is_object
FROM raw_json
WHERE NOT IS_OBJECT(data);
```

✔ Detects malformed JSON

---

### 🔹 Step 9: Validate Required Keys

```sql
SELECT *
FROM raw_json
WHERE data:order_id IS NULL;
```

---

## 5️⃣ Using Load Metadata Tables

### 🔹 Step 10: Query Load History Views

```sql
SELECT *
FROM SNOWFLAKE.ACCOUNT_USAGE.LOAD_HISTORY
WHERE STATUS != 'LOADED';
```

✔ Detects failed or partial loads

---

## 6️⃣ Production-Grade Validation Workflow (Real-World)

### Typical validation pipeline:

```
Stage → VALIDATION_MODE → COPY INTO
      → COPY_HISTORY
      → Row Count Check
      → Business Rule Checks
      → Alert / Retry
```

---

## 7️⃣ Handling Corrupt Data (What to Do)

If data is corrupt:

1. Identify bad rows using `VALIDATION_MODE`
2. Log rejected records to an error table
3. Fix upstream data or apply transformation logic
4. Reload only failed files (use `FORCE = TRUE` if needed)

---

## 🎯 Interview-Ready Summary Answer

> To validate whether data is corrupt or a load was successful in Snowflake, I first use `VALIDATION_MODE` in `COPY INTO` to check file structure before loading. During the load, I control error handling using `ON_ERROR` and capture rejected records. After the load, I verify success using `COPY_HISTORY` or `LOAD_HISTORY`, reconcile row counts, and run business rule checks such as NULL or range validations. This multi-layer validation approach ensures both technical and business data quality.

------------------------------------------------------------------------------------------------------------------------------------------------------------





Below is a **clear, interview-ready explanation** of **all `VALIDATION_MODE` options in Snowflake**, with **what each one does, when to use it, and SQL examples**. This is exactly the level interviewers expect for a **5–7 years experience** Snowflake candidate.

---

# ✅ VALIDATION_MODE in Snowflake (All Types + Examples)

`VALIDATION_MODE` is used with **`COPY INTO`** to **validate files without (or with limited) loading**, helping detect **corrupt data, format issues, and row-level errors**.

---

## 🔹 Why VALIDATION_MODE Is Important

* Prevents bad data from entering tables
* Identifies corrupt rows and files early
* Used heavily in **production pipelines**
* Common **interview topic**

---

## 📌 VALIDATION_MODE Options

Snowflake supports **three main validation modes**:

1. **`RETURN_ERRORS`**
2. **`RETURN_ALL_ERRORS`**
3. **`RETURN_n_ROWS`**

---

## 1️⃣ VALIDATION_MODE = `RETURN_ERRORS`

### 🔹 What it does

* Validates files
* Returns **first error per file**
* **Does not load any data**

### 🔹 When to use

* Quick validation before loading
* Identifying file-level format issues
* Faster than full validation

---

### 🔹 Example

```sql
COPY INTO sales
FROM @s3_stage
FILE_FORMAT = (TYPE = CSV)
VALIDATION_MODE = 'RETURN_ERRORS';
```

### 🔹 Sample Output

| FILE         | LINE | ERROR                              |
| ------------ | ---- | ---------------------------------- |
| sales_01.csv | 15   | Numeric value 'abc' not recognized |

📌 Stops scanning file after first error

---

## 2️⃣ VALIDATION_MODE = `RETURN_ALL_ERRORS` ⭐ (Most Detailed)

### 🔹 What it does

* Validates entire file
* Returns **all row-level errors**
* **Does not load any data**

### 🔹 When to use

* Deep data quality checks
* Debugging corrupt files
* Production issue analysis

---

### 🔹 Example

```sql
COPY INTO sales
FROM @s3_stage
FILE_FORMAT = (TYPE = CSV)
VALIDATION_MODE = 'RETURN_ALL_ERRORS';
```

### 🔹 Sample Output

| FILE         | LINE | ERROR              |
| ------------ | ---- | ------------------ |
| sales_01.csv | 15   | Invalid number     |
| sales_01.csv | 22   | Date parsing error |
| sales_02.csv | 9    | Missing column     |

📌 Slower but extremely useful

---

## 3️⃣ VALIDATION_MODE = `RETURN_n_ROWS`

### 🔹 What it does

* Validates **first N rows**
* Returns **errors only from those rows**
* **Does not load any data**

### 🔹 When to use

* Large files (GBs/TBs)
* Quick sampling
* Performance-sensitive checks

---

### 🔹 Example

```sql
COPY INTO sales
FROM @s3_stage
FILE_FORMAT = (TYPE = CSV)
VALIDATION_MODE = 'RETURN_100_ROWS';
```

📌 Only checks first 100 rows

---

## 🆚 Validation Modes Comparison

| Mode              | Loads Data | Errors Returned        | Use Case       |
| ----------------- | ---------- | ---------------------- | -------------- |
| RETURN_ERRORS     | ❌ No       | First error per file   | Quick check    |
| RETURN_ALL_ERRORS | ❌ No       | All errors             | Deep debugging |
| RETURN_n_ROWS     | ❌ No       | Errors in first N rows | Large files    |

---

## ⚠️ Important Notes (Interview Traps)

1. **VALIDATION_MODE never loads data**
2. Works only with **COPY INTO**
3. Not supported in **Snowpipe**
4. Cannot be combined with `ON_ERROR`
5. Used mainly for **pre-load validation**

---

## 🎯 Interview-Ready Summary Answer

> Snowflake provides three validation modes for `COPY INTO`.
> `RETURN_ERRORS` returns the first error per file and is used for quick validation.
> `RETURN_ALL_ERRORS` returns all row-level errors and is used for deep debugging.
> `RETURN_n_ROWS` validates only the first N rows, which is useful for very large files.
> None of these modes load data; they are strictly used to validate file integrity before ingestion.

-------------------------------------------------------------------------------------------------------------------------------------------------------------------





Below is a **clear, senior-level explanation** of **what happens when Snowflake skips files during a load**, why it happens, and how to control or override this behavior. This is a **very common interview question** around Snowflake data loading.

---

# ⏭️ What Happens When Snowflake Skips Files During a Load?

When you run a `COPY INTO` command, **Snowflake automatically tracks which files have already been loaded**. If a file has been successfully loaded before, **Snowflake skips it in subsequent loads by default**.

This behavior is **intentional** and is designed to **prevent duplicate data ingestion**.

---

## 1️⃣ Why Snowflake Skips Files

Snowflake maintains **load metadata** that records:

* File name
* File path
* Target table
* Load status (loaded / partially loaded / failed)
* Timestamp

If Snowflake detects:

* The **same file name**
* From the **same stage/location**
* Loaded into the **same target table**

➡️ **It skips the file automatically**.

---

## 2️⃣ What “Skipped” Means (Important)

When a file is skipped:

* ❌ No rows are read
* ❌ No data is reloaded
* ❌ No additional credits are consumed
* ✅ Load command completes successfully

The file is treated as **already ingested**.

---

## 3️⃣ Where Snowflake Tracks This Information

You can see skipped files in:

```sql
SELECT *
FROM TABLE(
  INFORMATION_SCHEMA.COPY_HISTORY(
    TABLE_NAME => 'SALES',
    START_TIME => DATEADD('hour', -24, CURRENT_TIMESTAMP())
  )
);
```

Look for:

* `STATUS = 'SKIPPED'`
* `ERROR_COUNT = 0`

---

## 4️⃣ Common Scenarios Where Files Get Skipped

### 🔹 Scenario 1: Re-running COPY INTO

```sql
COPY INTO sales
FROM @s3_stage;
```

➡️ Previously loaded files are skipped.

---

### 🔹 Scenario 2: Pipeline Retry After Partial Failure

* Successfully loaded files → skipped
* Failed files → retried

This makes pipelines **idempotent** by default.

---

### 🔹 Scenario 3: Snowpipe Auto-Ingest

Snowpipe also tracks files and **won’t ingest the same file twice**.

---

## 5️⃣ How Long Snowflake Remembers Loaded Files

* Snowflake retains load metadata for **64 days**
* After that period, files may be eligible for reload

📌 This is an important interview detail.

---

## 6️⃣ How to Force Reload a File (Override Skip Behavior)

### 🔹 Option 1: Use `FORCE = TRUE`

```sql
COPY INTO sales
FROM @s3_stage
FORCE = TRUE;
```

➡️ Reloads **all matching files**, even if already loaded.

⚠️ Risk: Duplicate data if not handled carefully.

---

### 🔹 Option 2: Rename or Move the File

* Change file name or path in cloud storage
* Snowflake treats it as a new file

---

### 🔹 Option 3: Load into a Different Table

File tracking is **per table**.

---

## 7️⃣ Best Practices to Avoid Issues

✔ Use **staging tables**
✔ Deduplicate data using **MERGE** or business keys
✔ Avoid `FORCE = TRUE` in production unless necessary
✔ Monitor skipped files using `COPY_HISTORY`
✔ Use Snowpipe for event-driven, exactly-once ingestion

---

## 8️⃣ Interview-Ready Summary Answer

> Snowflake automatically tracks loaded files and skips any file that has already been successfully ingested into the same table from the same stage. This prevents duplicate data loads and makes ingestion idempotent. Skipped files consume no compute and appear as ‘SKIPPED’ in the copy history. If needed, files can be reloaded using `FORCE = TRUE`, by renaming the file, or by loading into a different table, but this must be handled carefully to avoid duplicates.

---

## 💡 Senior-Level Insight (Very Important)

> This skip behavior is a key reason Snowflake ingestion pipelines are resilient to retries and failures. In production, we rely on this feature to safely rerun loads without worrying about duplicating data.

----------------------------------------------------------------------------------------------------------------------------------------------------------------








Below is a **clear, structured, interview-ready explanation** of **types of tables in Snowflake**, along with **Time Travel and Fail-safe**, and **how they are related to tables**. This is exactly the depth expected for a **5–7 years experience** Snowflake interview.

---

# 🗂️ Types of Tables in Snowflake

Snowflake supports **four main types of tables**, each with different **data retention, recovery, and cost characteristics**.

---

## 1️⃣ Permanent Tables (Default)

### 🔹 What is it?

* Standard Snowflake table
* Data is stored **permanently**
* Fully supports **Time Travel** and **Fail-safe**

### 🔹 Key Features

* Data persists until explicitly dropped
* Supports:

  * `Time Travel`
  * `Fail-safe`
  * Cloning
  * Data sharing

### 🔹 Retention

* **Time Travel**: up to **90 days** (Enterprise+)
* **Fail-safe**: fixed **7 days**

### 🔹 Use Case

✔ Production fact and dimension tables
✔ Historical and analytical data

---

## 2️⃣ Transient Tables

### 🔹 What is it?

* Non-permanent table
* Designed for **temporary or intermediate data**
* Supports **Time Travel only**
* ❌ No Fail-safe

### 🔹 Key Features

* Lower storage cost
* Reduced data recovery window
* Ideal for staging and intermediate tables

### 🔹 Retention

* Time Travel: **0–1 day (configurable)**
* Fail-safe: ❌ Not available

### 🔹 Use Case

✔ Staging tables
✔ ETL intermediate results
✔ Temporary business data

---

## 3️⃣ Temporary Tables

### 🔹 What is it?

* Exists only within a **session**
* Automatically dropped when session ends
* Supports **Time Travel within session only**
* ❌ No Fail-safe

### 🔹 Key Features

* Session-scoped
* Not visible to other users or sessions
* No long-term recovery

### 🔹 Retention

* Time Travel: session lifetime
* Fail-safe: ❌ Not available

### 🔹 Use Case

✔ Session-based transformations
✔ Debugging and testing
✔ Short-lived calculations

---

## 4️⃣ External Tables

### 🔹 What is it?

* Metadata-only table
* References data stored in **external cloud storage**
* Snowflake does **not own the data**

### 🔹 Key Features

* Data stays in S3/Azure/GCS
* Snowflake only stores metadata
* Supports querying external data

### 🔹 Retention

* Time Travel: ❌ Not supported
* Fail-safe: ❌ Not supported

### 🔹 Use Case

✔ Data lake querying
✔ Cost-sensitive workloads
✔ Hybrid architectures

---

## 🆚 Table Types Comparison

| Table Type | Time Travel    | Fail-safe | Storage Cost | Scope            |
| ---------- | -------------- | --------- | ------------ | ---------------- |
| Permanent  | ✅ Yes          | ✅ Yes     | High         | Persistent       |
| Transient  | ✅ Limited      | ❌ No      | Medium       | Persistent       |
| Temporary  | ✅ Session only | ❌ No      | Low          | Session          |
| External   | ❌ No           | ❌ No      | External     | External storage |

---

# ⏪ Time Travel in Snowflake

### 🔹 What is Time Travel?

Time Travel allows you to:

* Query **historical data**
* Recover **accidentally deleted or updated data**
* Restore tables to a previous state

### 🔹 Example

```sql
SELECT * FROM sales AT (OFFSET => -3600);
```

Restore dropped table:

```sql
UNDROP TABLE sales;
```

---

### 🔹 Time Travel Retention

* Standard Edition: **1 day**
* Enterprise+ Edition: **up to 90 days**
* Configurable per table

---

# 🛟 Fail-safe in Snowflake

### 🔹 What is Fail-safe?

Fail-safe is a **last-resort disaster recovery mechanism**.

* Applies **only to permanent tables**
* Data retained for **7 days**
* Not user-accessible
* Recovery requires Snowflake Support

---

### 🔹 When Fail-safe Is Used

* Accidental database or schema drop
* Catastrophic data loss
* Compliance-driven recovery

---

### 🔹 Fail-safe vs Time Travel

| Feature         | Time Travel         | Fail-safe         |
| --------------- | ------------------- | ----------------- |
| User accessible | ✅ Yes               | ❌ No              |
| Configurable    | ✅ Yes               | ❌ No              |
| Retention       | Up to 90 days       | Fixed 7 days      |
| Cost            | Included            | Storage only      |
| Use case        | Recovery & auditing | Disaster recovery |

---

# 🔗 How Tables, Time Travel & Fail-safe Are Related

| Table Type | Time Travel | Fail-safe |
| ---------- | ----------- | --------- |
| Permanent  | ✅ Yes       | ✅ Yes     |
| Transient  | ✅ Yes       | ❌ No      |
| Temporary  | Limited     | ❌ No      |
| External   | ❌ No        | ❌ No      |

📌 **Key Insight**:

* **Fail-safe only exists because Snowflake owns the storage**
* External tables and transient tables don’t qualify

---

# 🎯 Interview-Ready Summary Answer

> Snowflake supports permanent, transient, temporary, and external tables. Permanent tables support both Time Travel and Fail-safe, making them suitable for production data. Transient tables support limited Time Travel but no Fail-safe and are used for staging data. Temporary tables are session-scoped and short-lived. External tables reference data in cloud storage and do not support Time Travel or Fail-safe. Time Travel allows users to access historical data, while Fail-safe is a Snowflake-managed disaster recovery mechanism available only for permanent tables.

------------------------------------------------------------------------------------------------------------------------------------------------------------






Here is a **clear, concise, interview-ready explanation** of the **main differences between Transient and Temporary tables in Snowflake**, exactly how interviewers expect a **6-year experience** candidate to answer.

---

# 🔑 Main Difference Between Transient and Temporary Tables in Snowflake

Although both are **non-permanent tables**, they serve **very different purposes** in Snowflake.

---

## 🧾 1️⃣ Transient Table

### 🔹 What is it?

* A **persistent table** (exists beyond session)
* Designed for **intermediate or staging data**
* Lower storage cost than permanent tables

### 🔹 Key Characteristics

* Data persists until explicitly dropped
* Accessible by multiple users and sessions
* Supports **Time Travel (limited)**
* ❌ Does **not** support Fail-safe

### 🔹 Retention

* Time Travel: **0–1 day (configurable)**
* Fail-safe: ❌ Not available

### 🔹 Typical Use Cases

✔ Staging tables
✔ ETL intermediate data
✔ Temporary business data
✔ Large volumes with short retention needs

---

## ⏱️ 2️⃣ Temporary Table

### 🔹 What is it?

* A **session-scoped table**
* Exists only during the current user session
* Automatically dropped when the session ends

### 🔹 Key Characteristics

* Visible only within the session that created it
* Not accessible by other users
* Supports **Time Travel only during the session**
* ❌ No Fail-safe

### 🔹 Retention

* Time Travel: **session lifetime**
* Fail-safe: ❌ Not available

### 🔹 Typical Use Cases

✔ Session-based transformations
✔ Debugging / testing
✔ Ad-hoc analysis
✔ Short-lived calculations

---

## 🆚 Side-by-Side Comparison

| Feature               | Transient Table  | Temporary Table     |
| --------------------- | ---------------- | ------------------- |
| Persistence           | Persistent       | Session-only        |
| Dropped automatically | ❌ No             | ✅ Yes (session end) |
| Visible to others     | ✅ Yes            | ❌ No                |
| Time Travel           | ✅ Limited        | ✅ Session only      |
| Fail-safe             | ❌ No             | ❌ No                |
| Storage cost          | Lower            | Lower               |
| Production use        | Common (staging) | Rare                |

---

## 🎯 Interview-Ready One-Line Answer

> The main difference is that **transient tables persist beyond a session and are shared across users**, while **temporary tables exist only for the duration of a session and are automatically dropped when the session ends**. Transient tables are commonly used for staging data, whereas temporary tables are used for session-level processing and testing.

------------------------------------------------------------------------------------------------------------------------------------------------------------------



Below is a **clear, structured, interview-ready explanation** of the **difference between Bulk Loading and Continuous Loading in Snowflake**, with **definitions, examples, use cases, pros/cons, and interview tips**—perfect for a **5–7 years experience** profile.

---

# 🚚 Bulk Loading vs 🔁 Continuous Loading in Snowflake

Bulk loading and continuous loading are **two different data ingestion patterns** in Snowflake, chosen based on **data volume, latency requirements, and cost considerations**.

---

## 1️⃣ Bulk Loading

### 🔹 What is Bulk Loading?

Bulk loading is the process of loading **large volumes of data at scheduled intervals** (hourly, daily, weekly) into Snowflake using the `COPY INTO` command.

---

### 🔹 How it works

```
Source Files (S3 / Azure / GCS)
        ↓
External Stage
        ↓
COPY INTO (Batch)
        ↓
Snowflake Table
```

---

### 🔹 Tools Used

* `COPY INTO`
* External/Internal stages
* Tasks or external schedulers (Airflow, ADF, etc.)

---

### 🔹 When to Use Bulk Loading

✔ Large volumes of data
✔ Periodic ingestion (hourly/daily)
✔ Cost-sensitive workloads
✔ Data warehouses / BI reporting

---

### 🔹 Example

```sql
COPY INTO sales
FROM @s3_stage
FILE_FORMAT = (TYPE = CSV)
ON_ERROR = 'CONTINUE';
```

---

### 🔹 Pros

✅ High throughput
✅ Cost efficient
✅ Simple to manage
✅ Easy retries

---

### 🔹 Cons

❌ Higher latency
❌ Not near real-time

---

---

## 2️⃣ Continuous Loading

### 🔹 What is Continuous Loading?

Continuous loading ingests data **as soon as files arrive** in cloud storage, enabling **near real-time data ingestion**.

In Snowflake, this is implemented using **Snowpipe**.

---

### 🔹 How it works

```
Source System
   ↓
Cloud Storage (S3 / Azure / GCS)
   ↓ (Event Notification)
Snowpipe
   ↓
Snowflake Table
```

---

### 🔹 Tools Used

* **Snowpipe**
* External stages
* Cloud event notifications (SNS/SQS, Event Grid, Pub/Sub)

---

### 🔹 When to Use Continuous Loading

✔ Near real-time ingestion
✔ Event-driven architectures
✔ Streaming or micro-batch data
✔ Operational analytics

---

### 🔹 Example

```sql
CREATE PIPE sales_pipe AS
COPY INTO sales
FROM @s3_stage
FILE_FORMAT = (TYPE = JSON);
```

---

### 🔹 Pros

✅ Low latency
✅ Automated ingestion
✅ Scalable and fault tolerant
✅ No manual scheduling

---

### 🔹 Cons

❌ Higher cost per file
❌ Less control over batching

---

## 🆚 Bulk vs Continuous Loading (Comparison Table)

| Feature      | Bulk Loading          | Continuous Loading   |
| ------------ | --------------------- | -------------------- |
| Tool         | COPY INTO             | Snowpipe             |
| Data arrival | Scheduled             | Event-driven         |
| Latency      | Minutes to hours      | Seconds to minutes   |
| Cost         | Lower                 | Higher               |
| Use case     | Analytics / reporting | Real-time dashboards |
| Automation   | Scheduler-based       | Fully automated      |
| File size    | Large files           | Small/micro files    |

---

## 🎯 Interview-Ready Summary Answer

> Bulk loading uses `COPY INTO` to load large volumes of data at scheduled intervals and is optimized for cost and throughput. Continuous loading uses Snowpipe to ingest data automatically as files arrive, providing near real-time ingestion. Bulk loading is ideal for batch analytics, while continuous loading is suited for event-driven or real-time use cases.

---

## 💡 Senior-Level Interview Tips

Be ready to explain:

* **Why Snowpipe is more expensive** (per-file micro-batching)
* How Snowflake handles **file deduplication** in both
* When **micro-batching with COPY INTO** is better than Snowpipe
* How you chose ingestion method in **real projects**

------------------------------------------------------------------------------------------------





Below is a **senior-level, interview-ready explanation** covering **both parts of your question**:

1. **How Snowflake handles file de-duplication** (for `COPY INTO` and Snowpipe)
2. **When micro-batching with `COPY INTO` is better than Snowpipe**

This answer is commonly expected for **5–7+ years Snowflake interviews**.

---

# 🔁 How Snowflake Handles File De-duplication

Snowflake provides **automatic, metadata-based file de-duplication** for **both `COPY INTO` (bulk loads) and Snowpipe (continuous loads)**.

---

## 1️⃣ File De-duplication in `COPY INTO` (Bulk Loading)

### 🔹 How it works

When you run `COPY INTO`, Snowflake:

* Records metadata for each successfully loaded file:

  * File name
  * File path
  * Stage location
  * Target table
  * Load timestamp
* Stores this metadata internally

On subsequent `COPY INTO` executions:

* Files already loaded are **automatically skipped**
* Prevents duplicate data ingestion

---

### 🔹 Key Characteristics

* De-duplication is **table-specific**
* Metadata is retained for **64 days**
* Skipped files appear as `STATUS = 'SKIPPED'` in `COPY_HISTORY`

---

### 🔹 Example

```sql
COPY INTO sales
FROM @s3_stage;
```

Re-running this command:
➡️ Already loaded files are skipped automatically

---

### 🔹 Force reload (override)

```sql
COPY INTO sales
FROM @s3_stage
FORCE = TRUE;
```

⚠️ Use carefully — can cause duplicates

---

---

## 2️⃣ File De-duplication in Snowpipe (Continuous Loading)

### 🔹 How it works

Snowpipe also:

* Tracks ingested files using internal metadata
* Ensures **exactly-once file ingestion**

Even if:

* The same file event is sent multiple times
* The cloud notification is duplicated

➡️ Snowpipe loads the file **only once**

---

### 🔹 Key Characteristics

* Fully managed by Snowflake
* De-duplication handled automatically
* No user intervention required
* Same **64-day metadata retention**

---

### 🔹 Important Interview Point

> Snowpipe is **idempotent by design**, which is critical in event-driven architectures where duplicate notifications are common.

---

## 🆚 File De-duplication Comparison

| Feature                  | COPY INTO | Snowpipe |
| ------------------------ | --------- | -------- |
| File tracking            | ✅ Yes     | ✅ Yes    |
| Skips loaded files       | ✅ Yes     | ✅ Yes    |
| Duplicate events handled | N/A       | ✅ Yes    |
| FORCE reload option      | ✅ Yes     | ❌ No     |
| Metadata retention       | 64 days   | 64 days  |

---

# 🚀 When Micro-Batching with COPY INTO Is Better Than Snowpipe

Micro-batching means:

* Accumulating **small files** over time
* Loading them together in a single `COPY INTO` execution

---

## 1️⃣ Cost Efficiency (Biggest Reason)

### 🔹 Snowpipe cost model

* Billed **per file**
* High overhead for **many small files**

### 🔹 COPY INTO advantage

* One query processes **many files**
* Much lower credit usage

📌 **Interview statement**:

> For high-frequency small files, micro-batching with COPY INTO is significantly more cost-effective than Snowpipe.

---

## 2️⃣ File Arrival Pattern

### Use COPY INTO micro-batching when:

* Files arrive every few minutes
* Real-time ingestion is **not mandatory**
* SLA allows 5–15 minutes latency

---

## 3️⃣ Large Volume + Small File Problem

### Example scenario:

* 10,000 files/hour
* Each file is a few KBs

❌ Snowpipe → very expensive
✅ Micro-batch COPY → optimal

---

## 4️⃣ More Control Over Error Handling

With `COPY INTO` you can:

* Control `ON_ERROR`
* Validate before load (`VALIDATION_MODE`)
* Retry selectively
* Log errors explicitly

Snowpipe has **limited error-handling visibility**

---

## 5️⃣ Operational Simplicity

* One scheduled task (Snowflake Task or Airflow)
* Easier debugging
* Clear audit trail

---

## 🧠 Decision Matrix (Say This in Interview)

| Requirement                | Best Choice             |
| -------------------------- | ----------------------- |
| Near real-time (seconds)   | Snowpipe                |
| Cost-sensitive ingestion   | COPY INTO               |
| Large batch loads          | COPY INTO               |
| Many small files           | COPY INTO (micro-batch) |
| Event-driven pipeline      | Snowpipe                |
| Detailed validation needed | COPY INTO               |

---

## 🎯 Interview-Ready Summary Answer

> Snowflake handles file de-duplication for both `COPY INTO` and Snowpipe by tracking file metadata and skipping files that have already been successfully loaded. This metadata is retained for 64 days and ensures idempotent ingestion. Snowpipe additionally handles duplicate event notifications automatically. Micro-batching with `COPY INTO` is preferred over Snowpipe when cost efficiency is important, especially when dealing with a large number of small files and when near real-time ingestion is not required.

---

---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

Below is a **clear, end-to-end, interview-ready explanation** of **how Snowpipe works** and **when to choose it over batch loads**, written at the depth expected from a **6+ years Snowflake candidate**.

---

# 🔁 How Snowpipe Works in Snowflake

Snowpipe is Snowflake’s **continuous data ingestion service** that automatically loads data **as soon as files arrive** in cloud storage.

---

## 🧩 Snowpipe Architecture (Step-by-Step)

### **Step 1: Data lands in cloud storage**

* Files are dropped into:

  * AWS S3
  * Azure Blob Storage
  * Google Cloud Storage

---

### **Step 2: Cloud event notification**

* Storage service sends an event when a file is created:

  * AWS → S3 Event → SNS → SQS
  * Azure → Event Grid
  * GCP → Pub/Sub

This event contains:

* File name
* Path
* Stage location

---

### **Step 3: External stage**

* Snowflake external stage points to the cloud storage location
* Uses:

  * Storage integration (IAM-based security)
  * File format definition

---

### **Step 4: Snowpipe receives event**

* Snowpipe listens for file arrival events
* Triggers ingestion automatically
* No scheduler or manual trigger needed

---

### **Step 5: COPY INTO execution**

Internally, Snowpipe executes a `COPY INTO` command:

* Uses serverless compute (managed by Snowflake)
* Loads files into target tables
* Performs:

  * File parsing
  * Validation
  * Deduplication

---

### **Step 6: File de-duplication**

* Snowflake tracks loaded files
* Ensures **exactly-once ingestion**
* Duplicate notifications do NOT cause duplicate loads

---

### **Step 7: Monitoring & error handling**

* Use:

  * `LOAD_HISTORY`
  * `PIPE_USAGE_HISTORY`
  * `COPY_HISTORY`
* Errors logged automatically

---

## 🔧 Snowpipe Example

```sql
CREATE OR REPLACE PIPE sales_pipe
AUTO_INGEST = TRUE
AS
COPY INTO sales
FROM @s3_sales_stage
FILE_FORMAT = (TYPE = JSON);
```

---

# 🆚 When to Choose Snowpipe Over Batch Loads

---

## ✅ Choose Snowpipe When:

### 1️⃣ **Near Real-Time Ingestion Is Required**

* Data must be available in minutes or seconds
* Used for:

  * Operational dashboards
  * Real-time monitoring
  * Event-driven systems

📌 Example:

> Clickstream or IoT data used in near real-time dashboards

---

### 2️⃣ **Event-Driven Architecture**

* Data arrives unpredictably
* No fixed schedule
* Snowpipe reacts automatically

---

### 3️⃣ **Low Operational Overhead**

* No need to:

  * Manage schedulers
  * Write orchestration logic
* Fully serverless

---

### 4️⃣ **Small or Micro-Batch Files**

* Frequent file arrivals
* Each file triggers ingestion

---

### 5️⃣ **High Availability & Reliability**

* Automatic retries
* Idempotent ingestion
* Handles duplicate events gracefully

---

## ❌ Avoid Snowpipe When:

### 🚫 Cost Sensitivity Is High

* Snowpipe charges per file
* Expensive for:

  * Thousands of small files per hour

---

### 🚫 Real-Time Not Required

* SLA allows minutes/hours delay
* Batch loads are cheaper

---

### 🚫 Heavy Validation Required

* Snowpipe has limited validation controls
* Batch loads allow:

  * `VALIDATION_MODE`
  * Pre-load checks
  * Manual retries

---

## 🆚 Snowpipe vs Batch Loading (COPY INTO)

| Feature            | Snowpipe            | Batch Load     |
| ------------------ | ------------------- | -------------- |
| Latency            | Seconds–Minutes     | Minutes–Hours  |
| Trigger            | Event-based         | Scheduled      |
| Compute            | Serverless          | User warehouse |
| Cost               | Higher (per file)   | Lower          |
| Operational effort | Very low            | Moderate       |
| Validation control | Limited             | High           |
| Best for           | Real-time ingestion | Analytics & BI |

---

## 🎯 Interview-Ready Summary Answer

> Snowpipe is Snowflake’s continuous ingestion service that automatically loads data as soon as files arrive in cloud storage using event notifications. It executes serverless `COPY INTO` operations and ensures exactly-once ingestion through file deduplication. Snowpipe is chosen over batch loads when near real-time data availability is required, operational simplicity is important, and data arrives in small or unpredictable batches. Batch loads are preferred when cost efficiency and detailed validation are more important than low latency.

---

---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------


Below is a **step-by-step, interview-ready explanation** of **how to create a Snowpipe with auto-ingest** and **which AWS services are required**, exactly at the depth expected for a **6+ years Snowflake interview**.

---

# 🚀 Creating a Snowpipe with Auto-Ingest (AWS)

Snowpipe with **AUTO_INGEST = TRUE** uses **AWS event notifications** so that Snowflake automatically loads data as soon as files arrive in **S3**.

---

## 🔁 High-Level Architecture (AWS)

```
Source System
   ↓
Amazon S3 (new file arrives)
   ↓
S3 Event Notification
   ↓
SNS Topic
   ↓
SQS Queue
   ↓
Snowpipe (AUTO_INGEST)
   ↓
Snowflake Table
```

---

# 🧩 AWS Services Required

To enable **auto-ingest Snowpipe on AWS**, you need **four AWS components**:

### 1️⃣ Amazon S3

* Stores incoming data files
* Triggers events when new files arrive

---

### 2️⃣ Amazon SNS (Simple Notification Service)

* Receives S3 event notifications
* Publishes messages to SQS

---

### 3️⃣ Amazon SQS (Simple Queue Service)

* Queue that Snowflake polls
* Decouples S3 events from Snowpipe
* Ensures reliability and retry handling

---

### 4️⃣ IAM Role (via Storage Integration)

* Grants Snowflake permission to:

  * Read S3 data
  * Read from SQS queue
* Uses **least-privilege access**

📌 **Important Interview Point**
Snowflake **does NOT use access keys**. It uses **IAM role + trust policy**.

---

# 🛠️ Step-by-Step: Create Snowpipe with Auto-Ingest

---

## ✅ Step 1: Create a Storage Integration in Snowflake

```sql
CREATE OR REPLACE STORAGE INTEGRATION s3_int
TYPE = EXTERNAL_STAGE
STORAGE_PROVIDER = S3
ENABLED = TRUE
STORAGE_AWS_ROLE_ARN = 'arn:aws:iam::123456789012:role/snowflake_role'
STORAGE_ALLOWED_LOCATIONS = ('s3://my-bucket/data/');
```

Get Snowflake’s IAM user:

```sql
DESC INTEGRATION s3_int;
```

➡️ Use this to create **trust relationship** in AWS IAM

---

## ✅ Step 2: Create External Stage

```sql
CREATE OR REPLACE STAGE s3_stage
URL = 's3://my-bucket/data/'
STORAGE_INTEGRATION = s3_int
FILE_FORMAT = (TYPE = CSV);
```

---

## ✅ Step 3: Create Snowpipe with AUTO_INGEST

```sql
CREATE OR REPLACE PIPE sales_pipe
AUTO_INGEST = TRUE
AS
COPY INTO sales
FROM @s3_stage
FILE_FORMAT = (TYPE = CSV);
```

---

## ✅ Step 4: Get Snowpipe Notification Channel (Very Important)

```sql
DESC PIPE sales_pipe;
```

Output:

```text
notification_channel = arn:aws:sqs:us-east-1:123456789012:snowpipe_queue
```

---

## ✅ Step 5: Configure AWS Resources

### 🔹 Create SQS Queue

* Use ARN from `DESC PIPE`

---

### 🔹 Create SNS Topic

* Subscribe SQS queue to SNS topic

---

### 🔹 Configure S3 Event Notification

* On **ObjectCreated**
* Send event to SNS topic
* Filter by prefix/suffix if needed (e.g., `.csv`)

---

## 🧪 Verification

### Check pipe status:

```sql
SELECT SYSTEM$PIPE_STATUS('sales_pipe');
```

### View load history:

```sql
SELECT * 
FROM TABLE(INFORMATION_SCHEMA.LOAD_HISTORY(
  PIPE_NAME => 'sales_pipe'
));
```

---

# 🆚 AUTO_INGEST vs Manual Snowpipe

| Feature    | AUTO_INGEST | Manual               |
| ---------- | ----------- | -------------------- |
| Trigger    | S3 event    | REST API             |
| AWS setup  | Required    | Not required         |
| Latency    | Seconds     | Depends              |
| Complexity | Higher      | Lower                |
| Use case   | Real-time   | Controlled ingestion |

---

# 🎯 Interview-Ready Summary Answer

> To create a Snowpipe with auto-ingest on AWS, we first create a storage integration and external stage pointing to S3. Then we create a Snowpipe with `AUTO_INGEST = TRUE`. On AWS, we configure S3 event notifications to send object-created events to an SNS topic, which forwards them to an SQS queue. Snowflake listens to this SQS queue and automatically triggers the Snowpipe to load data. Required AWS services include S3, SNS, SQS, and IAM roles.

---
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------







Below is a **clear, structured, interview-ready explanation** of **common `COPY INTO` options**, especially **`ON_ERROR`** and **`VALIDATION_MODE`**, with **examples, use cases, and senior-level tips**—exactly what’s expected for a **6-year Snowflake interview**.

---

# 📥 Common `COPY INTO` Options in Snowflake

The `COPY INTO` command supports several options to **control error handling, validation, performance, and behavior during data loads**.

The **most commonly asked in interviews** are:

* `ON_ERROR`
* `VALIDATION_MODE`

I’ll also cover **other important options** briefly.

---

## 1️⃣ `ON_ERROR` – Error Handling Strategy

### 🔹 What is `ON_ERROR`?

Controls **what Snowflake does when it encounters errors** while loading data.

---

### 🔹 Common `ON_ERROR` Values

| Option                      | Behavior                     | When to Use                 |
| --------------------------- | ---------------------------- | --------------------------- |
| `ABORT_STATEMENT` (default) | Stops load immediately       | Strict data quality         |
| `CONTINUE`                  | Skips bad rows, loads rest   | Large files, tolerant loads |
| `SKIP_FILE`                 | Skips entire file on error   | File-level validation       |
| `SKIP_FILE_<n>`             | Skip file if errors exceed n | Controlled tolerance        |

---

### 🔹 Examples

#### Abort on first error (default)

```sql
COPY INTO sales
FROM @s3_stage
FILE_FORMAT = (TYPE = CSV)
ON_ERROR = 'ABORT_STATEMENT';
```

---

#### Continue loading valid rows

```sql
COPY INTO sales
FROM @s3_stage
ON_ERROR = 'CONTINUE';
```

---

#### Skip entire file if any error occurs

```sql
COPY INTO sales
FROM @s3_stage
ON_ERROR = 'SKIP_FILE';
```

---

#### Skip file only if errors exceed threshold

```sql
COPY INTO sales
FROM @s3_stage
ON_ERROR = 'SKIP_FILE_5';
```

---

### 🔹 Interview Insight

> `CONTINUE` may lead to **partial data loads**, so it’s often paired with **error logging and reconciliation**.

---

## 2️⃣ `VALIDATION_MODE` – Validate Without Loading

### 🔹 What is `VALIDATION_MODE`?

Allows you to **check data quality without inserting data** into the table.

---

### 🔹 Common Validation Modes

| Mode                | Purpose                          |
| ------------------- | -------------------------------- |
| `RETURN_ERRORS`     | Returns all row-level errors     |
| `RETURN_N_ROWS`     | Validates first N rows           |
| `RETURN_ALL_ERRORS` | Returns all errors (large files) |

---

### 🔹 Examples

#### Validate data and return errors

```sql
COPY INTO sales
FROM @s3_stage
VALIDATION_MODE = 'RETURN_ERRORS';
```

---

#### Validate first 100 rows only

```sql
COPY INTO sales
FROM @s3_stage
VALIDATION_MODE = 'RETURN_100_ROWS';
```

---

### 🔹 Important Notes

* **No data is loaded**
* Ideal for **pre-production validation**
* Common in CI/CD pipelines

---

## 3️⃣ Other Important `COPY INTO` Options (Interview-Relevant)

---

### 🔹 `FILE_FORMAT`

Specifies how to parse files:

```sql
FILE_FORMAT = (TYPE = CSV FIELD_DELIMITER = ',' SKIP_HEADER = 1);
```

---

### 🔹 `FORCE`

Forces reload of files already loaded:

```sql
COPY INTO sales
FROM @s3_stage
FORCE = TRUE;
```

⚠️ Can cause duplicates if not handled carefully.

---

### 🔹 `PATTERN`

Loads only matching files:

```sql
COPY INTO sales
FROM @s3_stage
PATTERN = '.*2025.*.csv';
```

---

### 🔹 `PURGE`

Deletes files from stage after successful load:

```sql
COPY INTO sales
FROM @s3_stage
PURGE = TRUE;
```

⚠️ Use cautiously—data loss risk.

---

### 🔹 `SIZE_LIMIT`

Limits total data size per load:

```sql
COPY INTO sales
FROM @s3_stage
SIZE_LIMIT = 50000000;
```

---

## 4️⃣ How You Typically Use These in Real Projects

### 🔹 Development / Testing

```sql
VALIDATION_MODE = 'RETURN_ERRORS'
```

---

### 🔹 Production (Large Files)

```sql
ON_ERROR = 'SKIP_FILE_10'
```

---

### 🔹 Real-Time / Snowpipe

```sql
ON_ERROR = 'CONTINUE'
```

---

## 🎯 Interview-Ready Summary Answer

> `ON_ERROR` controls how Snowflake handles errors during data loads, such as aborting the load, skipping bad rows, or skipping entire files. `VALIDATION_MODE` allows validating data without actually loading it, which is useful for pre-load quality checks. Together, these options provide flexibility to balance data quality, performance, and operational reliability during ingestion.

---






---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------



Below is a **practical, interview-ready explanation** of **how rejected or bad records are handled during Snowflake loads**, written at the level expected for a **5–7 years experience** candidate. I’ve included **mechanisms, examples, real-world patterns, and senior-level tips**.

---

# 🚫 Handling Rejected or Bad Records in Snowflake Loads

Snowflake provides **multiple mechanisms** to detect, handle, log, and recover **bad or rejected records** during data loading.

---

## 1️⃣ Use `ON_ERROR` to Control Load Behavior (First Line of Defense)

### 🔹 Purpose

Controls what Snowflake does **when it encounters bad records**.

### 🔹 Common Options

| Option            | Behavior                           |
| ----------------- | ---------------------------------- |
| `ABORT_STATEMENT` | Stop load on first error (default) |
| `CONTINUE`        | Skip bad rows, load good rows      |
| `SKIP_FILE`       | Skip entire file if any error      |
| `SKIP_FILE_<n>`   | Skip file after n errors           |

### 🔹 Example

```sql
COPY INTO sales
FROM @s3_stage
ON_ERROR = 'CONTINUE';
```

✔ Loads valid records
❌ Skips invalid rows

---

## 2️⃣ Pre-Validate Data Using `VALIDATION_MODE` (Before Load)

### 🔹 Why use it?

To **identify bad data before inserting anything**.

### 🔹 Example

```sql
COPY INTO sales
FROM @s3_stage
VALIDATION_MODE = 'RETURN_ERRORS';
```

✔ Returns:

* Row number
* Column name
* Error description

📌 **Common in pre-prod or CI/CD pipelines**

---

## 3️⃣ Capture Errors Using `COPY_HISTORY` and `LOAD_HISTORY`

### 🔹 Query rejected records

```sql
SELECT *
FROM TABLE(
  INFORMATION_SCHEMA.COPY_HISTORY(
    TABLE_NAME => 'SALES',
    START_TIME => DATEADD('hour', -1, CURRENT_TIMESTAMP())
  )
)
WHERE ERROR_COUNT > 0;
```

✔ Shows:

* Number of rejected rows
* Error messages
* File names

---

## 4️⃣ Load Data into a Staging (Raw) Table First (Best Practice)

### 🔹 Pattern

```
External Stage
   ↓
Raw Staging Table (VARCHAR / VARIANT)
   ↓
Validated Target Table
```

### 🔹 Why?

* Avoid load failures
* Capture all records (even bad ones)
* Apply business rules later

---

### 🔹 Example

```sql
COPY INTO raw_sales
FROM @s3_stage
FILE_FORMAT = (TYPE = CSV)
ON_ERROR = 'CONTINUE';
```

Then validate:

```sql
INSERT INTO sales
SELECT *
FROM raw_sales
WHERE TRY_TO_NUMBER(amount) IS NOT NULL;
```

Bad records can be isolated:

```sql
SELECT *
FROM raw_sales
WHERE TRY_TO_NUMBER(amount) IS NULL;
```

---

## 5️⃣ Use TRY_* Functions to Prevent Rejections

### 🔹 Common TRY functions

* `TRY_TO_NUMBER`
* `TRY_TO_DATE`
* `TRY_CAST`

### 🔹 Example

```sql
SELECT
  TRY_TO_DATE(order_date) AS order_date,
  TRY_TO_NUMBER(amount) AS amount
FROM raw_sales;
```

✔ Invalid values → `NULL`
✔ No query failure

---

## 6️⃣ Use Error Logging Tables (Enterprise Pattern)

### 🔹 Pattern

* Store rejected rows + error reason

```sql
INSERT INTO error_table
SELECT *, 'Invalid amount'
FROM raw_sales
WHERE TRY_TO_NUMBER(amount) IS NULL;
```

✔ Enables:

* Auditing
* Data correction
* Reprocessing

---

## 7️⃣ Snowpipe-Specific Handling

### 🔹 Snowpipe behavior

* Uses `ON_ERROR = CONTINUE` by default
* Bad records are skipped
* Errors logged in:

  * `LOAD_HISTORY`
  * `PIPE_USAGE_HISTORY`

📌 `VALIDATION_MODE` ❌ not supported in Snowpipe

---

## 8️⃣ Reprocess Rejected Records

### 🔹 Steps

1. Fix data in staging or error table
2. Reload using `COPY INTO` with `FORCE = TRUE`
   or
3. Insert cleaned records into target table

---

## 🎯 Interview-Ready Summary Answer

> Rejected or bad records in Snowflake are handled using a combination of `ON_ERROR` options, pre-load validation with `VALIDATION_MODE`, and post-load monitoring through `COPY_HISTORY`. A common best practice is to load data into a raw staging table first using tolerant error handling and then apply validations using `TRY_*` functions before inserting into the final table. This approach prevents load failures, allows error auditing, and enables easy reprocessing.

---

Below is a **practical, interview-ready explanation** of **how rejected or bad records are handled during Snowflake loads**, written at the level expected for a **5–7 years experience** candidate. I’ve included **mechanisms, examples, real-world patterns, and senior-level tips**.

---

# 🚫 Handling Rejected or Bad Records in Snowflake Loads

Snowflake provides **multiple mechanisms** to detect, handle, log, and recover **bad or rejected records** during data loading.

---

## 1️⃣ Use `ON_ERROR` to Control Load Behavior (First Line of Defense)

### 🔹 Purpose

Controls what Snowflake does **when it encounters bad records**.

### 🔹 Common Options

| Option            | Behavior                           |
| ----------------- | ---------------------------------- |
| `ABORT_STATEMENT` | Stop load on first error (default) |
| `CONTINUE`        | Skip bad rows, load good rows      |
| `SKIP_FILE`       | Skip entire file if any error      |
| `SKIP_FILE_<n>`   | Skip file after n errors           |

### 🔹 Example

```sql
COPY INTO sales
FROM @s3_stage
ON_ERROR = 'CONTINUE';
```

✔ Loads valid records
❌ Skips invalid rows

---

## 2️⃣ Pre-Validate Data Using `VALIDATION_MODE` (Before Load)

### 🔹 Why use it?

To **identify bad data before inserting anything**.

### 🔹 Example

```sql
COPY INTO sales
FROM @s3_stage
VALIDATION_MODE = 'RETURN_ERRORS';
```

✔ Returns:

* Row number
* Column name
* Error description

📌 **Common in pre-prod or CI/CD pipelines**

---

## 3️⃣ Capture Errors Using `COPY_HISTORY` and `LOAD_HISTORY`

### 🔹 Query rejected records

```sql
SELECT *
FROM TABLE(
  INFORMATION_SCHEMA.COPY_HISTORY(
    TABLE_NAME => 'SALES',
    START_TIME => DATEADD('hour', -1, CURRENT_TIMESTAMP())
  )
)
WHERE ERROR_COUNT > 0;
```

✔ Shows:

* Number of rejected rows
* Error messages
* File names

---

## 4️⃣ Load Data into a Staging (Raw) Table First (Best Practice)

### 🔹 Pattern

```
External Stage
   ↓
Raw Staging Table (VARCHAR / VARIANT)
   ↓
Validated Target Table
```

### 🔹 Why?

* Avoid load failures
* Capture all records (even bad ones)
* Apply business rules later

---

### 🔹 Example

```sql
COPY INTO raw_sales
FROM @s3_stage
FILE_FORMAT = (TYPE = CSV)
ON_ERROR = 'CONTINUE';
```

Then validate:

```sql
INSERT INTO sales
SELECT *
FROM raw_sales
WHERE TRY_TO_NUMBER(amount) IS NOT NULL;
```

Bad records can be isolated:

```sql
SELECT *
FROM raw_sales
WHERE TRY_TO_NUMBER(amount) IS NULL;
```

---

## 5️⃣ Use TRY_* Functions to Prevent Rejections

### 🔹 Common TRY functions

* `TRY_TO_NUMBER`
* `TRY_TO_DATE`
* `TRY_CAST`

### 🔹 Example

```sql
SELECT
  TRY_TO_DATE(order_date) AS order_date,
  TRY_TO_NUMBER(amount) AS amount
FROM raw_sales;
```

✔ Invalid values → `NULL`
✔ No query failure

---

## 6️⃣ Use Error Logging Tables (Enterprise Pattern)

### 🔹 Pattern

* Store rejected rows + error reason

```sql
INSERT INTO error_table
SELECT *, 'Invalid amount'
FROM raw_sales
WHERE TRY_TO_NUMBER(amount) IS NULL;
```

✔ Enables:

* Auditing
* Data correction
* Reprocessing

---

## 7️⃣ Snowpipe-Specific Handling

### 🔹 Snowpipe behavior

* Uses `ON_ERROR = CONTINUE` by default
* Bad records are skipped
* Errors logged in:

  * `LOAD_HISTORY`
  * `PIPE_USAGE_HISTORY`

📌 `VALIDATION_MODE` ❌ not supported in Snowpipe

---

## 8️⃣ Reprocess Rejected Records

### 🔹 Steps

1. Fix data in staging or error table
2. Reload using `COPY INTO` with `FORCE = TRUE`
   or
3. Insert cleaned records into target table

---

## 🎯 Interview-Ready Summary Answer

> Rejected or bad records in Snowflake are handled using a combination of `ON_ERROR` options, pre-load validation with `VALIDATION_MODE`, and post-load monitoring through `COPY_HISTORY`. A common best practice is to load data into a raw staging table first using tolerant error handling and then apply validations using `TRY_*` functions before inserting into the final table. This approach prevents load failures, allows error auditing, and enables easy reprocessing.

---




















