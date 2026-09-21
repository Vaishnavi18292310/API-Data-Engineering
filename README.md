# Mini API ETL Pipeline

A beginner-friendly **Data Engineering ETL pipeline** that extracts data from a REST API, transforms and validates the JSON response using Python and Pandas, logs pipeline activity, and loads the processed data into MySQL for storage and verification.

## 📌 Project Overview

This project demonstrates a complete **API → ETL → MySQL** workflow using the JSONPlaceholder REST API.

The pipeline takes nested JSON data from the API and converts it into a structured relational dataset suitable for storage and SQL-based analysis.

### Pipeline Architecture

```text
JSONPlaceholder REST API
          │
          ▼
      Extraction
          │
          ▼
   JSON Normalization
          │
          ▼
     Transformation
          │
          ▼
      Data Validation
          │
          ▼
        Logging
          │
          ▼
      MySQL Database
          │
          ▼
     SQL Verification
```

## 🎯 Objectives

* Extract data from a REST API using Python
* Handle nested JSON responses
* Normalize JSON data into a tabular structure
* Select relevant fields for downstream processing
* Convert data types appropriately
* Validate data quality before loading
* Implement pipeline logging
* Store processed data in a MySQL relational database
* Verify the loaded data using SQL queries

## 🛠️ Technologies Used

| Technology             | Purpose                                |
| ---------------------- | -------------------------------------- |
| Python                 | ETL development                        |
| Requests               | API extraction                         |
| Pandas                 | Data transformation and processing     |
| MySQL                  | Relational data storage                |
| MySQL Connector/Python | Python–MySQL connectivity              |
| Logging                | Pipeline monitoring and error tracking |
| Jupyter Notebook       | Development and execution              |
| SQL                    | Data loading and verification          |

## 🔄 ETL Workflow

### 1. Extract

Data is retrieved from the JSONPlaceholder REST API using the Python `requests` library.

```python
response = requests.get(url, timeout=10)
response.raise_for_status()

data = response.json()
```

The API returns user information containing nested structures such as:

* Address
* Geographic coordinates
* Company information

---

### 2. Transform

The nested JSON response is normalized using Pandas:

```python
df_flat = pd.json_normalize(data)
```

Relevant fields are then selected and renamed:

```text
address.city      → city
address.zipcode   → zipcode
address.geo.lat   → latitude
address.geo.lng   → longitude
company.name      → company_name
```

Latitude and longitude values are converted from strings to numeric data types for easier validation and analysis.

### Final Dataset

The processed dataset contains:

```text
id
name
username
email
city
zipcode
latitude
longitude
company_name
```

---

### 3. Data Validation

Before loading the data into MySQL, several data quality checks are performed.

#### Duplicate ID Validation

```python
duplicated_ids = selected_cols["id"].duplicated().sum()
```

Ensures customer IDs are unique.

#### Required Field Validation

```python
missing_values = selected_cols[
    ["id", "name", "email"]
].isnull().sum().sum()
```

Checks for missing values in required fields.

#### Latitude Validation

```python
invalid_latitude = selected_cols[
    (selected_cols["latitude"] < -90) |
    (selected_cols["latitude"] > 90)
]
```

Ensures latitude values fall within the valid geographic range.

#### Longitude Validation

```python
invalid_longitude = selected_cols[
    (selected_cols["longitude"] < -180) |
    (selected_cols["longitude"] > 180)
]
```

Ensures longitude values fall within the valid geographic range.

### Validation Result

The processed dataset passed all validation checks:

```text
Duplicate IDs: 0
Missing required values: 0
Invalid latitude records: 0
Invalid longitude records: 0
```

---

### 4. Logging

Python's `logging` module is used to record pipeline activity.

Example:

```python
logging.info(f"Duplicate IDs: {duplicated_ids}")
logging.info(f"Missing required values: {missing_values}")
logging.info(f"Invalid latitude records: {len(invalid_latitude)}")
logging.info(f"Invalid longitude records: {len(invalid_longitude)}")
```

The pipeline stores logs in:

```text
pipeline.log
```

This provides a basic mechanism for monitoring pipeline execution and identifying data-quality issues.

---

### 5. Load → MySQL

A dedicated MySQL database is created for the project:

```text
api_etl_db
```

The processed data is loaded into the:

```text
customers
```

table.

### Table Schema

| Column       | MySQL Type   | Description                |
| ------------ | ------------ | -------------------------- |
| id           | INT          | Unique customer identifier |
| name         | VARCHAR(100) | Customer name              |
| username     | VARCHAR(100) | Username                   |
| email        | VARCHAR(150) | Email address              |
| city         | VARCHAR(100) | Customer city              |
| zipcode      | VARCHAR(20)  | Postal code                |
| latitude     | FLOAT        | Geographic latitude        |
| longitude    | FLOAT        | Geographic longitude       |
| company_name | VARCHAR(150) | Company name               |

The `id` column is defined as the **PRIMARY KEY** to enforce uniqueness.

Parameterized SQL is used for inserting records:

```python
query = """
INSERT INTO customers
(id, name, username, email, city, zipcode,
 latitude, longitude, company_name)
VALUES (%s, %s, %s, %s, %s, %s, %s, %s, %s)
"""
```

Changes are persisted using:

```python
connection.commit()
```

---

### 6. Data Verification

After loading, the data is queried directly from MySQL to verify successful ingestion.

```sql
SELECT *
FROM customers;
```

The pipeline successfully loaded and verified **10 customer records**.

An additional SQL aggregation was performed:

```sql
SELECT city, COUNT(*) AS customer_count
FROM customers
GROUP BY city;
```

This demonstrates that the loaded data can be queried and analyzed using SQL after ingestion.

## 📊 Project Results

| Metric                    |      Result |
| ------------------------- | ----------: |
| API records extracted     |          10 |
| Final records loaded      |          10 |
| Duplicate IDs             |           0 |
| Missing required values   |           0 |
| Invalid latitude records  |           0 |
| Invalid longitude records |           0 |
| MySQL table               | `customers` |

## 📁 Project Structure

```text
Mini_API_ETL_Project/
│
├── mini_api_etl_project.ipynb
├── pipeline.log
└── README.md
```

> `pipeline.log` contains runtime logging generated during pipeline execution.

## 🔐 Security

No API key is required for the JSONPlaceholder API.

Database credentials should **not** be committed to GitHub.

For example, avoid pushing code containing:

```python
password = "my_actual_password"
```

For production-style projects, credentials should be stored using environment variables or a secrets-management solution.

## 🚀 Key Data Engineering Concepts Demonstrated

* REST API consumption
* HTTP requests
* JSON handling
* Nested JSON normalization
* Data transformation
* Data type conversion
* Data quality validation
* Duplicate detection
* Required-field validation
* Logging
* Relational database design
* MySQL connectivity
* Parameterized SQL
* Transaction commit
* SQL verification
* API-to-database ETL workflow

## 🔮 Future Improvements

Possible extensions to make the pipeline more production-oriented:

* Add API pagination handling
* Implement retry logic and exponential backoff
* Add incremental API extraction
* Introduce automated data-quality reporting
* Add duplicate-safe/upsert loading
* Separate extraction, transformation, validation, and loading into Python modules
* Use environment variables for database credentials
* Add automated tests
* Containerize the pipeline using Docker
* Schedule the pipeline using Apache Airflow
* Store raw API responses in cloud storage such as Amazon S3
* Extend the pipeline to a cloud data warehouse such as Snowflake

## 👩‍💻 Author

**Vaishnavi K**

Aspiring Data Engineer focused on Python, SQL, ETL, cloud technologies, and data engineering workflows.
