# SynapseWarehouse
# Data Ingestion, Transformation, Storage in Azure, ETL

This project focuses on ingesting, transforming, and storing data within **Azure Data Lake Storage (ADLS) Gen2**, leveraging **Azure Data Factory (ADF)**, **Azure Databricks**, and various **Azure services** for security, performance optimization, and scalability.
Then, transferring the processed.csv file from an Azure Blob Storage container (processed-data) to a **Synapse Analytics warehouse** for further data analysis.  The data is sourced from an **HTTP API** and **Microsoft SQL Server** and goes through a series of steps to clean, transform, and ETL for further data analysis.

## Prerequisites:

- **Azure Data Factory** (ADF) for orchestrating data pipelines.
- **Azure Data Lake Storage Gen2** for data storage.
- **Azure Key Vault** for securely managing credentials and secrets.
- **Azure Databricks** for data processing.
- **Azure Synapse Analytics** Used to manage the dedicated SQL pool for data storage and analysis..
- **GitHub** for version control and managing the source code.

---

## Steps Taken:

### 1. Setting Up Azure Data Factory (ADF)

#### 1.1 Linking ADF to GitHub
- **GitHub Repository**: Created a repository called **DataLake** for version control. This repository is connected to ADF, and the **Dev branch** is used for pipeline development.
- **Version Control**: This ensures that all changes to the pipeline are versioned and can be tracked in the GitHub repository.

#### 1.2 Creating Linked Services
- **HTTP API to ADLS Gen2**:
  - Created a linked service to fetch data from the **HTTP API** (https://jsonplaceholder.typicode.com/users).
  - Stored the data in the **raw-api** container of ADLS Gen2 in **Parquet** format.
  - **Azure Key Vault** was used for securely managing the **storage account key**.

- **Microsoft SQL Server to ADLS Gen2**:
  - Established a linked service to connect to the **SQL Server** using a **self-hosted integration runtime**.
  - Transferred data to the **raw-sql** container in ADLS Gen2 in **CSV** format.

#### 1.3 Creating Pipelines in ADF
- **Pipeline 1**: **HTTP API to ADLS Gen2**
  - Used the **Copy Data activity** to fetch data from the HTTP API and save it in **Parquet** format in the **raw-api** container.

- **Pipeline 2**: **SQL Server to ADLS Gen2**
  - Used the **Copy Data activity** to transfer data from **SQL Server** to the **raw-sql** container in **CSV** format.

---

### 2. Pipeline Triggers and Branching from Dev to QA

#### 2.1 Triggers for Data Pipelines
- **Scheduled Triggers**: 
  - Used **time-based triggers** to initiate data pipelines for periodic data ingestion.
  - A trigger was set for **Pipeline 1** to pull the HTTP API data at a scheduled time.
  - A trigger for **Pipeline 2** to pull data from SQL Server on a regular basis.

- **Manual Triggers**: 
  - For cases where immediate data ingestion is needed, manual triggers were employed to kick off the pipeline.
  
- **Trigger Dependencies**: 
  - **Trigger 1** (HTTP API to ADLS) must complete before **Trigger 2** (SQL Server to ADLS) can begin, ensuring data is ingested in the correct order.
  
#### 2.2 Branching from Dev to QA
- **Development (Dev) Branch**:
  - Created pipelines in the **Dev** environment to validate the end-to-end data flow from ingestion to storage.
  - All new changes and features were tested in the **Dev branch** first.
  
- **Quality Assurance (QA) Branch**:
  - Once the pipelines were successfully tested in **Dev**, a **pull request** (PR) was initiated to merge the Dev branch into the **QA branch**.
  - After approval, pipelines were deployed to the **QA environment** for further validation.

- **Version Control in GitHub**: 
  - Ensured that both **Dev** and **QA branches** are synchronized for consistent pipeline deployment across environments.

---

### 3. Data Processing and Transformation in Databricks

#### 3.1 Setting Up Databricks Workspace
- Mounted the **raw-api** and **raw-sql** containers from ADLS Gen2 to Databricks using **secret scopes** for secure access to storage account keys.
- Used the following Databricks code to mount the storage accounts:
  ```python
   dbutils.fs.mount(
      source="wasbs://raw-api@casestudy1new.blob.core.windows.net",
      mount_point = "/mnt/raw-api",
      extra_configs={"fs.azure.account.key.casestudy1new.blob.core.windows.net": dbutils.secrets.get(scope = "casestudy", key = "storage")}
  )

#### 3.2 Reading Data from Parquet and CSV 
- Read data from `raw-api` (Parquet format) and `raw-sql` (CSV format) using Spark in Databricks:
  ```python
  df_parquet = spark.read.parquet("/mnt/raw-api/users.parquet")
  df_csv = spark.read.csv("/mnt/raw-sql/dbo.football.csv", header=True, inferSchema=True)

#### 3.3 Data Cleaning and Transformation
**Parquet Data**:
- Filtered rows where the username was 'Samantha' and re-indexed the data based on the username.
  
**CSV Data**:
- Filtered out players with goals less than 80 and re-assigned the `Rank` column.

#### 3.4 Repartitioning and Coalescing
- Optimized performance by repartitioning and coalescing data:
  ```python
   df_cleaned_coalesced = df_cleaned.coalesce(1)

---
### 4. Data Saving to processed container

#### 4.1 Saving Processed Data to ADLS
- Saved cleaned and transformed data into the `processed-api` and `processed-sql` containers in ADLS Gen2:
  ```python
   df_cleaned_coalesced.write.mode('append').parquet("/mnt/processed-api/users_cleaned")
  
---
### 5. Transferring Processed Data to Synapse Warehouse for Analysis
**Set up Azure Synapse Analytics**: - Create a Synapse Analytics workspace to manage data pipelines and dedicated SQL pool.

**Create Linked Services**: - Use Azure Key Vault to securely store credentials for both the Azure Blob Storage and Synapse Analytics.

**Create Dedicated SQL Pool**: - Set up a dedicated SQL pool in Synapse Analytics to store and analyze the transferred data.

**Create Pipeline**: - In Synapse, create a new pipeline to transfer the processed.csv file from the processed-data container in Azure Blob Storage to Synapse Analytics.

**Configure Source and Sink**: - Set the source linked service to connect to the Blob Storage and the sink linked service to connect to the dedicated SQL pool in Synapse Analytics.

**Create Analysis Table**: - In the dedicated SQL pool, create a table to store the transferred data for further analysis.

**Debug and Test**: - Use the debugging feature in Synapse to test the pipeline and ensure the transfer works correctly.

**Manual Trigger**: - Trigger the pipeline manually to ensure it functions as expected and successfully transfers the file.

**Publish Pipeline**: - After successful testing, publish the pipeline to automate the process for future transfers.

**Validate Data Integrity**: - Verify that the data in the Synapse SQL pool matches the original data in the processed.csv file.

---

### 5. Security Best Practices

**Azure Key Vault**:
- Managed sensitive information such as storage account keys securely using Azure Key Vault. This ensures the credentials are not exposed in the code.

**Secret Scopes**:
- Databricks accessed the storage account keys securely using secret scopes to mount containers for data ingestion and saving.

---

### 6. Conclusion
- This project implements an end-to-end solution for ingesting, processing,storing data in Azure Data Lake Storage (ADLS) Gen2. Then, transferring the processed.csv file from an Azure Blob Storage container (processed-data) to a Synapse Analytics warehouse for further data analysis.
- The transfer of processed.csv from Blob Storage to Synapse Analytics was successfully implemented using Azure Synapse Analytics, Azure Blob Storage, Azure Key Vault, and the native Synapse pipeline orchestration.
- By employing triggers, manual interventions, and branching strategies between Dev and QA, this project ensures smooth deployments and updates to the pipeline. The data moves securely through each stage, from ingestion to transformation and finally to Syanapse warehouse, ready for analysis.
- This approach ensures optimal performance, security, and scalability for data processing in Azure, laying the groundwork for further data analysis and reporting.
- This setup allows for scalable data analysis and can be further automated and enhanced for future use cases.
