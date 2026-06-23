# 05 — Multimodal Models: Deep Dive

## What is a Multimodal Model?

A **Multimodal Model** is an AI model that can understand and process 
multiple types of input — text, images, audio, and video — together, 
in a single interaction.

Most early AI models were **unimodal** — text in, text out. 
Multimodal models broke this constraint.

> A human expert doesn't just read text. They look at diagrams, 
> listen to calls, watch recordings, and read documents — all at once. 
> Multimodal AI brings this same capability to machines.

---

## Why Multimodal Matters

The real world is not text only.

In financial services alone:
- Customers submit **images** of receipts as fraud evidence
- Agents handle **call recordings** that need to become structured notes
- Documents arrive as **PDFs, screenshots, scanned forms**
- Fraud evidence includes **transaction screenshots, ID photos, signatures**

Without multimodal capability, you need separate pipelines for each 
input type — complex, expensive, and fragile. Multimodal collapses 
this into a single, unified model.

---

## How Does a Multimodal Model Work?

### Processing Text
Already covered in earlier chapters — text is tokenised, 
converted to embeddings, processed by the Transformer.

### Processing Images — Step by Step

**Step 1 — Patch Extraction**
The image is split into small fixed-size patches — like cutting a 
photo into a grid of tiny squares.

Full Image

┌─────────────────┐

│  □  □  □  □  □  │

│  □  □  □  □  □  │  → Each □ is a patch

│  □  □  □  □  □  │

└─────────────────┘


**Step 2 — Vision Encoder**
Each patch is converted into a vector (list of numbers) by a 
**Vision Encoder** — a specialised embedding model for images.

Image Patch → Vision Encoder → [0.23, 0.87, 0.41, 0.12, ...]

The Vision Encoder learns during training that certain patterns of 
numbers represent edges, shapes, text, objects, and relationships.

**Step 3 — Modality Alignment**
This is the critical step most people miss.

Text embeddings and image embeddings live in different 
**vector spaces** — they speak different languages. A **projection 
layer** translates image embeddings into the same vector space as 
text embeddings so the Transformer can process them together.


Text:  "fraud receipt"  → [0.23, 0.87, 0.41]  ← text vector space

Image: receipt photo    → [0.91, 0.12, 0.67]  ← image vector space

↓

Projection Layer

↓

Both now in the same vector space → Transformer processes together

This is how the model learns that a **picture of a receipt** and 
the **words "receipt" or "transaction evidence"** mean the same thing.

**Step 4 — Cross-Attention**
Inside the Transformer, **Cross-Attention** allows text and image 
embeddings to interact with each other.

When you ask *"What is the transaction amount in this receipt?"*:
- The text query attends to relevant image patches 
  (the area showing the amount)
- The image patches attend to the question context
- Together they produce a grounded, accurate answer

Text query: "What is the transaction amount?"

↓

Cross-Attention finds relevant image patches

↓

Model focuses on the amount area of the receipt

↓

Output: "$247.50"


**Step 5 — Output Generation**
The Transformer decoder generates the response — same as text-only 
models, but now grounded in both text and image understanding.

---

## Processing Audio

Audio follows a similar pattern:

Audio recording

↓

Converted to spectrogram (visual representation of sound waves)

↓

Treated like an image — patch extraction + Vision Encoder

↓

Audio embeddings aligned with text embeddings

↓

Transformer processes audio + text together

> This is why Whisper (OpenAI's audio model) is so powerful — it 
> treats audio as a visual pattern, not just sound.

**Financial services use case:**

Customer call recording

↓

Whisper → Raw transcript

↓

PII stripping

↓

GPT-4o → Structured call summary + sentiment + key issues flagged

↓

Agent servicing tool


Result: Recordings become structured notes automatically — 
no manual effort, consistent quality, full audit trail.

---

## Processing Video

Video is the most complex modality — because it adds **time**.

Image = understanding one moment

Video = understanding change across time

**The challenge at scale:**
A 1 minute video at 30fps = 1,800 frames.
Processing every frame is expensive, slow, and unnecessary.

**How models handle this:**

Video

↓

Frame Sampling — select key frames intelligently

↓

Each frame processed as an image (patch → Vision Encoder)

↓

Temporal Attention — find patterns and relationships ACROSS frames

↓

Audio track processed separately (Whisper)

↓

All modalities combined → Output

**Temporal Attention** is what separates video understanding from 
image understanding — it tracks what changes between frames, 
identifying actions, events, and sequences over time.

**Why enterprises use multiple models for video:**

One model for content understanding  → what's in the video

One model for speech transcription   → Whisper

One model for moderation             → is this content safe?

One model for recommendations        → what to show next

No single model handles all of these well at the scale of 
YouTube or TikTok.

---

## Where Multimodal Models Struggle

Knowing limitations is as important as knowing capabilities.

**1. Complex Diagrams**
Models struggle with dense, interconnected diagrams — 
architecture flows, process maps with many nodes and arrows.

*Why:* Patch-based processing can miss relationships between 
elements far apart in the image. The model sees patches, 
not the whole diagram holistically.

*Workaround:* Break complex diagrams into sections. 
Add text annotations. Describe the flow in text alongside 
the image.

**2. Small Text in Images**
Fine print, small labels, watermarks — models frequently 
misread or miss these entirely.

*Why:* Small text occupies very few patches — not enough 
signal for accurate recognition.

*Workaround:* Use OCR tools to extract text first, 
then send extracted text + image together.

**3. Spatial Reasoning**
*"Is box A above or below box B?"* — models often get 
spatial relationships wrong.

*Why:* The model learns semantic meaning from patches, 
not precise spatial coordinates.

**4. Video at Long Duration**
Frame sampling means the model may miss important events 
that happen between sampled frames.

**5. Hallucination on Images**
If the image is unclear or ambiguous, the model will 
confidently describe things that aren't there — 
especially dangerous in regulated contexts like 
fraud evidence review.

> **PM Rule:** Never trust multimodal outputs on 
> critical decisions without a human review layer 
> or confidence scoring.

---

## Financial Services Use Cases — Present and Future

**Now — already being built:**

| Use Case | Modalities | How |
|---|---|---|
| Call summarisation | Audio + Text | Whisper → PII strip → LLM → structured notes |
| Fraud evidence review | Image + Text | Customer submits receipt screenshot → model extracts transaction details |
| Document processing | Image + Text | Scanned forms → extract fields → populate systems |
| KYC verification | Image + Text | ID document → verify fields → flag anomalies |

**Future — not yet mainstream:**

| Use Case | Modalities | Impact |
|---|---|---|
| Video dispute resolution | Video + Text | Customer records video of disputed transaction → AI analyses and resolves without agent |
| Real-time fraud detection | Image + Data + Text | Transaction screenshot + spending pattern + location → instant fraud decision |
| AI agent servicing | Audio + Text + Action | Customer calls → AI agent understands, decides, resolves — no human intervention |

---

### Understanding FPS — Why It Matters for AI Products

Video is not actually a moving image. It is thousands of 
**static images (frames) played rapidly** one after another, 
creating the illusion of motion.

1 second of video at 30 FPS = 30 individual static images

1 minute = 60 seconds

60 seconds × 30 frames = 1,800 individual images

Every frame is a separate image the model needs to process.

---

### FPS Standards

| FPS | Use Case |
|---|---|
| 24 FPS | Movies — cinematic standard |
| 30 FPS | Standard video — YouTube, WhatsApp, phone recordings |
| 60 FPS | Gaming, sports, slow motion |
| 120 FPS | High speed cameras, professional sports analysis |

Higher FPS = smoother video = more frames = significantly 
more expensive to process with AI.

---

### The Scale Problem at Enterprise Level

---

## Key Takeaway for PMs

Multimodal AI removes the constraint that AI only works with text. 

The real world speaks in images, audio, and video — and the best 
AI products meet users where their data actually lives.

Before choosing a multimodal model, always ask:
- What modalities does my use case actually need?
- Where will the model struggle with my specific data?
- What's my human review layer for high-stakes decisions?
- What does this cost at the scale I need?
- WhatsApp — 100 million videos shared daily

  

Average length — 1 minute at 30 FPS
100,000,000 videos

× 1,800 frames per video

= 180,000,000,000 frames per day
180 billion frames to process — every single day

Processing every frame with a large multimodal model is 
**financially and computationally impossible** at this scale.

---

### How Enterprises Solve This

**1. Frame Sampling**
Instead of processing every frame, the model intelligently 
selects key frames:

1,800 frames

↓

Sample 1 frame every 30 frames

↓

60 key frames to process

↓

97% reduction in compute cost

The model infers what happened between sampled frames 
based on context.

**2. Tiered Model Architecture**
Not one large model for everything:

Lightweight classifier first

↓

"Does this video need deep analysis?"

↓

No  → simple model handles it (cheap)
Yes → full multimodal model (expensive)

**3. Specialised Models Per Task**
Content understanding  → Vision model

Speech transcription   → Whisper

Content moderation     → Specialised safety model

Recommendations        → Ranking model

Each model is small, fast, and optimised for one task — 
far more efficient than one large model doing everything.

---

### Financial Services Application

Customer submits a video of a disputed transaction 
or fraudulent ATM interaction:

Video received

↓

Frame sampling — extract key frames

↓

Vision model — identify relevant evidence

(ATM screen, card, transaction details)

↓

Whisper — transcribe any audio

↓

LLM — synthesise evidence into structured fraud report

↓

Human review layer — agent validates before decision

**Why not process every frame?**
- A 2 minute ATM video at 30 FPS = 3,600 frames
- Processing all frames = high cost, high latency
- Key frames capture the evidence — the rest is context

---

### PM Question to Always Ask Vendors

When a vendor says *"our model processes video"* — ask:

- At what FPS does it sample?
- What is the maximum video length supported?
- What is the cost per minute of video processed?
- What happens to frames between samples — are they ignored?
- How does accuracy change at higher sampling rates?

The answers reveal the real capability, cost envelope, 
and limitations of their solution.

---

---

**Next: [06 — Enterprise Model Selection Framework →]
(06-enterprise-model-selection.md)**


