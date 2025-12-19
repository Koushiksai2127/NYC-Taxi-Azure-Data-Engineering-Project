# NYC-Taxi-Azure-Data-Engineering-Project
This repository provides a comprehensive example of building a modern, scalable Lakehouse-style data pipeline on Microsoft Azure. It utilizes real-world public data from NYC Taxi Trips and integrates Azure Data Factory, Databricks, ADLS Gen2, and Power BI to demonstrate best practices in cloud data engineering.

[![Azure Data Factory](https://img.shields.io/badge/Azure_Data_Factory-0078D4?logo=azure&logoColor=white)](https://learn.microsoft.com/en-us/azure/data-factory/)
[![Azure Databricks](https://img.shields.io/badge/Azure_Databricks-FF3900?logo=databricks&logoColor=white)](https://azure.microsoft.com/en-us/products/databricks)
[![ADLS Gen2](https://img.shields.io/badge/ADLS_Gen2-0089D6?logo=microsoftazure&logoColor=white)](https://learn.microsoft.com/en-us/azure/storage/blobs/data-lake-storage-introduction)
[![PySpark](https://img.shields.io/badge/PySpark-E25A1C?logo=apachespark&logoColor=white)](https://spark.apache.org/docs/latest/api/python/)
[![Delta Lake](https://img.shields.io/badge/Delta_Lake-00A651?logo=delta&logoColor=white)](https://delta.io/)
[![Power BI](https://img.shields.io/badge/Power%20BI-F2C811?logo=powerbi&logoColor=black)](https://powerbi.microsoft.com/)

## Summary
This project aims to gather NYC Taxi data from a public API, store it in a structured data lake, process it into useful datasets, and visualize the outcomes in Power BI. The pipeline is fully automated, parameterized, secure, and developed according to enterprise best practices.

The focus of this project was not just moving data, but ensuring **security** (via Managed Identities), **pipeline optimization** (dynamic expressions), and **data governance** (Unity Catalog).

## Architecture Overview

The pipeline follows a modern Lakehouse architecture pattern:

`[Web API Source]` ➔ `[Azure Data Factory]` ➔ `[ADLS Gen2 (Bronze)]` ➔ `[Databricks (Spark Transformation)]` ➔ `[Power BI (Reporting)]`

![NYC Architecture](https://github.com/Koushiksai2127/NYC-Taxi-Azure-Data-Engineering-Project/blob/main/Architecture%20and%20Reporting/NYC%20Architecture.png)

### Tech Stack & Tools
* **Ingestion:** Azure Data Factory (ADF) with Dynamic Pipelines.
* **Storage:** Azure Data Lake Storage Gen2 (ADLS) - Hierarchical Namespace enabled.
* **Compute & Transform:** Azure Databricks (PySpark, Delta Lake).
* **Governance:** Unity Catalog for secure access management.
* **Visualization:** Power BI connected via Partner Connect.

### Data Source
Check out the [NYC Taxi Source Data](https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page) here.

## Key Implementation Features

### 1. Infrastructure & Secure Connectivity
The foundation of this project is a secure, cloud-native infrastructure. Instead of using legacy Access Keys, which pose a security risk, I implemented **System-Assigned Managed Identities**.

* **Azure Storage (ADLS Gen2):** Provisioned with Hierarchical Namespace enabled to support directory-based storage (Bronze/Silver/Gold).
* **Azure Data Factory (ADF):** Enabled System-Assigned Managed Identity (SAMI) upon creation.

#### 🔐 Security Implementation: Managed Identity
To connect ADF to the Storage Account securely, I avoided hardcoding credentials.

1. **Identity Creation:** Enabled the System-Assigned Managed Identity for the Data Factory instance.
2. **RBAC Assignment:** Granted the ADF Managed Identity the specific role (e.g., *Storage Blob Data Contributor*) on the Storage Account.
3. **Linked Service Configuration:** Configured the Azure Data Lake Storage Gen2 Linked Service to use **"Managed Identity"** as the authentication method.

> **Why this matters:** This approach aligns with enterprise security best practices by ensuring that credentials are never stored in the code or pipeline definitions, and access is managed via Azure Active Directory (Entra ID).

<br>

<div align="center">
<img src="https://github.com/user-attachments/assets/87673c28-09cc-43a2-9756-9da2534a393d" width="700" />
<p><em>Figure 1: Linked Service configuration showing Managed Identity authentication.</em></p>

<br>

<img src="https://github.com/user-attachments/assets/8975977d-65ed-4b91-be25-bedec08c5b88" width="800" />
<img src="https://github.com/user-attachments/assets/71f1cabe-2d3f-4e06-af0d-6bea0b1f4a8a" width="800" />
<p><em>Figure 2: Provisioning the Azure Storage with Entra ID Authentication.</em></p>

</div>


### 2. Ingestion: Optimized Dynamic Pipelines
I constructed an dynamic ADF pipeline to fetch monthly data from the NYC Taxi open API.

* **Security First:** Authenticated to ADLS Gen2 using **System Assigned Managed Identity** to avoid hardcoding access keys.
* **Dynamic Iteration:** Implemented a `ForEach` activity to iterate through months (1-12) sequentially.

<div align="center">
<img width="975" height="567" alt="image" src="https://github.com/user-attachments/assets/745c7a7e-3a00-4844-ab26-7a1a1c1238f5" />
<br>
<img width="723" height="975" alt="image" src="https://github.com/user-attachments/assets/87058e36-f074-4bdf-942a-f825720c9860" />
</div>

### 3. Transformation: Databricks & Delta Lake
Data processing was handled in Azure Databricks using PySpark, following the Bronze-Silver-Gold (Medallion Architecture) layering.

* **Unity Catalog:** Configured Access Connectors and External Credentials to securely mount the Data Lake without exposing storage keys.
* **Write Mode Strategy:** Implemented robust write logic to ensure data integrity:
    * Used `overwrite` for the Silver layer to ensure a clean state during development.

<div align="center">
<img width="975" height="347" alt="image" src="https://github.com/user-attachments/assets/104e1654-16f7-45e9-b135-d52236e544db" />
<br>
<img width="975" height="464" alt="image" src="https://github.com/user-attachments/assets/a927cf5a-af32-479c-9fac-f2cc5286bbc2" />
<br>
<img width="975" height="366" alt="image" src="https://github.com/user-attachments/assets/503d6ef6-3e7c-4ddf-9c85-326f112b1cbe" />
</div>

### 4. Serving: Power BI Dashboard
The Gold layer (aggregated data) is connected to Power BI using Databricks Partner Connect.
* Generated Personal Access Tokens (PAT) for secure connectivity

<div align="center">
  <img width="975" height="419" alt="image" src="https://github.com/user-attachments/assets/21175ac8-9b2c-4a94-8542-8a98002d4248" />
</div>
<br>
* Created a visualization dashboard to analyze trip frequency and trends.

![PowerBi Report](https://github.com/Koushiksai2127/NYC-Taxi-Azure-Data-Engineering-Project/blob/main/Architecture%20and%20Reporting/Reporting.png)


## Key Takeaways & Learnings

### 🔧 Pipeline Optimization
I learned that "less is more" in cloud engineering. By replacing the `If` activity with a dynamic expression, I made the pipeline more robust. This aligns with ADF best practices for maintainability.

### 🛡️ Security & Governance
Moving away from Access Keys to **Managed Identities** and **Unity Catalog** gave me practical experience with modern Azure security standards (RBAC and Least Privilege).

### 📊 Spark Data Integrity
I gained a deep understanding of Spark Write Modes (`Overwrite` vs `Append` vs `ErrorIfExists`). Understanding when to overwrite data versus appending is critical for building idempotent pipelines that don't duplicate data on re-runs.

## Learning Background

<p> As part of my transition into data engineering, I explored Azure through online courses and documentation.This project reflects the skills I gained and my ability to apply them in a real cloud environment.</p>

## Final Thoughts

* Building this project has improved my practical skills in Azure Data Engineering and strengthened my understanding of cloud architecture design.
* It reflects my curiosity, persistence, and passion for developing real, end-to-end cloud data solutions.
* More Azure, Databricks, and data platform projects will be added soon.

<p> If you have suggestions, feedback, or opportunities related to data engineering, feel free to connect with me. I’m always excited to learn, collaborate, and work on meaningful data projects.</p>
