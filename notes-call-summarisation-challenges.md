### 10. Hallucination Challenges

**Challenge 1 — Bullet Count Violation**

System instruction: "Generate exactly 7 bullets"

↓

LLM produces 9 bullets

↓

Extra bullets contain:

Fabricated next steps
Invented resolution timelines
Made up agent commitments

↓

Agent misses extra bullets

↓

Customer expects commitments

that were never made

↓

Complaint and compliance risk

Root cause: LLM ignores strict count 
instructions when context is long —
more retrieved context = more output.

Fix:
- Output guardrail: count bullets before
  sending to GFS
- If ≠ 7 → regenerate automatically
- Add to system instruction:
  "You MUST generate EXACTLY 7 bullets.
   If you have more information —
   summarise into 7.
   Never exceed 7 under any circumstance."
- Reduce retrieved context chunks from 5 to 3

**Challenge 2 — Inaccurate Summaries**

Customer said:

"I think I may have made this transaction

but I'm not sure"

↓

LLM summary states:

"Customer confirmed transaction

was unauthorised"

↓

Dispute raised incorrectly

↓

Merchant charged back unfairly

↓

Regulatory and financial risk

Root cause: LLM resolves ambiguity
confidently — predicts most probable
outcome rather than preserving uncertainty.

Fix:
- Add to system instruction:
  "Preserve ambiguity exactly as stated.
   If customer said 'not sure' or 'maybe' —
   quote their exact words.
   Never resolve uncertainty into certainty."
- Temperature reduced to 0.0 for
  resolution and verification fields
- Faithfulness check: 
  every claim in summary must be 
  traceable to exact transcript line

---

### 11. Guardrail Failures

**Challenge 1 — Low Confidence Input Not Blocked**

STT confidence score: 0.61

(below 0.80 threshold)

↓

Input guardrail should block

and route to manual transcription

↓

Guardrail check failed silently:

Threshold check code had a bug
Score compared as string not number

"0.61" > "0.80" = True in string comparison

↓

Low confidence transcript

passed to LLM

↓

Inaccurate summary generated

↓

Agent approved without noticing

↓

Wrong information in GFS case record

Root cause: Guardrail logic error —
type mismatch in confidence score comparison.

Fix:
- Unit test every guardrail condition
- Explicit type casting:
  float(confidence_score) < 0.80 → block
- Integration test: 
  send known low confidence transcript
  verify pipeline blocks correctly
- Add guardrail health check to 
  observability dashboard
- Alert if guardrail block rate = 0%
  for 24 hours (likely broken)

**Challenge 2 — Output Guardrail Not Triggered**

LLM output contained 9 bullets

↓

Output guardrail should catch:

"bullet count ≠ 7 → regenerate"

↓

Guardrail checked for bullet symbol "•"

↓

LLM used "-" instead of "•"

↓

Guardrail counted 0 bullets

↓

Passed validation incorrectly

↓

9 bullet summary sent to GFS

Root cause: Guardrail brittle —
assumed specific bullet format.

Fix:
- Guardrail must detect ALL bullet formats:
  •, -, *, 1., 2., etc.
- Use LLM as guardrail judge:
  "Count the number of bullet points
   in this output. Return only a number."
- Simpler fix: count line breaks + 
  non-empty lines in output
- Regression test library:
  every guardrail failure becomes a test case

---

### 12. RAG Retrieval Failures

**Challenge 1 — Card Replacement Procedure Missed**

Customer call: CNP fraud confirmed

Agent needs to initiate card replacement

↓

RAG retrieves:

✅ Similar past CNP case        → 0.94

✅ CNP dispute procedure        → 0.91

❌ Card replacement steps       → 0.61

(below retrieval threshold)

↓

Card replacement procedure

not included in context

↓

LLM summary missing:

"Next steps: card replacement"

↓

Agent forgets to initiate replacement

↓

Customer calls back asking about new card

↓

Second call — avoidable

Root cause: Card replacement procedure
embedded with generic language —
low similarity to CNP fraud transcript.

Fix:
- Re-chunk card replacement procedure:
  add fraud type tags to each chunk:
  "Card replacement following CNP fraud..."
- Always retrieve card replacement steps
  as mandatory chunk for fraud calls —
  not similarity based, rule based:
  "If fraud_type confirmed →
   always include card replacement procedure"
- Hybrid retrieval:
  similarity search + rule based injection

**Challenge 2 — Fraud Dispute Procedure Missed**

Call classified: Account Takeover

↓

RAG metadata filter:

dispute_type = "account_takeover"

↓

Account takeover procedure:

only 3 cases in knowledge base

→ low training signal

→ low similarity scores

↓

RAG falls back to CNP procedure

(highest score available)

↓

Wrong procedure in context

↓

LLM generates CNP next steps

for an account takeover case

↓

Agent follows wrong process

↓

Compliance risk

Root cause: Sparse knowledge base
for less common fraud types —
not enough past cases to retrieve reliably.

Fix:
- Audit knowledge base coverage
  by fraud type:
  CNP: 450 cases ✅
  Card Present: 280 cases ✅
  Account Takeover: 3 cases 🔴
  ATM Dispute: 12 cases 🟡
        ↓
- Prioritise indexing more 
  account takeover cases
- Interim fix: manually curated 
  procedure always injected for
  low coverage fraud types
- Coverage threshold:
  < 50 cases per type → mandatory
  manual procedure injection

---

### RAG Coverage Dashboard

Track knowledge base coverage 
as a product metric:

| Fraud Type | Cases in KB | Coverage | Status |
|---|---|---|---|
| CNP Fraud | 450 | High | ✅ |
| Card Present | 280 | High | ✅ |
| ATM Dispute | 45 | Medium | 🟡 |
| Merchant Dispute | 38 | Medium | 🟡 |
| Subscription Fraud | 22 | Low | 🟡 |
| Account Takeover | 3 | Critical | 🔴 |

> 🔴 **PM Rule:** Never go live with RAG for a 
> fraud type with < 50 cases in knowledge base.
> Use rule based procedure injection as fallback
> until coverage is sufficient.

---

### Lessons Learned

Guardrails must be tested like product features

→ Unit tests, integration tests, regression tests

→ Not just implemented and assumed to work
RAG coverage must be audited before launch

→ Map every fraud type to case count in KB

→ Sparse coverage = wrong retrieval
Hallucinations hide in ambiguity

→ LLM resolves uncertainty confidently

→ Preserve ambiguity explicitly in instruction
Output format must not be assumed

→ LLM may use different bullet styles

→ Guardrails must handle all variations
Low confidence inputs are as dangerous

as low quality outputs

→ Block at input — not just output

→ Test input guardrails as rigorously

as output guardrails

