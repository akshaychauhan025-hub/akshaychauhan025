# 10 — Semantic Search: Deep Dive

## What is Semantic Search?

**Semantic Search** is a search capability that understands 
the **meaning and intent** behind a query — not just 
the exact words used.

Keyword Search          Semantic Search

↓                       ↓

Matches exact words     Matches meaning

"fraud dispute"    →    "fraud dispute" ✅

"chargeback" ✅

"unauthorised charge" ✅

"disputed transaction" ✅

"transaction I didn't make" ✅

> Keyword search finds what you typed.
> Semantic search finds what you meant.

---

## Keyword Search vs Semantic Search

| | Keyword Search | Semantic Search |
|---|---|---|
| **How it works** | Exact string matching | Meaning matching via embeddings |
| **Finds synonyms** | ❌ No | ✅ Yes |
| **Understands intent** | ❌ No | ✅ Yes |
| **Handles typos** | ❌ Limited | ✅ Better |
| **Multilingual** | ❌ No | ✅ Yes (with right model) |
| **Speed** | ✅ Very fast | 🟡 Slightly slower |
| **Infrastructure** | Simple database index | Vector DB required |
| **Cost** | Low | Higher |
| **Best for** | ID lookup, exact reference | Intent, concept, knowledge search |

---

## How Semantic Search Works — Step by Step


Step 1 — Index Building (done once)

All documents embedded and stored in Vector DB
Step 2 — Query arrives

"I didn't authorise this payment"
Step 3 — Query Embedding

Query converted to vector in real time

[0.34, 0.78, 0.12, 0.89, ...]
Step 4 — Vector Search

Cosine similarity calculated against

all document vectors in Vector DB
Step 5 — Ranking

Documents ranked by similarity score:

"unauthorised transaction dispute"  → 0.96 ✅

"fraud chargeback process"          → 0.91 ✅

"card not present fraud"            → 0.87 ✅

"account balance enquiry"           → 0.21 ❌
Step 6 — Results returned

Top K results surfaced to user or passed to LLM

---

## Types of Semantic Search

### 1. Dense Retrieval
Pure embedding based search — everything we've 
described above.

Query → Embed → Cosine search → Results

Fast, accurate for meaning-based search.
Struggles with exact matches (IDs, reference numbers).

### 2. Sparse Retrieval (BM25)
Traditional keyword based ranking — but smarter than 
simple matching. Scores documents by term frequency 
and rarity.

Query → keyword analysis → BM25 scoring → Results

Fast, great for exact matches.
Misses synonyms and intent.

### 3. Hybrid Search
Combines dense and sparse retrieval — best of both worlds.

Query

↓         ↓

Dense     Sparse

Search    Search

↓         ↓

Results   Results

↓   ↓

Reranker

(combines and reranks)

↓

Final Results

**Why hybrid wins in enterprise:**
- Dense handles meaning and intent
- Sparse handles exact IDs and references
- Together they cover all search scenarios

> **Most enterprise search systems use hybrid search** — 
> Azure AI Search, Elasticsearch, and Pinecone all 
> support it natively.

### 4. Reranking
A second model re-evaluates and reranks initial 
search results for higher precision.

Initial search returns top 20 results

↓

Reranker model reads query + each result

↓

Scores relevance more precisely

↓

Returns top 5 most relevant results

Adds latency but significantly improves quality — 
especially for RAG pipelines where precision matters.

---

## Semantic Search Architecture

Documents / Knowledge Base

↓

Chunking (split long docs into sections)

↓

Embedding Model

↓

Vector DB (stores embeddings)
+

User Query

↓

Embedding Model (same model as above)

↓

Vector Search

↓

Reranker (optional)

↓

Top K Results

↓

LLM (for RAG) or

Direct Results (for search UI)

---

## Chunking Strategy — Critical for Search Quality

Long documents must be split into chunks before embedding.
How you chunk determines search quality.

### Fixed Size Chunking
Split every N tokens regardless of content:

Policy document → chunk every 500 tokens

Simple but can split sentences mid-thought — 
losing context at boundaries.

### Sentence Chunking
Split at sentence boundaries:

Policy document → split at full stops

Better context preservation — but chunks vary in size.

### Semantic Chunking
Split where meaning changes — keeping related 
content together:

Policy document → split when topic shifts

Best quality — higher complexity to implement.

### Overlapping Chunks
Add overlap between chunks to preserve context 
at boundaries:

Chunk 1: tokens 1-500

Chunk 2: tokens 450-950   ← 50 token overlap

Chunk 3: tokens 900-1400  ← 50 token overlap

Prevents losing context at chunk boundaries — 
recommended for most RAG pipelines.

---

## Semantic Search in Financial Services

### 1. Agent Knowledge Search

Agent handling fraud dispute asks:

"What evidence do we need for a

high value card not present dispute?"

↓

Semantic search across policy library

↓

Finds relevant policy sections

even if exact words don't match

↓

Agent gets precise answer instantly

### 2. Customer Self Service

Customer types:

"Someone used my card without my permission"

↓

Semantic search understands intent:

fraud report

↓

Surfaces relevant:

How to report fraud
What happens next
Timeline for resolution

↓

Without customer knowing the

right words to search

### 3. Regulatory Compliance Search

Compliance team asks:

"What are our obligations for

notifying customers of data breaches?"

↓

Semantic search across:

Internal policies
Regulatory documents
Past rulings

↓

Surfaces relevant sections

across all sources

↓

LLM synthesises comprehensive answer

### 4. Fraud Pattern Detection

New fraud case description embedded

↓

Searched against historical fraud database

↓

Similar past cases surfaced:

Same merchant pattern → 0.94
Same geography → 0.89
Same amount range → 0.87

↓

Analyst sees similar cases instantly

↓

Faster, more informed decision

### 5. Transaction Dispute Routing

Customer dispute description:

"I cancelled my subscription

but was still charged"

↓

Semantic search against dispute categories

↓

Matches: "Recurring charge after cancellation"

↓

Auto-routed to correct team

↓

No manual reading and routing required

---

## Evaluation — How to Measure Search Quality

As a PM, you need to know if your semantic search 
is actually working. Key metrics:

| Metric | What it measures | How |
|---|---|---|
| **Precision@K** | Of top K results, how many are relevant? | Human evaluation |
| **Recall@K** | Of all relevant docs, how many in top K? | Human evaluation |
| **MRR** | Is the best result ranked first? | Automated |
| **NDCG** | Quality of full result ranking | Automated |
| **Time to answer** | How fast does agent find what they need? | Product analytics |
| **Search abandonment** | How often does user give up? | Product analytics |

> **PM Rule:** Track time to answer and search 
> abandonment before and after launching semantic search. 
> These are the business metrics that prove value.

---

## Common Semantic Search Pitfalls

**1. Wrong Chunking Strategy**
Chunks too large → embeddings average out meaning → 
poor precision.
Chunks too small → lose context → poor recall.

> Test multiple chunk sizes (256, 512, 1024 tokens) 
> and measure precision before committing.

**2. Wrong Embedding Model**
Using a general embedding model for a specialised domain 
(legal, medical, financial) reduces accuracy.

> Fine tune embedding model on domain specific data 
> for significantly better results.

**3. Not Using Hybrid Search**
Dense only search misses exact reference numbers.
Keyword only search misses synonyms.

> Default to hybrid search for enterprise products.

**4. Ignoring Reranking**
Initial vector search returns good candidates — 
not necessarily the best answer.

> Add reranking for RAG pipelines where 
> precision is critical.

**5. Static Index**
Documents updated but embeddings not regenerated — 
search returns stale results.

> Build automated re-embedding pipeline 
> triggered by document updates.

---

## Azure AI Search — Enterprise Semantic Search

For regulated enterprises on Azure:

Azure AI Search provides:

✅ Hybrid search (dense + sparse) out of the box

✅ Built in reranking (semantic ranker)

✅ Integration with Azure OpenAI embeddings

✅ Enterprise security and compliance

✅ 99.9% SLA

✅ Data stays within Azure environment

✅ Full audit logging

**Architecture with Azure:**

Documents

↓

Azure OpenAI Embedding Model

↓

Azure AI Search (Vector DB + Search)

↓

Hybrid search + Semantic Reranker

↓

Top results passed to Azure OpenAI LLM

↓

Grounded, accurate response

↓

Full audit trail in Azure Monitor

---

## Key Takeaway for PMs

Semantic search is the foundation of every 
intelligent search experience — from agent 
knowledge tools to customer self service 
to RAG pipelines.

Getting it right requires:
- Right chunking strategy
- Right embedding model
- Hybrid search (dense + sparse)
- Reranking for precision
- Continuous evaluation

The difference between good and great semantic 
search is not the model — it is the architecture 
and evaluation discipline around it.

---

**Next: [11 — RAG (Retrieval Augmented Generation) →]
(11-rag.md)**

