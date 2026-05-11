# GuauAI — Master Plan v1.0
*May 11, 2026 | Project Manager: Antigravity Digital*

---

## Strategic Decision: Zero-Shot First, Fine-Tune Later

The single most important call in this plan: we do NOT need to train a custom model to ship an MVP. Modern vision AI (Claude Vision, GPT-4V) can already identify dog behavior signals from video frames and photos with zero-shot prompting — no labeled dataset, no training pipeline, no GPU compute. We ship on zero-shot, measure accuracy with real users, and only invest in fine-tuning when we have the data to justify it. This cuts 8–12 weeks off the launch timeline.

---

## Phase 0: Research (Weeks 1–3)

### 0A. Zero-Shot Accuracy Test — FIRST THING WE DO

Before any other work, we validate the core assumption.

**Test protocol:**
1. Gather 50 short video clips of dogs showing clear behaviors (from YouTube, labeled channels)
2. Extract keyframes (1–3 frames per clip)
3. Send to Claude Vision with the behavior taxonomy prompt
4. Send same set to GPT-4V via OpenRouter
5. Score against ground-truth labels
6. Decision gate: >70% accuracy → ship on zero-shot; <50% → evaluate Google Video Intelligence + Amazon Rekognition

**Expected result:** >75% on clear behavioral signals (fear, aggression, play, resource guarding). Custom training is Phase 2.

---

### 0B. Behavior Taxonomy (15 Core Categories)

The classification vocabulary everything else is built on.

| ID | Behavior | Spanish Label | Priority |
|---|---|---|---|
| B01 | Resource guarding (food/toy/space) | Guardia de recursos | HIGH |
| B02 | Fear aggression | Agresión por miedo | HIGH |
| B03 | Dominance display | Demostración de dominancia | HIGH |
| B04 | Separation anxiety | Ansiedad por separación | HIGH |
| B05 | Excessive barking | Ladrido excesivo | HIGH |
| B06 | Leash reactivity | Reactividad con correa | HIGH |
| B07 | Destructive behavior | Comportamiento destructivo | MED |
| B08 | Play aggression / mouthing | Mordida en juego | MED |
| B09 | Submissive urination | Micción sumisa | MED |
| B10 | Calming signals | Señales de calma | MED |
| B11 | Prey drive / chasing | Impulso de presa | MED |
| B12 | Compulsive behavior | Comportamiento compulsivo | LOW |
| B13 | Attention-seeking | Búsqueda de atención | LOW |
| B14 | Jumping / counter-surfing | Saltar sobre personas | LOW |
| B15 | Inter-dog aggression | Agresión entre perros | HIGH |

---

### 0C. Knowledge Base Sources — What to Acquire

**Books to purchase in digital format (~$150 total)**

| Title | Author | Language | Why |
|---|---|---|---|
| Cómo hacer que tu perro te obedezca | Cesar Millan | ES | Spanish-native, massive brand recognition |
| El encantador de perros | Cesar Millan | ES | Pack dynamics, dominance signals |
| Don't Shoot the Dog | Karen Pryor | EN | Positive reinforcement science bible |
| The Other End of the Leash | Patricia McConnell | EN | Species-specific behavioral signals |
| On Talking Terms With Dogs: Calming Signals | Turid Rugaas | EN | Visual signal taxonomy (tail, eyes, posture) |
| Decoding Your Dog | ACVB | EN | Clinical, breed-specific, authoritative |
| How Dogs Learn | Burch & Bailey | EN | Learning theory for corrections |
| Mine! Resource Guarding Guide | Jean Donaldson | EN | Deep-dive one behavior (B01) |
| The Power of Positive Dog Training | Pat Miller | EN | Modern LIMA-based methods |
| Dominance: Fact or Fiction | Barry Eaton | EN | Counter-theory for balanced recommendations |

**Free online corpus sources (scrape/parse)**
- AKC breed-specific behavior guides → akc.org (200+ breed pages)
- ASPCA Animal Behavior resource library → aspca.org/pet-care/dog-care/dog-behavior
- IAABC position statements and articles → iaabc.org
- VCA Hospitals behavior library → vcahospitals.com
- AVSAB (American Vet Society of Animal Behavior) position papers
- r/Dogtraining — 5+ years of Q&A, real-world scenarios, 500K+ posts (Reddit API)

**Video corpus for future fine-tuning (Phase 2)**
- YouTube channels with labeled behavior clips:
  - Kikopup (Emily Larlham) — best label quality
  - Zak George's Dog Training Revolution
  - McCann Dog Training
  - Victoria Stilwell Positively
- Kaggle: search "dog behavior classification" + "dog action recognition"
- Academic: DECADE dataset (Dog Ethogram Categories And Data Extraction)
- User-generated: every beta session becomes labeled training data

---

### 0D. Competitive Landscape

| App | What It Does | Our Edge |
|---|---|---|
| GoodPup | Live video coaching with human trainers | We're AI, instant, 24/7, fraction of the cost |
| Puppr | Step-by-step trick training | We solve PROBLEM behaviors, not tricks |
| Dogo | Training plans + clicker | No vision AI, English-only, generic plans |
| Barkibu (ES market) | Vet + behavior Q&A | Text-only, no vision, no voice |
| Woofz | AI training plans | No behavior recognition, no Spanish |
| **GuauAI** | **Vision AI + Spanish voice + breed-aware RAG** | **Nobody is doing this combination** |

**Market size signal:**
- 65M+ dog-owning households in US
- 41M+ Hispanic Americans — grossly underserved by English-only dog apps
- LATAM: 80M+ pet owners, near-zero AI pet training tools in Spanish
- $5.4B global pet tech market growing 25% YoY

---

## Phase 1: Knowledge Base Construction (Weeks 2–4)

### 1A. Supabase Schema

```sql
-- Enable vector extension
CREATE EXTENSION IF NOT EXISTS vector;

-- Breed reference
CREATE TABLE breeds (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  name_es TEXT NOT NULL,
  name_en TEXT NOT NULL,
  group_type TEXT,
  common_behavior_ids TEXT[],  -- e.g. ['B01','B03','B06']
  temperament_es TEXT,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Behavior taxonomy
CREATE TABLE behaviors (
  id TEXT PRIMARY KEY,  -- 'B01' through 'B15'
  name_es TEXT NOT NULL,
  name_en TEXT NOT NULL,
  description_es TEXT,
  triggers TEXT[],
  severity_default INTEGER,  -- 1-5
  correction_approaches TEXT[]  -- ['positive_reinforcement','management','counter_conditioning']
);

-- RAG knowledge chunks
CREATE TABLE knowledge_chunks (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  source_title TEXT NOT NULL,
  behavior_id TEXT REFERENCES behaviors(id),
  breed_id uuid REFERENCES breeds(id),  -- NULL = all breeds
  content_es TEXT NOT NULL,
  content_en TEXT,
  embedding vector(1536),
  chunk_type TEXT,  -- 'correction','explanation','exercise','warning'
  quality_score FLOAT DEFAULT 0.8,
  created_at TIMESTAMPTZ DEFAULT NOW()
);
CREATE INDEX ON knowledge_chunks USING ivfflat (embedding vector_cosine_ops);

-- User analysis sessions
CREATE TABLE analysis_sessions (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id uuid,
  media_url TEXT,
  media_type TEXT,  -- 'video','photo','audio'
  detected_behavior_id TEXT REFERENCES behaviors(id),
  confidence_score FLOAT,
  breed_id uuid REFERENCES breeds(id),
  dog_name TEXT,
  recommendation_es TEXT,
  recommendation_en TEXT,
  audio_url TEXT,  -- ElevenLabs TTS output
  user_rating INTEGER,  -- 1-5 helpfulness feedback
  correction_worked BOOLEAN,  -- follow-up: did it work?
  created_at TIMESTAMPTZ DEFAULT NOW()
);
```

### 1B. RAG Embedding Strategy

**Embedding model: Cohere Embed v3 (multilingual)**
- Why Cohere over OpenAI: natively multilingual, Spanish embeddings are first-class citizens, not an afterthought
- Dimension: 1024 (fits pgvector, lower cost than 1536)
- Use Printing Press to generate the `guau-embed` CLI: `ingest-document`, `query-knowledge-base`, `update-chunk`

**Ingestion pipeline:**
```
PDF/HTML source
  → extract text (pypdf2 for books, trafilatura for web)
  → segment by topic/behavior (GPT-4o classification pass)
  → translate EN→ES if needed (DeepL API, ~$0.002 per 500 chars)
  → embed with Cohere Embed v3 (Spanish content)
  → upsert to Supabase knowledge_chunks with behavior_id tag
```

---

## Phase 2: Vision AI Pipeline (Weeks 3–6)

### 2A. The Core System Prompt (Spanish, Zero-Shot)

```
Eres un experto certificado en comportamiento canino con 20 años de experiencia.
Analiza este video/imagen y clasifica el comportamiento del perro.

CATEGORÍAS DISPONIBLES:
B01: Guardia de recursos | B02: Agresión por miedo | B03: Demostración de dominancia
B04: Ansiedad por separación | B05: Ladrido excesivo | B06: Reactividad con correa
B07: Comportamiento destructivo | B08: Mordida en juego | B09: Micción sumisa
B10: Señales de calma | B11: Impulso de presa | B12: Comportamiento compulsivo
B13: Búsqueda de atención | B14: Saltar sobre personas | B15: Agresión entre perros

SEÑALES A OBSERVAR: posición de cola, postura corporal, posición de orejas,
expresión facial, nivel de energía, contexto ambiental, tipo de vocalización.

Responde SOLO con este JSON:
{
  "behavior_id": "B0X",
  "confidence": 0.0-1.0,
  "signals_detected": ["señal específica 1", "señal específica 2"],
  "severity": 1-5,
  "breed_estimate": "raza o null",
  "reasoning_es": "explicación breve en español de máximo 2 oraciones"
}
```

### 2B. Full Request Pipeline

```
User uploads media (mobile)
  ↓
Vercel API route: /api/analyze
  ↓
If video: extract 3 keyframes (ffmpeg edge function)
  ↓
Claude Vision API → structured JSON {behavior_id, confidence, signals}
  ↓
Supabase RAG query: top 3 knowledge_chunks WHERE behavior_id = X
  (bonus: filter by breed if user has breed profile)
  ↓
Claude API: compose correction recommendation in Spanish
  (1 paragraph explanation + 3 concrete steps)
  ↓
ElevenLabs TTS: generate Spanish audio (warm female voice, ~30 seconds)
  ↓
Store session in analysis_sessions
  ↓
Return to mobile: {behavior_label_es, severity, recommendation_es, audio_url}
```

### 2C. Audio Analysis (Bark Classification)

For audio-only clips:
- Extract pitch (Hz), duration (ms), repetition rate (barks/sec)
- Map to behavior taxonomy based on Sophia Yin's bark frequency research:
  - High pitch, rapid → excitement / play
  - Low pitch, sustained → territorial / warning
  - Irregular, yelping → fear / pain
  - Rhythmic, medium pitch → attention-seeking / separation anxiety
- Send waveform + description to Claude for final classification

### 2D. Phase 2 — Fine-Tuned Model (Month 3–6, Data-Dependent)

Trigger: 500+ rated sessions in database with user_rating >= 4

Steps:
1. Export sessions → labeled video clips (behavior_id = ground truth)
2. Fine-tune on Google Vertex AI AutoML Video or Hugging Face ViT
3. A/B test fine-tuned vs. zero-shot per behavior category
4. Deploy winning model per category (ensemble if needed)

---

## Phase 3: App Interfaces (Weeks 5–10)

### 3A. Mobile App — Core Screens (React Native + Expo)

1. **Bienvenida / Onboarding** — dog name, breed, age → creates personalized profile
2. **Inicio** — big camera button, "¿Qué está haciendo tu perro ahora?" CTA, recent sessions
3. **Grabar** — 15-second video or photo, or audio-only option
4. **Analizando...** — animated loading with dog facts in Spanish
5. **Resultado** — behavior card (icon + name in Spanish), severity indicator, audio plays automatically
6. **Corrección** — 3-step correction plan, expandable detail, save to favorites
7. **Historial** — past sessions, behavior frequency chart ("tu perro ha mostrado B05 4 veces esta semana")
8. **Perfil de tu perro** — breed, age, behavior history, progress tracking

### 3B. Web Admin (Next.js on Vercel)

Internal dashboard for Mario:
- Session volume by day, behavior distribution pie chart
- User feedback ratings trend
- Knowledge base browser — add/edit/delete chunks
- Fine-tuning data export button
- Revenue + subscriber metrics

---

## Phase 4: Beta Launch (Weeks 10–14)

### Target: 50 Beta Users

**Segment breakdown:**
- 30 Spanish-speaking dog owners (RGV, SoCal, Florida, Chicago)
- 10 bilingual dog trainers (they give us quality feedback AND become Trainer Pro prospects)
- 10 Spanish-speaking rescues or shelters (high volume, real behaviors)

**Recruitment via Apollo.io:**
- Search: dog trainers in RGV, McAllen, Laredo, El Paso, San Antonio
- Filter: Spanish-speaking, small business
- Sequence: 3-email bilingual outreach, offer 6 months free Pro
- Target: 10 trainers signed up in Week 9

### Beta Metrics (4-Week Dashboard)

| Metric | Target | Green | Red |
|---|---|---|---|
| Sessions per user per week | 3+ | >3 | <1 |
| Behavior detection accuracy (self-reported) | 4+/5 | >4 | <3 |
| Voice response usefulness | 4+/5 | >4 | <3 |
| Week 2 retention | 70%+ | >70% | <40% |
| Top behavior categories | Know top 3 | Data collected | No data |

### Feedback Loop Automation

- Post-session: 3-question in-app survey (Spanish), stored in analysis_sessions
- Weekly: Telegram summary to Mario (behavior patterns, rating trends, churn signal)
- Week 4 debrief: Apollo email sequence to all beta users, 30-min interview offer

---

## Monetization Model

| Tier | Price | What You Get |
|---|---|---|
| Free | $0/mo | 5 analyses/month, text only, no voice |
| Pro | $9.99/mo | Unlimited analyses, Spanish voice, breed personalization, history |
| Familia | $14.99/mo | Up to 3 dogs, weekly pattern report, priority support |
| Entrenador Pro | $49/mo | White-label, client management, session export, API access |

**Revenue milestones:**
- Month 6: 200 Pro subscribers = $2,000 MRR
- Year 1: 1,000 subscribers (mix of tiers) = ~$12,000 MRR
- Year 2: 5,000 subscribers + Entrenador Pro tier = $60,000+ MRR
- Year 3: LATAM expansion → 25,000 subscribers = $250,000+ MRR

---

## Risk Register

| Risk | Probability | Mitigation |
|---|---|---|
| Zero-shot accuracy too low to ship | Medium | Test Week 1. If fails, pivot to Google Video Intelligence (proven on animal behavior) |
| Spanish training content gap | Medium | DeepL translation pipeline covers all English sources. Cesar Millan books are native ES. |
| App Store medical/behavioral advice rejection | Low | Disclaimer: "For reference only. Consult a certified trainer for serious behavior issues." |
| Competitor copies Spanish-first angle | Medium | Speed wins. 90-day MVP target. First-mover brand in RGV + LATAM establishes authority. |
| Training data copyright | Low | RAG summarizes/synthesizes, does not reproduce verbatim. Same as every RAG product. |
| ElevenLabs voice quality issues in Spanish | Low | Pre-select + lock a voice in Week 1. Test 5 voices, pick the best. Done. |

---

## Timeline

| Week | Deliverable | Owner |
|---|---|---|
| 1 | Zero-shot accuracy test (50 clips) | Claude |
| 1 | Behavior taxonomy finalized (B01–B15 with full Spanish definitions) | Claude |
| 2 | Supabase schema deployed, 3 books ingested | Claude |
| 3 | Full knowledge base (10 books + AKC/ASPCA), Printing Press embed CLI | Claude |
| 4 | Vision pipeline end-to-end: video → behavior_id → recommendation_es | Claude |
| 4 | ElevenLabs voice selected and integrated | Claude |
| 5 | Mobile app shell: camera → upload → result screen | Claude |
| 6 | Dog profile (breed, age) personalization layer | Claude |
| 7 | Audio bark analysis path | Claude |
| 8 | Web admin dashboard on Vercel | Claude |
| 9 | Apollo beta recruitment sequence launched | Mario + Claude |
| 10 | Beta launch: 50 users onboarded | Mario |
| 14 | Beta debrief, v1.1 priorities locked | Mario + Claude |
| 18 | Fine-tuning pipeline (if data threshold met) | Claude |
| 20 | App Store + Google Play submission | Mario + Claude |

---

*See mvp-sop/ folder for step-by-step execution SOP.*
