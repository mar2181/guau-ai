# 🐕 GuauAI

**Spanish-first AI dog behavior analysis and training recommendation platform.**

> *"Guau"* is the Spanish onomatopoeia for a dog's bark. GuauAI watches, listens, and knows what your dog is communicating — then tells you exactly what to do about it, in Spanish.

---

## What It Does

Record a short video or audio clip of your dog's behavior. GuauAI analyzes it — body language, tail position, ear posture, bark type, energy level — identifies the behavior category, and returns a plain-language explanation plus a step-by-step correction or training recommendation sourced from professional training literature.

**Languages:** Spanish (primary), English (secondary)
**Platforms:** Mobile (iOS + Android), Web

---

## Project Status

🔴 **Pre-MVP — Research & Planning Phase**

See [`PLAN.md`](./PLAN.md) for the full roadmap and research plan.
See [`TOOLS_INVENTORY.md`](./TOOLS_INVENTORY.md) for the full stack at our disposal.

---

## Repo Structure

```
guau-ai/
├── README.md              ← You are here
├── PLAN.md                ← Full research + execution plan
├── TOOLS_INVENTORY.md     ← Existing tools, APIs, skills at our disposal
├── research/              ← Research notes, corpus sources, competitive intel
├── knowledge-base/        ← Dog training literature, behavior taxonomy, prompts
├── mvp-sop/               ← Step-by-step SOP to build and ship the MVP
└── app/                   ← App code (when ready)
```

---

## Core Stack

| Layer | Tool |
|---|---|
| Vision AI (MVP) | Claude Vision / GPT-4V — zero-shot, no custom training needed |
| Vision AI (v2) | Fine-tuned model on labeled behavior dataset |
| Knowledge Base RAG | Supabase + pgvector + Cohere multilingual embeddings |
| LLM Orchestration | Claude API (Anthropic SDK with prompt caching) |
| Voice Output (Spanish) | ElevenLabs TTS |
| Mobile App | React Native (Expo) |
| Web Admin | Next.js on Vercel |
| CLI Tooling | Printing Press (internal — generates API clients on demand) |
| CRM / Beta | Apollo.io |

---

## Core Team
- Mario Elizondo — Product, Strategy, Partnerships
- Antigravity Digital AI Stack — Research, Architecture, Build, QA

---

*GitHub: [mar2181/guau-ai](https://github.com/mar2181/guau-ai)*
