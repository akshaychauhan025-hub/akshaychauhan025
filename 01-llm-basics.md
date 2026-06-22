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

## Real Example — Call & Notes Summarisation at American Express

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

---

## Key Takeaway for PMs

LLMs are powerful but they are **prediction engines**, not fact engines. They generate what's *likely* to be correct based on training data — which is why guardrails, system instructions, and evaluation frameworks matter as much as the model itself.

Choosing the right model, the right prompt, and the right infrastructure is the product decision — not just an engineering one.

---

**Next: [02 — System Instructions & Prompts →](02-system-instructions-prompts.md)**
