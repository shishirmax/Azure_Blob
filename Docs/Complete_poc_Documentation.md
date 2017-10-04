# Microsoft Azure
### Microsoft Azure Storage
- Blob Storage
- Table Storage
- Queue Storage
- File Storage

1. **Blob Storage**: Binary Large Object Storage for storing unstructured data.
2. **Table Storage**: Tables to store data, table have no schema, can be used table storage to develop based on fast changing requirement.
3. **Queue Storage**: Distributed app need to communicate async. Queueing is one mechanism for such communication.
4. **File Storage**: It is like a file share. Can be mapped using a network drive.
 
### Account Types 
1. **Locally Redundant (Standard - LRS)**: 3 data copies in single facility in single location.
2. **Zone Redundant (Standard - ZRS)**: 3 data copies in two or three facilities in a single location (BLOB only).
3. **Geo Redundant (Standard - GRS)**: LRS + 3 data copies stored in secondary location.
4. **Read - Access Geo Redundant (Standard - RAGRS)**: Same as GRS but at a failover state the data in secondary location is read only.
 

### XML XSD Validation

### Azure Blob using SSIS

### Azure Data Lake

### Azure Data Factory
>Azure Data Factory is a cloud-based data integration service that allows you to create data-driven workflows in the cloud for orchestrating and automating data movement and data transformation. Using Azure Data Factory, you can create and schedule data-driven workflows (called pipelines) that can ingest data from disparate data stores, process/transform the data by using compute services such as Azure HDInsight Hadoop, Spark, Azure Data Lake Analytics, and Azure Machine Learning, and publish output data to data stores such as Azure SQL Data Warehouse for business intelligence (BI) applications to consume.

### creating a pipeline to copy data( Azure Storage Account[Blob] to Azure Sql DB.
##### 1. Create an Azure data factory. [ADFTutorialPOCV1]
##### 2. Create linked services.
- We create linked services in a data factory to link your data stores and compute services to the data factory.We have used two data stores of type Azure Storage(source) and Azure SQL Database(destination).
- Two linked services named "AzureStorageLinkedServices" and "AzureSqlLinkedServices" of type: **AzureStorage** and **AzureSqlDatabase**.
- The AzureStorageLinkedService links your Azure storage account to the data factory. This storage account is the one in which you created a container and uploaded the data.
- AzureSqlLinkedService links your Azure SQL database to the data factory. The data that is copied from the blob storage is stored in this database.
- We have created 'emp' table in the database.
##### 3. Create Azure Storage linked service[AzureStorageLinkedService]
- We create a New data store(Azure storage). It create a JSON template for creating an Azure storage linked service.
- We will replace <accountname> and <accountkey> with the account name and account key values for our Azure storage account.
```JSON
{
    "name": "AzureStorageLinkedService",
    "properties": {
        "description": "",
        "hubName": "adftutorialpocv1_hub",
        "type": "AzureStorage",
        "typeProperties": {
            "connectionString": "DefaultEndpointsProtocol=https;AccountName=shishirmax;AccountKey=**********"
        }
    }
}
```
- After deploying it will be available under linked services.
##### 4. Create a linked service for the Azure SQL Database [AzureSqlLinkedService]
- we create a new data store(Azure SQL Database). It also generate a JSON template for creating the Azure SQL linked service.
- we will replace <servername>,<databasename>,<username>@<servername> and <password> with names of our Azure SQL server, database, user account and password.
```JSON
{
    "name": "AzureSqlLinkedService",
    "properties": {
        "description": "",
        "hubName": "adftutorialpocv1_hub",
        "type": "AzureSqlDatabase",
        "typeProperties": {
            "connectionString": "Data Source=tcp:shishirazurepoc.database.windows.net,1433;Initial Catalog=azurepocdb;Integrated Security=False;User ID=shishiradmin;Password=**********;Connect Timeout=30;Encrypt=True"
        }
    }
}
```
- After deploying it will be available under linked services.

##### 5. Create datasets
Now we will define two datasets named InputDataset and OUtputDataset that represent input and output data that is stored in the data stores referred by AzureStorageLinkedService and AzureSqlLinkedService respectively.

The Azure storage linked service specifies the connection string that Data Factory service uses at run time to connect to our Azure storage account. And, the input blob dataset (InputDataset) specifies the container and the folder that contains the input data.
Similarly, the Azure SQL Database linked service specifies the connection string that Data Factory service uses at run time to connect to out Azure SQL database. And, the output SQL table dataset (OututDataset) specifies the table in the database to which the data from the blob storage is copied.

- **Create input dataset**
Create a dataset named InputDataset that points to a blob file (inputEmp.txt) in the root folder of a blob container (txtblob/txtdatafiles) in the Azure Storage represented by the AzureStorageLinkedService linked service. If you don't specify a value for the fileName (or skip it), data from all blobs in the input folder are copied to the destination.

To create click ... More, click New dataset, and click Azure Blob storage from the drop-down menu.
Replace/update the JSON.

```JSON
{
    "name": "InputDataset",
    "properties": {
        "structure": [
            {
                "name": "FirstName",
                "type": "String"
            },
            {
                "name": "LastName",
                "type": "String"
            }
        ],
        "published": false,
        "type": "AzureBlob",
        "linkedServiceName": "AzureStorageLinkedService",
        "typeProperties": {
            "fileName": "inputEmp.txt",
            "folderPath": "txtblob/txtdatafiles/",
            "format": {
                "type": "TextFormat",
                "columnDelimiter": "|"
            }
        },
        "availability": {
            "frequency": "Hour",
            "interval": 1
        },
        "external": true,
        "policy": {}
    }
}
```
Click Deploy

- **Create output dataset**
The Azure SQL Database linked service specifies the connection string that Data Factory service uses at run time to connect to your Azure SQL database. The output SQL table dataset (OutputDataset) we create in this step specifies the table in the database to which the data from the blob storage is copied.

To create click ... More, click New dataset, and click Azure SQL from the drop-down menu.
Replace/update the JSON.

```JSON
{
    "name": "OutputDataset",
    "properties": {
        "structure": [
            {
                "name": "FirstName",
                "type": "String"
            },
            {
                "name": "LastName",
                "type": "String"
            }
        ],
        "published": false,
        "type": "AzureSqlTable",
        "linkedServiceName": "AzureSqlLinkedService",
        "typeProperties": {
            "tableName": "emp"
        },
        "availability": {
            "frequency": "Hour",
            "interval": 1
        }
    }
}
```
Click Deploy

##### 6. Create pipeline [ADFTutorialPOCV1Pipeline]

we will create a pipeline with a copy activity that uses InputDataset as an input and OutputDataset as an output.
To create click ... More, and click New pipeline. 
Replace/Update JSON

```JSON
{
    "name": "ADFTutorialPOCV1Pipeline",
    "properties": {
        "description": "Copy data from a blob to Azure SQL table",
        "activities": [
            {
                "type": "Copy",
                "typeProperties": {
                    "source": {
                        "type": "BlobSource"
                    },
                    "sink": {
                        "type": "SqlSink",
                        "writeBatchSize": 10000,
                        "writeBatchTimeout": "60.00:00:00"
                    }
                },
                "inputs": [
                    {
                        "name": "InputDataset"
                    }
                ],
                "outputs": [
                    {
                        "name": "OutputDataset"
                    }
                ],
                "policy": {
                    "timeout": "01:00:00",
                    "concurrency": 1,
                    "executionPriorityOrder": "NewestFirst"
                },
                "scheduler": {
                    "frequency": "Hour",
                    "interval": 1
                },
                "name": "CopyFromBlobToSQL"
            }
        ],
        "start": "2017-05-11T00:00:00Z",
        "end": "2017-05-12T00:00:00Z",
        "isPaused": true,
        "hubName": "adftutorialpocv1_hub",
        "pipelineMode": "Scheduled"
    }
}
```
Click deploy.

**Complete Reference:** https://docs.microsoft.com/en-us/azure/data-factory/v1/data-factory-introduction






---
**Reference Links**:
- https://docs.microsoft.com/en-us/azure/data-factory/connector-azure-blob-storage
- https://docs.microsoft.com/en-us/azure/data-factory/copy-activity-overview
- https://docs.microsoft.com/en-us/azure/data-factory/quickstart-create-data-factory-portal
- https://docs.microsoft.com/en-us/azure/data-factory/tutorial-copy-data-dot-net
- https://www.slideshare.net/ShawnIsmail/session-38-azure-storage-part-1-introduction
- https://docs.microsoft.com/en-us/azure/data-factory/
- https://docs.microsoft.com/en-us/azure/data-factory/v1/data-factory-copy-activity-tutorial-using-azure-portal