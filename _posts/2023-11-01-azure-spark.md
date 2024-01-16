---
title: Azure Databricks & Spark for Data Engineers (PySpark / SQL)
author: kato
date: 2023-11-01 14:10:00 +0800
categories: [Microsoft Azure, Databricks]
tags: [Pyspark, Databricks, Microsoft Azure, Azure Data Factory, Azure Data Lake Storage, Azure Key Vault]
render_with_liquid: false
published: true
---

## Introduction
This personal project, I use `PySpark` on `Databrick` on `Azure` to build data warehouse with Medallion architecture

**Git repo:** &nbsp; [<img src="https://git-scm.com/images/logos/1color-lightbg@2x.png" width="45">](https://github.com/Katoo2706/AzureDataEngineer/tree/production/src/Fomular1)

## General concept

### Hive metastore

> Reference: &nbsp; [Hive Tables](https://spark.apache.org/docs/latest/sql-data-sources-hive-tables.html#hive-tables)

`Spark SQL` uses a `Hive metastore` to manage the metadata of persistent relational entities (e.g. databases, tables, columns, partitions) in a relational database (for fast access).

A Hive metastore warehouse (aka spark-warehouse) is the directory where Spark SQL persists tables whereas a Hive metastore (aka `metastore_db`) is a relational database to manage the metadata of the persistent relational entities, e.g. databases, tables, columns, partitions.

By default, `Spark SQL` uses the embedded deployment mode of a Hive metastore with a `Apache Derby` database.

### Hive metastore on Databricks

Databricks manages Meta Store:
- By Databricks default
- Or by External Meta Store (Azure SQL, My SQL, PostgreSQL, MariaDB etc)


## Project architecture
> Techstack using on this project: `Databricks`, `Pyspark`, `Delta Lake`, `Azure Data Lake Storage`, `Azure data Factory`, `Azure Key Vault`

![Project architecture](/assets/post/azure/azure_ach.png)

> Azure databricks Analytics architecture reference.

![azure_db_analytics_archi.png](/assets/post/azure/azure_db_analytics_archi.png)
 
## 1. Set up the environments

### Create Databricks Cluster
Databricks / Compute / Create compute (with below configuration):
- Policy: Unrestricted (Can create custom policy): [Cluster policy definitions](https://adb-1999765807836905.5.azuredatabricks.net/?o=1999765807836905#create/cluster)

For example: Set up value for spark version ![Choose the policy](/assets/post/azure/policy.png) with below policy config
```
{
  "spark_version": {
    "type": "fixed",
    "value": "auto:latest-lts",
    "hidden": true  # to hide the version
  }
}
```
- Access mode: No isolation shared
- Node type: Standard_DS3_v2 
- Termination time: 18 minutes (saving money for activity)
- Logging: dbfs:/cluster-logs/0821-080317-lq06a367

### Create budgets & Cost alerts
Budget Details
- Name: az-admin-exceed-30-pounds
- Reset period: Reset period
- Amount (Threshold): 10
Alert conditions:
- Actual - 90 (%) 
- Alert recipients (email): katoo2706@gmail.com

### Create Azure Storage Account
![Storage account](/assets/post/azure/create_storage_account.png)

*Select hierarchical to enables file and directory semantics, accelerates big data analytics workloads, and enables access control lists (ACLs)*
![Storage account](/assets/post/azure/create_storage_account_2.png)

*Choose the storage account*
![Storage account](/assets/post/azure/storage_3.png)

*Download storage browser*
![Storage account](/assets/post/azure/storage_brower.png)
[Download](https://azure.microsoft.com/en-us/products/storage/storage-explorer/)

### Case 1: Session Scoped Authentication (Via notebook)

#### 1. Access Azure Data Lake storage using access key
```bash
abfss://<container>@<storageAccountName>.dfs.core.windows.net
```
![Access ky](/assets/post/azure/access_key.png)
[The Azure Blob Filesystem driver (ABFS)](https://learn.microsoft.com/en-us/azure/storage/blobs/data-lake-storage-abfs-driver)
**Set up the access key**
https://learn.microsoft.com/en-us/azure/storage/blobs/data-lake-storage-abfs-driver
```
spark.conf.set(
    "fs.azure.account.key.datalake01course.dfs.core.windows.net",
    "**access_key**"
)
```

#### 2. Access Azure Data Lake storage using Shared access signature
- [Connect to Azure Data Lake Storage Gen2 using SAS token](https://learn.microsoft.com/en-us/azure/databricks/storage/azure-storage#sastokens:~:text=Active%20Directory%20application.-,Sas%C2%A0tokens,-You%20can%20configure)
- [Grant limited access to Azure Storage resources using shared access signatures (SAS)](https://learn.microsoft.com/en-us/azure/storage/common/storage-sas-overview)
    ```python
    # Set up the access key
    spark.conf.set("fs.azure.account.auth.type.datalake01course.dfs.core.windows.net", "SAS")
    spark.conf.set("fs.azure.sas.token.provider.type.datalake01course.dfs.core.windows.net", "org.apache.hadoop.fs.azurebfs.sas.FixedSASTokenProvider")
    spark.conf.set("fs.azure.sas.fixed.token.datalake01course.dfs.core.windows.net", blob_sas_token)
    ```
Generate SAS token from container
  ![generate sas token](/assets/post/azure/sas_token.png)

  ![generate sas token](/assets/post/azure/grant_privileges.png)

Otherwise, SAS can be generated from Microsoft Azure Storage Explorer
![SAS via Microsoft Azure Storage Explorer](/assets/post/azure/sas_app.png)

#### 3. Access Azure Data Lake using Service Principal
[Azure service principal](https://learn.microsoft.com/en-us/azure/databricks/storage/azure-storage#adls-gen2-oauth-20-with-azure-service-principals-notebook:~:text=or%20a%20notebook%3A-,Azure%C2%A0service%C2%A0principal,-Use%20the%20following)

**Step to follow:**

- Create app: Choose service Azure Active Directory / Choose App registrations / New registration (Choose default setting)
![App registration](/assets/post/azure/app_registration.png)
Copy client_id, tenant_id
```python
  client_id = "***"
  tenant_id = "***"
  
  # To get client secret, choose Certificates & secrets / + New client secret -> copy value
  client_secret = "***" 
```

- Assign role to app: In storage account, choose Access Control (IAM) / Add / Add role assignment (Storage Blob Data Contributor)
![Add role assignment](/assets/post/azure/storage_blob.png)

### Case 2: Cluster Scoped Authentication (Via cluster)
#### 1. Enable credential passthrough for user-level data access
![Add role assignment](/assets/post/azure/cluster-scoped.png)

> Same as Access Azure Data Lake using Service Principal, we need to grant access to creator account (who owns cluster) in Storage account\
> Even though I'm the owner, I do not have the role to read data from the storage account.
-> Assign role to Cluster: In storage account, choose Access Control (IAM) / Add / Add role assignment (Storage Blob Data Contributor)


## 2. Secure Access to Azure Data Lake
### Create Azure Key Vault

```markdown
// Basics
Resource group: databrickcourse-rg
Key vault name: fomula1-key-vault-course
Region: Southeast Asia
Pricing tier: Standard

// Access configuration
Permission model: Vault access policy
```

Create access token in Key Vault
![Secret access token in Key vault](/assets/post/azure/create_secret.png)

### Create secret scope in Databrick
will get scope-name

Reference: [Link](https://learn.microsoft.com/en-us/azure/databricks/security/secrets/secret-scopes#create-an-azure-key-vault-backed-secret-scope-using-the-ui:~:text=all%20networks.-,Create%20an%20Azure%20Key%20Vault%2Dbacked%20secret%20scope%20using%20the%20UI,-Go%20to%20https)

Go to `https://<databricks-instance>#secrets/createScope`. This URL is case sensitive; scope in createScope must be uppercase.
![Secret access token in Key vault](/assets/post/azure/secret-scope.png)

For `DNS Name` & `Resource ID`, go to key vault / Properties to get value for that
```markdown
DNS Name = Vault URI
Resource ID = Resource ID
```
![Secret access token in Key vault](/assets/post/azure/key-vault.png)

## 3. Mount the container onto Hive MetaStore.

## 4. Convert raw files into Parquet tables, Delta tables.

## 5. Establish either a temporary view or manage tables/external tables, utilizing SparkSQL for data visualization.

## 6. Azure Data Factory
### Learning resource:
- **Tutorial**: [Visit here](https://learn.microsoft.com/en-us/azure/data-factory/data-factory-tutorials)
- **Video**: [Visit here](https://www.youtube.com/channel/UC2S0k7NeLcEm5_IhHUwpN0g/featured)

![data_factories.png](/assets/post/azure/data_factories.png)

### Create CI/CD with git
![adf.png](/assets/post/azure/adf.png)

Or after launching the Studio, choose Linked service and set up the git repo
![adf-git.png](/assets/post/azure/adf-git.png)

Create pipeline
![adf-pipeline.png](/assets/post/azure/adf-pipeline.png)

**Set up the linked Databrick Notebook**
![adf-config-task.png](/assets/post/azure/adf-config-task.png)

To use Managed Service Identity
- Change access (IAM) in Azure Databricks Service - workspace
- Privileged Administrator Roles / Contributor
![adf-access-databrick.png](/assets/post/azure/adf-access-databrick.png)
- Add menber etl-adls-course (Data Factory)
![adf-add-member.png](/assets/post/azure/adf-add-member.png)
- After that, choose the access from existing cluster
![adf-config-task-access.png](/assets/post/azure/adf-config-task-access.png)
*If it's not loaded, click other cluster and re-click the existing cluster for refresh UI*

- Add the dynamic parameters by using Variables from the pipeline
![adf-dynamic-parameter-using-var.png](/assets/post/azure/adf-dynamic-parameter-using-var.png)
- Or using parameter (Can not be changed)
![adf-dynamic-parameter-using-parameter.png](/assets/post/azure/adf-dynamic-parameter-using-parameter.png)
- Config the DAGs
![adf-DAGs.png](/assets/post/azure/adf-DAGs.png)

### Add Get metadata to check if file exists

| Set up the storage account                | Get parameter as directory                 |
|-------------------------------------------|--------------------------------------------|
| ![Left Image](.//assets/post/azure/adf-metadata-1.png) | ![Right Image](.//assets/post/azure/adf-metadata-2.png) |

**Check if data exists or not**
![adf-if-condition.png](/assets/post/azure/adf-if-condition.png)

Then, click True (edit) and paste the Notebook into Window
![adf-if-true.png](/assets/post/azure/adf-if-true.png)

### Final: Create master execute pipeline
![adf-execute-pipeline.png](/assets/post/azure/adf-execute-pipeline.png)

> Updating...
{: .prompt-info }

## Learn More

For more knowledge about my posts, reach me via [katoo2706@gmail.com](mailto:katoo2706@gmail.com)
