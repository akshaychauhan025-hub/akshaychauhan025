Raw SOR Data
─────────────
CASE_ID: FRD-2024-00123
DISP_TYP_CD: CNP
CUST_SGMT: PLAT
RSLT_CD: 04
AGT_NTS: "cust clm unauth txn mcht XYZ $247 dt 12/06"
        ↓
Preprocessing Layer
─────────────────────
1. Map codes to readable text:
   DISP_TYP_CD: CNP → "Card Not Present Fraud"
   RSLT_CD: 04 → "Dispute Approved"
   CUST_SGMT: PLAT → "Platinum Customer"

2. Expand abbreviations:
   "cust clm unauth txn" → 
   "Customer claimed unauthorised transaction"

3. Structure the data:
   Case ID: FRD-2024-00123
   Dispute Type: Card Not Present Fraud
   Customer Segment: Platinum
   Outcome: Dispute Approved
   Agent Notes: "Customer claimed unauthorised 
   transaction at merchant XYZ for $247 on 12th June"
        ↓
Now LLM understandable ✅
        ↓
Embedding model converts to vectors
        ↓
Stored in Vector DB


**ETL — Extract, Transform, Load
**ETL is a three stage pipeline:**

Extract          Transform          Load
   ↓                 ↓                ↓
Pull data        Clean and          Store in
from SOR         convert it         Vector DB

Stage 1 — Extract
Pull raw data from GFS SOR:

GFS SOR Database
(Oracle / SQL Server / PostgreSQL)
        ↓
SQL Query runs on schedule:

SELECT 
  CASE_ID,
  DISP_TYP_CD,
  CUST_SGMT,
  RSLT_CD,
  AGT_NTS,
  CREATED_DT
FROM FRAUD_CASES
WHERE CREATED_DT >= LAST_EXPORT_DATE

        ↓
Raw data extracted:
CASE_ID    | DISP_TYP_CD | RSLT_CD | AGT_NTS
FRD-00123  | CNP         | 04      | "cust clm unauth txn..."
FRD-00124  | CP          | 02      | "cust sd crd stolen..."


Tools

Azure Data Factory  → enterprise, drag and drop, 
                      connects to any database
Apache Airflow      → open source, Python based,
                      highly flexible
dbt                 → transforms data inside 
                      the database before extraction


Stage 2 — Transform
Convert raw SOR data into LLM understandable text:


Raw record:
CASE_ID    → FRD-00123
DISP_TYP_CD → CNP
CUST_SGMT  → PLAT
RSLT_CD    → 04
AGT_NTS    → "cust clm unauth txn mcht XYZ $247"

        ↓ Step 1: Code mapping
DISP_TYP_CD: CNP  → Card Not Present Fraud
RSLT_CD: 04       → Dispute Approved
CUST_SGMT: PLAT   → Platinum Customer

        ↓ Step 2: Abbreviation expansion
"cust clm unauth txn mcht XYZ $247"
→ "Customer claimed unauthorised 
   transaction at merchant XYZ for $247"

        ↓ Step 3: PII stripping
"Customer claimed unauthorised 
transaction at merchant [MERCHANT] for $247"

        ↓ Step 4: Structured text generation
"Case FRD-00123: A Card Not Present Fraud dispute 
was raised by a Platinum customer who claimed an 
unauthorised transaction of $247 at a merchant. 
The dispute was reviewed and approved."

        ↓ Step 5: Chunking (if long)
Split into 500 token chunks with overlap

Stage 3 — Load
Store transformed data into Vector DB:

Clean, structured text
        ↓
Embedding model
(Azure OpenAI text-embedding-3-large)
        ↓
Text converted to vector:
[0.23, 0.87, 0.41, 0.12, ...]
        ↓
Stored in Vector DB (Azure AI Search)
with metadata:

{
  vector: [0.23, 0.87, 0.41...],
  text: "Case FRD-00123: Card Not Present...",
  metadata: {
    case_id: "FRD-00123",
    dispute_type: "Card Not Present Fraud",
    outcome: "Approved",
    date: "12/06/2024"
  }
}

Full Picture — Azure Data Factory Example

Azure Data Factory Pipeline:

Trigger: Every night 2am
        ↓
Activity 1 — Extract
Connect to GFS Oracle DB
Run SQL query
Pull new fraud cases since last run
        ↓
Activity 2 — Transform
Map codes → readable text
Expand abbreviations
Strip PII (Microsoft Presidio)
Generate structured text
        ↓
Activity 3 — Embed
Send text to Azure OpenAI
text-embedding-3-large
Get vectors back
        ↓
Activity 4 — Load
Store vectors + metadata
in Azure AI Search
        ↓
Activity 5 — Log
Record:
- How many records processed
- Any errors
- Timestamp
- Audit trail for compliance
- 

