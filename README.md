Below is a full Azure Data Factory (ADF) project with a **sample dataset**, code, and detailed steps for creating a pipeline to move data from Azure Blob Storage to an Azure SQL Database, including transformations.

---

## **Objective**
Move and transform data from Azure Blob Storage (CSV file) to an Azure SQL Database using Azure Data Factory.

---

## **Sample Dataset**
Save this dataset as a `sample_data.csv` file and upload it to an Azure Blob Storage container.

```csv
EmployeeID,Name,Department,Salary
101,John Doe,Engineering,60000
102,Jane Smith,Marketing,55000
103,Michael Brown,Sales,45000
104,Linda White,HR,50000
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
         Name NVARCHAR(50),
         Department NVARCHAR(50),
         Salary INT
     );
     ```
4. Install **Azure Data Factory Studio** (available via the Azure Portal).

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

---

### **Step 3: Create Linked Services**
Linked Services are used to connect ADF to external resources.

#### 1. Blob Storage Linked Service
1. In ADF Studio, go to **Manage > Linked Services**.
2. Click **+ New** and select **Azure Blob Storage**.
3. Configure:
   - **Account Selection Method**: Enter manually or use a subscription.
   - **Storage Account Name**: Enter the storage account.
4. Test the connection and save.

#### 2. Azure SQL Database Linked Service
1. Create another Linked Service for Azure SQL Database.
2. Configure:
   - **Server Name**: Enter the server address.
   - **Database Name**: Enter the database name.
   - **Authentication Type**: SQL authentication.
   - **Username** and **Password**: Enter credentials.
3. Test the connection and save.

---

### **Step 4: Create Datasets**
Datasets represent the data structure in the source and destination.

#### 1. Blob Dataset
1. Go to **Author > Datasets**, click **+ New Dataset**.
2. Select **Azure Blob Storage** and **DelimitedText**.
3. Configure:
   - **Linked Service**: Select Blob Storage.
   - **File Path**: Point to `sample_data.csv`.
   - Enable **First Row as Header**.
4. Save as `BlobInputDataset`.

#### 2. SQL Dataset
1. Add another dataset for the Azure SQL Database.
2. Select **Azure SQL Database** and configure:
   - **Linked Service**: Select SQL Database.
   - **Table Name**: Choose `Employee`.
3. Save as `SQLSinkDataset`.

---

### **Step 5: Create the Pipeline**
1. In **Author > Pipelines**, click **+ New Pipeline**.
2. Drag and drop the **Copy Data** activity onto the canvas.
3. Configure the activity:
   - **Source**:
     - Select `BlobInputDataset`.
   - **Sink**:
     - Select `SQLSinkDataset`.
   - **Mapping**:
     - Map source columns to sink columns:
       - `EmployeeID → EmployeeID`
       - `Name → Name`
       - `Department → Department`
       - `Salary → Salary`.

---

### **Step 6: Debug and Run**
1. Click **Debug** to test the pipeline.
2. Monitor the progress in the **Output** window.

---

### **Step 7: Publish and Trigger**
1. Click **Publish All** to save changes.
2. Add a trigger:
   - Manual: Use **Trigger Now**.
   - Scheduled: Configure a schedule in **Add Trigger > New/Edit**.

---

## **Code Snippets**

### Pipeline JSON
```json
{
  "name": "CopyPipeline",
  "properties": {
    "activities": [
      {
        "name": "Copy Data from Blob to SQL",
        "type": "Copy",
        "typeProperties": {
          "source": {
            "type": "DelimitedTextSource",
            "additionalColumns": []
          },
          "sink": {
            "type": "AzureSqlSink"
          }
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
      }
    ]
  }
}
```

### Blob Dataset JSON
```json
{
  "name": "BlobInputDataset",
  "properties": {
    "linkedServiceName": {
      "referenceName": "AzureBlobStorageLinkedService",
      "type": "LinkedServiceReference"
    },
    "type": "DelimitedText",
    "typeProperties": {
      "location": {
        "type": "AzureBlobStorageLocation",
        "container": "sample-container",
        "fileName": "sample_data.csv"
      },
      "columnDelimiter": ",",
      "firstRowAsHeader": true
    }
  }
}
```

### SQL Dataset JSON
```json
{
  "name": "SQLSinkDataset",
  "properties": {
    "linkedServiceName": {
      "referenceName": "AzureSQLDatabaseLinkedService",
      "type": "LinkedServiceReference"
    },
    "type": "AzureSqlTable",
    "typeProperties": {
      "tableName": "Employee"
    }
  }
}
```

---

## **Enhancements**
1. **Transformation**:
   - Add a Data Flow for complex transformations.
2. **Error Handling**:
   - Use **Try-Catch** blocks for pipeline errors.
3. **Parameterization**:
   - Add pipeline parameters for dynamic dataset paths or table names.
4. **Monitoring**:
   - Enable alerts via Azure Monitor.

---

This full project demonstrates creating a functional Azure Data Factory pipeline with datasets, linked services, and transformations. Let me know if you'd like additional customizations!
