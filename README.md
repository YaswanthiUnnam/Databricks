
# End-to-End Data Engineering Pipeline in Azure Databricks

## Overview
The implementation of a data engineering pipeline using **Azure Databricks** for earthquake data processing. The pipeline follows a **medallion architecture** (bronze, silver, and gold layers) and utilizes an **Azure Storage Account** for data lake storage.

## 1. Azure Resource Setup
- **Azure Account**: Required for setting up resources.
- **Azure Databricks Workspace**: Created within a specified subscription and resource group with a premium pricing tier to enable **Role-Based Access Control (RBAC)**.
- **Azure Storage Account**: Configured with **Azure Data Lake Storage Gen2** and a hierarchical namespace enabled.
- **Storage Containers**:
  ![Container](https://github.com/YaswanthiUnnam/Databricks/blob/205eb783fd487f7413daf4c371f123c8bc79093d/Images/containers.png)
  - **Bronze**: Stores raw, unprocessed data.
  - **Silver**: Stores cleaned and transformed data.
  - **Gold**: Stores enriched and structured data for business use.

## 2. Connecting Databricks to Storage Account
- **RBAC Implementation**:
  - Databricks Unity Catalog **managed identity** granted **Storage Blob Data Contributor** role.
- **Credential Setup**:
  - Created in Databricks using the managed identity’s resource ID.
- **External Locations**:
  - Defined in Databricks Unity Catalog for each storage container (bronze, silver, gold).

## 3. Data Transformation Logic (Databricks Notebooks)
### Bronze Notebook
![bronze](https://github.com/YaswanthiUnnam/Databricks/blob/205eb783fd487f7413daf4c371f123c8bc79093d/Images/bronze.png)
- Mounts the **bronze container**.
- Retrieves raw earthquake data from an **external API** using a time-based query.
- Saves raw **JSON data** into the bronze container with filenames including the start date.
- Outputs storage paths for use in later processing stages. ![Bronze Notebook](https://github.com/YaswanthiUnnam/Databricks/blob/205eb783fd487f7413daf4c371f123c8bc79093d/Bronze%20Notebook.ipynb)

### Silver Notebook
![silver](https://github.com/YaswanthiUnnam/Databricks/blob/205eb783fd487f7413daf4c371f123c8bc79093d/Images/silver.png)
- Reads raw JSON data from the **bronze container** into a **Spark DataFrame**.
- Performs **data cleaning and transformation**:
  - Renames columns (e.g., coordinates → latitude, longitude, elevation).
  - Handles missing values.
  - Converts **Unix timestamps** to **datetime** format.
- Saves transformed data as a **Parquet file** in the silver container.
- Outputs the path to the transformed **silver Parquet file**. ![Silver Notebook](https://github.com/YaswanthiUnnam/Databricks/blob/205eb783fd487f7413daf4c371f123c8bc79093d/Silver%20Notebook.ipynb)

### Gold Notebook
![gold](https://github.com/YaswanthiUnnam/Databricks/blob/205eb783fd487f7413daf4c371f123c8bc79093d/Images/gold.png)
- Reads **transformed Parquet data** from the silver container.
- **Enrichment Process**:
  - Uses a **reverse geocoding library** to determine the country based on latitude and longitude.
  - Categorizes earthquake **significance (Sig)** into levels (low, moderate, high).
- Saves the enriched data as a **Parquet file** in the gold container. ![Gold Notebook](https://github.com/YaswanthiUnnam/Databricks/blob/205eb783fd487f7413daf4c371f123c8bc79093d/Gold%20Notebook.ipynb)

## 4. Workflow Orchestration
- **Databricks Workflow (Job)** created to automate notebook execution.![Pipeline_run](https://github.com/YaswanthiUnnam/Databricks/blob/205eb783fd487f7413daf4c371f123c8bc79093d/Images/Pipeline_run.png)
- **Tasks**:
  - Bronze Notebook → Silver Notebook → Gold Notebook.
- **Dependencies**:
  - The **Silver Notebook** runs after the Bronze Notebook.
  - The **Gold Notebook** runs after the Silver Notebook.
- **Parameter Passing**:
  - Utilizes **task values** to transfer storage paths and file references between tasks.
- **Monitoring**:
  - Job runs are monitored for errors and execution status.

## Conclusion
This pipeline showcases a **serverless, fully managed** data engineering solution within Azure Databricks. It demonstrates how to **ingest, transform, and enrich** earthquake data using Databricks workflows, without requiring **Azure Data Factory** for core pipeline logic. The **gold layer** data is optimized for **business intelligence and reporting**.

## Contributors
<strong>Yaswanthi Unnam</strong>

## Credits
<strong>Luke J Byrne</strong>

## References
[https://youtu.be/zIS_ssTQmO0?si=tqZd1EZSY6FHtHyv]
