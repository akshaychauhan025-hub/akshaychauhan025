# 06 — Enterprise Model Selection Framework

## Why Model Selection is a Product Decision

Choosing the wrong model doesn't just affect quality — it can:
- Expose customer PII to external servers
- Fail regulatory audit
- Create unpredictable outputs at scale
- Lock you into a vendor with no exit strategy
- Cost millions more than budgeted at scale

> Model selection is not an engineering decision. 
> It is a product, commercial, and compliance decision 
> that engineering executes.

---

## The 7 Layer Enterprise Selection Framework

Before committing to any AI model, evaluate across all 7 layers:

Layer 1 — Use Case

Layer 2 — Scale

Layer 3 — Cost

Layer 4 — Latency

Layer 5 — Compliance

Layer 6 — Build vs Buy

Layer 7 — Fallback

---

## Layer 1 — Use Case

Start here. The use case determines everything else.

**Key questions:**
- What modalities do you need? Text, image, audio, video?
- Are you generating text or classifying it?
- Is the output customer facing or internal?
- Does the model need to reason deeply or respond quickly?

**Model mapping by use case:**

| Use Case | Best Model | Why |
|---|---|---|
| General Q&A / Chat | GPT-4o, Claude 3.5 Sonnet | Strong reasoning, instruction following |
| Long document analysis | Claude 3.5 Sonnet | 200K context window |
| Summarisation | GPT-4o, Claude 3.5 Sonnet | Consistent structured outputs |
| Classification / Search | BERT, embedding models | Encoder only — fast, cheap, accurate |
| Audio transcription | Whisper | Purpose built for speech |
| Video understanding | Gemini 1.5 Pro | Native video processing |
| Image + Text | GPT-4o, Gemini 1.5 Pro | Strong vision + language |
| Real time decisions | GPT-4o mini, Mistral, Phi-3 | Small, fast, cheap |
| Complex reasoning | GPT o1, Claude extended thinking | Step by step thinking |
| RAG pipelines | Command R+, Llama 3 | Optimised for retrieval |
| Embeddings | text-embedding-3-large, BGE | Purpose built for vector similarity |
| On device / Edge | Phi-3, Gemini Nano, Llama 3.1 8B | Runs without cloud |

---

## Layer 2 — Scale

What works for 100 users breaks at 10 million.

**Key questions:**
- How many requests per day at peak?
- What is the average input/output length in tokens?
- Is traffic consistent or spiky?

**Scale considerations:**


Low scale (< 10K requests/day)

→ Closed source API fine

→ Cost manageable

→ No infrastructure needed

Medium scale (10K - 1M requests/day)

→ Evaluate open source vs closed source cost

→ Consider reserved capacity with vendor

→ Start optimising prompt length

High scale (> 1M requests/day)

→ Open source self hosted likely cheaper

→ Need dedicated infrastructure team

→ Caching, batching, compression become critical

**Real example — Video at Scale:**

WhatsApp — 100M videos per day at 30 FPS

= 180 billion frames to process daily
Solution:

Frame sampling (not every frame)
Tiered models (lightweight first, heavy only when needed)
Specialised models per task (not one model for everything)

---

## Layer 3 — Cost

Cost of AI has three components most PMs miss:

**1. Inference Cost**
Cost per API call — charged per token (input + output).

GPT-4o         → $5 per 1M input tokens

GPT-4o mini    → $0.15 per 1M input tokens

Claude 3.5     → $3 per 1M input tokens

Llama 3 (self) → ~$0.10 per 1M tokens (infrastructure cost)

At 1M requests/day with 1K tokens each:

GPT-4o         → $5,000/day   → $1.8M/year

GPT-4o mini    → $150/day     → $55K/year

Llama 3 (self) → ~$100/day    → $36K/year

**2. Infrastructure Cost**
Self hosted open source models need GPU servers:
- Setup cost: high
- Running cost: lower per call at scale
- Maintenance cost: your engineering team

**3. Hidden Costs**
- Fine tuning — training a model on your data
- Prompt engineering — iterations before production quality
- Evaluation — testing outputs at scale
- Human review layer — agents reviewing AI outputs
- MRM validation — independent model validation cost

> **PM Rule:** Always calculate Total Cost of Ownership (TCO) 
> — not just API cost. Hidden costs often exceed inference cost.

---

## Layer 4 — Latency

Latency = time between sending prompt and receiving response.

**Why it matters:**
- Fraud detection needs a decision in < 300ms
- Customer chat expects response in < 2 seconds
- Background summarisation can wait 10-30 seconds

**Latency by model type:**

| Model | Typical Latency | Best For |
|---|---|---|
| GPT-4o mini | 300-800ms | Real time, customer facing |
| Mistral 7B | 200-500ms | High speed, low cost |
| Phi-3 | 100-300ms | Edge, on device |
| GPT-4o | 1-3 seconds | Quality over speed |
| Claude 3.5 Sonnet | 1-4 seconds | Long documents |
| GPT o1 | 10-30 seconds | Deep reasoning tasks |

**Latency optimisation strategies:**
- Use smaller models for simple tasks
- Cache frequent responses
- Stream outputs (show text as it generates)
- Batch non-urgent requests
- Run model closer to user (edge deployment)

> **PM Rule:** Match model size to task complexity. 
> Using GPT-4o for a simple classification task is like 
> using a truck to deliver a letter.

---

## Layer 5 — Compliance

In regulated environments, compliance is often the deciding factor.

### The 7 Reasons Models Get Rejected

**1. Data Residency Violation**
Model processes data outside approved country/region.

| Model | Risk | Mitigation |
|---|---|---|
| OpenAI direct API | 🔴 High | Use Azure OpenAI instead |
| Gemini direct API | 🔴 High | Use Google Vertex AI instead |
| Azure OpenAI | 🟢 Low | Runs in your Azure region |
| Self hosted Llama | 🟢 Low | Your environment, full control |

**2. PII Exposure**
Customer data sent to external model without masking.

> Rule: Always strip PII before any data leaves 
> your environment — names, card numbers, account 
> details, addresses, phone numbers.

**3. No Audit Trail**
Cannot log every input and output for regulatory review.

| Model | Audit Capability |
|---|---|
| OpenAI direct API | 🟡 Limited — logs owned by OpenAI |
| Azure OpenAI | 🟢 Full logging in your environment |
| Self hosted | 🟢 You control all logs |

**4. Model Explainability**
Black box decisions not acceptable for customer-facing outcomes.

> This is why fraud DECISIONS are made by traditional 
> ML models — LLMs assist, they do not decide.

LLM                    Traditional ML

↓                        ↓

Assists agent          Makes the decision

Summarises call        Approves/declines claim

Suggests resolution    Sets credit limit

↓                        ↓

Human reviews          MRM validated

↓                        ↓

Human decides          Auditable, explainable

**5. No SLA Guarantee**

| Model | SLA |
|---|---|
| OpenAI direct API | 🔴 No enterprise SLA on standard tier |
| Azure OpenAI | 🟢 Microsoft enterprise SLA 99.9%+ |
| AWS Bedrock | 🟢 AWS enterprise SLA |

**6. Vendor Security Assessment Failure**
Vendor fails internal security review — missing SOC2, 
ISO27001, or penetration testing certification.

**7. Model Versioning**
Vendor updates model without notice — outputs change overnight.

| Model | Version Control |
|---|---|
| OpenAI direct API | 🔴 Updates without notice |
| Azure OpenAI | 🟢 Pin to specific version |
| Self hosted | 🟢 You control updates |

---

## Model Risk Management (MRM)

Every AI model used in a regulated financial institution 
must go through **Model Risk Management** — a formal 
framework to control risks from model-based decisions.

Governed by:
- **SR 11-7** — US Federal Reserve
- **SS1/23** — Bank of England
- **EBA Guidelines** — European Banking Authority

### 3 Core MRM Requirements

**1. Model Validation**
Independent team validates the model before deployment:
- Is it accurate?
- Is it fair across customer segments?
- Does it do what it claims?

**2. Model Inventory**
Every model registered in a central inventory:
- What does it do?
- Who owns it?
- When was it last validated?

**3. Ongoing Monitoring**
Continuous monitoring post deployment:
- Performance tracked against baseline
- Model drift detected and flagged
- Below threshold → model pulled or retrained

### MRM Challenges Specific to LLMs

| MRM Requirement | LLM Challenge |
|---|---|
| Validation | No single right answer — how do you validate free text? |
| Explainability | Cannot explain token by token decisions to regulators |
| Fairness | Hard to prove equal treatment across customer segments |
| Inventory | Must register every GPT integration as a model |
| Monitoring | Need evaluation framework for output quality at scale |

### Internal Stakeholders for AI Sign-Off

In a regulated enterprise, you need approval from:

| Stakeholder | What They Care About |
|---|---|
| **Legal** | Liability, IP, vendor contracts, data rights |
| **Compliance** | Regulatory requirements, audit readiness, PII |
| **Engineering** | Integration, infrastructure, security, SLAs |
| **Model Risk** | Validation, explainability, monitoring framework |
| **Finance** | TCO, cost at scale, ROI |

> **PM insight:** Bring Legal and Compliance in at the 
> start — not at the end. Every week they are excluded 
> is a week of rework when they find issues at launch.

---

## Layer 6 — Build vs Buy

**The honest framework:**

Buy (third party API)           Build (self hosted / fine tune)

↓                                  ↓

Faster to market                Full data control

Lower upfront cost              Lower cost at scale

Vendor manages updates          You manage updates

Data leaves environment         Data stays internal

Vendor lock-in risk             Full flexibility

Best for: prototyping,          Best for: scale, compliance,

speed, low volume               regulated data, differentiation
 
**Decision framework:**

Is compliance the #1 concern?

→ Yes → Self hosted open source or enterprise cloud wrapper

Is speed to market the #1 concern?

→ Yes → Buy (Azure OpenAI, AWS Bedrock)

Is cost at scale the #1 concern?

→ Yes → Evaluate open source at your volume

Do you need custom behaviour on proprietary data?

→ Yes → Fine tune open source model

Are you prototyping?

→ Yes → Always buy first, build later

> **PM Rule:** Prototype with closed source. 
> Scale with open source. Never build what you can buy 
> until scale and compliance demand it.

---

## Layer 7 — Fallback

What happens when the model fails, is slow, or returns 
a bad output? This is the layer most PMs forget — 
until it becomes an incident.

**Fallback strategies:**

**1. Model Fallback**
Primary model fails → automatically route to backup model

GPT-4o fails

↓

Route to Claude 3.5 Sonnet

↓

If that fails → route to GPT-4o mini

↓

If that fails → human agent handles

**2. Graceful Degradation**
AI feature unavailable → product still works without it

Call summarisation AI down

↓

Agent manually writes summary

↓

AI feature shows "unavailable" — not a blank screen

**3. Confidence Thresholds**
Low confidence output → don't show it, escalate to human

Model confidence < 70%

↓

Don't surface AI output

↓

Route to human review

**4. Caching**
Store frequent responses — serve from cache when model is slow

**5. Circuit Breaker**
If model error rate exceeds threshold → stop sending 
requests, activate fallback immediately

---

## The Enterprise Model Selection Checklist

Before approving any AI model for production:

Use Case

□ Modalities confirmed (text, image, audio, video)

□ Generation vs classification confirmed

□ Output format defined
Scale

□ Peak requests per day estimated

□ Average token count calculated

□ Traffic pattern understood (consistent vs spiky)
Cost

□ Inference cost calculated at scale

□ Infrastructure cost included

□ Hidden costs identified (fine tuning, evaluation, MRM)
Latency

□ Latency requirement defined (ms or seconds)

□ Model latency tested under load

□ Streaming considered for UX
Compliance

□ PII stripping layer confirmed

□ Data residency requirement met

□ Audit logging in place

□ MRM validation planned

□ Legal, Compliance, Engineering signed off

□ Vendor security assessment passed

□ Model version pinned
Build vs Buy

□ Build vs buy decision made with TCO analysis

□ Vendor contract reviewed by Legal

□ Exit strategy defined
Fallback

□ Fallback model identified

□ Graceful degradation designed

□ Confidence threshold set

□ Circuit breaker implemented

□ Human review layer confirmed


---

## Key Takeaway for PMs

The best model is not the most powerful model — 
it is the model that fits your use case, 
passes your compliance requirements, 
works at your scale, 
fits your cost envelope, 
and has a fallback when it fails.

In regulated environments — compliance always wins. 
Build your model selection process around it, 
not as an afterthought.

**Human Review Layer**
Human agents (fraud analysts, customer service staff) 
validate AI outputs before decisions are made or 
surfaces to customers. Critical in regulated environments 
where AI output has financial or compliance consequence.

**AI Agent Review Layer**
A second AI model evaluates the first model's output — 
checking for quality, accuracy, hallucinations, and 
policy violations before surfacing to humans.

AI Output

↓

AI Agent Review    ← catches errors at scale automatically

↓

Human Review       ← validates before consequential decisions

↓

Final Decision

---

**Next: [07 — Reasoning Models →](07-reasoning-models.md)**

