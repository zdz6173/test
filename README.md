# CoreAnalyticsTool: SubEquipment Alarms Pipeline Documentation

## 1. Objective
The goal of this pipeline is to extract alarm data related to sub-equipment from the SQL database, process it, and export the results to Google BigQuery (`analyst-246813:CoreAnalyticsTool.subequipment_alarms`) for further analysis and visualization in Power BI or other tools.

---

## 2. Architecture Overview
- **Source:** Cloud SQL Database (CareAlarm)
- **Destination:** BigQuery Table (`CoreAnalyticsTool.subequipment_alarms`)
- **Execution Environment:**
  - Local execution allowed only for testing (BigQuery write disabled locally)

---

## 3. Pipeline Steps

### Step 1: Initialization
- Get sure that you are connected to the good proxy
  ```
  .\cloud_sql_proxy.exe data-244201:europe-west2:production --port=3307
  ```
- Logger setup using `GCPLogSession`.
- Load DB configuration from the `config/` folder.

### Step 2: Data Extraction
- Lauch the file "Carealarm_ETL.py"
- Will in part :
    - Connect to SQL via `initiate_db`.
    - Execute query using `DataCloudSqlIO`:
      ```
      connector.query_subequipment_alarms(crt=None)
      ```
  - Load results into a Pandas DataFrame.


### Step 3: Data Validation
- Verify that data was extracted:
  ```python
  if len(df) > 0:
  ```
- Preview data and log extracted row count.
![{3DF84372-05BA-461D-8167-743BA75D15BD}](https://github.com/user-attachments/assets/b946f9d1-bd8e-4a39-ad48-aaca399ff9a1)
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
- You will get where the data has been extracted showing the project_id and the destination table

---

## 4. Folder Structure
```
/Test_subequipment_alarms[test_subequipment_alarms]
 
 ── Carealarm_ETL.py               # Main ETL script
 ── dataa.py                       # SQL query logic
 ── config/                        # DB config files
 ── log_config/                    # Logging configuration
 ── bq_tables/                     # BigQuery table schemas
    ── schemas/CoreAnalyticsTool/subequipment/*.json
```

---

## 5. Notes
- **BigQuery Write Restriction:** Writing to BigQuery is only allowed inside GCP VMs. Make sure to put comments on the code just to resend data to BigQuery
- **Schema Control:** DataFrame columns must align with the schema defined in the corresponding JSON schema file.

---

## 7. Maintainer
- **Author:** Zakaria Sidi Bachir
- **Contact:** zakaria.sidibachir@sidel.com

