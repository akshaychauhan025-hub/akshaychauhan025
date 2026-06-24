# Notes — SOR Data Extraction and ETL Pipeline

## Overview

This document covers how to extract data from a 
System of Record (SOR), transform it into LLM 
understandable language, and load it into a Vector DB 
for use in RAG pipelines.

---

## Step 1 — Raw SOR Data

Data as it exists in the System of Record:

CASE_ID: FRD-2024-00123

DISP_TYP_CD: CNP

CUST_SGMT: PLAT

RSLT_CD: 04

AGT_NTS: "cust clm unauth txn mcht XYZ $247 dt 12/06"

Raw SOR data is:
- Full of system codes
- Abbreviated agent notes
- Not readable by LLMs
- Not embeddable as-is

---

## Step 2 — Preprocessing Layer

Transform raw data into LLM understandable language:

**1. Map codes to readable text:**

DISP_TYP_CD: CNP  → "Card Not Present Fraud"

RSLT_CD: 04       → "Dispute Approved"

CUST_SGMT: PLAT   → "Platinum Customer"

**2. Expand abbreviations:**

"cust clm unauth txn mcht XYZ $247 dt 12/06"

↓

"Customer claimed unauthorised transaction

at merchant XYZ for $247 on 12th June"

**3. Structure the data:**

Case ID:          FRD-2024-00123

Dispute Type:     Card Not Present Fraud

Customer Segment: Platinum

Outcome:          Dispute Approved

Agent Notes:      "Customer claimed unauthorised

transaction at merchant XYZ

for $247 on 12th June"

**Result — Now LLM understandable ✅**

"Case FRD-2024-00123: A Card Not Present Fraud

dispute was raised by a Platinum customer who

claimed an unauthorised transaction of $247 at

a merchant on 12th June. The dispute was

reviewed and approved."

↓

Embedding model converts to vectors

↓

Stored in Vector DB

---

## ETL Pipeline — Extract, Transform, Load

ETL is the three stage pipeline that automates 
the entire process on a schedule:

Extract          Transform          Load

↓                 ↓                ↓

Pull data        Clean and          Store in

from SOR         convert it         Vector DB

---

## Stage 1 — Extract

Pull raw data from GFS SOR on a schedule:

GFS SOR Database

(Oracle / SQL Server / PostgreSQL)

↓

SQL Query runs automatically:
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

**Tools:**

Azure Data Factory  → enterprise, drag and drop,

connects to any database

Apache Airflow      → open source, Python based,

highly flexible

dbt                 → transforms data inside

the database before extraction

---

## Stage 2 — Transform

Convert raw SOR data into LLM understandable text:

Raw record:

CASE_ID      → FRD-00123

DISP_TYP_CD  → CNP

CUST_SGMT    → PLAT

RSLT_CD      → 04

AGT_NTS      → "cust clm unauth txn mcht XYZ $247"

↓ Step 1: Code mapping

DISP_TYP_CD: CNP  → Card Not Present Fraud

RSLT_CD: 04       → Dispute Approved

CUST_SGMT: PLAT   → Platinum Customer

↓ Step 2: Abbreviation expansion

"cust clm unauth txn mcht XYZ $247"

→ "Customer claimed unauthorised

transaction at merchant XYZ for $247"

↓ Step 3: PII stripping

"Customer claimed unauthorised transaction

at merchant [MERCHANT] for $247"

↓ Step 4: Structured text generation

"Case FRD-00123: A Card Not Present Fraud dispute

was raised by a Platinum customer who claimed an

unauthorised transaction of $247 at a merchant.

The dispute was reviewed and approved."

↓ Step 5: Chunking (if document is long)

Split into 500 token chunks with 50 token overlap


---

## Stage 3 — Load

Store transformed data into Vector DB:

Clean, structured text

↓

Embedding Model

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

---

## Full Azure Data Factory Pipeline

Trigger: Every night at 2am

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

Chunk long documents

↓

Activity 3 — Embed

Send clean text to Azure OpenAI

text-embedding-3-large

Receive vectors

↓

Activity 4 — Load

Store vectors + metadata

in Azure AI Search

↓

Activity 5 — Log

Record:

How many records processed
Any errors or failures
Timestamp
Audit trail for compliance


---

## Tool Comparison

| Tool | Type | Best For |
|---|---|---|
| Azure Data Factory | Enterprise | Regulated environments, Azure stack |
| Apache Airflow | Open Source | Flexible, complex pipelines |
| dbt | Transformation | Pre-cleaning inside database |
| Microsoft Presidio | PII Stripping | Financial services, open source |
| Azure OpenAI | Embedding | Enterprise compliant embedding |
| Azure AI Search | Vector DB | Enterprise search and RAG |
| Azure Monitor | Logging | Audit trail, compliance |

---

## Which Tool for Regulated Enterprise

Pre-cleaning inside GFS DB    → dbt

Orchestration and scheduling  → Azure Data Factory

PII stripping                 → Azure AI Language 

Embedding                     → Azure OpenAI

Vector storage                → Azure AI Search

Audit logging                 → Azure Monitor

---

## Key PM Considerations

| Concern | Action |
|---|---|
| Data quality | Preprocessing quality = RAG quality. Garbage in, garbage out. |
| Freshness | Define re-embedding schedule — daily for active cases |
| PII compliance | Strip before embedding — never store PII in Vector DB |
| Audit trail | Log every extraction from SOR — compliance requirement |
| Cost | Budget for initial embedding + ongoing re-embedding |
| Legal approval | Using SOR data for AI requires Legal and Compliance sign off |
| Access control | Read only access to SOR — no write permissions |

---

## Scheduling Strategy

| Data Type | Update Frequency | Why |
|---|---|---|
| Active fraud cases | Nightly | Cases updated daily by agents |
| Fraud procedures | Weekly | Procedures change less frequently |
| Past decisions | Weekly | Historical — less time sensitive |
| Regulatory policies | On change | Updated when regulations change |
| Fraud templates | On change | Updated when templates revised |

