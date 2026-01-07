
Core Concepts — Expected Beginner & Foundation Questions

Describe the steps to load data into Snowflake.e
Create table → Define file format → Create stage → Upload files (if internal) → COPY INTO → Validate → Handle errors → Transform → Automate → Monitor
Step 1: Prepare the Target Table
Before loading data, you must create a destination table that matches the structure of the incoming data.
Key considerations:
•	Correct data types
•	Handling nullable vs non-nullable columns  ?
•	For semi-structured data, use VARIANT, OBJECT, or ARRAY
•	Consider future schema evolution
________________________________________
Step 2: Define a File Format
Snowflake needs to understand how to read the incoming file.
Common file formats:
•	CSV
•	JSON
•	Parquet
•	XML
File format defines:
•	Delimiter - 
•	Header rows
•	Compression
•	Null handling

📌 File formats can be:
•	Defined once using CREATE FILE FORMAT
•	Or inline within the COPY INTO command
________________________________________
Step 3: Stage the Data
A stage is a location where Snowflake reads data files from.
Types of stages:
1.	Internal Stage
o	Managed by Snowflake
o	Requires PUT command to upload files
o	Used mostly for small/manual loads
2.	External Stage
o	Cloud storage like AWS S3, Azure Blob, GCS
o	Preferred for production pipelines
o	Uses storage integration for secure access
3.	Table Stage
o	Associated with a specific table
o	Simplifies load syntax
📌 In enterprise environments, external stages are most commonly used.
________________________________________
Step 4: Upload Data (If Using Internal Stage)
This step applies only to internal stages.
•	Use the PUT command to upload local files into Snowflake’s internal storage
•	Supports compression and parallel uploads
📌 This step is not required for external stages.
________________________________________
Step 5: Load Data Using COPY INTO
This is the core data loading step.
COPY INTO:
•	Reads data from a stage
•	Parses files using the file format
•	Loads records into the target table
Key COPY INTO options:
•	ON_ERROR – controls behavior on errors
•	VALIDATION_MODE – validate without loading
•	PURGE – delete files after successful load
•	FORCE – reload files even if loaded earlier
•	PATTERN – load only specific files
📌 Snowflake automatically:
•	Parallelizes loading
•	Scales compute
•	Tracks loaded files to avoid duplicates
________________________________________
Step 6: Validate the Load
After loading, always verify data correctness.
Validation methods:
•	VALIDATION_MODE = RETURN_ERRORS
•	VALIDATION_MODE = RETURN_ALL_ERRORS
•	COPY_HISTORY / LOAD_HISTORY
•	Querying rejected records
•	Row count reconciliation
📌 This step is critical in production pipelines.
________________________________________
Step 7: Handle Errors and Rejected Records
Snowflake allows granular error handling.
Error strategies:
•	Skip bad rows and load good data
•	Log rejected rows for analysis
•	Fix data issues and reload
Common errors include:
•	Data type mismatches
•	Extra or missing columns
________________________________________
Step 8: Transform and Merge Data (Optional but Common)
In real-world projects:
•	Data is first loaded into raw/staging tables
•	Then transformed into final tables using:
o	INSERT
o	MERGE
o	CTAS
📌 This supports:
•	Deduplication
•	Incremental loads
•	Business logic enforcement
________________________________________
Step 9: Automate the Load (Production Setup)
For ongoing ingestion, Snowflake provides:
Automation options:
•	Snowpipe – continuous, near real-time ingestion
•	Tasks – scheduled batch loads
•	Streams + Tasks – incremental processing
•	External orchestrators (Airflow, ADF, etc.)
📌 Snowpipe is ideal for event-driven ingestion, while Tasks suit batch pipelines.
________________________________________
Step 10: Monitor and Optimize
Finally, monitor and optimize loads.
Monitoring tools:
•	Query history
•	Load history views
•	Warehouse credit usage
•	Snowflake UI dashboards
Optimization strategies:
•	Right-size warehouses
•	Batch files efficiently
•	Use clustering when needed
•	Avoid unnecessary reloads
















What types of stages exist in Snowflake and when would you use each?
Types of Stages in Snowflake and When to Use Each -
A stage in Snowflake is a storage location used to load data into tables or unload data from tables. Stages act as an abstraction layer between Snowflake and data files.
Snowflake supports three main types of stages:
1.	Internal Stages
2.	External Stages
3.	Table Stages


1. Internal Stage
An internal stage is a location within Snowflake itself. You don’t need external storage like S3, Azure Blob, or Google Cloud Storage. Files can be uploaded here temporarily for loading data.
Steps to Create:
Step 1: Create a named internal stage
CREATE STAGE my_internal_stage
  FILE_FORMAT = (TYPE = CSV FIELD_OPTIONALLY_ENCLOSED_BY='"');
•	FILE_FORMAT is optional but recommended.
•	By default, the stage is empty and ready to store files.
Step 2: Upload files to the stage
PUT file:///local/path/myfile.csv @my_internal_stage;
•	PUT uploads local files to the stage.
•	@my_internal_stage references the internal stage.
Step 3: Load data into a table
COPY INTO my_table
FROM @my_internal_stage
FILE_FORMAT = (TYPE = CSV FIELD_OPTIONALLY_ENCLOSED_BY='"');
________________________________________
2. External Stage
An external stage points to a storage outside Snowflake, such as AWS S3, Azure Blob, or GCP Cloud Storage. Useful for large datasets and shared storage.
Steps to Create:
Step 1: Create a named external stage
CREATE STAGE my_external_stage
  URL='s3://mybucket/data/'
  STORAGE_INTEGRATION = my_s3_integration
  FILE_FORMAT = (TYPE = CSV FIELD_OPTIONALLY_ENCLOSED_BY='"');
•	STORAGE_INTEGRATION is needed to securely connect Snowflake to S3 or other cloud storage.
•	URL points to the bucket or folder in your cloud storage.
Step 2: List files in the stage
LIST @my_external_stage;
Step 3: Load data into a table
COPY INTO my_table
FROM @my_external_stage
FILE_FORMAT = (TYPE = CSV FIELD_OPTIONALLY_ENCLOSED_BY='"');
________________________________________
3. Table Stage
Every table in Snowflake has its own internal stage, called a table stage. This is convenient for temporary storage without creating a named stage.
Steps to Use a Table Stage:
Step 1: Put file into table stage
PUT file:///local/path/myfile.csv @%my_table;
•	@%my_table references the table stage.
•	No need to create it explicitly; it exists automatically.
Step 2: Load data into the table
COPY INTO my_table
FROM @%my_table
FILE_FORMAT = (TYPE = CSV FIELD_OPTIONALLY_ENCLOSED_BY='"');
Step 3: Optional – List files in table stage
LIST @%my_table;
________________________________________

Exactly! ✅
You do not have to create a table stage manually in Snowflake. Every table automatically comes with its own internal table stage as soon as the table is created.
Here’s the breakdown:
•	Internal / Named Stage: You create manually with CREATE STAGE. You can share it across tables and users.
•	Table Stage: Auto-created when you create a table.
o	Access it using @%table_name.
o	Useful for temporary uploads specifically for that table.
o	No CREATE STAGE command needed.
Quick Example
-- Create table
CREATE TABLE my_table (
    id INT,
    name STRING
);

-- Upload file to table stage (no need to create stage)
PUT file:///local/path/data.csv @%my_table;

-- Load into table
COPY INTO my_table
FROM @%my_table
FILE_FORMAT = (TYPE=CSV);
So, table stages are “built-in” internal stages for each table, perfect for quick loads without managing extra stages.

Summary Table
Stage Type	Location	Creation Step	Example Reference
Internal Stage	Snowflake	CREATE STAGE ...	@my_internal_stage
External Stage	Cloud Storage	CREATE STAGE ... URL=...	@my_external_stage
Table Stage	Specific Table	Auto-created (no CREATE)	@%my_table

________________________________________
1️⃣ Internal Stages
🔹 What is it?
Internal stages are Snowflake-managed storage locations where data files are stored inside Snowflake.


Snowflake takes care of:
•	Storage
•	Security
•	Encryption
•	Access control
🔹 Types of Internal Stages
Type	Description
User Stage (@~)	Each user gets a personal stage
Named Internal Stage	Explicitly created stage
Table Stage (@%table_name)	Automatically created for each table
________________________________________
🔹 How data is loaded
•	Files are uploaded using the PUT command
•	Then loaded using COPY INTO
________________________________________
🔹 When to use Internal Stages
✅ Small to medium data loads
✅ Ad-hoc or manual data uploads
✅ Proof of concept (POC) or testing
✅ When cloud storage is not available
📌 Not recommended for production pipelines due to manual file uploads and limited scalability.
________________________________________
🔹 Example scenario
A data analyst needs to load a one-time CSV file received via email into Snowflake.
________________________________________


🔹 Pros
✔ Simple to use
✔ No cloud setup required
✔ Secure and encrypted
🔹 Cons
❌ Requires manual file upload
❌ Not ideal for automation
❌ Limited for large-scale ingestion
________________________________________
2️⃣ External Stages ⭐ (Most Common in Production)
🔹 What is it?
External stages reference cloud object storage outside Snowflake:
•	AWS S3
•	Azure Blob Storage
•	Google Cloud Storage (GCS)
Snowflake reads data directly from these locations.
________________________________________
🔹 Security & Access
•	Uses Storage Integration (recommended)
•	Or legacy credentials (not recommended)
Storage integration provides:
•	IAM-based access
________________________________________
🔹 How data is loaded
•	Files land in cloud storage
•	Snowflake reads them via COPY INTO or Snowpipe
________________________________________

🔹 When to use External Stages
✅ Production data pipelines
✅ Large-volume data ingestion
✅ Automated and scheduled loads
✅ Integration with data lakes
✅ Snowpipe (auto-ingest)
📌 This is the industry standard approach.
________________________________________
🔹 Example scenario
Daily sales data is dropped into S3 by upstream systems and loaded into Snowflake every hour using Snowpipe.
________________________________________
🔹 Pros
✔ Highly scalable
✔ Fully automated
✔ Integrates with cloud ecosystems
✔ Supports continuous ingestion
🔹 Cons
❌ Requires cloud configuration
❌ Slightly more complex to set up
________________________________________
3️⃣ Table Stages
🔹 What is it?
A table stage is an internal stage automatically created for every table.
Syntax:
@%table_name
Files staged here are tightly coupled to that table.
________________________________________

🔹 How data is loaded
•	Files uploaded using PUT
•	Loaded using COPY INTO table_name
________________________________________
🔹 When to use Table Stages
✅ Quick loads specific to a table
✅ Temporary or intermediate data
✅ Simple staging use cases
📌 Not commonly used in enterprise pipelines.
________________________________________
🔹 Example scenario
A developer loads test data into a specific table during development.
________________________________________
🔹 Pros
✔ No need to create a stage explicitly
✔ Easy syntax
🔹 Cons
❌ Not reusable across tables
❌ Same limitations as internal stages





















Explain the difference between PUT and COPY INTO.
(When and why you use each, and whether they are needed for internal vs external stages) 
🔄 Difference Between PUT and COPY INTO in Snowflake
PUT and COPY INTO are two different steps in Snowflake’s data loading process, and they serve very different purposes.

________________________________________
🔹 High-Level Difference (One-Line Answer)
PUT uploads files into a Snowflake internal stage, while COPY INTO loads data from a stage into a Snowflake table.
________________________________________
1️⃣ PUT Command
🔹 What is PUT?
PUT is a client-side command used to upload local files from your machine into a Snowflake internal stage.
It does NOT load data into tables.
________________________________________
🔹 When do you use PUT?
You use PUT only when working with internal stages.
Typical use cases:
•	Ad-hoc or one-time data loads
•	Testing or development
•	Small datasets
•	When cloud storage (S3/Azure/GCS) is not available
________________________________________
🔹 Where does PUT work?
✅ Internal stages
❌ External stages (not allowed)
________________________________________
🔹 Example
PUT file://local/path/data.csv @my_internal_stage;
This uploads the file to Snowflake-managed storage.
________________________________________


🔹 Key Characteristics of PUT
Feature	Details
Uploads files	✅ Yes
Loads data into table	❌ No
Works with internal stages	✅ Yes
Works with external stages	❌ No
Used in production	Rare
________________________________________
2️⃣ COPY INTO Command
🔹 What is COPY INTO?
COPY INTO is a server-side command that reads data from a stage and loads it into a Snowflake table.
This is the actual data ingestion step.
________________________________________
🔹 When do you use COPY INTO?
You use COPY INTO whenever you want to load data into a Snowflake table, regardless of stage type.
Typical use cases:
•	Batch data ingestion
•	Scheduled loads
•	Production pipelines
•	Loading structured or semi-structured data
________________________________________
🔹 Where does COPY INTO work?
✅ Internal stages
✅ External stages
________________________________________
🔹 Example
COPY INTO sales
FROM @my_stage
FILE_FORMAT = (TYPE = CSV);
________________________________________
🔹 Key Characteristics of COPY INTO
Feature	Details
Uploads files	❌ No
Loads data into table	✅ Yes
Supports internal stages	✅ Yes
Supports external stages	✅ Yes
Production ready	✅ Yes
________________________________________
🔁 Internal Stage vs External Stage Usage
📌 Internal Stage Flow
Local File
   ↓ PUT
Internal Stage
   ↓ COPY INTO
Snowflake Table
•	Both PUT and COPY INTO are required
________________________________________
📌 External Stage Flow
Cloud Storage (S3/Azure/GCS)
   ↓ COPY INTO (or Snowpipe)
Snowflake Table
•	PUT is NOT used
•	Files are already in cloud storage

How does Snowflake handle semi-structured data (like JSON)? And how would you load it into a VARIANT column? 
🌐 Handling Semi-Structured Data in Snowflake
Snowflake provides native support for semi-structured data, including:
•	JSON
•	AVRO
•	ORC
•	Parquet
•	XML
________________________________________
1️⃣ Key Features
1.	VARIANT Data Type
o	Snowflake uses the VARIANT column type to store any semi-structured data.
o	Supports nested objects, arrays, and mixed types.
2.	Native Parsing
o	Snowflake can ingest JSON directly without needing schema definition.
o	You can query nested fields using:
	Dot notation: data.field1
	Bracket notation: data['field1']
o	Flatten arrays or nested structures using FLATTEN function.
3.	Schema-on-Read
o	Unlike traditional relational tables, you don’t need to define every key in advance.
o	Flexible for evolving JSON structures.
________________________________________
2️⃣ Loading JSON into a VARIANT Column
Snowflake provides two main approaches:
A. Loading from staged JSON files (bulk load)
1.	Create table with VARIANT column
CREATE TABLE raw_json_data (
    id INT AUTOINCREMENT,
    data VARIANT
);
2.	Create file format
CREATE FILE FORMAT json_ff
    TYPE = JSON;
3.	Create stage (internal or external)
CREATE STAGE json_stage FILE_FORMAT = json_ff;
4.	Upload JSON files to stage
PUT file://local/path/file1.json @json_stage;
5.	Load JSON into VARIANT column
COPY INTO raw_json_data (data)
FROM @json_stage
FILE_FORMAT = (TYPE = JSON)
ON_ERROR = 'CONTINUE';
✅ After this, each JSON object in the file is stored as one row in the data VARIANT column.
________________________________________
B. Loading JSON directly from a string or query (inline)
INSERT INTO raw_json_data (data)
VALUES (PARSE_JSON('{"name":"John","age":30,"address":{"city":"NY"}}'));
•	PARSE_JSON() converts a JSON string into VARIANT type
•	Useful for small inserts or API-based ingestion
________________________________________
3️⃣ Querying JSON in VARIANT Column
1.	Access top-level keys
SELECT data:name AS name, data:age AS age
FROM raw_json_data;
2.	Access nested objects
SELECT data:address.city AS city
FROM raw_json_data;
3.	Flatten arrays
SELECT f.value AS item
FROM raw_json_data,
LATERAL FLATTEN(input => data:items) f;
________________________________________
4️⃣ Loading JSON with COPY INTO from S3 (External Stage)
CREATE STAGE s3_json_stage
  URL = 's3://mybucket/jsonfiles/'
  STORAGE_INTEGRATION = s3_int
  FILE_FORMAT = (TYPE = JSON);

COPY INTO raw_json_data (data)
FROM @s3_json_stage;
•	Each JSON file object becomes a row in the table
•	Works with nested arrays and objects
________________________________________
5️⃣ Handling Semi-Structured Data Best Practices
•	Use VARIANT columns for raw data ingestion
•	Store raw JSON first, then transform to structured tables if needed
•	Use FLATTEN and LATERAL joins to normalize data
•	For large-scale pipelines, combine external stage + COPY INTO + VARIANT for high performance
•	Use Streams + Tasks for incremental JSON ingestion

-------------------
1️⃣ Sample JSON Data (Stored in VARIANT)
Assume we have JSON like this coming from an API:
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
________________________________________
2️⃣ Table with VARIANT Column
CREATE TABLE orders_raw (
    data VARIANT
);
Load the JSON into this table (via COPY INTO or INSERT).
________________________________________
3️⃣ Why FLATTEN Is Needed
•	JSON arrays (like items) cannot be queried as normal rows
•	FLATTEN explodes array elements into multiple rows
•	LATERAL allows FLATTEN to access each row’s JSON data
________________________________________
4️⃣ Basic FLATTEN Example (Array → Rows)
SELECT
    f.value:item_id::STRING AS item_id,
    f.value:product::STRING AS product,
    f.value:price::NUMBER  AS price
FROM orders_raw o,
     LATERAL FLATTEN(input => o.data:items) f;
🔎 What happens?
order	item_id	product	price
101	P1	Laptop	1200
101	P2	Mouse	25
Each element in the items array becomes one row.

________________________________________
5️⃣ Including Parent Fields (Very Common Interview Pattern)
SELECT
    o.data:order_id::INT             AS order_id,
    o.data:customer.name::STRING    AS customer_name,
    o.data:customer.country::STRING AS country,
    f.value:item_id::STRING          AS item_id,
    f.value:product::STRING          AS product,
    f.value:price::NUMBER            AS price
FROM orders_raw o,
     LATERAL FLATTEN(input => o.data:items) f;
This is how you denormalize JSON into relational format.
________________________________________
6️⃣ Using Explicit LATERAL JOIN Syntax
Equivalent and more readable:
SELECT
    o.data:order_id::INT AS order_id,
    f.value:product::STRING AS product
FROM orders_raw o
JOIN LATERAL FLATTEN(input => o.data:items) f;
________________________________________
7️⃣ FLATTEN with Nested Arrays (Advanced Example)
JSON with nested arrays:
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
Query:
SELECT
    o.data:user_id::INT AS user_id,
    s.value:session_id::STRING AS session_id,
    e.value::STRING AS event
FROM sessions_raw o,
     LATERAL FLATTEN(input => o.data:sessions) s,
     LATERAL FLATTEN(input => s.value:events) e;
________________________________________
8️⃣ Important FLATTEN Columns (Interview Favorite)
FLATTEN returns multiple metadata columns:
Column	Meaning
VALUE	Actual array element
INDEX	Position in array
PATH	JSON path
THIS	Full original object
Example:
SELECT
    f.index,
    f.value
FROM orders_raw o,
     LATERAL FLATTEN(input => o.data:items) f;
________________________________________
9️⃣ When to Use FLATTEN + LATERAL (Say This in Interview)
I use FLATTEN with LATERAL when dealing with JSON arrays in a VARIANT column.
FLATTEN explodes array elements into rows, and LATERAL allows the function to reference columns from the outer query row.
________________________________________
10️⃣ Real-World Use Case (Senior-Level Answer)
•	Load raw JSON into a VARIANT column
•	Use FLATTEN + LATERAL to normalize arrays
•	Insert into structured fact/dimension tables
•	Use Streams + Tasks for incremental processing
________________________________________
🎯 Interview-Ready One-Line Summary
FLATTEN is used to convert JSON arrays into multiple rows, and LATERAL allows the flatten operation to access each row’s VARIANT data in Snowflake.

How do you validate if a load was successful?
(Check LOAD_HISTORY, COPY_HISTORY, error reporting, etc.) 
1️⃣ Pre-Load Validation (Detect Corrupt Data Before Load)
🔹 Step 1: Validate File Structure (Dry Run)
Use VALIDATION_MODE to check files without loading data.
COPY INTO sales
FROM @s3_stage
FILE_FORMAT = (TYPE = CSV)
VALIDATION_MODE = 'RETURN_ERRORS';

✔ Detects:
•	Wrong delimiter
•	Missing columns
•	Invalid data types
•	Malformed rows
________________________________________
🔹 Step 2: Check File Presence & Size
LIST @s3_stage;
✔ Ensures:
•	Files exist
•	File sizes are reasonable
•	Correct folder path
________________________________________
2️⃣ During Load Validation (COPY INTO Controls)
🔹 Step 3: Use ON_ERROR Strategically
Option	Behavior
ABORT_STATEMENT	Stop load on first error
CONTINUE	Skip bad rows, load good ones
SKIP_FILE	Skip entire bad file
SKIP_FILE_n	Skip file after n errors
COPY INTO sales
FROM @s3_stage
ON_ERROR = 'CONTINUE';
✔ Allows partial loads with error tolerance
________________________________________
🔹 Step 4: Capture Rejected Records
COPY INTO sales
FROM @s3_stage
VALIDATION_MODE = 'RETURN_ALL_ERRORS';
✔ Returns detailed error messages per row
________________________________________
3️⃣ Post-Load Validation (Confirm Load Success)
🔹 Step 5: Check COPY History (Most Important)
SELECT *
FROM TABLE(
  INFORMATION_SCHEMA.COPY_HISTORY(
    TABLE_NAME => 'SALES',
    START_TIME => DATEADD('hour', -2, CURRENT_TIMESTAMP())
  )
);
✔ Shows:
•	Files loaded
•	Rows loaded
•	Rows rejected
•	Error counts
•	Load status
📌 Interview tip: This is your primary proof of load success.
________________________________________
🔹 Step 6: Reconcile Row Counts
Compare:
•	Source file rows
•	Target table rows
SELECT COUNT(*) FROM sales;
✔ Confirms completeness
________________________________________
🔹 Step 7: Check for NULLs & Invalid Values
SELECT COUNT(*)
FROM sales
WHERE amount IS NULL
   OR amount < 0;
✔ Detects corrupt or unexpected data
________________________________________
4️⃣ Validating Semi-Structured (JSON) Data
🔹 Step 8: Validate JSON Structure
SELECT
  data,
  IS_OBJECT(data) AS is_object
FROM raw_json
WHERE NOT IS_OBJECT(data);
✔ Detects malformed JSON
________________________________________
🔹 Step 9: Validate Required Keys
SELECT *
FROM raw_json
WHERE data:order_id IS NULL;
________________________________________
5️⃣ Using Load Metadata Tables
🔹 Step 10: Query Load History Views
SELECT *
FROM SNOWFLAKE.ACCOUNT_USAGE.LOAD_HISTORY
WHERE STATUS != 'LOADED';
✔ Detects failed or partial loads
________________________________________
6️⃣ Production-Grade Validation Workflow (Real-World)
Typical validation pipeline:
Stage → VALIDATION_MODE → COPY INTO
      → COPY_HISTORY
      → Row Count Check
      → Business Rule Checks
      → Alert / Retry
________________________________________
7️⃣ Handling Corrupt Data (What to Do)
If data is corrupt:
1.	Identify bad rows using VALIDATION_MODE
2.	Log rejected records to an error table
3.	Fix upstream data or apply transformation logic
4.	Reload only failed files (use FORCE = TRUE if needed)


✅ VALIDATION_MODE in Snowflake (All Types + Examples)
VALIDATION_MODE is used with COPY INTO to validate files without (or with limited) loading, helping detect corrupt data, format issues, and row-level errors.
________________________________________
🔹 Why VALIDATION_MODE Is Important
•	Prevents bad data from entering tables
•	Identifies corrupt rows and files early
•	Used heavily in production pipelines
•	Common interview topic
________________________________________
📌 VALIDATION_MODE Options
Snowflake supports three main validation modes:
1.	RETURN_ERRORS
2.	RETURN_ALL_ERRORS
3.	RETURN_n_ROWS
________________________________________
1️⃣ VALIDATION_MODE = RETURN_ERRORS
🔹 What it does
•	Validates files
•	Returns first error per file
•	Does not load any data
🔹 When to use
•	Quick validation before loading
•	Identifying file-level format issues
•	Faster than full validation
________________________________________
🔹 Example
COPY INTO sales
FROM @s3_stage
FILE_FORMAT = (TYPE = CSV)
VALIDATION_MODE = 'RETURN_ERRORS';
🔹 Sample Output
FILE	LINE	ERROR
sales_01.csv	15	Numeric value 'abc' not recognized
📌 Stops scanning file after first error
________________________________________
2️⃣ VALIDATION_MODE = RETURN_ALL_ERRORS ⭐ (Most Detailed)
🔹 What it does
•	Validates entire file
•	Returns all row-level errors
•	Does not load any data
🔹 When to use
•	Deep data quality checks
•	Debugging corrupt files
•	Production issue analysis
________________________________________
🔹 Example
COPY INTO sales
FROM @s3_stage
FILE_FORMAT = (TYPE = CSV)
VALIDATION_MODE = 'RETURN_ALL_ERRORS';
🔹 Sample Output
FILE	LINE	ERROR
sales_01.csv	15	Invalid number
sales_01.csv	22	Date parsing error
sales_02.csv	9	Missing column
📌 Slower but extremely useful
________________________________________
3️⃣ VALIDATION_MODE = RETURN_n_ROWS
🔹 What it does
•	Validates first N rows
•	Returns errors only from those rows
•	Does not load any data
🔹 When to use
•	Large files (GBs/TBs)
•	Quick sampling
•	Performance-sensitive checks
________________________________________
🔹 Example
COPY INTO sales
FROM @s3_stage
FILE_FORMAT = (TYPE = CSV)
VALIDATION_MODE = 'RETURN_100_ROWS';
📌 Only checks first 100 rows
________________________________________
🆚 Validation Modes Comparison
Mode	Loads Data	Errors Returned	Use Case
RETURN_ERRORS	❌ No	First error per file	Quick check
RETURN_ALL_ERRORS	❌ No	All errors	Deep debugging
RETURN_n_ROWS	❌ No	Errors in first N rows	Large files



What happens when Snowflake skips files during a load?
(e.g., it won’t reload data already ingested unless forced) 
1️⃣ Why Snowflake Skips Files
Snowflake maintains load metadata that records:
•	File name
•	File path
•	Target table
•	Load status (loaded / partially loaded / failed)
•	Timestamp
If Snowflake detects:
•	The same file name
•	From the same stage/location
•	Loaded into the same target table
➡️ It skips the file automatically.
________________________________________
2️⃣ What “Skipped” Means (Important)
When a file is skipped:
•	❌ No rows are read
•	❌ No data is reloaded
•	❌ No additional credits are consumed
•	✅ Load command completes successfully
The file is treated as already ingested.
________________________________________
3️⃣ Where Snowflake Tracks This Information
You can see skipped files in:
SELECT *
FROM TABLE(
  INFORMATION_SCHEMA.COPY_HISTORY(
    TABLE_NAME => 'SALES',
    START_TIME => DATEADD('hour', -24, CURRENT_TIMESTAMP())
  )
);
Look for:
•	STATUS = 'SKIPPED'
•	ERROR_COUNT = 0
________________________________________
4️⃣ Common Scenarios Where Files Get Skipped
🔹 Scenario 1: Re-running COPY INTO
COPY INTO sales
FROM @s3_stage;
➡️ Previously loaded files are skipped.
________________________________________
🔹 Scenario 2: Pipeline Retry After Partial Failure
•	Successfully loaded files → skipped
•	Failed files → retried
This makes pipelines idempotent by default.
________________________________________
🔹 Scenario 3: Snowpipe Auto-Ingest
Snowpipe also tracks files and won’t ingest the same file twice.
________________________________________
5️⃣ How Long Snowflake Remembers Loaded Files
•	Snowflake retains load metadata for 64 days
•	After that period, files may be eligible for reload
📌 This is an important interview detail.
________________________________________
6️⃣ How to Force Reload a File (Override Skip Behavior)
🔹 Option 1: Use FORCE = TRUE
COPY INTO sales
FROM @s3_stage
FORCE = TRUE;
➡️ Reloads all matching files, even if already loaded.
⚠️ Risk: Duplicate data if not handled carefully.
________________________________________
🔹 Option 2: Rename or Move the File
•	Change file name or path in cloud storage
•	Snowflake treats it as a new file
________________________________________
🔹 Option 3: Load into a Different Table
File tracking is per table.
________________________________________
7️⃣ Best Practices to Avoid Issues
✔ Use staging tables
✔ Deduplicate data using MERGE or business keys
✔ Avoid FORCE = TRUE in production unless necessary
✔ Monitor skipped files using COPY_HISTORY
✔ Use Snowpipe for event-driven, exactly-once ingestion

