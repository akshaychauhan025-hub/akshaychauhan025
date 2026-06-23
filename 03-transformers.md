LLM / Transformer        → the brain — understands and generates language

Embedding Model          → the translator — converts text to numbers

Vector DB                → the memory — stores those numbers for retrieval


# 03 — Transformers

## What is a Transformer?

A **Transformer** is the core architecture that powers modern LLMs like GPT-4, Claude, and Gemini.

It is responsible for one thing: **understanding the meaning and relationships within language** — and using that understanding to predict the right output.

> Before Transformers (pre-2017), AI struggled with long text. Transformers solved this by processing entire sentences at once rather than word by word.

---

## How Does a Transformer Work?

A Transformer has two main components:

**Encoder** — reads and understands the input
- Takes your prompt and builds a deep understanding of what it means
- Identifies relationships between words — even words far apart in the text

**Decoder** — generates the output
- Uses the encoder's understanding to predict the response
- Generates output one token at a time until the response is complete

- Input Prompt
↓
Encoder (understands meaning)
↓
Decoder (generates response)
↓
Output

> Not all models use both. GPT models are decoder-only. BERT (used in search) is encoder-only. Models like T5 use both.

---

## The Attention Mechanism

The secret ingredient inside every Transformer is **Attention**.

Attention allows the model to focus on the most relevant parts of the input when generating each word of the output — similar to how humans pay attention to key words when reading a long sentence.

**Example:**

*"The customer called to dispute a charge but said the card was not lost or stolen."*

WWhen generating a summary, Attention helps the model understand:
- *"dispute a charge"* is the core issue
- *"not lost or stolen"* is a critical qualifier
- These two phrases are related even though they are far apart

WWithout Attention, the model would treat every word equally — missing the nuance.

---

## How Text Becomes Numbers

Transformers don't read words — they read numbers. Here's the full conversion flow:

**Step 1 — Tokenisation**
Text is broken into chunks called **tokens**. A token is roughly a word or part of a word.

"Fraud dispute resolution" → ["Fraud", "dispute", "resolution"]

**Step 2 — Embedding**
Each token is converted into a number (or a list of numbers called a vector) by an **Embedding Model**. These numbers capture *meaning*, not just spelling.
"Fraud" → [0.23, 0.87, 0.41, ...]

"Scam"  → [0.24, 0.85, 0.43, ...]  ← similar numbers = similar meaning

**Step 3 — Transformer Processing**
The Transformer processes these numbers using Attention — finding relationships across the entire input.

**Step 4 — Output Generation**
The Decoder predicts output tokens one by one, converting them back to text.

Text → Tokens → Embeddings (numbers) → Transformer → Output numbers → Text

---

## Transformer vs Vector DB — What's the Difference?

A common confusion:

| | Transformer | Vector DB |
|---|---|---|
| **What it is** | The AI model architecture | An external database |
| **What it stores** | Nothing — it processes | Embeddings (numbers) |
| **Where similarity happens** | Inside the model (Attention) | Outside the model (Cosine Search) |
| **Part of the LLM?** | Yes | No |

Transformers use attention *internally* to find relationships within your prompt.
Vector DBs store embeddings *externally* and are used in RAG to retrieve relevant context before sending to the LLM.

> Same concept — similarity between meanings — but at different layers of the system.

---

## Why PMs Need to Understand This

You don't need to build a Transformer — but you need to know:

- **Context window** — Transformers can only process a limited amount of text at once (e.g. 8K, 32K, 128K tokens). If your use case involves long documents, this is a product constraint.
- **Encoder vs Decoder** — Encoder models are better for search and classification. Decoder models are better for generation. Choosing the wrong type affects quality.
- **Token limits affect cost** — every token processed costs money. Longer prompts = higher cost. This is a product and commercial decision.

---

## Key Takeaway for PMs

Transformers are the engine. Embeddings are the fuel. Attention is what makes it intelligent.

Understanding this helps you ask the right questions — about context limits, model choice, cost, and why your AI produces the outputs it does.

1. Parametric Knowledge — baked in during training

This is everything the model learned during training — books, internet, Wikipedia, research papers etc. It lives inside the model's parameters (billions of numbers).

It's static — frozen after training
The model can't update it without retraining
Think of it as long term memory

2. Context Window — what you send in the prompt

This is the live knowledge you inject at runtime — your system instruction, RAG context, transcript, user input etc.

It's dynamic — changes with every request
Limited in size (token limit)
Think of it as short term memory

3. Vector DB — external knowledge (via RAG)

This is your organisation's own data — policies, documents, past cases etc. — stored as embeddings and retrieved when relevant.

Not inside the LLM — sits outside
Retrieved at runtime and added to the prompt
Think of it as a reference library the model can look things up in

Parametric Knowledge   → what the model already knows (training)
Context Window         → what you tell it right now (prompt)
Vector DB              → what it can look up from your data (RAG)

---

## How the Decoder Predicts the Next Token

This is where the magic actually happens — and understanding this 
deeply separates good AI PMs from great ones.

### The Knowledge Lives in the Parameters

During training, the model processed billions of sentences from 
books, internet, Wikipedia, research papers, and more.

For every word — it learned to predict what comes next.

Read sentence → predict next word
Get it wrong  → adjust parameters
Get it right  → reinforce parameters
Repeat this trillions of times

↓

Parameters now contain compressed knowledge of language, facts, reasoning, and relationship

GPT-4 has approximately **1.7 trillion parameters** — each one a 
number, tuned during training to capture a pattern in language.

> Think of parameters as billions of dials — each adjusted during 
> training to encode how language, facts, and reasoning work.

---

### During Inference — What Actually Happens

When you send a prompt, the decoder doesn't look anything up. 
It calculates:

Your prompt arrives

↓

Transformer processes all tokens using Attention

↓

For each position the model asks:

"Given everything before this token,

what is the most likely next token?"

↓

Searches its parameters

(everything learned during training)

↓

Calculates probability score

for every possible next token

↓

Picks the highest probability token

↓

Appends it to the sequence

↓

Repeats until response is complete

---

### Probability in Action

Input: *"The customer called to report a fraud"*

Next token probabilities:

"transaction"  → 34%

"dispute"      → 28%

"claim"        → 19%

"case"         → 12%

other tokens   → 7%


Model picks *"transaction"* — appends it — then calculates 
probabilities for the token after that — and so on until 
the full response is generated.

---

### Temperature — Controlling Randomness

Temperature controls how the model chooses between 
probable tokens:

| Temperature | Behaviour | Use Case |
|---|---|---|
| 0 | Always picks highest probability — fully deterministic | Regulated products, call summaries, compliance outputs |
| 0.3 | Slight variation — consistent but not robotic | Internal tools, structured reports |
| 0.7 | Balanced creativity and accuracy | General purpose assistants |
| 1.0+ | High randomness — creative but unpredictable | Creative writing, brainstorming |

> **PM Rule for regulated environments:** Always set temperature 
> between 0 and 0.3. You need consistent, auditable, 
> predictable outputs — not creative ones.

---

### Why Hallucination Happens — The Root Cause

The model doesn't retrieve facts. It predicts probable tokens 
based on patterns in training data.

Prompt pushes model into unfamiliar territory

↓

Model has no reliable pattern to follow

↓

Still predicts the most probable next token

↓

Produces a confident, fluent, plausible sentence

↓

That is factually wrong

This is not a bug — it is the fundamental nature of how 
token prediction works.

**Three root causes of hallucination:**

**1. Knowledge gaps**
The topic wasn't well represented in training data — 
model fills the gap with plausible but invented content.

**2. Prompt ambiguity**
Vague or poorly structured prompts push the model toward 
generic, sometimes incorrect responses.

**3. Conflicting training data**
If the internet contained contradictory information on a 
topic, the model learned conflicting patterns — 
and may produce either version.

**Why this matters for regulated products:**

A hallucinated call summary could:
- Misrepresent what a customer said
- Create incorrect resolution records
- Expose the company to compliance and legal risk

> This is why grounding (RAG), guardrails, system instructions 
> with hard constraints, and human review layers are 
> non-negotiable in financial services AI products.

---

**Next: [04 — Embeddings & Models →](04-embeddings-models.md)**
