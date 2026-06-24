# 07 — Reasoning Models: Deep Dive

## What is a Reasoning Model?

A **Reasoning Model** is an AI model designed to **think through 
problems step by step** before producing an answer — rather than 
instantly predicting the next token.

Standard LLMs respond immediately — fast but sometimes wrong 
on complex problems.

Reasoning models slow down deliberately — trading speed for 
accuracy on problems that require logic, multi-step thinking, 
or nuanced judgement.

> Think of it like this:
> Standard LLM = gut reaction
> Reasoning Model = thinking it through carefully before answering

---

## Why Reasoning Models Exist

Standard LLMs fail on complex problems because they predict 
the most probable next token — not the most correct answer.

**Example:**

Question: *"A customer disputes a $500 charge. They have 
disputed 3 times in the last 6 months. The merchant has 
provided delivery confirmation. Should we approve the dispute?"*

**Standard LLM:**
Instantly generates a plausible answer based on token 
probability — may miss critical nuances.

**Reasoning Model:**

Step 1: Analyse dispute history — 3 disputes in 6 months

is above normal threshold

Step 2: Evaluate merchant evidence — delivery confirmation

is strong counter evidence

Step 3: Consider regulatory requirement — customer has

right to dispute

Step 4: Weigh risk — approving may encourage abuse,

declining may be unfair

Step 5: Recommend — escalate to human agent with

full context rather than auto-decide


Better outcome. More defensible. More auditable.

---

## How Reasoning Models Work

### Chain of Thought (CoT)

**Chain of Thought** is the core technique behind reasoning models.

Instead of directly predicting the answer, the model generates 
intermediate reasoning steps — like showing its working in maths.

Standard LLM:

Question → Answer
Chain of Thought:

Question → Step 1 → Step 2 → Step 3 → Answer

**How it was discovered:**
Researchers found that simply adding *"Let's think step by step"* 
to a prompt dramatically improved accuracy on complex problems.

The model wasn't smarter — it was just forced to reason 
before answering.

---

### Tree of Thought (ToT)

Chain of Thought follows one reasoning path.
**Tree of Thought** explores multiple paths simultaneously — 
like a chess player thinking several moves ahead.

Question
                   ↓
      ┌────────────┼────────────┐
    Path A       Path B       Path C
      ↓            ↓            ↓
   Explore      Explore      Explore
      ↓            ↓            ↓
   Dead end     Promising    Dead end
                   ↓
             Best Answer

             
             Used when:
- Multiple valid approaches exist
- Problem has high stakes
- Single path reasoning risks missing the best solution

---

### How o1 and o3 Were Trained Differently

OpenAI's o1 and o3 models are not just prompted to reason — 
they were **trained specifically to reason** using 
Reinforcement Learning.

Standard LLM Training:

Predict next token → get it right → reinforce
o1/o3 Training:

Generate reasoning chain

↓

Evaluate if reasoning led to correct answer

↓

Reward good reasoning chains

↓

Penalise bad reasoning chains

↓

Model learns to reason better over time


The model learned that **thinking carefully leads to 
better outcomes** — not just predicting probable tokens.

---

## Reasoning Models vs Standard LLMs

| | Standard LLM | Reasoning Model |
|---|---|---|
| **Speed** | Fast (1-3 seconds) | Slow (10-30 seconds) |
| **Cost** | Lower per call | Higher per call |
| **Simple tasks** | Excellent | Overkill |
| **Complex reasoning** | Often wrong | Much more accurate |
| **Multi-step problems** | Struggles | Designed for this |
| **Auditability** | Low — no reasoning shown | High — steps visible |
| **Best for** | Summarisation, chat, classification | Complex decisions, analysis, risk assessment |

---

## When to Use Reasoning Models

**Use reasoning models when:**
- Problem requires multiple steps to solve correctly
- Wrong answer has high consequence (financial, regulatory)
- You need an auditable reasoning trail
- Task involves ambiguous trade-offs
- Speed is less important than accuracy

**Do not use reasoning models when:**
- Task is simple (summarisation, classification)
- Real time response is required (< 2 seconds)
- Cost at scale is a concern
- Problem has a clear, single correct answer

---

## Reasoning Models in Financial Services

**High value use cases:**

**1. Complex Dispute Resolution**

Customer dispute with conflicting evidence

↓

Reasoning model analyses:

Transaction history
Merchant evidence
Customer dispute pattern
Regulatory requirements

↓

Produces step by step recommendation

↓

Human agent reviews reasoning + decides

**2. Fraud Risk Assessment**

Unusual transaction flagged

↓

Reasoning model evaluates:

Is location consistent with history?
Is amount within normal range?
Has merchant been flagged before?
What is customer's dispute history?

↓

Reasoned risk score with explanation

↓

Human or ML model makes final decision

**3. Regulatory Policy Interpretation**

New regulation published

↓

Reasoning model reads full document

↓

Step by step analysis:

What changed?
Which products are affected?
What needs to change in our systems?
What is the timeline?

↓

Structured impact assessment for PM and Legal

---

## When Reasoning Models Fail

**1. Overthinking Simple Problems**
Reasoning models sometimes over-complicate straightforward 
questions — producing long unnecessary reasoning chains 
for simple tasks.

**2. Slow for Real Time**
10-30 second response time is unacceptable for 
customer-facing real time products.

**3. Expensive at Scale**
Reasoning tokens cost significantly more — 
not viable for high volume, low complexity tasks.

**4. Reasoning Can Still Be Wrong**
More steps does not guarantee correct answer — 
the model can reason confidently down a wrong path.

> **PM Rule:** Reasoning models are not always better — 
> they are better for specific tasks. 
> Match the model to the problem.

---

## Current Reasoning Models

| Model | Company | Latency | Best For |
|---|---|---|---|
| o1 | OpenAI | 15-30s | Complex reasoning, coding, analysis |
| o3 | OpenAI | 20-60s | Hardest reasoning tasks |
| o3 mini | OpenAI | 5-15s | Balanced speed and reasoning |
| Claude 3.5 Sonnet (extended thinking) | Anthropic | 10-30s | Regulated environments, long documents |
| Gemini 2.0 Flash Thinking | Google | 8-20s | Multimodal reasoning |
| DeepSeek R1 | DeepSeek | 10-25s | Open source reasoning, self hosted |

---

## Reasoning Models and MRM

Reasoning models have a significant advantage in regulated 
environments — **the reasoning chain is the audit trail.**

Standard LLM:

Input → Output

"Why did it say that?" → Unknown
Reasoning Model:

Input → Step 1 → Step 2 → Step 3 → Output

"Why did it say that?" → Read the reasoning chain

This makes reasoning models far more compatible with 
Model Risk Management requirements — explainability 
is built in.

> **Interview insight:** 
> *"Reasoning models are particularly valuable in regulated 
> environments because the reasoning chain provides the 
> explainability that MRM requires — something standard 
> LLMs cannot offer."*

---

## Key Takeaway for PMs

Reasoning models trade speed and cost for accuracy 
and explainability.

Use them where being wrong is expensive.
Don't use them where being slow is expensive.

In financial services — complex dispute analysis, 
policy interpretation, and risk assessment are where 
reasoning models deliver the most value.

---

**Next: [08 — Small Language Models →](08-small-language-models.md)**



