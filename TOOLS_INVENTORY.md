# GuauAI — Tools & Infrastructure Inventory

What we already have that directly applies to this project. Zero procurement needed for MVP.

---

## AI / Intelligence Layer

| Tool | What We Use It For | Status |
|------|-------------------|--------|
| **Claude API (Anthropic)** | Core reasoning, RAG Q&A, behavior analysis from vision, Spanish/English generation | Live — `claude-api` skill |
| **Claude Vision** | Zero-shot video frame + image analysis for dog behavior | Live via API |
| **GPT-4V (via OpenRouter)** | Backup vision model, second opinion on behavior classification | Live — OPENROUTER_API_KEY in .env |
| **ElevenLabs** | Spanish TTS — speak the recommendation back to the user in natural Spanish | Live — Zapier MCP + direct API |
| **Higgsfield** | Video generation for demo/marketing content | Live — MCP connected |

---

## Backend / Data Layer

| Tool | What We Use It For | Status |
|------|-------------------|--------|
| **Supabase** | Knowledge base vector store, user profiles, behavior log, training session history, feedback loop data | Live — MCP connected |
| **Vercel** | Web app hosting, serverless API routes for vision pipeline | Live — MCP + CLI |
| **Cloudflare** | CDN, KV for caching breed lookups, D1 for lightweight edge data | Live — MCP connected |

---

## Build / Automation Layer

| Tool | What We Use It For | Status |
|------|-------------------|--------|
| **Printing Press** | Generate production-ready CLI clients for any API we integrate (vision APIs, training datasets, breed registries) | Live — skill in vault |
| **Claude API skill** | Build + optimize the core intelligence pipeline with prompt caching | Live |
| **GitHub (Zapier MCP)** | Repo management, file commits, issue tracking, PR workflow | Live |
| **Vercel Bootstrap skill** | Spin up new Next.js app linked to Vercel + Supabase in one shot | Live |

---

## CRM / Go-to-Market Layer

| Tool | What We Use It For | Status |
|------|-------------------|--------|
| **Apollo.io** | Beta user outreach — dog trainers, shelters, veterinary clinics, bilingual pet communities | Live — MCP connected |
| **Gmail MCP** | Outbound campaign management for beta signups | Live |
| **Google Calendar** | Scheduling beta user interviews, onboarding calls | Live |

---

## Key API Keys Already in .env

Located at: `C:/Users/mario/.gemini/antigravity/scratch/gravity-claw/.env`

- `ANTHROPIC_API_KEY` — Claude vision + RAG
- `OPENROUTER_API_KEY` — GPT-4V backup, model routing
- `ELEVENLABS_API_KEY` — Spanish TTS
- `SUPABASE_URL` + `SUPABASE_SERVICE_KEY` — database + vector store
- `TELEGRAM_BOT_TOKEN` — notifications during build

---

## What We Still Need to Procure

| Item | Purpose | Est. Cost |
|------|---------|-----------|
| **pgvector extension on Supabase** | Vector similarity search for RAG | Free (built-in) |
| **Fal.ai account** | Image generation for training illustration content | ~$0.003/image (already used for blog writer) |
| **App Store Developer Account** | iOS distribution | $99/yr |
| **Google Play Developer Account** | Android distribution | $25 one-time |
| **React Native / Expo** | Mobile app framework | Free |
