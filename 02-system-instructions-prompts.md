# 02 — System Instructions & Prompts

## What is a System Instruction?

A **System Instruction** (also called a system prompt) is a set of rules and context you give to an LLM *before* any user interaction begins.

It defines:
- **Who the AI is** — its role and expertise
- **What it should do** — the task and expected output format
- **What it must never do** — guardrails to prevent wrong or harmful outputs

Think of it as the job description you hand to an employee on day one — before they touch any work.

> Without a system instruction, an LLM is a generalist. With one, it becomes a specialist.

---

## What is a Prompt?

A **prompt** is the actual input you send to the LLM to generate a response.

It carries everything the model needs to produce the right output:
- The system instruction (role + rules)
- A template (structure of the expected output)
- RAG context (relevant knowledge retrieved from your data)
- The user input or raw data (e.g. a stripped transcript, a question, a document)

## Prompt = System Instruction + Template + RAG Context + User Input

---

## System Instruction vs Prompt — What's the Difference?

| | System Instruction | Prompt |
|---|---|---|
| **What it is** | Rules and role for the LLM | The actual input sent for a response |
| **When it's set** | Once, at setup | Every time a request is made |
| **Who writes it** | PM + Engineer together | Engineer / PM |
| **Changes often?** | No — stable | Yes — varies per use case |

---

## Real Example — Call Summarisation

**System Instruction:**
You are an expert assistant for a financial services company.

Your job is to generate a call summary in exactly 7 bullet points.

Rules:

Do not fabricate any information
Do not create a resolution if one was not reached on the call
Use only the information provided in the transcript
Use the reference summary format provided

**Prompt (what gets sent to the LLM):**

[System Instruction above]
[Reference summary template]
[RAG context — relevant policy or product information]
[Stripped transcript — PII removed]
Generate the call summary now.

**Output:**
A structured 7-bullet call summary covering issue, key discussion points, and next steps — ready for the agent to review.

---

## Why System Instructions Matter for PMs

System instructions are where you **encode your product requirements into the AI**.

Every constraint you'd normally write in a PRD — tone, format, what to include, what to avoid — goes here. Getting this right is the difference between an AI that behaves predictably and one that goes off-script.

**Common mistakes PMs make:**
- Too vague — *"Be helpful and professional"* tells the model nothing specific
- No constraints — without rules, the model will fill gaps with hallucinated content
- No output format — if you don't specify structure, outputs will be inconsistent
- Set and forget — system instructions need testing and iteration like any product feature

---

## Best Practices

**Be specific about the role**
❌ *"You are an AI assistant"*
✅ *"You are a financial services expert specialising in fraud dispute resolution"*

**Define the output format explicitly**
❌ *"Summarise the call"*
✅ *"Summarise the call in exactly 7 bullet points covering: issue raised, actions taken, resolution status, and next steps"*

**Set hard constraints**
✅ *"Do not fabricate information. If something is unclear in the transcript, say 'unclear' — do not guess."*

**Use reference examples**
Give the model a sample of a good output. It will follow the pattern far more reliably than instructions alone.

---

## A/B Prompting

Just like A/B testing in product, **A/B prompting** means testing two different prompts against each other to find which one produces better outputs.

**How it works:**
- Write two versions of a prompt — same goal, different instructions or structure
- Send both to the same model with the same input
- Compare outputs and let users or evaluators decide which is better

**Example — Call Summarisation:**

## Prompt A: Summarise the call in 7 bullet points covering issue, actions, and next steps.

## Prompt B: Summarise the call in 3 sections: Problem, What Was Done, Next Steps. Be concise. Max 2 sentences per section.

Both produce a summary — but tone, structure, and usefulness may differ significantly.

**How do you decide which wins?**
- **User feedback** — agents rate which summary was more useful
- **Accuracy** — how often does the output match what actually happened on the call
- **Consistency** — which prompt produces reliable outputs across 100+ transcripts
- **Revision rate** — how often does the agent edit the summary before saving it

> Revision rate is one of the most honest signals — if agents are constantly rewriting the AI output, the prompt isn't working.

**PM Tip:** Never go live with a single prompt. Always test at least two versions before committing — prompt quality directly impacts product quality.

---

## Key Takeaway for PMs

Prompts and system instructions are your primary lever for controlling AI behaviour. Before asking engineering to change the model, always ask: *can we fix this with a better system instruction?*

Most quality issues in AI products are prompt problems, not model problems.

---

**Next: [03 — Transformers →](03-transformers.md)**

