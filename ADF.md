# Azure Data Factory (ADF) Project

## Introduction to Azure Data Factory (ADF)
Azure Data Factory (ADF) is a cloud-based data integration service that allows you to create, schedule, and orchestrate data workflows. It is designed to move data between different systems, transform it as needed, and support end-to-end data solutions.

### Core Concepts in ADF:
1. **Datasets**: Represent the data you want to work with, such as tables, files, or blobs.
2. **Pipelines**: A logical grouping of activities that perform a unit of work. For example, copying data from one location to another.
3. **Activities**: The steps in a pipeline (e.g., Copy Activity, Data Flow Activity).
4. **Linked Services**: Connections to your data sources and destinations.
5. **Triggers**: Define the schedule or event that starts the pipeline.

---

## Steps to Create an Azure Data Factory Project

### 1. **Set up the Azure Data Factory Instance**
   1. Go to the [Azure Portal](https://portal.azure.com/).
   2. Click on “Create a resource” and search for "Data Factory."
   3. Fill in the required details:
      - **Subscription**: Select your Azure subscription.
      - **Resource Group**: Choose an existing resource group or create a new one.
      - **Region**: Select the region where the ADF instance will be hosted.
      - **Data Factory Name**: Provide a unique name.
   4. Click “Review + Create” and then “Create” to deploy the Data Factory.

---

### 2. **Create a Pipeline in ADF**
#### a. Access the Data Factory Studio
   1. In the Azure Portal, navigate to your Data Factory instance.
   2. Click on "Author & Monitor" to open the Data Factory Studio.

#### b. Create Linked Services
   1. Navigate to **Manage** > **Linked Services**.
   2. Click “New” to define connections to your source and destination data systems (e.g., Azure Blob Storage, SQL Database).

#### c. Define Datasets
   1. Go to **Author** > **Datasets**.
   2. Click “New Dataset” and select the appropriate data format (e.g., Azure Blob Storage > CSV).
   3. Provide details about the dataset, such as linked service and file path.

#### d. Create a Pipeline
   1. In the **Author** section, click “New Pipeline.”
   2. Drag activities (e.g., Copy Data) onto the canvas.
   3. Configure the activities by specifying source and destination datasets.

---

### 3. **Schedule and Execute Pipelines**
#### a. Add a Trigger
   1. In the pipeline canvas, click **Add Trigger** > **New/Edit.**
   2. Choose to run manually, on a schedule, or based on an event.
   3. Configure trigger details and click “OK.”

#### b. Debug and Publish
   1. Use the **Debug** option to test your pipeline.
   2. Once verified, click “Publish All” to deploy your changes.

---

## Moving Data in ADF
1. Use the **Copy Activity** to move data from source to destination.
2. Configure options such as file format conversion, compression, and data mapping.

---

## Transformation and Analysis
1. **Mapping Data Flows**: Perform data transformations using a graphical interface.
   - Aggregate, filter, join, and derive columns.
2. **Custom Code**: Use Azure Functions or Databricks for custom logic.

---

## Monitoring and Managing Pipelines
1. Navigate to the **Monitor** tab in Data Factory Studio.
2. View pipeline runs, trigger runs, and activity details.
3. Set up alerts and diagnostics for error handling and performance monitoring.

---

## Common Use Cases for ADF
1. **Data Migration**: Moving on-premises data to Azure.
2. **Data Integration**: Combining data from multiple sources.
3. **Data Warehousing**: Loading and transforming data for analytics.
4. **Event-Driven Workflows**: Triggering pipelines based on events (e.g., new file upload).

---

## Sample Code for Deployment
Below is a basic example using Azure CLI and PowerShell to deploy ADF components:

### Azure CLI Example:
```bash
# Create Resource Group
az group create --name myResourceGroup --location eastus

# Create Data Factory
az datafactory create --resource-group myResourceGroup \
                      --name myDataFactory

# Create Linked Service
az datafactory linked-service create --resource-group myResourceGroup \
                                     --factory-name myDataFactory \
                                     --name myLinkedService \
                                     --properties '{"type": "AzureBlobStorage", "typeProperties": {"connectionString": "<your-connection-string>"}}'

# Create Pipeline
az datafactory pipeline create --resource-group myResourceGroup \
                               --factory-name myDataFactory \
                               --name myPipeline \
                               --pipeline "<pipeline-json-content>"
```

### PowerShell Example:
```powershell
# Import Azure Module
Import-Module Az.DataFactory

# Create Resource Group
New-AzResourceGroup -Name "myResourceGroup" -Location "East US"

# Create Data Factory
New-AzDataFactoryV2 -ResourceGroupName "myResourceGroup" -Name "myDataFactory" -Location "East US"

# Create Linked Service
Set-AzDataFactoryV2LinkedService -ResourceGroupName "myResourceGroup" \
                                  -DataFactoryName "myDataFactory" \
                                  -Name "myLinkedService" \
                                  -Properties @{type = 'AzureBlobStorage'; typeProperties = @{connectionString = '<your-connection-string>'}}

# Create Pipeline
Set-AzDataFactoryV2Pipeline -ResourceGroupName "myResourceGroup" \
                             -DataFactoryName "myDataFactory" \
                             -Name "myPipeline" \
                             -DefinitionFile "<path-to-pipeline-json>"
```

---

This guide provides an overview of setting up and working with Azure Data Factory. Customize the pipeline activities and datasets according to your specific requirements.

