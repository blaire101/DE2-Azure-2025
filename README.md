# 📦 Azure Data Engineering Overview

> 📚 Motivation: Data without architecture is just noise；architect it well，and it becomes knowledge.

🌅 **Microsoft Certified：Azure Data Engineer Associate（DP‑203）**

## Preface

In modern data architecture，**Azure** provides a comprehensive set of tools to support the **full data lifecycle** —— from ingestion and storage to processing，orchestration，governance，and serving。

> Solid line → main path（core data flow），Dashed line → optional／supplementary path；CDC：Change Data Capture

---

## 1）Simple Version

```mermaid
flowchart LR
    %% ===== Layers =====
    subgraph L1[🔁 Data Ingestion]
        ADF[[📥 Azure Data Factory<br>（Copy Data ／ CDC）]]:::ing
        EH[[📡 Azure Event Hubs<br>（Streaming）]]:::ing
    end

    subgraph L2[🗃️ Data Lake]
        ADLS[(🪣 Azure Data Lake Storage Gen2)]:::stor
    end

    subgraph L3[⚙️ Transform]
        DFE[[🧪 Azure Data Factory Mapping Data Flows<br>（Transform）]]:::proc
        SPARK[[⚡ Azure Databricks<br>（Spark Processing）]]:::proc
    end

    subgraph L4[🏛️ Warehouse]
        SYN[(Azure Synapse Analytics<br>（Warehouse）)]:::wh
    end

    subgraph L5[📊 Serving]
        SA[[🔍 Azure Synapse Serverless SQL<br>（Query on ADLS）]]:::srv
        PP[[📈 Power BI<br>（Dashboards）]]:::srv
        AS[[📊 Azure Analysis Services<br>（Semantic Models）]]:::srv
    end

    %% ===== Governance =====
    PURVIEW[[📚 Microsoft Purview<br>（Catalog ／ Governance）]]:::gov

    %% ===== Core paths =====
    ADF --> ADLS
    ADLS --> DFE
    DFE --> SYN
    ADLS --> SA
    SYN --> PP

    %% ===== Streaming optional =====
    EH -.-> SPARK
    EH -.-> AS

    %% ===== Governance wiring =====
    PURVIEW -.-> ADLS
    PURVIEW -.-> SYN

    %% ===== Styles =====
    classDef ing  fill:#d0f0fd,stroke:#0078D4,stroke-width:2px,color:#000;
    classDef stor fill:#fde2d0,stroke:#ca5010,stroke-width:2px,color:#000;
    classDef proc fill:#e6d0fd,stroke:#5c2d91,stroke-width:2px,color:#000;
    classDef wh   fill:#ffe8b3,stroke:#986f0b,stroke-width:2px,color:#000;
    classDef srv  fill:#d9f7be,stroke:#107c10,stroke-width:2px,color:#000;
    classDef gov  fill:#f0f5ff,stroke:#0078D4,stroke-width:2px,color:#000;
```

---

## 2）Middle Version

```mermaid
flowchart LR
    subgraph L1["Data Ingestion"]
        ADF["Azure Data Factory <br>（Copy Data ／ CDC）"]
        EH["Azure Event Hubs <br>（Streaming）"]
    end

    subgraph L2["Raw Zone（ADLS Gen2）"]
        RAW["Raw Data"]
    end

    subgraph L3["Staging Zone（ADLS ／ ODS）"]
        STG["Staging Data"]
    end

    subgraph L4["Curated Zone（ADLS）"]
        DIL["DIL - Detailed Facts"]
        DIM["DIM - Dimensions"]
        DWS["DWS - Aggregates"]
    end

    subgraph L5["Azure Synapse Analytics（Warehouse ／ Serving）"]
        SYN["DIM ＆ DWS for BI"]
    end

    subgraph L6["Serving ＆ Analytics"]
        SA["Synapse Serverless SQL"]
        PP["Power BI"]
    end

    %% Paths
    ADF --> RAW
    RAW --> DFE1["ADF Mapping Data Flows ／ Databricks"] --> STG
    STG --> DFE2["ADF Mapping Data Flows ／ Databricks"] --> DIL
    STG --> DFE2 --> DIM
    STG --> DFE2 --> DWS

    DIL --> SYN
    DIM --> SYN
    DWS --> SYN

    DIL --> SA
    DIM --> SA
    DWS --> SA
    SYN --> PP

    %% Optional streaming
    EH -.-> RT1["Stream Analytics ／ Databricks（Real-time）"] -.-> API["API Management ／ Power BI DirectQuery"]

    %% Styles
    classDef ing  fill:#d0f0fd,stroke:#0078D4,stroke-width:2px,color:#000;
    classDef stor fill:#fde2d0,stroke:#ca5010,stroke-width:2px,color:#000;
    classDef proc fill:#e6d0fd,stroke:#5c2d91,stroke-width:2px,color:#000;
    classDef wh   fill:#ffe8b3,stroke:#986f0b,stroke-width:2px,color:#000;
    classDef srv  fill:#d9f7be,stroke:#107c10,stroke-width:2px,color:#000;

    class ADF,EH ing
    class RAW,STG,DIL,DIM,DWS stor
    class DFE1,DFE2,RT1 proc
    class SYN wh
    class SA,PP,API srv
```

---

## 3）Detailed Version

```mermaid
flowchart LR
    %% ===== L1: Ingestion =====
    subgraph L1[🔁 Data Ingestion]
        ADF[[📥 Azure Data Factory<br>（Copy Data ／ CDC）]]:::ing
        DFEG[[🧪 Dataflow Gen2<br>（Batch ／ Transform）]]:::ing
        EH[[📡 Azure Event Hubs ／ IoT Hub<br>（Streaming Events）]]:::ing
    end

    %% ===== L2: Processing =====
    subgraph L2[⚙️ Data Processing]
        DBX[[⚡ Azure Databricks<br>（Spark ／ ML）]]:::proc
        ASA[[🔷 Azure Stream Analytics<br>（Flink-like Streaming）]]:::proc
        FUNC[[λ Azure Functions<br>（Light Stream Transform）]]:::proc
        SYNPIPE[[🛠️ Synapse Pipelines<br>（Orchestration）]]:::proc
    end

    %% ===== L3: Lake Storage Zones =====
    subgraph L3[🗃️ Data Lake（ADLS Gen2）]
        RAW[(🪣 Raw Zone)]:::stor
        STG[(🪣 Staging Zone)]:::stor
        CUR[(🪣 Curated Zone)]:::stor
    end

    %% ===== L3.5: Warehouse =====
    subgraph L35[🏛️ Warehouse]
        SYN[(Azure Synapse Dedicated SQL Pool)]:::wh
    end

    %% ===== L4: Serving ＆ Analytics =====
    subgraph L4[📊 Serving ＆ Analytics]
        SA[[🔍 Synapse Serverless SQL<br>（Query on ADLS）]]:::srv
        PP[[📈 Power BI<br>（Dashboards）]]:::srv
        API[[🔌 API Management ＆ Functions<br>（Data APIs）]]:::srv
        AS[[📊 Azure Analysis Services<br>（Semantic Models）]]:::srv
        AML[[🤖 Azure Machine Learning<br>（ML Training ／ Inference）]]:::srv
    end

    %% ===== Governance =====
    subgraph GOV[🗂️ Governance]
        PURVIEW[[📚 Microsoft Purview<br>（Catalog ／ Lineage ／ Security）]]:::gov
    end

    %% ===== Batch core =====
    ADF --> RAW
    DFEG --> RAW
    RAW --> DBX
    RAW --> SYNPIPE
    DBX --> STG
    SYNPIPE --> STG
    STG --> DBX
    DBX --> CUR
    SYNPIPE --> CUR
    CUR --> SYN

    %% ===== Streaming core =====
    EH -.-> ASA
    EH -.-> FUNC
    ASA -.-> STG
    FUNC -.-> STG
    ASA -.-> SYN

    %% ===== Serving =====
    CUR --> SA
    SYN --> PP
    CUR -.-> AS
    SYN -.-> API
    CUR -.-> AML

    %% ===== Streaming → Serving direct =====
    ASA -.-> AS
    FUNC -.-> API

    %% ===== Governance wiring =====
    PURVIEW -.-> RAW
    PURVIEW -.-> STG
    PURVIEW -.-> CUR
    PURVIEW -.-> SYN

    %% Optional source
    SQLDB[(🗄️ Azure SQL Database ／ Managed Instance<br>（OLTP Source）)]:::src
    SQLDB -.-> ADF

    %% ===== Styles =====
    classDef ing  fill:#d0f0fd,stroke:#0078D4,stroke-width:2px,rx:10,ry:10;
    classDef proc fill:#e6d0fd,stroke:#5c2d91,stroke-width:2px,rx:10,ry:10;
    classDef stor fill:#fde2d0,stroke:#ca5010,stroke-width:2px,rx:10,ry:10;
    classDef wh   fill:#ffe8b3,stroke:#986f0b,stroke-width:2px,rx:10,ry:10;
    classDef srv  fill:#d9f7be,stroke:#107c10,stroke-width:2px,rx:10,ry:10;
    classDef gov  fill:#f0f5ff,stroke:#0078D4,stroke-width:2px,rx:10,ry:10;
    classDef src  fill:#fff1f0,stroke:#a80000,stroke-width:2px,rx:10,ry:10;
```

---

## Azure Zones Definition

1. **Raw Zone（ADLS Gen2）**  
   - Exact copy of source data（immutable）。  
   - For audit and reprocessing。  

2. **Staging Zone（ADLS Gen2 ／ ODS）**  
   - Lightly cleaned，standardized schema。  
   - Temporary before transformation。  

3. **Curated Zone（ADLS ／ Synapse）**  
   - **DIL**：detailed，cleaned facts。  
   - **DIM**：dimensions for joins。  
   - **DWS**：aggregated，business‑ready tables。  
   - Stored in ADLS for Serverless SQL or in Synapse for high performance。  

4. **Synapse Analytics**  
   - Stores DIM／DWS for fast BI queries。  
   - Powers dashboards，APIs，and advanced analytics。  

---

## AWS → Azure Service Comparison Table (Concise Version)

| Function                      | AWS                                               | Azure                                                              |
| ----------------------------- | ------------------------------------------------- | ------------------------------------------------------------------ |
| Batch ingestion/orchestration | AWS Glue Ingest / Glue Workflows / Step Functions | Azure Data Factory / Synapse Pipelines                             |
| CDC (database replication)    | AWS DMS                                           | ADF Change Data Capture (SQL/MI) / third-party (Debezium/Fivetran) |
| Streaming ingestion           | Amazon Kinesis / MSK                              | Azure Event Hubs / IoT Hub                                         |
| Stream processing             | Kinesis Data Analytics (Flink)                    | Azure Stream Analytics / Databricks Structured Streaming           |
| Data lake storage             | Amazon S3                                         | Azure Data Lake Storage Gen2                                       |
| Batch processing engine       | AWS Glue ETL (Spark) / EMR                        | Azure Databricks / ADF Mapping Data Flows                          |
| Metadata catalog              | AWS Glue Data Catalog                             | Microsoft Purview                                                  |
| Data warehouse                | Amazon Redshift                                   | Azure Synapse Dedicated SQL Pool                                   |
| Lakehouse ad-hoc queries      | Amazon Athena                                     | Synapse Serverless SQL                                             |
| Search / logging              | Amazon OpenSearch                                 | Azure Cognitive Search / Log Analytics                             |
| Visualization                 | Amazon QuickSight                                 | Power BI                                                           |
| Machine learning              | Amazon SageMaker                                  | Azure Machine Learning                                             |
| API exposure                  | API Gateway & Lambda                              | API Management & Functions                                         |

---

## Synapse vs Hive vs Databricks SQL

| Feature | Synapse Dedicated SQL Pool | Hive | Databricks SQL |
|--------|-----------------------------|------|----------------|
| Type | Managed MPP Data Warehouse | Hadoop SQL Engine | Spark‑based SQL |
| Storage | Internal columnar store | HDFS | HDFS ／ ADLS ／ S3 |
| Latency | Fast | Slow | Fast |
| Deployment | Fully managed | Self‑hosted Hadoop | Managed ／ Serverless |
| Best for | BI star／snowflake，ELT，materialized data marts | Legacy batch ETL | Interactive lakehouse BI，SQL endpoints |

---

### Notes (Practical Tips)

- **Recommended architecture**: ADLS (Raw → Staging → Curated) + Databricks (Transform) + Synapse (Serving) + Purview (Governance) + ADF/Synapse Pipelines (Orchestration).
- **Cost control**: Use Serverless SQL for exploration; use Dedicated SQL Pool only for stable reporting; enable auto-termination for Databricks workspaces.
- **Access governance**: Use Purview as the catalog source, combined with R...

