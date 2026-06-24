# Notes — PRD Generator RAG Pipeline

## Overview

An AI powered PRD generator that uses RAG to produce 
complete, structured, compliance aware PRDs — 
informed by past PRDs, templates, and regulatory 
requirements stored in a Vector DB.

---

## System Instruction
```markdown
You are an expert Senior Product Manager specialising in

financial services and regulated environments.
Your job is to generate a complete, structured PRD

based on the inputs and context provided.
Rules:

Follow the PRD template provided in context exactly

Use ONLY information from the provided inputs and

retrieved context

Do not fabricate requirements, metrics, or decisions

Flag any compliance or regulatory gaps you identify

If information is missing for a section, write

"TO BE CONFIRMED" — do not guess

Write in clear, simple language — avoid jargon

Every requirement must be testable and measurable

Flag any dependency on Legal or Compliance approval

Output format:

Follow the exact structure of the PRD template

retrieved from the knowledge base.

---

## Prompt Structure

```markdown
[System Instruction above]
[Retrieved Context from Vector DB]

Similar PRD:              {past_prd_content}

PRD Template:             {standard_template}

Compliance requirements:  {relevant_compliance_reqs}

Related user stories:     {user_story_library}

Applicable regulations:   {regulatory_requirements}
[PM Inputs]

Product name:             {product_name}

Meeting notes:            {meeting_notes}

User stories:             {user_stories}

Requirements:             {requirements}

Flow diagram description: {visio_description}

Target users:             {target_users}

Success metrics:          {success_metrics}

Timeline:                 {timeline}
Generate a complete PRD following the template.

For each section:

Use provided inputs first
Enrich with relevant past PRD context
Flag compliance gaps clearly
Mark missing information as TO BE CONFIRMED

---

## PRD Output Structure

```markdown
# PRD — {Product Name}

## 1. Problem Statement
What problem are we solving and for whom?

## 2. Goals and OKRs
What does success look like?
OKR 1: ...
OKR 2: ...

## 3. User Stories
As a [user] I want to [action] so that [benefit]

## 4. Functional Requirements
What must the product do?
REQ-001: ...
REQ-002: ...

## 5. Non Functional Requirements
Performance, scalability, availability

## 6. Compliance Considerations
⚠️ COMPLIANCE GAP: ...
Legal approval required for: ...
Regulatory requirements: ...

## 7. Technical Requirements
APIs, integrations, dependencies

## 8. Success Metrics
How will we measure success?
Metric 1: ...
Metric 2: ...

## 9. GTM Approach
How will we launch?
Phased rollout / A/B test / Full launch

## 10. Dependencies
What do we need before we can start?

## 11. Open Questions
TO BE CONFIRMED: ...
TO BE CONFIRMED: ...

## 12. Out of Scope
What are we explicitly NOT building?
```

---

## Compliance Flagging Instruction

Add this to the system instruction to auto catch 
compliance gaps:

After generating each section, review it against

the compliance requirements retrieved from context.
If any requirement touches:

Customer data or PII     → flag for Data Privacy review
Payment processing       → flag for PCI DSS compliance
Customer communications  → flag for Legal review
New product feature      → flag for Compliance approval
AI/ML components         → flag for Model Risk Management

Format flags as:

⚠️ COMPLIANCE FLAG: [what needs review]

→ [which team to involve]

→ [estimated approval timeline]

---

## A/B Prompting

Test two prompt versions before committing:

**Prompt A — Detailed:**

Generate a complete PRD with full detail

in every section. Include rationale for

every requirement.

**Prompt B — Concise:**

Generate a PRD with clear, actionable

requirements only. No rationale needed.

Flag gaps and missing info.

Measure PM revision rate for each version.
Lower revision rate = better prompt.

---

## Guardrails

| Risk | Guardrail |
|---|---|
| LLM fabricates requirements | "Use ONLY provided context. Mark gaps as TO BE CONFIRMED" |
| LLM misses compliance req | Auto flag sections touching PII, payments, AI, customer data |
| LLM uses wrong template | Pin specific template version in system instruction |
| Confidential PRD details leaked | Strip sensitive data before adding to Vector DB |
| Untestable requirements | "Every requirement must be measurable and testable" |
| Output too long / short | Define max/min length per section |

---

## Evaluation Metrics

| Metric | What it measures | Target |
|---|---|---|
| Completeness | Are all 12 sections populated? | 100% |
| Compliance coverage | Did it flag all relevant compliance gaps? | > 95% |
| Template adherence | Does output follow PRD template? | 100% |
| Requirement quality | Are requirements testable and measurable? | > 90% |
| PM revision rate | How much does PM edit before approving? | < 20% |
| Time saved | PRD generation vs manual writing | > 60% |
| Accuracy | Do requirements match PM inputs? | > 90% |

---

## Feedback Loops

PM approves PRD with no/minor edits

↓

High quality signal

↓

Store PRD in Vector DB as high quality example

Future RAG retrieval improves ✅
PM makes significant edits

↓

Low quality signal

↓

Capture what was changed:

Improve system instruction
Update PRD template
Improve chunking of past PRDs
Adjust prompt

Compliance team flags a gap

↓

Add gap type to compliance

flagging instruction permanently

↓

Future PRDs auto catch this gap ✅

---

## Full Quality Loop

PM inputs

↓

RAG retrieves:

Similar past PRDs
PRD template
Compliance requirements
User story library

↓

GPT-4o generates PRD

↓

Guardrails check:

✅ All 12 sections complete?

✅ Compliance gaps flagged?

✅ Template followed?

✅ No fabricated content?

↓

PM reviews and edits

↓

Revision rate tracked

↓

High quality PRD → added to Vector DB

↓

Future PRDs get better ✅

---

## Document Extraction Pipeline

Past PRDs scattered across multiple sources:

| Source | Extraction Tool | Output |
|---|---|---|
| Confluence | REST API | Markdown |
| Visio | GPT-4o Vision | Markdown description |
| PDF (text) | pdfplumber / Docling | Markdown |
| PDF (scanned) | Azure Document Intelligence | Markdown |
| Word (.docx) | mammoth | Markdown |
| Excel (.xlsx) | pandas | Markdown tables |

### Unified Extraction Pipeline

Confluence / Visio / PDF / Word / Excel

↓

Extraction layer

(API / Docling / GPT-4o Vision)

↓

Preprocessing:

Convert all formats → markdown
Remove boilerplate
Preserve structure
Strip PII
Quality check

↓

Chunking:
Split by section headings
500 token chunks
50 token overlap

↓

Azure OpenAI text-embedding-3-large

↓

Azure AI Search (Vector DB)

with metadata:

{

source: "confluence",

document_type: "PRD",

product: "Card Activation",

date: "2024-03",

version: "v2.1"

}

### Visio Diagram Handling

**Option 1 — Text extraction (simple diagrams):**

vsdx file → unzip → read XML

↓

Extract text nodes and labels

↓

Structured text description

**Option 2 — Vision AI (complex diagrams):**

.vsdx → export as PNG

↓

GPT-4o Vision:

"Describe this process flow in detail.

List each step, decision point,

and connection in order."

↓

Rich markdown description ✅

---

## ETL Schedule

| Data Type | Update Frequency | Tool |
|---|---|---|
| Historical PRD backfill | One time | Azure Databricks |
| New PRDs created | On creation (webhook) | Azure Event Hubs |
| Updated PRDs | On update (webhook) | Azure Event Hubs |
| Template updates | On change | Azure Data Factory |
| Compliance requirements | On change | Azure Data Factory |

---

## Key PM Considerations

| Concern | Action |
|---|---|
| Document quality varies | Quality scoring — skip low quality docs |
| Outdated PRDs | Date metadata — filter by recency |
| Confidential PRDs | Role based access control on Vector DB |
| Visio complexity | Test both approaches — choose by accuracy |
| Excel with formulas | Extract values not formulas |
| Confluence permissions | Respect source permissions in RAG |
| Version control | Store document version in metadata |

---

## Interview One Liner

> "Our PRD generator uses a RAG pipeline — past PRDs, 
> templates, and compliance requirements extracted from 
> Confluence, Visio, PDF, Word and Excel using 
> Azure Document Intelligence and Docling, 
> converted to markdown, chunked, embedded via 
> Azure OpenAI, and stored in Azure AI Search. 
> At runtime, GPT-4o retrieves relevant context, 
> follows our standard template, auto flags compliance 
> gaps, and generates a complete PRD first draft 
> in minutes. PM revision rate is our key quality metric."
>


---

## Challenges

### 1. Content Challenges

**Outdated PRDs in Knowledge Base**

Old PRDs contain deprecated requirements,

sunset features, and outdated compliance rules

↓

RAG retrieves outdated context

↓

LLM generates PRD with wrong requirements

Fix: Date metadata filter — only retrieve PRDs 
from last 2 years. Archive older PRDs separately.

**Inconsistent PRD Quality**

Some PRDs written by senior PMs — high quality

Some written by junior PMs — incomplete

RAG retrieves both equally

↓

Low quality PRDs contaminate outputs

Fix: Quality scoring on ingestion — 
only index PRDs above quality threshold.
Human review gate before adding to Vector DB.

**Missing PM Inputs**

PM provides minimal inputs:

"Build a fraud detection feature"

↓

LLM fills gaps with assumptions

↓

PRD looks complete but is fabricated

Fix: Mandatory input validation before 
RAG pipeline runs. Block generation if 
critical fields empty. Prompt PM for 
missing information first.

**Conflicting Past PRDs**

PRD from 2022: "Card limit is $500"

PRD from 2024: "Card limit is $1000"

Both retrieved by RAG

↓

LLM confused — may use wrong value

Fix: Version metadata + date filtering.
Always retrieve most recent version.
Flag conflicts explicitly in output.

---

### 2. Technical Challenges

**Fragmented Documents**
PRDs scattered across:

Confluence / Visio / PDF / Word / Excel

↓

Different formats, structures, quality

↓

Extraction pipeline complex to maintain

↓

Any source change breaks extraction

Fix: Unified extraction layer (Docling + 
Azure Document Intelligence). 
Webhook triggers for new/updated documents.

**Visio Diagram Information Loss**

Complex swim lane diagrams

↓

Text extraction misses:

Spatial relationships
Arrow directions
Decision points
Parallel flows

↓

PRD missing critical flow details

Fix: GPT-4o Vision for all complex diagrams.
Always verify diagram description accuracy
before adding to Vector DB.

**Chunking Breaking Requirements**

Long functional requirement split

across two chunks:

Chunk 1: "The system must validate

customer identity..."

Chunk 2: "...using two factor authentication

within 30 seconds"

↓

RAG retrieves only Chunk 1

↓

Incomplete requirement in PRD

Fix: Chunk by section headings not token count.
Add 50-100 token overlap between chunks.
Keep individual requirements intact.

**Embedding Model Domain Gap**

Standard embedding model trained on

general internet text

↓

Financial services terminology

treated as general text:

"CNP fraud" ≈ "card not present fraud"

may have low similarity score

↓

Relevant PRDs not retrieved

Fix: Fine tune embedding model on 
financial services corpus.
Or use domain specific embedding model.

---

### 3. Compliance Challenges

**Confidential PRDs in Knowledge Base**

PRD for unreleased product

embedded in Vector DB

↓

Any PM can query and retrieve

confidential product details

↓

Information security violation

Fix: Role based access control on Vector DB.
Filter retrieval by PM's access level.
Separate Vector DBs for different 
classification levels.

**Stale Compliance Requirements**

Regulation changes in January

↓

Compliance requirement document

not updated in Vector DB until March

↓

PRDs generated in February

miss new regulatory requirement

↓

Compliance violation risk

Fix: Compliance documents flagged 
as highest priority for re-indexing.
Webhook triggers immediate re-indexing
when compliance documents updated.
Alert PM if compliance doc is > 30 days old.

**LLM Misreading Regulatory Nuance**

Regulation states:

"Customer must be notified within

5 business days UNLESS transaction

is under investigation"

↓

LLM generates requirement:

"Notify customer within 5 business days"

↓

Critical exception missed

↓

Compliance violation in production

Fix: Compliance sections always reviewed
by human. Never auto approve compliance
flagging without Legal review.
Add explicit instruction:
"Quote regulation verbatim — 
do not paraphrase compliance requirements"

**MRM Validation Required**

PRD generator is an AI system

used in a regulated environment

↓

Must be registered in model inventory

↓

Independent validation required

↓

Ongoing monitoring needed

↓

Timeline: 3-6 months before production

Fix: Engage Model Risk team at project start.
Not at launch.

---

### 4. Adoption Challenges

**PM Trust Gap**

PM receives AI generated PRD

↓

"This doesn't sound like me"

"These requirements are too generic"

"I don't trust AI for something this important"

↓

PM ignores tool, writes PRD manually

↓

Low adoption, wasted investment

Fix: Position as first draft assistant —
not a replacement. Show revision rate data.
Start with sections PMs hate most
(non functional requirements, compliance).
Let PMs choose which sections to generate.

**Engineering Pushback**

Engineers receive AI generated PRD

↓

"Requirements are ambiguous"

"These acceptance criteria are untestable"

"Who approved this?"

↓

Engineering trust in PRD quality drops

↓

More back and forth, slower delivery

Fix: Add engineering review step before 
PRD finalised. Track and measure 
requirement clarity scores from engineers.
Use feedback to improve system instruction.

**Legal and Compliance Skepticism**

"AI flagged this as compliant"

"How do we know it didn't miss something?"

"We can't rely on AI for compliance review"

↓

Legal insists on full manual review anyway

↓

Time saving eliminated

Fix: Position AI compliance flagging as 
first pass — not final review. 
Show evidence of gaps caught vs missed.
Build trust incrementally over time.

---

### 5. Operational Challenges

**Knowledge Base Staleness**

New PRDs added regularly

Old PRDs never retired

↓

Vector DB grows with outdated content

↓

RAG retrieves mix of current

and deprecated requirements

↓

PRD quality degrades over time



