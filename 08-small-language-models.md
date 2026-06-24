# 08 — Small Language Models (SLMs): Deep Dive

## What is a Small Language Model?

A **Small Language Model (SLM)** is a compact, efficient AI model 
designed to run on limited hardware with lower cost and latency — 
optimised for specific tasks rather than general purpose capability.

LLM                          SLM

GPT-4 — 1.7T parameters     Phi-3 — 3.8B parameters

Requires massive GPU cluster  Runs on a laptop or phone

General purpose              Task specific

High cost                    Low cost

High latency                 Low latency

> Size is not everything. A well trained SLM often outperforms 
> a large general model on the specific task it was built for.

---

## Why SLMs Exist

LLMs are powerful but come with real constraints:

- **Cost** — $5 per 1M tokens adds up fast at scale
- **Latency** — 1-3 seconds too slow for real time use cases
- **Privacy** — data must leave device to reach cloud model
- **Connectivity** — cloud models need internet — not always available
- **Overkill** — using GPT-4 to classify a sentence is like 
  using a truck to deliver a letter

SLMs solve all of these — at the cost of some capability.

---

## How SLMs Are Built

### 1. Training from Scratch
Build a smaller model with fewer parameters from the beginning.

Massive dataset

↓

Train model with fewer parameters

↓

Smaller model — less capable generally

↓

Fine tune on specific task

↓

Highly capable on that task specifically

---

### 2. Knowledge Distillation

The most important technique — a large model teaches a small model.

Large Teacher Model (GPT-4)

↓

Generates high quality outputs

on thousands of examples

↓

Small Student Model

learns to replicate those outputs

↓

Student captures ~80% of teacher capability

at 10% of the size

> Think of it as a senior expert training a junior — 
> the junior won't know everything the senior knows 
> but becomes very good at the core job.

**Why this works:**
The large model's outputs contain richer information 
than just the correct answer — they encode confidence, 
nuance, and reasoning patterns the small model absorbs.

---

### 3. Quantisation

Reducing the precision of model weights to compress size 
without retraining.

Standard model weights: 32-bit numbers

↓

Quantised to: 8-bit or 4-bit numbers

↓

Model size: 4-8x smaller

Speed: significantly faster

Quality loss: minimal for most tasks

**Real world impact:**

Llama 3 70B full precision  → requires 8 x A100 GPUs

Llama 3 70B quantised (4bit) → runs on 2 x consumer GPUs

Makes powerful models accessible without enterprise 
GPU infrastructure.

---

### 4. Pruning

Removing redundant connections in the model that 
contribute little to output quality.

Full model: 1 billion connections

↓

Identify connections with minimal impact

↓

Remove them

↓

Pruned model: 700M connections

↓

Retrain briefly to recover quality

↓

Smaller, faster, nearly as capable

---

## SLMs vs LLMs — When Each Wins

| Scenario | SLM | LLM |
|---|---|---|
| Simple classification | ✅ SLM wins | Overkill |
| Sentiment analysis | ✅ SLM wins | Overkill |
| Keyword extraction | ✅ SLM wins | Overkill |
| Real time response < 500ms | ✅ SLM wins | Too slow |
| On device / no internet | ✅ SLM wins | Not possible |
| High volume, low cost | ✅ SLM wins | Too expensive |
| Complex reasoning | ❌ LLM wins | SLM struggles |
| Long document analysis | ❌ LLM wins | Context too small |
| Creative generation | ❌ LLM wins | SLM limited |
| Multi-step problem solving | ❌ LLM wins | SLM struggles |

---

## Edge Deployment — Running AI Without the Cloud

**Edge deployment** means running an AI model directly 
on the device — phone, laptop, ATM, IoT sensor — 
without sending data to a cloud server.

Cloud Model (LLM)              Edge Model (SLM)

↓                              ↓

Data sent to server            Data stays on device

Server processes               Device processes

Response sent back             Response instant

Latency: 1-3 seconds           Latency: 50-200ms

Needs internet                 Works offline

Privacy risk                   No privacy risk

**Why this matters for financial services:**

ATM fraud detection

↓

SLM running on ATM hardware

↓

Analyses transaction in real time

↓

Decision in < 100ms

↓

No data sent to cloud

↓

Works even if internet is down

---

## Current SLMs Worth Knowing

| Model | Company | Parameters | Best For |
|---|---|---|---|
| Phi-3 Mini | Microsoft | 3.8B | On device, real time tasks |
| Phi-3 Medium | Microsoft | 14B | Balanced capability + speed |
| Gemini Nano | Google | ~1.8B | Mobile devices, Android |
| Llama 3.1 8B | Meta | 8B | Self hosted, fine tuning |
| Mistral 7B | Mistral | 7B | Fast, efficient, open source |
| TinyLlama | Open Source | 1.1B | Ultra lightweight deployment |

---

## SLMs in Financial Services

**1. Real Time Fraud Detection**

Transaction initiated

↓

SLM on device or edge server

analyses transaction pattern

↓

Risk score in < 100ms

↓

Flag or approve

↓

Full LLM analysis only if flagged

**2. ATM and Branch Interactions**

Customer interacts with ATM

↓

SLM running locally

understands intent

↓

No cloud dependency

↓

Works offline

↓

Escalates to cloud only for complex requests

**3. Mobile Banking Assistant**

Customer opens banking app

↓

SLM running on phone (Gemini Nano)

handles simple queries

↓

"What's my balance?" → answered locally

"Why was I charged?" → escalated to cloud LLM

**4. Document Classification at Scale**

10M documents per day

↓

SLM classifies each document

(invoice, statement, ID, form)

↓

Cost: fraction of LLM

Speed: milliseconds per document

↓

Only complex documents escalated to LLM

---

## The Tiered Model Architecture

The most efficient enterprise AI systems use 
SLMs and LLMs together in a tiered structure:

Request arrives

↓

Tier 1: SLM (fast, cheap)

Can this be handled simply?

↓

Yes → SLM responds (90% of requests)

No  → escalate to Tier 2

↓

Tier 2: LLM (powerful, expensive)

Handles complex requests (10% of requests)

↓

Still complex? → escalate to Tier 3

↓

Tier 3: Reasoning Model

Handles hardest problems (1% of requests)

**Cost impact:**

Without tiering: 100% requests → LLM → high cost

With tiering:    90% requests  → SLM → low cost

9% requests   → LLM → medium cost

1% requests   → Reasoning → high cost
Overall cost: 60-70% lower

---

## SLMs and Compliance

SLMs have a natural compliance advantage:

| Compliance Concern | SLM Advantage |
|---|---|
| Data residency | Runs on device — data never leaves |
| PII exposure | No external API call — no exposure risk |
| Audit trail | All processing local — fully logged |
| Latency for real time decisions | Fast enough for < 100ms requirements |
| Offline capability | Works without internet — no dependency |

---

## Key Takeaway for PMs

SLMs are not inferior LLMs — they are purpose built 
tools for specific jobs.

The best AI product architectures use both:
- SLMs for speed, cost, privacy, and scale
- LLMs for complexity, reasoning, and quality
- Reasoning models for the hardest problems

Knowing when to use each — and how to tier them — 
is what separates good AI PMs from great ones.

---

**Next: [09 — Embeddings Deep Dive →](09-embeddings-deep-dive.md)**

