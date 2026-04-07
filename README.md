# 🛒 Ecommerce Data Pipeline (Azure)

A cloud-native, end-to-end data engineering pipeline built on **Microsoft Azure** that ingests raw ecommerce data, processes it through a multi-layer medallion architecture using Azure Databricks, and serves curated insights via Power BI.

---

## 📐 Architecture Overview

```
Ecommerce Data  →  Azure Data Lake Storage (ADLS)  →  Azure Databricks (ETL/ELT)  →  Medallion Layers  →  Power BI
     (CSV)               (Centralized Data Lake)          (Unity Catalog)            Bronze / Silver / Gold
```



---

## 🧱 Architecture Components

### 1. Source System — Ecommerce Data
- The ecommerce platform exports transactional and product data as **CSV files**.
- These files are pushed to the centralized data lake as the raw ingestion layer.

### 2. Azure Data Lake Storage (ADLS)
- Acts as the **Centralized Data Lake** and landing zone for all incoming raw CSV data.
- Secure access is enforced via an **Access Connector** using Azure Managed Identity, ensuring no credentials are hardcoded.
- All raw files are stored here before any transformation begins.

### 3. Azure Databricks (ETL/ELT Engine)
- Core transformation engine running **ETL and ELT** workloads on Apache Spark.
- Integrated with **Unity Catalog** for unified data governance, access control, and lineage tracking across all data assets.
- Connects to ADLS via the **Secure Access Connector**, removing the need for storage account keys.

### 4. Medallion Architecture (Bronze → Silver → Gold)

| Layer   | Description |
|---------|-------------|
| 🟫 **Bronze** | Raw, unprocessed data ingested directly from ADLS. Full historical fidelity retained. |
| ⬜ **Silver** | Cleaned, validated, and conformed data. Deduplication, type casting, and business rules applied. |
| 🟡 **Gold**   | Aggregated, business-ready data models (facts & dimensions). Optimized for analytical queries. |

### 5. Analytics & Reporting Layer — Power BI
- Connects directly to the **Gold layer** for reporting and dashboarding.
- Enables business stakeholders to explore KPIs, sales trends, customer behavior, and operational metrics through interactive reports.

---

## 🛠️ Tech Stack

| Technology | Role |
|---|---|
| Azure Data Lake Storage Gen2 | Raw data storage and landing zone |
| Azure Databricks | Distributed data processing (Spark-based ETL/ELT) |
| Unity Catalog | Data governance, cataloging, and access control |
| Azure Managed Identity / Access Connector | Secure, credential-free ADLS access |
| Delta Lake | Storage format for Bronze / Silver / Gold tables |
| Power BI | Business intelligence and reporting |
| Python / PySpark | Transformation logic and pipeline scripts |

---

## 📁 Project Structure

```
├── medallion_processing_dim/              # Dimension table processing
│   ├── 01_dim_bronze.ipynb                # Ingest raw dimension data → Bronze
│   ├── 02_dim_silver.ipynb                # Clean & conform dimensions → Silver
│   └── 03_dim_gold.ipynb                  # Final dimension models → Gold
│
├── medallion_processing_fact/             # Fact table processing
│   ├── order_items_processing/
│   │   ├── 01_fact_bronze.ipynb           # Raw order items → Bronze
│   │   ├── 02_fact_silver.ipynb           # Cleaned order items → Silver
│   │   ├── 03_fact_gold.ipynb             # Aggregated order items → Gold
│   │   └── 04_daily_summary.ipynb         # Daily order items summary
│   ├── order_returns_processing/
│   │   ├── 01_facts_order_returns_bronze.ipynb   # Raw returns → Bronze
│   │   ├── 02_facts_order_returns_silver.ipynb   # Cleaned returns → Silver
│   │   └── 03_facts_order_returns_gold.ipynb     # Aggregated returns → Gold
│   └── order_shipments_processing/
│       ├── 01_facts_order_shipments_bronze.ipynb  # Raw shipments → Bronze
│       ├── 02_facts_order_shipments_silver.ipynb  # Cleaned shipments → Silver
│       └── 03_facts_order_shipments_gold.ipynb    # Aggregated shipments → Gold
│
├── setup/                                 # One-time environment setup
│   ├── setup-catalog.ipynb                # Unity Catalog setup (schemas, tables)
│   └── setup_raw_external_volume.ipynb    # ADLS external volume registration
│
└── README.md
```

---

## 🚀 Getting Started

### Prerequisites

- Azure Subscription with the following resources provisioned:
  - Azure Data Lake Storage Gen2
  - Azure Databricks Workspace
  - Unity Catalog enabled on Databricks
  - Access Connector for Azure Databricks
- Power BI Desktop (for local report development)

### Setup Steps

1. **Clone the repository**
   ```bash
   git clone https://github.com/PratikTiwari5995/Ecommerce-data-pipeline.git
   cd Ecommerce-data-pipeline
   ```

2. **Configure ADLS Access**
   - Assign the **Storage Blob Data Contributor** role to the Databricks Access Connector's managed identity on your ADLS account.
   - Update `configs/pipeline_config.json` with your storage account name and container paths.

3. **Set up Unity Catalog & External Volume**
   - Open your Databricks workspace and run the setup notebooks in order:
     1. `setup/setup_raw_external_volume.ipynb` — registers ADLS as an external volume in Unity Catalog
     2. `setup/setup-catalog.ipynb` — creates the catalog, schemas (`bronze`, `silver`, `gold`), and required tables

4. **Upload Raw Data**
   - Place ecommerce CSV exports into the `raw/` container path in ADLS.

5. **Run the Pipeline**
   - Execute notebooks in the following order:

   **Dimensions (run first):**
   1. `medallion_processing_dim/01_dim_bronze.ipynb`
   2. `medallion_processing_dim/02_dim_silver.ipynb`
   3. `medallion_processing_dim/03_dim_gold.ipynb`

   **Facts — Order Items:**
   1. `medallion_processing_fact/order_items_processing/01_fact_bronze.ipynb`
   2. `medallion_processing_fact/order_items_processing/02_fact_silver.ipynb`
   3. `medallion_processing_fact/order_items_processing/03_fact_gold.ipynb`
   4. `medallion_processing_fact/order_items_processing/04_daily_summary.ipynb`

   **Facts — Order Returns:**
   1. `medallion_processing_fact/order_returns_processing/01_facts_order_returns_bronze.ipynb`
   2. `medallion_processing_fact/order_returns_processing/02_facts_order_returns_silver.ipynb`
   3. `medallion_processing_fact/order_returns_processing/03_facts_order_returns_gold.ipynb`

   **Facts — Order Shipments:**
   1. `medallion_processing_fact/order_shipments_processing/01_facts_order_shipments_bronze.ipynb`
   2. `medallion_processing_fact/order_shipments_processing/02_facts_order_shipments_silver.ipynb`
   3. `medallion_processing_fact/order_shipments_processing/03_facts_order_shipments_gold.ipynb`

6. **Connect Power BI**
   - Open Power BI Desktop.
   - Use the **Azure Databricks** connector to point to your Gold layer tables.
   - Refresh and publish your reports.

---

## 📊 Data Flow

```
[Ecommerce CSV Data]
        │
        ▼
[ADLS Raw Container]  ──(Secure Access Connector)──▶  [Databricks + Unity Catalog]
                                                              │
                                              ┌───────────────┼───────────────┐
                                              ▼               ▼               ▼
                                          [Bronze]        [Silver]         [Gold]
                                         (Raw Delta)   (Cleaned Delta)  (Aggregated Delta)
                                                                              │
                                                                              ▼
                                                                         [Power BI]
```

---

## 🔐 Security

- No storage account keys are used. All ADLS access is via **Azure Managed Identity** through the Access Connector.
- Unity Catalog enforces **row-level and column-level** security across all layers.
- Databricks secrets are managed via **Azure Key Vault** integration (recommended).

---

## 📈 Future Enhancements

- Add incremental/CDC-based ingestion to replace full batch loads
- Integrate Azure Data Factory for pipeline orchestration and scheduling
- Add data quality checks using Great Expectations or Databricks Data Quality
- Enable real-time streaming ingestion via Azure Event Hubs + Structured Streaming

---

## 🤝 Contributing

Pull requests are welcome. For major changes, please open an issue first to discuss what you would like to change.

---

