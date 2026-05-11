# GuauAI — Master Plan
*Version 1.0 — May 11, 2026 | Project Manager: Claude (DONALD)*

---

## Executive Summary

We are building a Spanish-first AI dog behavior recognition and training recommendation app. The MVP uses zero-shot multimodal vision AI (no custom model training required) combined with a RAG knowledge base built from top dog training literature. We do NOT need to train a custom model to ship. We leverage what we already have: Claude API, Supabase, Vercel, ElevenLabs, Printing Press, and the Antigravity automation stack.

---

## Phase 0: Research (Weeks 1–3)

### 0A. Training Data & Knowledge Base Sources

**Primary Literature to Acquire (PDFs / Digital)**
| Book | Author | Why It Matters |
|---|---|---|
| Cómo hacer que tu perro te obedezca | Cesar Millan (ES) | Spanish-first, massive following, behavior taxonomy |
| El encantador de perros | Cesar Millan (ES) | Pack dynamics, dominance signals |
| Don't Shoot the Dog | Karen Pryor | Positive reinforcement science |
| The Other End of the Leash | Patricia McConnell | Species-specific behavior signals |
| On Talking Terms With Dogs: Calming Signals | Turid Rugaas | Visual signal taxonomy (tail, eyes, posture) |
| Decoding Your Dog | American College of Veterinary Behaviorists | Clinical, breed-specific |
| How Dogs Learn | Mary Burch & Jon Bailey | Learning theory for corrections |
| Dominance: Fact or Fiction | Barry Eaton | Counter-theory (important for balance) |
| Mine! A Practical Guide to Resource Guarding | Jean Donaldson | One specific behavior deep-dive |
| The Power of Positive Dog Training | Pat Miller | Modern LIMA-based methods |

**Online / Open Sources**
- AKC breed-specific behavior guides (akc.org — scrapeable)
- ASPCA Animal Behavior resource library (aspca.org)
- IAABC (Int'l Assoc. of Animal Behavior Consultants) articles
- VCA Hospitals behavior library (vcahospitals.com)
- r/Dogtraining (Reddit) — real-world Q&A corpus (5+ years, thousands of cases)
- American Veterinary Society of Animal Behavior (avsab.org) position statements

**Video Corpus for Future Model Fine-Tuning**
- YouTube channels with labeled clips:
  - Zak George's Dog Training Revolution (labeled behavior types in titles)
  - Victoria Stilwell Positively
  - McCann Dog Training
  - Kikopup (Emily Larlham) — excellent behavior labeling
- Kaggle datasets: search "dog behavior classification"
- Academic: "DECADE" dataset (Dog Ethogram Categories And Data Extraction)
- Self-labeled: record 500+ behavior clips with beta users, label in-app

### 0B. Existing Model Evaluation (Do We Need to Train?)

**Zero-Shot Testing (Week 1 — Start Here)**
Run these tests BEFORE deciding to fine-tune:
1. Send 50 labeled dog behavior video clips to Claude Vision API
2. Send same clips to GPT-4V
3. Score accuracy against ground truth labels
4. If accuracy > 70% zero-shot → MVP ships on zero-shot, fine-tuning is Phase 2
5. If accuracy < 50% → evaluate Google Video Intelligence API and Amazon Rekognition

**Hypothesis:** Zero-shot accuracy on clear behavioral signals (barking, growling, tail position, play bow, cowering) will exceed 75%. Custom training is Phase 2, not MVP blocker.

**Fine-Tuning Path (Phase 2 — Only If Needed)**
- Platform: Google Vertex AI (AutoML Vision) or Hugging Face fine-tuning
- Minimum dataset: 2,000 labeled video clips across 12 behavior categories
- Expected accuracy after fine-tuning: 85–92%
- Timeline: 8–12 weeks to gather + label + train
- Cost estimate: $500–2,000 for labeling, $200–500 for compute

### 0C. Behavior Taxonomy (What We're Classifying)

Define the 15 core behavior categories the system recognizes:

| ID | Behavior | Spanish Label | Priority |
|---|---|---|---|
| B01 | Resource guarding (food/toy/space) | Guardia de recursos | HIGH |
| B02 | Fear aggression | Agresión por miedo | HIGH |
| B03 | Dominance display | Demostración de dominancia | HIGH |
| B04 | Separation anxiety | Ansiedad por separación | HIGH |
| B05 | Excessive barking | Ladrido excesivo | HIGH |
| B06 | Leash reactivity | Reactividad con correa | HIGH |
| B07 | Destructive behavior | Comportamiento destructivo | MED |
| B08 | Play aggression (mouthing/biting) | Mordida en juego | MED |
| B09 | Submissive urination | Micción sumisa | MED |
| B10 | Calming signals | Señales de calma | MED |
| B11 | Prey drive / chasing | Impulso de presa | MED |
| B12 | Compulsive behavior | Comportamiento compulsivo | LOW |
| B13 | Attention-seeking | Búsqueda de atención | LOW |
| B14 | Counter-surfing / jumping | Saltar / robar en cocina | LOW |
| B15 | Inter-dog aggression | Agresión entre perros | HIGH |

### 0D. Competitive Intelligence

| App | What It Does | Gap We Fill |
|---|---|---|
| GoodPup | Video coaching with human trainers | We're AI, 24/7, instant |
| Puppr | Step-by-step trick training | We analyze PROBLEMS, not tricks |
| Dogo | Training plans + clicker | No vision AI, English-only |
| Barkibu | Vet/behavior Q&A (ES market) | Text-only, no vision |
| Woofz | AI training plans | No behavior recognition, no Spanish voice |
| **GuauAI** | Vision AI + Spanish voice + RAG | **Nobody is doing this** |

---

## Phase 1: Knowledge Base Construction (Weeks 2–4)

### 1A. Supabase Schema

```sql
-- Breeds reference table
CREATE TABLE breeds (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  name_es TEXT NOT NULL,
  name_en TEXT NOT NULL,
  group_type TEXT, -- herding, working, toy, etc.
  common_behaviors TEXT[], -- array of B01-B15 IDs common to this breed
  temperament_notes TEXT
);

-- Behavior taxonomy
CREATE TABLE behaviors (
  id TEXT PRIMARY KEY, -- B01 through B15
  name_es TEXT NOT NULL,
  name_en TEXT NOT NULL,
  description_es TEXT,
  triggers TEXT[],
  severity_default INTEGER -- 1-5
);

-- Knowledge chunks for RAG
CREATE TABLE knowledge_chunks (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  source TEXT NOT NULL, -- book title or URL
  behavior_id TEXT REFERENCES behaviors(id),
  breed_id uuid REFERENCES breeds(id), -- nullable = applies to all breeds
  content_es TEXT,
  content_en TEXT,
  embedding vector(1536), -- for pgvector similarity search
  correction_type TEXT -- positive_reinforcement | management | counter_conditioning
);

-- User sessions
CREATE TABLE analysis_sessions (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id uuid,
  media_url TEXT,
  media_type TEXT, -- video | photo | audio
  detected_behavior_id TEXT REFERENCES behaviors(id),
  confidence_score FLOAT,
  breed_id uuid REFERENCES breeds(id),
  recommendation_es TEXT,
  recommendation_en TEXT,
  user_rating INTEGER, -- 1-5 feedback
  created_at TIMESTAMPTZ DEFAULT NOW()
);
```

### 1B. RAG Pipeline (Printing Press CLI)

Use Printing Press to generate a CLI for the embedding + retrieval pipeline:
- Target API: OpenAI Embeddings API (or Cohere for multilingual — better for Spanish)
- CLI functions: `ingest-document`, `query-knowledge-base`, `update-embedding`
- **Cohere Embed v3 is the call here** — natively multilingual, Spanish embeddings are first-class

### 1C. Content Ingestion Workflow

```
PDF/Web source 
  → extract text (pypdf2 / trafilatura)
  → chunk by behavior category (GPT-4 classification pass)
  → translate to Spanish if English-only (DeepL API)
  → embed with Cohere Embed v3
  → upsert to Supabase pgvector
```

---

## Phase 2: Vision AI Pipeline (Weeks 3–6)

### 2A. MVP — Zero-Shot (Ship First)

```
User uploads video/photo/audio
  → preprocess (extract keyframes if video)
  → send to Claude Vision API with behavior taxonomy prompt
  → parse structured response: {behavior_id, confidence, signals_detected}
  → query Supabase RAG: SELECT top 3 knowledge chunks WHERE behavior_id = X AND breed = Y
  → compose recommendation in Spanish
  → ElevenLabs TTS → return audio response
```

**The system prompt for vision analysis:**
```
Eres un experto en comportamiento canino. Analiza este video/imagen y clasifica el comportamiento del perro usando solo estas categorías: [B01-B15 list in Spanish]. 

Devuelve JSON: {
  "behavior_id": "B0X",
  "confidence": 0.0-1.0,
  "signals_detected": ["señal1", "señal2"],
  "breed_estimate": "optional",
  "severity": 1-5,
  "reasoning_es": "brief explanation in Spanish"
}
```

### 2B. Audio Analysis

For bark classification specifically:
- Google Cloud Speech-to-Text → context clues from any human speech in clip
- Bark audio analysis: pitch (Hz), duration, repetition rate → maps to behavior categories
- Reference: Sophia Yin's bark frequency research (documented in literature)

### 2C. Phase 2 — Fine-Tuned Model (Months 3–6)

Once we have 500+ labeled sessions from beta users:
- Export session data → labeled video clips
- Fine-tune on Google Vertex AI AutoML or Hugging Face ViT
- A/B test fine-tuned vs. zero-shot, keep whichever scores higher per behavior category
- Deploy fine-tuned model as Vercel Edge Function or Cloud Run endpoint

---

## Phase 3: App Interfaces (Weeks 5–10)

### 3A. Mobile App (React Native + Expo)

Core screens:
1. **Home** — camera button, recent sessions, tip of the day (Spanish)
2. **Record** — 15-second video or photo capture
3. **Analysis** — loading state → behavior detected card → correction recommendation
4. **Voice** — ElevenLabs audio plays automatically, Spanish first
5. **Profile** — dog profile (breed, age, name) for personalization
6. **History** — past sessions, patterns over time

### 3B. Web Admin (Next.js on Vercel)

For Mario to monitor:
- Session volume, behavior distribution, confidence scores
- User feedback ratings
- Knowledge base management (add/edit/delete chunks)
- Fine-tuning data export

---

## Tools & Assets We Already Have

| Tool | How It's Used in GuauAI |
|---|---|
| **Claude API (Anthropic SDK)** | Core vision analysis, RAG orchestration, Spanish generation |
| **Supabase** | pgvector knowledge base, user sessions, breed database |
| **Vercel** | Web dashboard hosting, edge functions for API |
| **ElevenLabs** | Spanish TTS voice output — warm, clear female voice |
| **Printing Press** | Generate CLIs for: Cohere Embeddings API, Google Cloud Vision, any new API we add |
| **GitHub (mar2181)** | This repo — all code + plans |
| **Apollo.io** | CRM for beta user recruitment (dog trainers, owners) |
| **Zapier** | Webhook automation between services |
| **HeyGen** | Marketing videos — Spanish-speaking avatar demo of the app |
| **Higgsfield** | Generate app store screenshots, training illustration imagery |

**What We Still Need to Acquire**
- Cohere API key (multilingual embeddings — $20/mo for MVP volume)
- DeepL API key (English→Spanish document translation — $5/mo)
- Dog training books in digital format (PDF purchase: ~$150 total for 10 books)
- 1 ElevenLabs voice selected and locked for GuauAI brand (Spanish female, warm)
- Apple Developer Account ($99/yr) for iOS distribution
- Google Play Developer Account ($25 one-time) for Android distribution

---

## Phase 4: Beta Launch (Weeks 10–14)

### Target Beta Users (50 people)
- 30 Spanish-speaking dog owners (RGV, SoCal, Florida networks)
- 10 dog trainers (bilingual) — use Apollo to find and recruit
- 10 Spanish-speaking rescues / shelters

### Beta Metrics to Track
- Sessions per user per week (target: 3+)
- Behavior detection accuracy rating (target: 4+/5)
- Voice response usefulness rating (target: 4+/5)
- Most common behavior categories detected (informs content priority)
- Churn / retention at week 2 and week 4

### Feedback Loop Automation
- Post-session survey: 3 questions, in-app, Spanish
- Weekly Telegram summary to Mario (behavior patterns, top issues)
- Apollo sequence for beta user follow-up emails (bilingual)

---

## Monetization Model

| Tier | Price | What You Get |
|---|---|---|
| Free | $0 | 5 analyses/month, text only |
| Pro | $9.99/mo | Unlimited analyses, voice output, breed personalization |
| Family | $14.99/mo | Up to 3 dogs, history, weekly pattern report |
| Trainer Pro | $49/mo | White-label, client management, session export |

**Year 1 target:** 500 Pro subscribers = $5,000 MRR
**Year 2 target:** 2,500 subscribers + Trainer Pro tier = $30,000+ MRR

---

## Risk Register

| Risk | Probability | Mitigation |
|---|---|---|
| Zero-shot accuracy too low | Medium | Test Week 1 before committing to architecture |
| Spanish content gap in literature | Medium | DeepL translation pipeline from English sources |
| App Store rejection | Low | No medical claims, add "for entertainment/reference only" disclaimer |
| Competitor copies us | Medium | Speed wins. Ship in 90 days. |
| Training data copyright issues | Low | RAG summarization doesn't reproduce copyrighted text verbatim |

---

## Timeline Summary

| Week | Deliverable |
|---|---|
| 1 | Zero-shot accuracy test (50 clips × 2 models) |
| 2 | Supabase schema live, 5 books ingested to RAG |
| 3 | Full knowledge base (10 books + AKC/ASPCA), Printing Press CLI for embeddings |
| 4 | Vision AI pipeline end-to-end (video → behavior → recommendation) |
| 5 | ElevenLabs Spanish voice integrated |
| 6 | Mobile app shell (camera → upload → response) |
| 8 | Web dashboard live on Vercel |
| 10 | Beta launch: 50 users |
| 14 | Beta debrief, v1.1 priorities locked |
| 20 | App Store + Google Play submission |

---

*Next step: See SOP_MVP.md for the exact execution playbook.*
