# CoreAnalyticsTool: SubEquipment Alarms Pipeline Documentation

## 1. Objective
The goal of this pipeline is to extract alarm data related to sub-equipment from the SQL database, process it, and export the results to Google BigQuery (`analyst-246813:CoreAnalyticsTool.subequipment_alarms`) for further analysis and visualization in Power BI or other tools.

---

## 2. Architecture Overview
- **Source:** Cloud SQL Database (CareAlarm)
- **Destination:** BigQuery Table (`CoreAnalyticsTool.subequipment_alarms`)
- **Execution Environment:**
  - Designed for execution inside Google Cloud VM
  - Local execution allowed only for testing (BigQuery write disabled locally)

---

## 3. Pipeline Steps

### Step 1: Initialization
- Logger setup using `GCPLogSession`.
- Get sure that you are connected to the good proxy
  ```Windows Powershell
-   .\cloud_sql_proxy.exe data-244201:europe-west2:production --port=3307
  ```
- Load DB configuration from the `config/` folder.

### Step 2: Data Extraction
- Connect to SQL via `initiate_db`.
- Execute query using `DataCloudSqlIO`:
  ```python
  connector.query_subequipment_alarms(crt=None)
  ```
- Load results into a Pandas DataFrame.

### Step 3: Data Validation
- Verify that data was extracted:
  ```python
  if len(df) > 0:
  ```
- Preview data and log extracted row count.

### Step 4: BigQuery Export
- Export data to BigQuery using:
  ```python
  BigQueryIO(bq_client).send_data_to_bigquery(df, destination_table)
  ```
- Schema enforced from JSON definitions located at:
  ```
  cat_core/bq_tables/schemas/CoreAnalyticsTool/subequipment/
  ```
- Write operation restricted to GCP environments using `is_running_on_gcp()`.

---

## 4. Folder Structure
```
/ASulpr_nom
│
├── Carealarm_ETL.py               # Main ETL script
├── dataa.py                       # SQL query logic
├── config/                        # DB config files
├── log_config/                    # Logging configuration
├── bq_tables/                     # BigQuery table schemas
│   └── schemas/CoreAnalyticsTool/subequipment/*.json
```

---

## 5. Error Handling & Logging
- Managed via `GCPLogSession`.
- Logged events:
  - `init_session_logging`
  - `success_bigquery_insert`
  - `error_bigquery_insert`
  - `table_not_found`
  - `wrong_query`
  - `wrong_dataset`

---

## 6. Notes
- **BigQuery Write Restriction:** Writing to BigQuery is only allowed inside GCP VMs.
- **Schema Control:** DataFrame columns must align with the schema defined in the corresponding JSON schema file.

---

## 7. Maintainer
- **Author:** Zakaria Sidi Bachir
- **Contact:** zakaria.sidibachir@sidel.com

