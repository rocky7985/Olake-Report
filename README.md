# OLake Data Pipeline with PostgreSQL, Apache Iceberg & Spark SQL

This project demonstrates how to set up a local data pipeline using PostgreSQL (via Docker), Apache Iceberg, and Spark SQL to create and query Iceberg tables backed by a JDBC catalog.

---

## ✅ Setup Steps

### 1. Prerequisites Installed

- [x] **Docker & Docker Compose**
- [x] **PostgreSQL CLI (`psql`)**
- [x] **Java 8+**
- [x] **Apache Spark 3.4+ (used Spark 3.4.0)**
- [x] **Python 3**

---

### 2. PostgreSQL via Docker

A `docker-compose.yml` was used to spin up PostgreSQL:

```yaml
version: '3.8'
services:
  postgres:
    image: postgres:15
    container_name: olake_postgres
    restart: always
    ports:
      - "5432:5432"
    environment:
      POSTGRES_USER: olake_user
      POSTGRES_PASSWORD: olake_pass
      POSTGRES_DB: olake
    volumes:
      - pgdata:/var/lib/postgresql/data
volumes:
  pgdata:
To start PostgreSQL:

bash
Copy
Edit
docker-compose up -d
3. Create Sample Table & Insert Data
Logged into PostgreSQL:

bash
Copy
Edit
psql -h localhost -U olake_user -d olake
Created table:

sql
Copy
Edit
CREATE TABLE orders (
  id INT PRIMARY KEY,
  product VARCHAR(100),
  quantity INT
);
4. Spark SQL Setup with Iceberg JDBC Catalog
Ran Spark SQL with Iceberg runtime:

bash
Copy
Edit
spark-sql ^
  --packages org.apache.iceberg:iceberg-spark-runtime-3.3_2.12:1.4.2 ^
  --conf spark.sql.catalog.postgres_catalog=org.apache.iceberg.spark.SparkCatalog ^
  --conf spark.sql.catalog.postgres_catalog.catalog-impl=org.apache.iceberg.jdbc.JdbcCatalog ^
  --conf spark.sql.catalog.postgres_catalog.uri=jdbc:postgresql://host.docker.internal:5432/olake ^
  --conf spark.sql.catalog.postgres_catalog.jdbc.user=olake_user ^
  --conf spark.sql.catalog.postgres_catalog.jdbc.password=olake_pass ^
  --conf spark.sql.catalog.postgres_catalog.warehouse=/mnt/warehouse
Created Iceberg table and inserted 10 records:

sql
Copy
Edit
CREATE TABLE postgres_catalog.default.orders (
  id INT,
  product STRING,
  quantity INT
)
USING iceberg;

INSERT INTO postgres_catalog.default.orders VALUES 
  (1, 'Laptop', 2),
  (2, 'Mouse', 5),
  (3, 'Keyboard', 3),
  (4, 'Monitor', 1),
  (5, 'Headphones', 4),
  (6, 'Webcam', 2),
  (7, 'USB Cable', 10),
  (8, 'Charger', 6),
  (9, 'Tablet', 2),
  (10, 'Phone', 3);

SELECT * FROM postgres_catalog.default.orders;
📸 Screenshots
1. Spark SQL

    ![Screenshot 2025-06-04 184042](https://github.com/user-attachments/assets/db18d269-d83e-4fff-bc0e-9c8200ab5efe)


❗ Challenges Faced
Class Compatibility Issue: Spark wasn't recognizing SparkCatalog until I added the correct Iceberg runtime using --packages.

Java Version Conflicts: Faced UnsupportedClassVersionError due to JDK 23; switching to JDK 17 resolved it.

Docker Networking: Used host.docker.internal for JDBC URI inside Spark CLI on Windows.

💡 Improvements Suggested
Automate setup using a shell script or Makefile.

Use a pre-bundled Spark Docker image to avoid manual local Spark setup.

Integrate Apache Superset or similar for data visualization on Iceberg tables.


✅ Status
All components successfully integrated:

PostgreSQL via Docker ✅

Iceberg JDBC Catalog setup ✅

Spark SQL query on Iceberg table ✅
