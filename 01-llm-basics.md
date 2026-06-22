# 01 — LLMs: What They Are and How They Work

## What is an LLM?

A **Large Language Model (LLM)** is an AI model trained on massive amounts of text data — books, websites, Wikipedia, YouTube transcripts, research papers, and more.

Through this training, it learns to **understand the meaning of language** and **predict responses** in a way that feels natural and human.

When you ask it a question, it doesn't look up an answer in a database. It generates a response word by word, based on patterns it learned during training.

> Think of it as a very well-read colleague who has consumed more information than any human ever could — and can explain it back to you in plain language.

---

## How Does an LLM Actually Work?

Here's what happens under the hood when you send a message to an LLM:

**Step 1 — Your text becomes numbers**
Computers don't understand words. So your input (called a **prompt**) is first broken into chunks called **tokens**, then converted into numbers called **embeddings** via an embedding model. 
These numbers capture the *meaning* of words, not just the spelling.

**Step 2 — Transformers find relationships**
The **Transformer** architecture (the T in GPT) processes these numbers and identifies relationships between words and concepts — even across long pieces of text. It understands that "fraud" and "dispute" are related, or that "activate card" and "onboarding" belong in the same context.

**Step 3 — The model predicts the output**
Based on everything it learned during training, the model predicts the most relevant response — token by token.

**Step 4 — Numbers become text again**
The output numbers are converted back into human-readable text via the Transformer decoder.

Your prompt (text)
↓
Tokens + Embeddings (numbers)
↓
Transformer (finds meaning & relationships)
↓
Output tokens (numbers)
↓
Response (text)

---

## Real Example — Call & Notes Summarisation at financial services company 

**The problem:** After every customer service call, agents had to manually write call summaries — time-consuming, inconsistent, and a significant source of handling effort.

**The solution:** An LLM-powered summarisation capability.

Here's how it worked:

1. Raw call transcript is generated from the conversation
2. **PII is stripped** from the transcript (names, card numbers, addresses) before any data leaves the environment — critical in a regulated context
3. The cleaned transcript is sent to **GPT-4 via Azure OpenAI** — Azure ensures data residency and compliance requirements are met
4. A **System Instruction** defines the AI's role: *"You are a financial services assistant. Summarise the following call into structured notes covering issue, resolution, and next steps."*
5. A **Prompt** passes the cleaned transcript as input
6. GPT-4 returns a structured call summary
7. The summary is surfaced to the agent in the servicing tool

**Result:** 20% reduction in call handling effort.


## Choosing the Right Model

Not all LLMs are equal — and choosing the wrong one can create cost, compliance, and operational problems down the line.

### Open Source vs Closed Source

| | Open Source | Closed Source |
|---|---|---|
| **Examples** | Llama (Meta), Mistral, Falcon | GPT-4 (OpenAI), Claude (Anthropic), Gemini (Google) |
| **Cost** | Lower — you run it yourself | Pay per API call — costs scale with usage |
| **Control** | Full — hosted in your own environment | Limited — model runs on vendor infrastructure |
| **Compliance** | Easier — data never leaves your environment | Requires careful vendor assessment |
| **Capability** | Catching up fast | Generally stronger out of the box |
| **Maintenance** | Your team manages it | Vendor manages updates |

### Key Factors for PMs

**1. Cost**
Closed source models charge per token — every word in and every word out costs money. At scale (millions of users, thousands of calls per day) this adds up fast. Open source models have upfront infrastructure costs but lower per-call cost at scale.

> Rule of thumb: prototype with closed source, evaluate open source at scale.

**2. Compliance**
In regulated industries, this is often the deciding factor:

- **PII must never be sent to an external LLM** — strip all personal data before any prompt leaves your environment
- **Data residency** — where is the data processed and stored? Some regulations require data to stay in-country
- **Audit trails** — can you log every input and output for regulatory review?
- **Regulatory approval** — AI systems in financial services often require formal sign-off before deployment; factor this into your roadmap

> Using a vendor like Azure OpenAI or AWS Bedrock can help — they provide closed source model capability (GPT-4, Claude) within your own cloud environment, satisfying many compliance requirements.

**3. Latency**
Latency is the time between sending a prompt and receiving a response. This matters enormously for user-facing products.

- A fraud decision that takes 5 seconds will frustrate users and break the experience
- Smaller, faster models are often better for real-time decisions even if slightly less capable
- Larger models are fine for background tasks (summarisation, report generation) where speed is less critical

**4. Vendor Lock-in**
When you build entirely on one provider's model (e.g. GPT-4), you depend on their pricing, availability, and roadmap. If they raise prices or deprecate a model, your product is affected.

Mitigation strategies:
- Abstract your AI layer so you can swap models without rewriting your product
- Evaluate at least 2 models before committing
- Monitor the open source landscape — the gap with closed source is closing fast

---

## Key Takeaway for PMs

LLMs are prediction engines, not fact engines. They generate what's *likely* to be correct — which is why guardrails, system instructions, and evaluation frameworks matter as much as the model itself.

Choosing the right model is a product decision, not just an engineering one.

---

**Next: [02 — System Instructions & Prompts →](02-system-instructions-prompts.md)**
