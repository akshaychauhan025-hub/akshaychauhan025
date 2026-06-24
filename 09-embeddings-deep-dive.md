# 09 — Embeddings: Deep Dive

## What is an Embedding?

An **embedding** is a numerical representation of meaning.

It converts text (words, sentences, documents) into a list 
of numbers — called a **vector** — that captures the semantic 
meaning of that text in a way computers can process and compare.

"Fraud dispute"    → [0.23, 0.87, 0.41, 0.12, 0.67, ...]

"Chargeback"       → [0.24, 0.85, 0.43, 0.11, 0.65, ...]

"Pizza recipe"     → [0.91, 0.12, 0.03, 0.78, 0.22, ...]

**Fraud dispute** and **Chargeback** have similar numbers 
because they have similar meaning.

**Pizza recipe** has completely different numbers because 
it means something entirely different.

> Embeddings turn meaning into maths.

---

## Why Embeddings Matter

Before embeddings, computers matched text by exact keywords:


Search: "fraud dispute"

Keyword match finds: "fraud dispute" ✅

Keyword match misses: "chargeback" ❌

"disputed transaction" ❌

"unauthorised charge" ❌

With embeddings, computers match by meaning:

Search: "fraud dispute"

Embedding search finds: "fraud dispute" ✅

"chargeback" ✅

"disputed transaction" ✅

"unauthorised charge" ✅

This is the foundation of **semantic search** — 
understanding intent, not just matching words.

---

## How Embeddings Are Created

### Step 1 — Tokenisation
Text is broken into tokens (words or subwords):

"The customer disputed the charge"

→ ["The", "customer", "disputed", "the", "charge"]

### Step 2 — Token Embeddings
Each token is converted to a vector by the embedding model:

"customer"  → [0.12, 0.45, 0.78, 0.23, ...]

"disputed"  → [0.34, 0.67, 0.12, 0.89, ...]

"charge"    → [0.56, 0.23, 0.45, 0.67, ...]

### Step 3 — Contextual Embeddings
The Transformer processes all token vectors together 
using Attention — producing contextual embeddings 
that capture meaning in context, not just in isolation.

"charge" in "disputed the charge"    → financial meaning

"charge" in "charge the battery"     → electrical meaning

Same word — different embedding — because context is different.

### Step 4 — Sentence / Document Embedding
All token embeddings are pooled into a single vector 
representing the entire sentence or document:

"The customer disputed the charge"

→ [0.23, 0.87, 0.41, 0.12, 0.67, 0.34, ...]

← one vector representing the whole sentence →

---

## Vector Dimensions — Going Deeper

Each embedding is a vector with hundreds or thousands 
of dimensions — each dimension capturing a different 
aspect of meaning.

text-embedding-3-large → 3,072 dimensions

text-embedding-ada-002 → 1,536 dimensions

Sentence Transformers  → 384-768 dimensions

**What do dimensions represent?**

No single dimension has a human-readable meaning — 
they collectively encode:
- Topic (finance, technology, healthcare)
- Sentiment (positive, negative, neutral)
- Intent (question, complaint, request)
- Entity type (person, organisation, product)
- Relationship between concepts
- And thousands of other semantic properties

---

## Cosine Similarity — How Embeddings Are Compared

To find similar meanings, embeddings are compared 
using **cosine similarity** — measuring the angle 
between two vectors.

Same meaning    → angle close to 0°  → similarity score ~1.0

Related meaning → angle ~45°         → similarity score ~0.7

Different meaning → angle ~90°       → similarity score ~0.0

Opposite meaning → angle ~180°       → similarity score ~-1.0

**Real example:**

Query: "customer reported unauthorised transaction"
Cosine similarity scores:

"fraud dispute"              → 0.94 ✅ very similar

"chargeback request"         → 0.91 ✅ very similar

"card not present fraud"     → 0.87 ✅ similar

"account balance enquiry"    → 0.31 ❌ not similar

"pizza delivery complaint"   → 0.04 ❌ completely different

Set a threshold (e.g. > 0.80) and you have a 
powerful semantic search or classification system.

---

## Types of Embedding Models

### Text Embedding Models
Convert text to vectors for search, RAG, and classification.

| Model | Dimensions | Best For |
|---|---|---|
| text-embedding-3-large | 3,072 | Highest accuracy, complex search |
| text-embedding-3-small | 1,536 | Balanced cost and accuracy |
| text-embedding-ada-002 | 1,536 | Legacy, widely deployed |
| BGE-large | 1,024 | Open source, strong performance |
| E5-large | 1,024 | Open source, multilingual |
| Sentence Transformers | 384-768 | Lightweight, self hosted |

### Multimodal Embedding Models
Convert both text and images into the same vector space — 
enabling cross-modal search.

| Model | Company | Capability |
|---|---|---|
| CLIP | OpenAI | Text + Image in same vector space |
| ImageBind | Meta | Text + Image + Audio + Video |
| Gecko | Google | Text + Image, multilingual |

**Cross modal search example:**

Search with text: "customer holding card at ATM"

CLIP finds:       matching images from CCTV footage

### Code Embedding Models
Specialised for understanding and searching code.

| Model | Best For |
|---|---|
| text-embedding-3-large | General code search |
| CodeBERT | Code understanding, bug detection |
| StarEncoder | Code search, clone detection |

---

## Embedding Model Selection for Enterprise

| Factor | Consideration |
|---|---|
| **Accuracy** | More dimensions = generally more accurate |
| **Cost** | More dimensions = more storage + compute cost |
| **Latency** | Smaller models = faster embedding generation |
| **Language** | Multilingual models for global products |
| **Modality** | Multimodal if handling images + text |
| **Compliance** | Self hosted embedding models for PII data |

**For regulated financial services:**

PII in documents?

↓

Yes → Self hosted embedding model

(Sentence Transformers, BGE)

Data never leaves environment

↓

No  → OpenAI text-embedding-3-large

via Azure OpenAI

---

## Embeddings in Financial Services

### 1. Fraud Detection

Historical fraud transaction descriptions

↓

Embedded and stored in Vector DB

↓

New transaction arrives

↓

Embedded in real time

↓

Cosine similarity search:

"Does this match known fraud patterns?"

↓

High similarity → flag for review

Low similarity  → approve

### 2. Policy and Compliance Search

Entire regulatory policy library embedded

↓

Stored in Vector DB

↓

Compliance officer asks:

"What is our policy on cross border transactions?"

↓

Query embedded

↓

Cosine search finds most relevant policy sections

↓

LLM synthesises answer from retrieved sections

### 3. Dispute Classification

Customer raises dispute

↓

Dispute description embedded

↓

Compared to embeddings of dispute categories:

Card not present fraud
Merchant error
Subscription cancellation
ATM dispute

↓

Highest cosine similarity = correct category

↓

Routed to correct team automatically

### 4. Customer Intent Detection

Customer message: "I didn't make this payment"

↓

Embedded in real time

↓

Compared to intent embeddings:

Fraud report      → 0.94 ✅
Payment query     → 0.31
Balance enquiry   → 0.12

↓

Intent: Fraud report

↓

Routed to fraud servicing flow

### 5. Knowledge Base Search (RAG)

Agent asks: "What documents does customer need

for a high value dispute?"

↓

Query embedded

↓

Vector DB searched for relevant policy

↓

Top 3 policy sections retrieved

↓

LLM generates precise answer

↓

Agent gets accurate, grounded response

---

## Embeddings vs Keywords — When to Use Each

| Scenario | Keywords | Embeddings |
|---|---|---|
| Exact ID or reference number search | ✅ Keywords | ❌ |
| Finding synonyms and related concepts | ❌ | ✅ Embeddings |
| Regulatory compliance search | ❌ | ✅ Embeddings |
| Customer intent detection | ❌ | ✅ Embeddings |
| Fraud pattern matching | ❌ | ✅ Embeddings |
| Simple database lookup | ✅ Keywords | ❌ |
| Semantic document classification | ❌ | ✅ Embeddings |

> **PM Rule:** If the user knows exactly what they're 
> looking for — keywords. If they know what they 
> mean but not the exact words — embeddings.

---

## Common Embedding Pitfalls for PMs

**1. Embedding Staleness**
Embeddings are generated at a point in time. 
If your documents change — embeddings must be regenerated.

> Build a re-embedding pipeline for updated content.

**2. Dimension Mismatch**
You cannot mix embeddings from different models — 
they live in different vector spaces.

> Pick one embedding model and stick to it across 
> your entire pipeline.

**3. Context Length Limits**
Embedding models have token limits — long documents 
must be chunked before embedding.

Long policy document (50 pages)

↓

Split into chunks (500 tokens each)

↓

Each chunk embedded separately

↓

All chunks stored in Vector DB

↓

Search retrieves most relevant chunks

**4. Language Mismatch**
Standard embedding models work best in English. 
For multilingual products — use multilingual 
embedding models (E5, mBERT).

**5. Cost at Scale**
Embedding 10M documents at launch is a one-time cost.
Embedding every new document in real time is ongoing.

> Budget for both initial embedding and 
> ongoing re-embedding costs.

---

## Key Takeaway for PMs

Embeddings are the bridge between human language 
and machine understanding.

Every semantic search, RAG pipeline, intent detection 
system, and fraud pattern matcher is built on embeddings.

Understanding how they work — and where they fail — 
lets you design better AI products, ask better 
questions of your engineering team, and make smarter 
decisions about model selection and architecture.

Your application
        ↓
Azure Private Network
        ↓
Azure OpenAI / Azure AI Studio
        ↓
Embedding model runs in your Azure region
        ↓
Vector returned to your application
        ↓
Stored in Azure Vector DB
(Azure AI Search or PostgreSQL with pgvector)
        ↓
Data never leaves your Azure environment

---

**Next: [10 — Semantic Search →](10-semantic-search.md)**

