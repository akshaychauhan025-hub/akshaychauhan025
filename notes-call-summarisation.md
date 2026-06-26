# Notes — Call & Notes Summarisation: E2E Pipeline

## Overview

An enterprise grade AI pipeline that captures fraud 
servicing calls, converts speech to text, strips PII, 
classifies call type, retrieves relevant context via RAG, 
and generates structured summaries — automatically 
populated into the Global Fraud Servicing (GFS) system.

---

## End to End Architecture

LAYER 1 — VOICE CAPTURE

↓

LAYER 2 — SPEECH TO TEXT

↓

LAYER 3 — PII STRIPPING

↓

LAYER 4 — CALL CLASSIFICATION

↓

LAYER 5 — RAG RETRIEVAL

↓

LAYER 6 — SUMMARISATION

↓

LAYER 7 — OUTPUT TO GFS

↓

LAYER 8 — OBSERVABILITY

---

## Layer 1 — Voice Capture

Genesys / Nuance Voice AI Gateway

↓

Captures the fraud servicing call

Stores raw audio to secure storage

(Azure Blob Storage — encrypted at rest)

↓

Triggers downstream pipeline on call end

**Key considerations:**
- Audio stored with case ID metadata
- Retention policy aligned with regulatory requirements
- Access control — only authorised systems can read audio

---

## Layer 2 — Speech to Text

Nuance STT (Speech to Text)

↓

Converts call audio → raw transcript

↓

Confidence scoring per word/phrase:

High confidence   → transcript used as-is

Low confidence    → flagged for human review

↓

Speaker diarisation:

Agent:    "Can I take your card number?"

Customer: "I didn't make this transaction"

↓

Labelled transcript output

**Why speaker diarisation matters:**

Without diarisation:

"didn't make this transaction"

→ unclear who said it

→ LLM may attribute to wrong party

→ Incorrect summary
With diarisation:

Customer: "I didn't make this transaction"

→ clearly customer's claim

→ accurate summary

**Output:**

{

call_id: "CALL-2024-00123",

duration: "4:32",

transcript: [

{speaker: "Agent",    text: "Thank you for calling..."},

{speaker: "Customer", text: "I didn't authorise..."},

{speaker: "Agent",    text: "I can see the transaction..."}

],

confidence_scores: [0.97, 0.94, 0.98],

low_confidence_flags: []

}

---

## Layer 3 — PII Stripping

Three layer defence to ensure zero PII reaches the LLM:

Raw transcript

↓

Layer 1: Nuance built-in PII redaction

(catches structured PII at source)

↓

Layer 2: Azure AI Language (NER)

Named Entity Recognition identifies:

Person names
Organisations
Locations
Dates (DOB)
Phone numbers

↓

Layer 3: Microsoft Presidio

Validation scan — catches anything missed:
Card numbers      → [CARD_NUMBER]
Account numbers   → [ACCOUNT_NUMBER]
SSN               → [SSN]
Email addresses   → [EMAIL]
Addresses         → [ADDRESS]

↓

Clean, PII-free transcript ✅

**What is retained (not PII):**

✅ Transaction amounts  ($247)

✅ Merchant category    (online retailer)

✅ Transaction dates    (12th June)

✅ Fraud type          (Card Not Present)

✅ Call outcomes       (escalated / resolved)

**Audit log entry per call:**

{

call_id: "CALL-2024-00123",

pii_entities_removed: 4,

entity_types: ["CARD_NUMBER", "PERSON", "ADDRESS", "PHONE"],

stripping_layers_passed: 3,

timestamp: "2024-06-12T14:32:00Z"

}

---

## Layer 4 — Fraud Call Classification

Before RAG retrieval — classify the call type to 
ensure correct procedures and past cases are retrieved.

Clean transcript

↓

GPT-4o mini

(small, fast, cheap — classification only)

↓

System Instruction:

"Classify this fraud call into exactly one category:

Card Not Present (CNP) Fraud
Card Present Fraud
Account Takeover
ATM Dispute
Merchant Dispute
Subscription Fraud
Other

Return only the category name. Nothing else."

↓

Output: "Card Not Present (CNP) Fraud"

↓

Used as metadata filter in RAG retrieval

**Why classify before RAG?**

Without classification:

RAG searches all procedures and past cases

→ May retrieve card present procedure

for a CNP fraud call

→ Wrong next steps in summary
With classification:

RAG filters by fraud type FIRST

→ Only retrieves CNP procedures and cases

→ Accurate, relevant summary

---

## Layer 5 — RAG Pipeline

### Knowledge Base Setup

GFS SOR (Oracle DB)

↓

Azure Data Factory (orchestration — nightly)

↓

Azure Databricks / Spark (transformation)

Code mapping     (CNP → Card Not Present Fraud)
Abbreviation expansion
PII stripping    (Microsoft Presidio)
Structured text generation
Chunking         (500 tokens, 50 token overlap)

↓

Azure OpenAI text-embedding-3-large

↓

Azure AI Search (Vector DB)

Knowledge base contains:

✅ Past fraud summaries (by type)

✅ Fraud templates

✅ Historical decisions

✅ Card replacement procedures

✅ Fraud dispute procedures

✅ Compliance policies

✅ Regulatory requirements

### Runtime Retrieval

Classified call type: CNP Fraud

↓

Metadata filter applied:

dispute_type = "card_not_present"

↓

Clean transcript embedded:

Azure OpenAI text-embedding-3-large

↓

Hybrid search — Azure AI Search:

Dense search   → semantic similarity

Sparse search  → keyword matching

↓

Semantic reranker scores results

↓

Top K chunks retrieved:
Similar past CNP case        → 0.96

CNP dispute procedure        → 0.91

Card replacement steps       → 0.88

Summary template (CNP)       → 0.85

Relevant compliance policy   → 0.82

---

## Layer 6 — Summarisation

### Prompt Builder

Assembles all components into final prompt:

PROMPT =

System Instruction

Retrieved Context (RAG)
Clean Transcript
Output Format Template

### System Instruction

You are an expert fraud servicing assistant

for a financial services company.
Your job is to generate a structured call summary

in exactly 7 bullet points.
Rules:

Use ONLY information from the provided transcript

and retrieved context
Do not fabricate any information
Do not create a resolution if one was not reached
Do not paraphrase compliance requirements —

quote verbatim from retrieved context
If information is unclear write "UNCLEAR" —

do not guess
Follow the summary template exactly
Flag any compliance or regulatory considerations
Maximum 2 sentences per bullet point
Temperature: 0.1 — consistency over creativity

Output format:

Follow the exact template provided in context.

### Full Prompt Structure

[SYSTEM INSTRUCTION]

As above
[RETRIEVED CONTEXT]

Similar past case:        {similar_cnp_case}

Applicable procedure:     {cnp_dispute_procedure}

Card replacement steps:   {card_replacement_steps}

Summary template:         {cnp_summary_template}

Compliance policy:        {relevant_compliance_policy}
[CLEAN TRANSCRIPT]

Call ID:      CALL-2024-00123

Duration:     4:32

Call type:    Card Not Present Fraud

Transcript:

Agent:    "Thank you for calling fraud services..."

Customer: "I didn't authorise a transaction of

$247 on [DATE] at [MERCHANT_CATEGORY]"

Agent:    "I can see the transaction on your account.

I'll raise a dispute for you..."
[OUTPUT INSTRUCTIONS]

Generate exactly 7 bullets following the template.

For each bullet:

Use transcript information first
Reference retrieved procedure where applicable
Flag compliance considerations
Mark unclear information as UNCLEAR

### LLM Call

Model:       GPT-4o

Deployment:  Azure OpenAI

(data stays within Azure boundary)

Temperature: 0.1

Max tokens:  500

Top P:       0.9

Timeout:     10 seconds

Retry logic: 3 attempts with exponential backoff

Fallback:    GPT-4o mini if primary unavailable

### Output Format

Call Summary — CALL-2024-00123

Issue: Customer reported an unauthorised transaction

of $247 on [DATE] at an online merchant.
Fraud Type: Card Not Present (CNP) Fraud —

transaction made without physical card present.
Verification: Customer identity verified via

security questions. Card ownership confirmed.
Actions Taken: Dispute raised in GFS.

Transaction flagged for investigation per

CNP dispute procedure v2.3.
Resolution Status: Dispute submitted —

pending merchant investigation.

No resolution reached on this call.
Next Steps: Card replacement initiated.

Customer to receive new card in 3-5 business days.
Follow Up: Customer to be contacted within

5 business days with dispute outcome.

⚠️ COMPLIANCE FLAG: CNP disputes > $200 require

merchant chargeback filing within 45 days

per Regulation E guidelines.

→ Compliance team to confirm timeline.

---

## Layer 7 — Output to GFS

Structured summary validated

↓

Guardrails check passed

↓

Auto populated into GFS fraud case record:

Case overview field    ← bullet 1-2
Actions taken field    ← bullet 4
Resolution field       ← bullet 5
Next steps field       ← bullet 6-7
Compliance flags       ← flagged separately

↓

Agent notification:

"Summary ready for review — Case CALL-2024-00123"

↓

Agent reviews summary

↓

Agent approves or edits

↓

Final summary saved to case record

↓

Audit log entry created

---

## Layer 8 — Observability

### Real Time Monitoring

Splunk / Azure Monitor tracks per call:
Latency metrics:

STT processing time      (target < 30s)
PII stripping time       (target < 2s)
Classification time      (target < 1s)
RAG retrieval time       (target < 2s)
LLM generation time      (target < 5s)
Total pipeline time      (target < 45s)

Quality metrics:

STT confidence score     (target > 90%)
RAG retrieval score      (target > 0.80)
Guardrails pass rate     (target 100%)
Agent revision rate      (target < 20%)

Cost metrics:

Tokens used per call
Cost per call
Daily/monthly spend

Error metrics:

Pipeline failures
LLM timeouts
PII stripping failures
GFS write failures

### Alerting Rules

Alert 1: Pipeline failure rate > 1%

→ Page on-call engineer immediately
Alert 2: Average latency > 60 seconds

→ Notify engineering team
Alert 3: Agent revision rate > 30%

→ Notify PM — prompt quality review needed
Alert 4: PII stripping failure detected

→ CRITICAL — page security team immediately

→ Halt pipeline until resolved
Alert 5: LLM cost spike > 20% day on day

→ Notify PM and Finance

### Observability Dashboard

Real time view:

┌─────────────────────────────────────┐

│ Call Summarisation — Live Dashboard │

├─────────────────────────────────────┤

│ Calls processed today:    1,247     │

│ Avg pipeline latency:     38s ✅    │

│ Guardrails pass rate:     99.8% ✅  │

│ Agent revision rate:      17% ✅    │

│ Cost per call:            $0.043 ✅ │

│ Pipeline errors:          2 ⚠️      │

│ PII failures:             0 ✅      │

└─────────────────────────────────────┘

---

## Guardrails

### Input Guardrails (before LLM)

| Check | Rule | Action if Failed |
|---|---|---|
| Audio quality | Confidence score > 80% | Flag for manual transcription |
| PII check | Zero PII entities in transcript | Block — re-run stripping |
| Transcript length | Min 50 words | Flag as incomplete call |
| Call classification | Must match known category | Default to "Other" — manual review |
| RAG retrieval score | Top result > 0.75 | Fallback to general procedure |
| Token limit | Prompt < 4,000 tokens | Truncate transcript — keep ends |

### Output Guardrails (after LLM)

| Check | Rule | Action if Failed |
|---|---|---|
| Bullet count | Exactly 7 bullets | Regenerate |
| PII in output | Zero PII in summary | Block — strip and regenerate |
| Fabrication check | All facts traceable to transcript | Flag for human review |
| Template adherence | All required fields present | Regenerate missing fields |
| Resolution check | No resolution if not in transcript | Remove resolution bullet |
| Compliance flag | Auto check for regulatory triggers | Add flag if missing |
| Length check | Max 500 tokens | Truncate per section limit |

---

## Evaluation Framework

### Automated Metrics (every call)

| Metric | What it measures | Target | How measured |
|---|---|---|---|
| Faithfulness | Summary matches transcript | > 95% | LLM judge |
| Hallucination rate | Fabricated content | 0% | Fact check vs transcript |
| PII detection rate | PII removed before LLM | 100% | Automated scan |
| Template adherence | All 7 bullets present | 100% | Rule based check |
| Guardrails pass rate | All checks passed | > 99% | Pipeline logs |
| Latency | Total pipeline time | < 45s | Azure Monitor |
| Cost per call | Token cost | < $0.05 | Azure Monitor |

### Human Metrics (sampled weekly)

| Metric | What it measures | Target | How measured |
|---|---|---|---|
| Agent revision rate | How often agent edits | < 20% | GFS edit tracking |
| Revision depth | How much agent changes | < 10% of content | Diff analysis |
| Compliance flag accuracy | Correct flags raised | > 90% | Compliance review |
| Agent satisfaction | Do agents find it useful? | > 4/5 | Weekly survey |
| Handling time reduction | Time saved vs manual | > 20% | Before/after AHT |

---

## Revision Rate Tracking

Revision rate is the most honest signal of quality.

Agent receives AI summary

↓

Agent makes edits

↓

System captures diff:

Words added
Words removed
Sections rewritten

↓

Revision rate calculated:

(words changed / total words) × 100

< 10%  → Excellent — minor tweaks only

10-20% → Good — acceptable quality

20-30% → Needs improvement — review prompt

30%  → Poor — escalate to PM immediately

### Revision Rate by Category

Track separately by fraud type:

CNP Fraud revision rate:       15% ✅

Card Present revision rate:    18% ✅

Account Takeover revision rate: 28% ⚠️

ATM Dispute revision rate:     22% ⚠️

Different revision rates by category reveal 
which fraud types need better RAG context 
or prompt tuning.

---

## Feedback Loop

SIGNAL 1 — Agent approves with no edits

↓

High quality example

↓

Store summary + transcript in Vector DB

as high quality reference case

Future RAG retrieval improves ✅
SIGNAL 2 — Agent makes significant edits

↓

Capture original vs edited version

↓

Diff analysis identifies pattern:

"Agent always adds merchant category"

"Agent always removes resolution statement"

↓

Update system instruction to fix pattern

↓

A/B test new vs old prompt

↓

Deploy if revision rate improves
SIGNAL 3 — Compliance flag missed

↓

Add missed flag type to

compliance checking instruction

↓

All future summaries catch this ✅
SIGNAL 4 — Pipeline error

↓

Root cause analysis:

STT failure? PII failure? LLM timeout?

↓

Fix root cause

↓

Add regression test to pipeline
SIGNAL 5 — Agent marks UNCLEAR

↓

Review source:

Audio quality issue?

STT confidence too low?

PII stripping too aggressive?

↓

Fix at correct layer---

## Prompt Improvement Cycle

Week 1: Baseline revision rate measured

↓

Week 2: Pattern analysis — what do agents change?

↓

Week 3: Prompt updated — A/B test launched

↓

Week 4: Results compared

↓

Lower revision rate → deploy new prompt

Higher revision rate → revert, try again

↓

Repeat monthly

---

## Compliance and Audit

Every call logged in Azure Monitor:
{

call_id:              "CALL-2024-00123",

timestamp:            "2024-06-12T14:32:00Z",

agent_id:             "AGT-456",

fraud_type:           "Card Not Present",
pipeline_log: {

stt_confidence:     0.97,

pii_entities_removed: 4,

classification:     "CNP Fraud",

rag_top_score:      0.96,

chunks_retrieved:   5,

llm_model:          "gpt-4o",

tokens_used:        387,

latency_ms:         38420,

guardrails_passed:  true

},
inputs: {

transcript_hash:    "sha256:abc123...",

prompt_version:     "v2.3"

},
outputs: {

summary_hash:       "sha256:def456...",

compliance_flags:   ["Regulation E — 45 day filing"],

revision_rate:      0.12

},
agent_action: {

reviewed:           true,

approved:           true,

edits_made:         true,

final_summary_hash: "sha256:ghi789..."

}

}

Retained for: regulatory required period
Access: audit team, compliance, legal only

---

## Challenges

### 1. Audio Quality

Poor recording quality

↓

Low STT confidence scores

↓

Inaccurate transcript

↓

Summary based on wrong information

Fix: Audio quality gate before STT.
Minimum quality threshold enforced.
Low quality → manual transcription queue.

### 2. Speaker Diarisation Errors

Agent and customer talk simultaneously

↓

STT confuses speakers

↓

Wrong statements attributed to wrong party

↓

"Agent said they didn't authorise

the transaction" — incorrect

Fix: Diarisation confidence scoring.
Low confidence → flag for human review.
Do not send ambiguous transcript to LLM.

### 3. PII Stripping Too Aggressive

Presidio removes merchant names incorrectly

"[MERCHANT]" instead of "online retailer"

↓

Summary missing context

Agent cannot understand what happened

Fix: Tune Presidio entity rules.
Whitelist non-PII entities.
Test on 1,000 sample transcripts before production.

### 4. Wrong Procedure Retrieved

CNP call classified correctly

↓

RAG retrieves card present procedure

due to low similarity score

↓

Wrong next steps in summary

Fix: Mandatory metadata filter by fraud type.
Minimum retrieval score threshold (> 0.80).
Below threshold → fallback to general procedure
+ flag for human review.

### 5. Hallucination in Resolution Field

Resolution unclear in transcript

↓

LLM generates plausible resolution

"Dispute approved — refund in 5 days"

↓

Resolution was NOT approved on call

↓

Customer expects refund → complaint

→ Regulatory risk

Fix: Hard guardrail on resolution field.
"NEVER create resolution unless explicitly 
stated in transcript."
Temperature: 0.1
Human review mandatory for resolution field.

### 6. Stale Knowledge Base

Fraud procedure updated

↓

Nightly re-indexing not run yet

↓

RAG retrieves old procedure

↓

Summary recommends deprecated steps

→ Compliance risk

Fix: Procedure updates trigger immediate 
re-indexing via webhook.
Never wait for nightly batch for 
compliance critical documents.

### 7. Agent Over-Reliance

Agents stop listening carefully

↓

AI will summarise anyway

↓

Agent misses nuance not captured in summary

↓

Customer experience degrades

Escalations increase

Fix: Train agents — AI is assistant not replacement.
Track escalation rate alongside revision rate.
Escalation spike → retrain agents.

### 8. Latency Spikes

Azure OpenAI peak demand

↓

LLM response time: 15+ seconds

↓

Total pipeline: > 60 seconds

↓

Agent waiting — productivity loss

Fix: Reserved capacity on Azure OpenAI.
Fallback to GPT-4o mini if primary slow.
Async processing — agent can start next call
while summary generates.

---

## Key PM Metrics Dashboard

| Metric | Current | Target | Status |
|---|---|---|---|
| Calls processed daily | 1,247 | 1,500 | 🟡 Scaling |
| Avg pipeline latency | 38s | < 45s | ✅ |
| Agent revision rate | 17% | < 20% | ✅ |
| Handling time reduction | 22% | > 20% | ✅ |
| Guardrails pass rate | 99.8% | 100% | 🟡 |
| PII failures | 0 | 0 | ✅ |
| Cost per call | $0.043 | < $0.05 | ✅ |
| Agent satisfaction | 4.1/5 | > 4/5 | ✅ |

---

## Interview One Liner

> "Our call summarisation pipeline is an 8 layer 
> enterprise RAG system — Genesys captures the call, 
> Nuance STT transcribes with speaker diarisation, 
> three layer PII stripping via Nuance, Azure AI 
> Language and Presidio, GPT-4o mini classifies 
> fraud type, Azure AI Search retrieves relevant 
> procedures and past cases, GPT-4o generates a 
> structured 7 bullet summary at temperature 0.1, 
> auto populated into GFS for agent review. 
> Agent revision rate is our primary quality metric, 
> Splunk and Azure Monitor track latency, cost and 
> accuracy, and every input and output is logged 
> for full regulatory audit trail."
>
> 



















