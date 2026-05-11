# GuauAI 🐕

> **Spanish-first AI dog behavior analysis and training recommendation platform**

GuauAI watches your dog — via video, audio, or photo — identifies the behavior in real time, and returns a precise, breed-aware correction recommendation in Spanish (and English). Powered by multimodal vision AI, a curated RAG knowledge base from top dog training literature, and a feedback loop that gets smarter with every session.

---

## The Problem

Dog owners — especially Spanish-speaking families — have no real AI tool that can watch their dog misbehave and tell them exactly what to do. Generic advice on YouTube doesn't account for breed, age, environment, or the specific trigger. Personal trainers cost $100/hr. Books are written in English. The gap is wide open.

## The Solution

1. **Record the behavior** — video clip, photo, or audio on mobile
2. **GuauAI analyzes it** — vision AI classifies the behavior (fear, dominance, resource guarding, play aggression, separation anxiety, etc.)
3. **RAG retrieves the correction** — sourced from 15+ top training books and breed-specific guides
4. **ElevenLabs delivers it in Spanish** — spoken, warm, clear

## Core Stack

| Layer | Tool |
|---|---|
| Vision AI (Phase 1) | Claude Vision / GPT-4V (zero-shot) |
| Vision AI (Phase 2) | Fine-tuned model on labeled behavior dataset |
| Knowledge Base | Supabase + pgvector (RAG) |
| LLM Orchestration | Claude API (Anthropic SDK) |
| Voice Output | ElevenLabs (Spanish TTS) |
| Mobile App | React Native (Expo) |
| Web Dashboard | Next.js on Vercel |
| Database | Supabase (Postgres + pgvector) |
| CLI Tooling | Printing Press (internal) |
| CRM / Beta Users | Apollo.io |
| Automation | Zapier |

## Language Strategy

- **Spanish first** — UI, voice output, training content, marketing
- **English secondary** — toggle available, same underlying model
- **Target market:** Spanish-speaking dog owners in US (RGV, SoCal, Florida, Chicago) + Mexico

## Status

- [ ] Research phase (see MASTER_PLAN.md)
- [ ] Knowledge base construction
- [ ] Vision AI pipeline (zero-shot MVP)
- [ ] Mobile app shell
- [ ] ElevenLabs voice integration
- [ ] Beta launch (50 users)

---

*Built by Antigravity Digital — Mario Elizondo + Claude*
