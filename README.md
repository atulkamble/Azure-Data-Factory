Below is a full Azure Data Factory (ADF) project with a **sample dataset**, code, and detailed steps for creating a pipeline to move data from Azure Blob Storage to an Azure SQL Database, including transformations.

---

## **Objective**
Demonstrate a complete Azure Data Factory (ADF) ETL pipeline that:
- Extracts data from Azure Blob Storage (CSV format)
- Transforms data with validation and enrichment
- Loads data into Azure SQL Database
- Implements monitoring, error handling, and security best practices
- Showcases modern ADF features like Data Flows and pipeline orchestration

---

## **Sample Dataset**
Save this dataset as a `sample_data.csv` file and upload it to an Azure Blob Storage container.

```csv
EmployeeID,Name,Department,Salary,HireDate,IsActive,ManagerID
101,John Doe,Engineering,60000,2020-01-15,true,201
102,Jane Smith,Marketing,55000,2021-03-22,true,202
103,Michael Brown,Sales,45000,2019-11-08,true,203
104,Linda White,HR,50000,2020-07-12,true,204
105,Robert Johnson,Engineering,65000,2018-05-03,false,201
106,Sarah Davis,Marketing,52000,2022-01-10,true,202
```

---

## **Steps to Create the ADF Project**

### **Step 1: Prerequisites**
1. **Azure Subscription**: Ensure you have access.
2. **Azure Blob Storage**: Create a storage account, container, and upload the `sample_data.csv` file.
3. **Azure SQL Database**:
   - Create a table for the data:
     ```sql
     CREATE TABLE Employee (
         EmployeeID INT PRIMARY KEY,
         Name NVARCHAR(100) NOT NULL,
         Department NVARCHAR(50) NOT NULL,
         Salary DECIMAL(10,2) NOT NULL,
         HireDate DATE NOT NULL,
         IsActive BIT NOT NULL DEFAULT 1,
         ManagerID INT NULL,
         CreatedDate DATETIME2 DEFAULT GETUTCDATE(),
         ModifiedDate DATETIME2 DEFAULT GETUTCDATE()
     );
     
     -- Create index for better query performance
     CREATE NONCLUSTERED INDEX IX_Employee_Department ON Employee(Department);
     CREATE NONCLUSTERED INDEX IX_Employee_ManagerID ON Employee(ManagerID);
     ```
4. Install **Azure Data Factory Studio** (available via the Azure Portal).
5. https://learn.microsoft.com/en-us/azure-data-studio/download-azure-data-studio?tabs=win-install%2Cwin-user-install%2Credhat-install%2Cwindows-uninstall%2Credhat-uninstall#download-azure-data-studio

---

### **Step 2: Create the Data Factory**
1. Navigate to the **Azure Portal**.
2. Search for **Data Factory** and click **Create**.
3. Fill in:
   - **Subscription**: Select your subscription.
   - **Resource Group**: Create or select one.
   - **Region**: Choose a nearby region.
   - **Data Factory Name**: Provide a unique name.
4. Click **Review + Create**, then **Create**.
5. **Security Configuration**:
   - Enable **Managed Identity** for the Data Factory
   - Configure **Azure Key Vault** for storing connection strings securely
   - Set up **Private Endpoints** if required for enhanced security

---

### **Step 2.5: Security Best Practices**
1. **Use Managed Identity**: Enable system-assigned managed identity for ADF
2. **Key Vault Integration**: Store sensitive information in Azure Key Vault
3. **Network Security**: Configure firewall rules and private endpoints
4. **RBAC**: Assign appropriate roles (Data Factory Contributor, etc.)
5. **Monitoring**: Enable diagnostic logging and alerts

---

### **Step 3: Create Linked Services**
Linked Services are used to connect ADF to external resources.

#### 1. Azure Key Vault Linked Service (Recommended First)
1. In ADF Studio, go to **Manage > Linked Services**.
2. Click **+ New** and select **Azure Key Vault**.
3. Configure:
   - **Azure subscription**: Select your subscription
   - **Azure key vault name**: Select your Key Vault
   - **Authentication method**: Use **Managed Identity**
4. Test the connection and save as `AzureKeyVaultLS`.

#### 2. Blob Storage Linked Service
1. Click **+ New** and select **Azure Blob Storage**.
2. Configure:
   - **Account Selection Method**: From Azure subscription
   - **Storage Account Name**: Select your storage account
   - **Authentication method**: Use **Managed Identity** (recommended)
3. Test the connection and save as `AzureBlobStorageLS`.

#### 3. Azure SQL Database Linked Service
1. Create another Linked Service for Azure SQL Database.
2. Configure:
   - **Server Name**: Enter the server FQDN
   - **Database Name**: Enter the database name
   - **Authentication Type**: **Azure SQL Database authentication**
   - **Username**: Reference from Key Vault: `@{linkedService().AzureKeyVaultLS.secretName('sql-username')}`
   - **Password**: Reference from Key Vault: `@{linkedService().AzureKeyVaultLS.secretName('sql-password')}`
3. Test the connection and save as `AzureSQLDatabaseLS`.

---

### **Step 4: Create Datasets**
Datasets represent the data structure in the source and destination.

#### 1. Blob Dataset (Source)
1. Go to **Author > Datasets**, click **+ New Dataset**.
2. Select **Azure Blob Storage** and **DelimitedText**.
3. Configure:
   - **Linked Service**: Select `AzureBlobStorageLS`
   - **File Path**: Container: `data-container`, File: `sample_data.csv`
   - **Column delimiter**: Comma (,)
   - **Row delimiter**: Default (\r\n or \n)
   - Enable **First Row as Header**
   - **Schema**: Import schema to validate data structure
4. **Preview Data** to ensure correct parsing
5. Save as `BlobInputDataset`.

#### 2. SQL Dataset (Sink)
1. Add another dataset for the Azure SQL Database.
2. Select **Azure SQL Database** and configure:
   - **Linked Service**: Select `AzureSQLDatabaseLS`
   - **Table Name**: `dbo.Employee` (include schema prefix)
   - **Import Schema**: Yes, to validate table structure
3. **Test Connection** to ensure table accessibility
4. Save as `SQLSinkDataset`.

---

### **Step 5: Create the Pipeline with Data Transformation**

#### Option A: Simple Copy Pipeline
1. In **Author > Pipelines**, click **+ New Pipeline**.
2. Name the pipeline `EmployeeDataProcessing`.
3. Add **Variables** for dynamic configuration:
   - `SourceContainer`: String, default value: "data-container"
   - `ProcessedDate`: String, default value: `@utcnow()`

#### Add Copy Data Activity:
1. Drag and drop the **Copy Data** activity onto the canvas.
2. Name it `CopyEmployeeData`.
3. Configure the **Source** tab:
   - Dataset: Select `BlobInputDataset`
   - **Data Consistency Verification**: Enable checksum validation
4. Configure the **Sink** tab:
   - Dataset: Select `SQLSinkDataset`
   - **Write Behavior**: Upsert (recommended for data updates)
   - **Key Columns**: EmployeeID
5. Configure **Mapping**:
   - Auto-map columns or manually map:
     - `EmployeeID → EmployeeID`
     - `Name → Name`
     - `Department → Department`
     - `Salary → Salary`
     - `HireDate → HireDate`
     - `IsActive → IsActive`
     - `ManagerID → ManagerID`

#### Option B: Enhanced Pipeline with Data Flow
1. Create a **Mapping Data Flow** for advanced transformations:
   - Add **Data Flow** activity to pipeline
   - Create transformations: Filter active employees, validate data, add audit columns

#### Add Error Handling:
1. **Validation Activity**: Add before Copy Data
   - Validate source file exists and has data
   - Check SQL Database connectivity
2. **Try-Catch Logic**:
   - Wrap Copy Data in **Try** block
   - Add **Catch** block with:
     - **Web Activity** to send failure notifications
     - **Stored Procedure** to log errors

---

### **Step 6: Data Quality and Validation**
1. **Add Data Validation**:
   - **Lookup Activity**: Count source records
   - **If Condition**: Proceed only if records > 0
   - **Data Flow**: Add data quality checks (null validation, format validation)

2. **Add Audit Logging**:
   - Create audit table in SQL Database
   - Log pipeline start/end times, record counts, status

---

### **Step 7: Debug and Run**
1. **Pipeline Validation**: Click **Validate All** to check for errors
2. **Debug Mode**: Click **Debug** to test the pipeline
3. **Monitor Progress**: Watch the **Output** window and activity details
4. **Data Verification**: Query destination table to verify data accuracy

---

### **Step 8: Publish and Trigger**
1. **Pre-Publish Validation**:
   - Run **Validate All** to check for any issues
   - Test in Debug mode one final time

2. **Publish**: Click **Publish All** to save changes to the service

3. **Create Triggers**:
   
   **Manual Trigger**:
   - Use **Trigger Now** for ad-hoc runs
   
   **Schedule Trigger**:
   ```json
   {
     "name": "DailyEmployeeSync",
     "type": "ScheduleTrigger",
     "typeProperties": {
       "recurrence": {
         "frequency": "Day",
         "interval": 1,
         "startTime": "2024-01-01T06:00:00Z",
         "timeZone": "UTC"
       }
     }
   }
   ```
   
   **Event-Based Trigger** (when new file uploaded):
   ```json
   {
     "name": "FileUploadTrigger",
     "type": "BlobEventsTrigger",
     "typeProperties": {
       "blobPathBeginsWith": "/data-container/employee",
       "blobPathEndsWith": ".csv",
       "ignoreEmptyBlobs": true,
       "events": ["Microsoft.Storage.BlobCreated"]
     }
   }
   ```

---

## **Code Snippets**

### Enhanced Pipeline JSON
```json
{
  "name": "EmployeeDataProcessing",
  "properties": {
    "parameters": {
      "SourceContainer": {
        "type": "string",
        "defaultValue": "data-container"
      },
      "ProcessDate": {
        "type": "string",
        "defaultValue": "@utcnow('yyyy-MM-dd')"
      }
    },
    "variables": {
      "RecordCount": {
        "type": "Integer",
        "defaultValue": 0
      }
    },
    "activities": [
      {
        "name": "ValidateSourceFile",
        "type": "Validation",
        "typeProperties": {
          "dataset": {
            "referenceName": "BlobInputDataset",
            "type": "DatasetReference"
          },
          "timeout": "0.00:05:00",
          "sleep": 30,
          "minimumSize": 1
        }
      },
      {
        "name": "CopyEmployeeData",
        "type": "Copy",
        "dependsOn": [
          {
            "activity": "ValidateSourceFile",
            "dependencyConditions": ["Succeeded"]
          }
        ],
        "typeProperties": {
          "source": {
            "type": "DelimitedTextSource",
            "storeSettings": {
              "type": "AzureBlobStorageReadSettings",
              "recursive": false,
              "enablePartitionDiscovery": false
            },
            "formatSettings": {
              "type": "DelimitedTextReadSettings",
              "skipLineCount": 0
            }
          },
          "sink": {
            "type": "AzureSqlSink",
            "preCopyScript": "TRUNCATE TABLE dbo.Employee_Staging",
            "writeBehavior": "upsert",
            "upsertSettings": {
              "useTempDB": true,
              "keys": ["EmployeeID"]
            },
            "sqlWriterUseTableLock": false,
            "tableOption": "autoCreate"
          },
          "enableStaging": false,
          "dataIntegrationUnits": 4
        },
        "inputs": [
          {
            "referenceName": "BlobInputDataset",
            "type": "DatasetReference"
          }
        ],
        "outputs": [
          {
            "referenceName": "SQLSinkDataset",
            "type": "DatasetReference"
          }
        ]
      },
      {
        "name": "LogPipelineExecution",
        "type": "SqlServerStoredProcedure",
        "dependsOn": [
          {
            "activity": "CopyEmployeeData",
            "dependencyConditions": ["Succeeded"]
          }
        ],
        "typeProperties": {
          "storedProcedureName": "sp_LogPipelineExecution",
          "storedProcedureParameters": {
            "PipelineName": {
              "value": "@pipeline().Pipeline",
              "type": "String"
            },
            "RunId": {
              "value": "@pipeline().RunId",
              "type": "String"
            },
            "RecordsProcessed": {
              "value": "@activity('CopyEmployeeData').output.rowsCopied",
              "type": "Int32"
            },
            "Status": {
              "value": "Success",
              "type": "String"
            }
          }
        },
        "linkedServiceName": {
          "referenceName": "AzureSQLDatabaseLS",
          "type": "LinkedServiceReference"
        }
      }
    ]
  }
}
```

### Enhanced Blob Dataset JSON
```json
{
  "name": "BlobInputDataset",
  "properties": {
    "linkedServiceName": {
      "referenceName": "AzureBlobStorageLS",
      "type": "LinkedServiceReference"
    },
    "parameters": {
      "Container": {
        "type": "string",
        "defaultValue": "data-container"
      },
      "FileName": {
        "type": "string",
        "defaultValue": "sample_data.csv"
      }
    },
    "type": "DelimitedText",
    "typeProperties": {
      "location": {
        "type": "AzureBlobStorageLocation",
        "container": {
          "value": "@dataset().Container",
          "type": "Expression"
        },
        "fileName": {
          "value": "@dataset().FileName",
          "type": "Expression"
        }
      },
      "columnDelimiter": ",",
      "rowDelimiter": "\n",
      "firstRowAsHeader": true,
      "quoteChar": "\"",
      "escapeChar": "\\"
    },
    "schema": [
      {
        "name": "EmployeeID",
        "type": "String"
      },
      {
        "name": "Name",
        "type": "String"
      },
      {
        "name": "Department",
        "type": "String"
      },
      {
        "name": "Salary",
        "type": "String"
      },
      {
        "name": "HireDate",
        "type": "String"
      },
      {
        "name": "IsActive",
        "type": "String"
      },
      {
        "name": "ManagerID",
        "type": "String"
      }
    ]
  }
}
```

### Enhanced SQL Dataset JSON
```json
{
  "name": "SQLSinkDataset",
  "properties": {
    "linkedServiceName": {
      "referenceName": "AzureSQLDatabaseLS",
      "type": "LinkedServiceReference"
    },
    "parameters": {
      "TableName": {
        "type": "string",
        "defaultValue": "Employee"
      },
      "SchemaName": {
        "type": "string",
        "defaultValue": "dbo"
      }
    },
    "type": "AzureSqlTable",
    "typeProperties": {
      "schema": {
        "value": "@dataset().SchemaName",
        "type": "Expression"
      },
      "table": {
        "value": "@dataset().TableName",
        "type": "Expression"
      }
    },
    "schema": [
      {
        "name": "EmployeeID",
        "type": "int",
        "precision": 10
      },
      {
        "name": "Name",
        "type": "nvarchar",
        "precision": 100
      },
      {
        "name": "Department",
        "type": "nvarchar",
        "precision": 50
      },
      {
        "name": "Salary",
        "type": "decimal",
        "precision": 10,
        "scale": 2
      },
      {
        "name": "HireDate",
        "type": "date"
      },
      {
        "name": "IsActive",
        "type": "bit"
      },
      {
        "name": "ManagerID",
        "type": "int",
        "precision": 10
      },
      {
        "name": "CreatedDate",
        "type": "datetime2",
        "scale": 7
      },
      {
        "name": "ModifiedDate",
        "type": "datetime2",
        "scale": 7
      }
    ]
  }
}
```

---

### **Step 9: Monitoring and Alerting**

#### Set up Monitoring:
1. **Azure Monitor Integration**:
   - Enable diagnostic settings for Data Factory
   - Configure Log Analytics workspace
   - Set up custom dashboards

2. **Alert Rules**:
   ```json
   {
     "alertName": "Pipeline Failure Alert",
     "condition": "PipelineRuns | where Status == 'Failed'",
     "threshold": "Greater than 0",
     "actionGroup": "DataFactoryAlerts"
   }
   ```

3. **Performance Monitoring**:
   - Track pipeline duration trends
   - Monitor data throughput
   - Set up cost alerts

#### Troubleshooting Guide:
1. **Common Issues**:
   - Authentication failures: Check managed identity permissions
   - Data type mismatches: Verify schema mapping
   - Connection timeouts: Check network configuration
   - Performance issues: Optimize data integration units (DIUs)

2. **Diagnostic Queries**:
   ```kusto
   // Failed pipeline runs in last 24 hours
   PipelineRuns
   | where TimeGenerated > ago(24h)
   | where Status == "Failed"
   | project TimeGenerated, PipelineName, RunId, ErrorMessage
   ```

---

## **Advanced Enhancements**

### 1. **Data Flow Transformations**:
- **Aggregate**: Calculate department-wise salary summaries
- **Conditional Split**: Separate active/inactive employees
- **Derived Column**: Add calculated fields (tenure, salary grade)
- **Filter**: Remove test data or apply business rules

### 2. **CI/CD Integration**:
- Use **Azure DevOps** or **GitHub Actions** for pipeline deployment
- Implement environment-specific configurations (Dev/Test/Prod)
- Version control ADF artifacts using Git integration

### 3. **Parameterization Best Practices**:
- Environment-specific parameters
- Dynamic file paths with date patterns
- Configurable retry policies
- External configuration management

### 4. **Security Enhancements**:
- Private endpoints for all data sources
- Customer-managed keys for encryption
- Network isolation with VNet integration
- Regular access reviews and least privilege principles

### 5. **Cost Optimization**:
- Right-size integration runtime
- Use Auto-pause for Spark clusters
- Implement data lifecycle policies
- Monitor and optimize DIU usage

---

## **Additional SQL Scripts**

### Audit Table Creation
```sql
CREATE TABLE dbo.PipelineExecutionLog (
    LogID BIGINT IDENTITY(1,1) PRIMARY KEY,
    PipelineName NVARCHAR(100) NOT NULL,
    RunId NVARCHAR(50) NOT NULL,
    StartTime DATETIME2 DEFAULT GETUTCDATE(),
    EndTime DATETIME2 NULL,
    Status NVARCHAR(20) NOT NULL,
    RecordsProcessed INT NULL,
    ErrorMessage NVARCHAR(MAX) NULL,
    CreatedDate DATETIME2 DEFAULT GETUTCDATE()
);

GO

-- Stored procedure for logging
CREATE PROCEDURE sp_LogPipelineExecution
    @PipelineName NVARCHAR(100),
    @RunId NVARCHAR(50),
    @RecordsProcessed INT = NULL,
    @Status NVARCHAR(20) = 'Success',
    @ErrorMessage NVARCHAR(MAX) = NULL
AS
BEGIN
    INSERT INTO dbo.PipelineExecutionLog 
    (PipelineName, RunId, Status, RecordsProcessed, ErrorMessage, EndTime)
    VALUES 
    (@PipelineName, @RunId, @Status, @RecordsProcessed, @ErrorMessage, GETUTCDATE());
END;
```

### Data Quality Views
```sql
-- View for data quality metrics
CREATE VIEW vw_EmployeeDataQuality AS
SELECT 
    Department,
    COUNT(*) as TotalEmployees,
    COUNT(CASE WHEN IsActive = 1 THEN 1 END) as ActiveEmployees,
    AVG(Salary) as AvgSalary,
    MIN(HireDate) as EarliestHireDate,
    MAX(HireDate) as LatestHireDate
FROM dbo.Employee
GROUP BY Department;
```

---

## **Performance Optimization Tips**

1. **Data Integration Units (DIUs)**:
   - Start with 4 DIUs for small datasets
   - Scale up to 32+ DIUs for large datasets (>1GB)
   - Monitor performance and adjust accordingly

2. **Parallel Processing**:
   - Use degree of copy parallelism for large tables
   - Implement data partitioning strategies
   - Consider using multiple copy activities for very large datasets

3. **Staging and Compression**:
   - Enable staging for better performance with large datasets
   - Use appropriate compression (gzip, snappy) for blob storage
   - Consider Parquet format for analytical workloads

4. **Incremental Loading**:
   - Implement change data capture (CDC) patterns
   - Use watermark columns for incremental sync
   - Schedule regular full refreshes for data integrity

---

This comprehensive project demonstrates creating a production-ready Azure Data Factory pipeline with security, monitoring, error handling, and performance optimization. The solution follows Azure best practices and includes all necessary components for enterprise-grade data integration.
