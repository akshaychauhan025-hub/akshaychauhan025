# 11 — RAG (Retrieval Augmented Generation): Deep Dive

## What is RAG?

**Retrieval Augmented Generation (RAG)** is a technique 
that grounds LLM outputs in your own data — by retrieving 
relevant context from a knowledge base before generating 
a response.

Without RAG:

Question → LLM → Answer based on training data only

(may hallucinate, no access to your data)
With RAG:

Question → Retrieve relevant context → LLM → Answer

from YOUR knowledge base    grounded in your data

> RAG gives the LLM a reference library to read 
> before answering — instead of relying purely 
> on memory.

---

## Why RAG Exists

LLMs have three fundamental limitations:

**1. Knowledge Cutoff**
Trained on data up to a certain date.
Knows nothing about events, cases, or decisions after that.

**2. No Access to Your Data**
GPT-4 doesn't know your fraud procedures,
past cases, internal policies, or customer history.

**3. Hallucination**
When the model doesn't know something —
it confidently makes something up.

RAG solves all three:

Knowledge cutoff    → Vector DB always has latest data

No your data        → Vector DB contains YOUR knowledge base

Hallucination       → LLM answers from retrieved facts

not from memory

---

## RAG vs Fine Tuning

A common question — when do you RAG vs fine tune?

| | RAG | Fine Tuning |
|---|---|---|
| **What it does** | Retrieves context at runtime | Bakes knowledge into model weights |
| **Data freshness** | Always current | Frozen at training time |
| **Cost** | Lower — no retraining | High — retraining is expensive |
| **Speed to implement** | Fast — days to weeks | Slow — weeks to months |
| **Best for** | Dynamic, frequently updated data | Teaching model a new style or behaviour |
| **Explainability** | High — can show retrieved source | Low — knowledge baked in |
| **Compliance** | Easier — data stays in Vector DB | Harder — data baked into model |

> **For regulated financial services:**
> RAG is almost always preferred over fine tuning —
> data stays in your controlled Vector DB,
> sources are explainable, and knowledge updates
> don't require retraining.

---

## How RAG Works — Step by Step

### Phase 1 — Indexing (done once, updated regularly)

Your Knowledge Base:

Past fraud summaries
Fraud templates
Fraud decisions
Card replacement procedures
Fraud dispute procedures

↓

ETL Pipeline (Azure Data Factory + Databricks)

↓

Preprocessing:
Code mapping
Abbreviation expansion
PII stripping
Structured text generation
Chunking (500 tokens, 50 overlap)

↓

Embedding Model

(Azure OpenAI text-embedding-3-large)

↓

Vectors stored in Vector DB

(Azure AI Search)

with metadata

### Phase 2 — Retrieval (every query)

Agent query or transcript arrives

↓

Query embedded using same model

↓

Hybrid search in Vector DB:

Dense search  → finds similar meaning

Sparse search → finds exact references

↓

Reranker scores results

↓

Top K most relevant chunks retrieved:
Result 1: Similar past fraud case   → 0.96

Result 2: Relevant dispute procedure → 0.91

Result 3: Card replacement steps    → 0.87

Result 4: Relevant fraud template   → 0.84

### Phase 3 — Generation (every query)

Retrieved context

+

System instruction

+

Clean transcript / query

↓

Combined into prompt

↓

GPT-4 via Azure OpenAI

↓

Structured, grounded response

↓

GFS updated

---

## Your Real World RAG Pipeline — GFS Fraud Servicing

### Knowledge Base Setup

GFS SOR (Oracle DB)

↓

Azure Data Factory (orchestration)

↓

Azure Databricks / Spark (transformation)

┌─────────────────────────────────┐

│ Code mapping (CNP → Card Not    │

│ Present Fraud)                  │

│ Abbreviation expansion          │

│ PII stripping (Presidio)        │

│ Structured text generation      │

│ Chunking                        │

└─────────────────────────────────┘

↓

Azure OpenAI text-embedding-3-large

↓

Azure AI Search (Vector DB)
Knowledge base contains:

✅ Past fraud summaries

✅ Fraud templates

✅ Historical decisions

✅ Card replacement procedures

✅ Fraud dispute procedures


### Runtime — Every Call

Customer call recording

↓

Whisper → Raw transcript

↓

PII stripping (Presidio)

↓

Clean transcript embedded

↓

Hybrid search in Azure AI Search:
Retrieved context:

Similar past fraud case     → 0.96
Applicable dispute procedure → 0.91
Card replacement steps      → 0.88
Relevant fraud template     → 0.85

↓

Prompt assembled:

[System Instruction]

You are a fraud servicing expert.

Generate a structured case summary

following our standard template.

Do not fabricate. Use only provided context.
[Retrieved Context]

Similar case: FRD-00089 — CNP fraud approved...

Procedure: Card Not Present dispute steps...

Template: Summary format...
[Clean Transcript]

Customer reported unauthorised transaction

of $247 at [MERCHANT] on [DATE]..

↓

GPT-4 via Azure OpenAI

↓

Structured call summary:

Fraud type: Card Not Present
Issue: Unauthorised transaction $247
Procedure followed: CNP dispute process
Decision: Escalated for review
Next steps: Card replacement initiated

↓

Summary fed into GFS ✅

Agent reviews and approves ✅

---

## RAG Architecture Patterns

### Naive RAG
Basic retrieval — embed, search, generate.
Good for prototypes. Quality issues at scale.

Query → Embed → Search → Generate

### Advanced RAG
Adds query rewriting, reranking, hybrid search.
Production grade quality.


Query

↓

Query rewriting

(improve search quality)

↓

Hybrid search (dense + sparse)

↓

Reranker

↓

Generate


### Modular RAG
Full pipeline with feedback loops,
multiple retrievers, routing logic.
Enterprise grade.

Query

↓

Router (which knowledge base?)

↓

Multiple retrievers in parallel

↓

Fusion and reranking

↓

Generate

↓

Evaluate output quality

↓

Feedback loop


---

## Query Rewriting — Why It Matters

Agents and customers don't always phrase queries well.
Query rewriting improves retrieval quality before searching.

Original agent query:

"what do we do when customer says

they didnt make payment"

↓

Query rewriter (small LLM):

"Unauthorised transaction dispute

procedure for Card Not Present fraud"

↓

Better search → better retrieval → better answer


---

## Chunking Strategy for Your Pipeline

How you chunk determines RAG quality.

**For fraud procedures (structured documents):**

Split by section headings

Each section = one chunk

Preserves procedural logic

**For past fraud summaries (short records):**

Each case summary = one chunk

No splitting needed

Metadata filters by case type

**For long policy documents:**

500 token chunks

50 token overlap

Prevents context loss at boundaries

---

## Metadata Filtering — Precision Retrieval

Don't just search all documents — filter first,
then search within relevant subset.

New call: Card Not Present fraud

↓

Metadata filter:

dispute_type = "Card Not Present"

↓

Search only within CNP cases

and CNP procedures

↓

Higher precision retrieval

Lower noise in context

**Metadata to store with each chunk:**


{

vector: [...],

text: "...",

metadata: {

document_type: "fraud_procedure",

dispute_type: "card_not_present",

outcome: "approved",

date: "2024-06",

case_id: "FRD-00123",

version: "v2.1"

}

}


---

## RAG Evaluation — How to Know It's Working

| Metric | What it measures | Target |
|---|---|---|
| **Faithfulness** | Does answer match retrieved context? | > 90% |
| **Answer relevance** | Does answer address the question? | > 85% |
| **Context precision** | Are retrieved chunks relevant? | > 80% |
| **Context recall** | Did we retrieve all needed context? | > 75% |
| **Revision rate** | How often agent edits summary? | < 20% |
| **Handling time** | Time saved vs manual process | > 20% reduction |

> **Revision rate is your most honest signal.**
> If agents constantly edit AI summaries —
> your RAG pipeline needs improvement.

---

## Common RAG Failures and Fixes

**1. Retrieving wrong context**

Problem:  Search returns irrelevant chunks

Fix:      Improve chunking strategy

Add metadata filtering

Use hybrid search + reranker

**2. Context window overflow**

Problem:  Too much retrieved context →

exceeds LLM token limit

Fix:      Retrieve fewer chunks (top 3-5)

Use reranker to select best chunks

Compress context before sending

**3. Hallucination despite RAG**

Problem:  LLM ignores retrieved context

and makes things up

Fix:      Stronger system instruction:

"Use ONLY the provided context.

If answer not in context, say so."

Lower temperature (0.1-0.2)

**4. Stale knowledge base**

Problem:  Vector DB not updated —

RAG retrieves outdated procedures

Fix:      Automated re-indexing pipeline

Version control on documents

Freshness metadata filter

**5. PII in Vector DB**

Problem:  PII stored in embeddings —

compliance violation

Fix:      Mandatory PII stripping

before embedding

Never embed raw SOR data

---

## RAG in Regulated Environments — Compliance Checklist

Before going live:
Data

□ PII stripped before embedding

□ Data residency confirmed (Azure region)

□ SOR access approved by Legal/Compliance

□ Re-embedding schedule defined
Security

□ Vector DB access controlled

□ Audit logging enabled (Azure Monitor)

□ Encryption at rest and in transit
Model Risk

□ RAG pipeline registered in model inventory

□ Independent validation completed

□ Faithfulness evaluation framework in place

□ Human review layer confirmed
Operations

□ Fallback if Vector DB unavailable

□ Monitoring dashboard in place

□ Revision rate tracking enabled

□ Re-indexing pipeline automated

---

## Key Takeaway for PMs

RAG is the most practical way to build 
AI products on your organisation's data —
without fine tuning, without retraining,
and without exposing PII to external models.

The quality of your RAG pipeline depends on:
- Quality of your knowledge base
- Quality of your preprocessing
- Quality of your chunking strategy
- Quality of your retrieval (hybrid + reranker)
- Quality of your system instructions
- Rigour of your evaluation framework

RAG is not a one time build —
it is a product that needs continuous improvement
like any other feature you ship.


## Real World Example 2 — PRD Generator

### The Problem
Writing PRDs from scratch is time consuming —
pulling together user stories, requirements,
meeting notes, diagrams, compliance requirements,
and past PRD templates into one structured document.

### The RAG Pipeline

Knowledge Base (indexed once, updated regularly):

Past PRDs (successful products)
PRD templates and standards
Compliance requirements by product type
User story library
Product principles and OKRs
Regulatory requirements

↓

Azure OpenAI Embeddings → Azure AI Search

Runtime (every PRD generation):
Inputs provided by PM:

Meeting notes
User stories
Requirements
Flow diagrams (multimodal)
Existing PRD drafts

↓

All inputs embedded

↓

Hybrid search retrieves:
Most relevant past PRDs        → 0.95
Applicable PRD template        → 0.92
Relevant compliance reqs       → 0.89
Related user stories           → 0.86
Applicable regulations         → 0.83

↓

Prompt assembled:

[System Instruction]

You are an expert product manager in

financial services. Generate a complete PRD

following our standard template.

Include: problem statement, user stories,

success metrics, compliance considerations,

technical requirements, and GTM approach.

Use ONLY provided context and inputs.

Flag any compliance gaps.
[Retrieved Context]

Similar past PRD: Card Activation feature...

PRD template: Standard format v3.1...

Compliance requirements: PCI DSS, GDPR...

Related user stories: Activation flow...
[PM Inputs]

Meeting notes: "Customer wants to..."

User stories: "As a customer I want to..."

Flow diagram: [multimodal — diagram image]

Requirements: "Must support iOS and Android..."

↓

GPT-4 via Azure OpenAI

↓

Complete structured PRD:

Problem Statement
Goals and OKRs
User Stories
Success Metrics
Functional Requirements
Non Functional Requirements
Compliance Considerations ← auto flagged
Technical Requirements
GTM Approach
Open Questions

↓

PM reviews, edits, approves ✅

### Why RAG Makes PRD Generation Better

| Without RAG | With RAG |
|---|---|
| Generic PRD structure | Follows YOUR organisation's template |
| No compliance flags | Auto flags relevant compliance requirements |
| No precedent | Informed by similar past PRDs |
| PM writes from scratch | PM edits and approves |
| Hours of work | Minutes to first draft |
| Inconsistent quality | Consistent, structured output |

### Multimodal Input — Flow Diagrams

PM uploads backend flow diagram

↓

GPT-4o Vision processes image:

Reads flow steps
Identifies system components
Maps user journey

↓

Combined with text inputs and retrieved context

↓

PRD includes accurate technical flow

based on actual diagram

### Results
- PRD first draft in minutes vs hours
- Consistent format across all products
- Compliance requirements auto surfaced
- Past PRD patterns inform new ones
- PM focuses on strategy not formatting

---

## Comparing Both Pipelines

| | Call Summarisation | PRD Generator |
|---|---|---|
| **Input type** | Audio + Text | Text + Image |
| **Knowledge base** | Fraud cases, procedures | PRDs, templates, compliance |
| **Model** | Whisper + GPT-4 | GPT-4o (multimodal) |
| **Output** | Structured call summary | Complete PRD document |
| **User** | Fraud servicing agent | Product Manager |
| **RAG benefit** | Policy compliant summaries | Organisation aligned PRDs |
| **Human review** | Agent approves | PM reviews and edits |
| **Update frequency** | Nightly (new cases) | On change (new PRDs/policies) |

---



---

**Next: [12 — Vector Databases →](12-vector-db.md)**

