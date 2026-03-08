## Azure Data Factory – Basic Project with Code Snippets

![Image](https://learn.microsoft.com/en-us/azure/data-factory/media/copy-activity-performance/adf-copy-scalability.png)

![Image](https://learn.microsoft.com/en-us/azure/data-factory/media/author-visually/properties-pane.png)

![Image](https://miro.medium.com/v2/resize%3Afit%3A1400/1%2AVO3hSPUcXorXwPxig_YsUQ.png)

![Image](https://miro.medium.com/1%2A5OmJt9YY8VYiXmcTpfcUAw.jpeg)

Below is a **simple beginner project** demonstrating how to use **Azure Data Factory** to move data from a database to storage.

This kind of project is useful for **cloud / data engineering training, interviews, and practical labs**.

---

# Basic Project

## Project Name

**ADF ETL Pipeline – SQL Database → Blob Storage**

### Architecture

```
Azure SQL Database
        ↓
Azure Data Factory Pipeline
        ↓
Copy Activity
        ↓
Azure Blob Storage
```

Destination file format: **CSV / Parquet**

---

# Step 1 — Create Resource Group

### Azure CLI

```bash
az group create \
--name adf-demo-rg \
--location eastus
```

---

# Step 2 — Create Storage Account

This will store exported data.

```bash
az storage account create \
--name adfdemostorage986 \
--resource-group adf-demo-rg \
--location eastus \
--sku Standard_LRS
```

---

# Step 3 — Create Azure Data Factory

```bash
az datafactory create \
--factory-name adf-demo-project986 \
--resource-group adf-demo-rg \
--location eastus
```

---

# Step 4 — Create SQL Database (Source)

```bash
az sql server create \
--name adf-sql-server-demo \
--resource-group adf-demo-rg \
--location eastus \
--admin-user azureuser \
--admin-password Password1234!
```

Create database:

```bash
az sql db create \
--resource-group adf-demo-rg \
--server adf-sql-server-demo \
--name salesdb \
--service-objective S0
```

---

# Step 5 — Create Sample Table

Connect using SQL client.

```sql
CREATE TABLE customers (
    id INT,
    name VARCHAR(50),
    city VARCHAR(50)
);

INSERT INTO customers VALUES
(1,'Atul','Pune'),
(2,'Rahul','Mumbai'),
(3,'Neha','Delhi');
```

---

# Step 6 — Create Linked Services

Linked services connect ADF to external systems.

Example JSON for **SQL Database Linked Service**

```json
{
  "name": "AzureSqlLinkedService",
  "properties": {
    "type": "AzureSqlDatabase",
    "typeProperties": {
      "connectionString": "Server=tcp:adf-sql-server-demo.database.windows.net;Database=salesdb;User ID=azureuser;Password=Password1234!"
    }
  }
}
```

---

Example **Blob Storage Linked Service**

```json
{
  "name": "BlobStorageLinkedService",
  "properties": {
    "type": "AzureBlobStorage",
    "typeProperties": {
      "connectionString": "DefaultEndpointsProtocol=https;AccountName=adfdemostorage123;"
    }
  }
}
```

---

# Step 7 — Create Dataset

Dataset represents data inside a source.

### SQL Dataset

```json
{
  "name": "CustomerTableDataset",
  "properties": {
    "linkedServiceName": {
      "referenceName": "AzureSqlLinkedService",
      "type": "LinkedServiceReference"
    },
    "type": "AzureSqlTable",
    "typeProperties": {
      "tableName": "customers"
    }
  }
}
```

---

### Blob Dataset

```json
{
  "name": "BlobOutputDataset",
  "properties": {
    "linkedServiceName": {
      "referenceName": "BlobStorageLinkedService",
      "type": "LinkedServiceReference"
    },
    "type": "DelimitedText",
    "typeProperties": {
      "fileName": "customers.csv",
      "columnDelimiter": ","
    }
  }
}
```

---

# Step 8 — Create Pipeline

Pipeline with **Copy Activity**

```json
{
  "name": "CopyCustomersPipeline",
  "properties": {
    "activities": [
      {
        "name": "CopySQLToBlob",
        "type": "Copy",
        "inputs": [
          {
            "referenceName": "CustomerTableDataset",
            "type": "DatasetReference"
          }
        ],
        "outputs": [
          {
            "referenceName": "BlobOutputDataset",
            "type": "DatasetReference"
          }
        ],
        "typeProperties": {
          "source": {
            "type": "AzureSqlSource"
          },
          "sink": {
            "type": "BlobSink"
          }
        }
      }
    ]
  }
}
```

---

# Step 9 — Run Pipeline

Run manually in ADF Studio.

Or trigger using REST API.

Example:

```bash
az datafactory pipeline create-run \
--factory-name adf-demo-project \
--resource-group adf-demo-rg \
--name CopyCustomersPipeline
```

---

# Step 10 — Output Result

After pipeline runs:

```
Blob Storage
   └── customers.csv
```

File contents:

```
id,name,city
1,Atul,Pune
2,Rahul,Mumbai
3,Neha,Delhi
```

---

# Real-World Use Case

Typical enterprise pipeline:

```
On-Prem SQL Server
        ↓
Azure Data Factory
        ↓
Data Transformation
        ↓
Azure Data Lake
        ↓
Azure Synapse
        ↓
Power BI Dashboard
```

Uses services like:

* Azure Synapse Analytics
* Azure Data Lake Storage
* Power BI

---

# Interview Questions (Very Common)

**Q1:** What is Azure Data Factory?
A cloud ETL service used to **create and orchestrate data pipelines**.

**Q2:** What is a pipeline?
A logical group of activities performing a task.

**Q3:** What is Integration Runtime?
The compute infrastructure used to **execute data movement and transformations**.

**Q4:** What is a dataset?
A representation of **data inside a data store**.

**Q5:** Difference between Copy Activity and Data Flow?

| Feature | Copy Activity       | Data Flow      |
| ------- | ------------------- | -------------- |
| Purpose | Move data           | Transform data |
| Compute | Integration Runtime | Spark          |

---
