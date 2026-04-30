<div align="center">

<br />

# ☀️ Brighter Day

**The AI intelligence platform for people who want to know first.**

Signals · Morning Brief · Genius Radar · Relationship Graph · Lab Intel

<br />

[![iOS](https://img.shields.io/badge/iOS-18%2B-black?style=flat-square&logo=apple)](https://apple.com)
[![Swift](https://img.shields.io/badge/Swift-6.0-F05138?style=flat-square&logo=swift)](https://swift.org)
[![Python](https://img.shields.io/badge/Python-3.12-3776AB?style=flat-square&logo=python)](https://python.org)
[![FastAPI](https://img.shields.io/badge/FastAPI-0.115-009688?style=flat-square&logo=fastapi)](https://fastapi.tiangolo.com)
[![License](https://img.shields.io/badge/license-MIT-blue?style=flat-square)](LICENSE)
[![Status](https://img.shields.io/badge/status-in%20development-yellow?style=flat-square)]()

<br />

<img src="assets/mockups.svg" alt="Brighter Day app screens — Signal Feed, Morning Brief, Rising Minds" width="100%"/>

<br />

</div>

---

## What is Brighter Day?

Most people find out about the next big thing in AI **after** it already blew up.

**Brighter Day** is an iOS app that watches GitHub stars accelerating, arXiv papers dropping, and Hacker News threads forming — and surfaces the signal **before** it becomes noise. It also tracks the *people* behind breakthroughs: researchers and builders who are quietly doing the work that everyone will talk about in 12 months.

Five layers. One app.

| Layer | What it does |
|---|---|
| 📡 **Signal Feed** | Early detection of repos, papers, and projects gaining traction |
| ⚡ **Morning Brief** | AI-synthesized news digest from 20+ sources, every morning |
| 🧠 **Rising Minds** | Discovers researchers and builders before they're famous |
| 🕸️ **Relationship Graph** | Visualize talent flows and co-authorship clusters across labs |
| 🔍 **Lab Intel** | Job postings, API changelogs, and patent filings from top AI labs |

<br />

## Features

### 📡 Signal Feed — catch the wave early

- Monitors GitHub for repos gaining **500+ stars in under 48 hours**
- Tracks arXiv submission velocity by topic cluster (LLMs, agents, vision, robotics…)
- Watches Hacker News, Product Hunt, and Reddit for breakout threads
- Configurable alert thresholds: star velocity, fork ratio, issue activity
- Real-time delivery via Supabase Realtime — no polling, no delay

### ⚡ Morning Brief

- Aggregates Hacker News, arXiv, AI blogs, and Reddit into one daily digest
- **Claude-powered synthesis:** top 7 stories, no filler, no hype
- **Voice mode:** listen like a podcast on your commute (OpenAI TTS)
- Tap any story to go deeper — full thread, paper abstract, related signals
- Delivered as a push notification at the exact time you choose

### 🧠 Rising Minds — the Genius Radar

The most unique feature. Brighter Day scores every emerging researcher across four signal dimensions:

| Dimension | What it measures |
|---|---|
| 📄 **Paper signals** | Citation growth rate, quality of citing authors, topic relevance |
| 💻 **Code signals** | GitHub stars, commit frequency, fork-to-star ratio |
| 📣 **Social signals** | Who is engaging with their work, are top researchers taking notice |
| 🔗 **Network signals** | Lab affiliation, advisor reputation, co-author graph |

The output is a card: *"Watch this person. They'll be everywhere in 2 years."*

### 🕸️ Relationship Graph

- Visualize who works with whom across labs, universities, and companies
- Track researcher migrations: *3 hires from one MIT group → that group is worth watching*
- Trace co-authorship clusters and how ideas are flowing between institutions
- Identify advisor lineages and prolific research trees

### 🔍 Lab Intel

- Job postings from Anthropic, OpenAI, DeepMind, xAI, Mistral, and 20+ labs
- API changelog monitoring — what changed quietly in the docs this week
- Patent filings tracker for major AI IP activity

<br />

## Architecture

```
╔══════════════════════════════════════════════════════════╗
║                    iOS App (SwiftUI)                     ║
║   Signal Feed · Morning Brief · Rising Minds · Graph     ║
║                Lab Intel · Alerts · Search               ║
╚══════════════════╤═══════════════════════════════════════╝
                   │  HTTPS REST  /  WebSocket (Realtime)
╔══════════════════▼═══════════════════════════════════════╗
║               API Server (FastAPI)                       ║
║  /feed  /brief  /minds  /graph  /labs  /auth  /search    ║
╚═══════╤══════════════╤════════════════╤══════════════════╝
        │              │                │
╔═══════▼══════╗ ╔═════▼══════╗ ╔══════▼══════════════════╗
║  Collectors  ║ ║   Ranker   ║ ║    Synthesis Engine      ║
║  (Celery +   ║ ║  (scoring  ║ ║  Claude API → digest     ║
║   Redis)     ║ ║  + ranking)║ ║  OpenAI TTS → audio      ║
╚═══════╤══════╝ ╚═════╤══════╝ ╚══════╤══════════════════╝
        │              │                │
╔═══════▼══════════════▼════════════════▼══════════════════╗
║                      Supabase                            ║
║         Postgres · pgvector · Realtime · Storage         ║
╚══════════════════════════════════════════════════════════╝
        │
╔═══════▼══════════════════════════════════════════════════╗
║               External Data Sources                      ║
║  GitHub · arXiv · Hacker News · Reddit · RSS             ║
║  Exa.ai · Semantic Scholar · Job Boards                  ║
╚══════════════════════════════════════════════════════════╝
```

### Data Flow

```
Source APIs
    │
    ▼
Celery Collectors (scheduled tasks, rate-limited)
    │  raw JSON
    ▼
Normaliser (pydantic models, deduplication)
    │  canonical events
    ▼
Ranker (velocity scoring, novelty scoring, genius scoring)
    │  scored items
    ▼
Postgres + pgvector (persistence + semantic search)
    │
    ├──► Realtime channel → iOS app (live feed)
    └──► Synthesis Engine (daily brief job)
              │  Claude API prompt
              ▼
          Brief text → TTS → audio file → Supabase Storage
              │
              ▼
          Push notification → iOS app
```

### Data Sources

| Source | What we collect | Method | Frequency |
|---|---|---|---|
| GitHub | Stars, forks, commits, topics, contributors | REST API v3 | Every 15 min |
| arXiv | New papers, authors, abstracts, subjects | OAI-PMH feed | Every hour |
| Hacker News | Posts, scores, comment velocity | Algolia API | Every 10 min |
| Reddit | r/MachineLearning, r/LocalLLaMA, r/artificial | Reddit API | Every 30 min |
| Semantic Scholar | Citations, author h-index, paper influence | S2 API | Daily |
| RSS | 30+ AI labs and researcher blogs | Feed parser | Every 30 min |
| Exa.ai | Twitter/X mentions, signal extraction | Exa API | Every hour |
| Job boards | Anthropic, OpenAI, DeepMind, xAI, Mistral… | Scraper | Daily |

<br />

## Project Structure

```
pheme/
├── ios/                          # Swift / SwiftUI iOS app
│   ├── BrighterDay/
│   │   ├── App/                  # App entry point, DI container
│   │   ├── Features/
│   │   │   ├── Feed/             # Signal Feed screen
│   │   │   ├── Brief/            # Morning Brief + audio player
│   │   │   ├── Minds/            # Rising Minds cards
│   │   │   ├── Graph/            # Relationship graph view
│   │   │   ├── LabIntel/         # Lab Intel screen
│   │   │   └── Alerts/           # Custom alert settings
│   │   ├── Core/
│   │   │   ├── Networking/       # API client, WebSocket manager
│   │   │   ├── Models/           # Shared data models
│   │   │   ├── Storage/          # SwiftData / UserDefaults
│   │   │   └── Extensions/
│   │   └── Config.swift          # Environment configuration
│   └── BrighterDay.xcodeproj
│
├── backend/                      # Python / FastAPI backend
│   ├── api/
│   │   ├── routers/              # FastAPI route handlers
│   │   │   ├── feed.py
│   │   │   ├── brief.py
│   │   │   ├── minds.py
│   │   │   ├── graph.py
│   │   │   ├── labs.py
│   │   │   └── auth.py
│   │   ├── models/               # Pydantic request/response schemas
│   │   └── dependencies.py       # Auth, rate limiting, DB deps
│   ├── collectors/               # Celery tasks for each data source
│   │   ├── github.py
│   │   ├── arxiv.py
│   │   ├── hackernews.py
│   │   ├── reddit.py
│   │   ├── semantic_scholar.py
│   │   └── rss.py
│   ├── ranker/
│   │   ├── velocity.py           # Star/view/comment velocity scoring
│   │   ├── genius.py             # Rising Minds multi-dimension scorer
│   │   └── novelty.py            # Semantic novelty via pgvector
│   ├── synthesis/
│   │   ├── brief.py              # Claude prompt + daily brief pipeline
│   │   └── tts.py                # OpenAI TTS → audio generation
│   ├── db/
│   │   ├── migrations/           # Supabase migrations (SQL)
│   │   └── queries/              # Typed query helpers
│   ├── core/
│   │   ├── config.py             # Pydantic settings from env
│   │   ├── celery_app.py         # Celery + Redis setup
│   │   └── supabase_client.py
│   ├── main.py                   # FastAPI app factory
│   ├── requirements.txt
│   └── .env.example
│
├── CLAUDE.md                     # Architecture guide for AI assistants
└── README.md
```

<br />

## Tech Stack

### iOS
| Tool | Purpose |
|---|---|
| Swift 6 / SwiftUI | UI framework |
| Combine + async/await | Reactive state & async networking |
| AVFoundation | Voice brief playback |
| Swift Charts | Signal velocity graphs |
| SwiftData | Local caching |

### Backend
| Tool | Purpose |
|---|---|
| Python 3.12 + FastAPI | API server |
| Celery + Redis | Async job queue for collectors |
| Supabase (Postgres + pgvector) | Persistence + semantic search |
| Supabase Realtime | WebSocket push to iOS client |
| Claude API (claude-opus-4-5) | Synthesis, scoring, summarisation |
| OpenAI TTS | Voice brief audio generation |
| Railway | Deployment (API + Celery workers) |

<br />

## Roadmap

### Phase 1 — Foundation
- [x] Concept & architecture
- [ ] Supabase schema + migrations
- [ ] Data collectors — GitHub, HN, arXiv, RSS
- [ ] Velocity ranking engine
- [ ] FastAPI core routes (`/feed`, `/brief`, `/auth`)

### Phase 2 — Intelligence
- [ ] Claude synthesis pipeline
- [ ] Morning brief with TTS voice
- [ ] Genius scoring model v1 (Rising Minds)
- [ ] Semantic Scholar integration
- [ ] pgvector semantic novelty scoring

### Phase 3 — iOS App
- [ ] Signal Feed screen with real-time updates
- [ ] Morning Brief screen + audio player
- [ ] Rising Minds cards
- [ ] Relationship graph (SwiftUI + force-directed layout)
- [ ] Push notifications & custom alerts

### Phase 4 — Growth
- [ ] App Store launch
- [ ] Pro subscription via RevenueCat
- [ ] Lab Intel module
- [ ] Teams & API access (CSV export, Slack integration)
- [ ] Android / web companion

<br />

## Getting Started

### Prerequisites

| Tool | Version |
|---|---|
| Xcode | 16+ |
| Python | 3.12+ |
| Redis | 7+ |
| Supabase account | — |
| Anthropic API key | — |
| OpenAI API key | — |

### Backend

```bash
git clone https://github.com/mythosic/pheme
cd pheme/backend

python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt

cp .env.example .env
# fill in SUPABASE_URL, SUPABASE_KEY, ANTHROPIC_API_KEY, OPENAI_API_KEY, etc.

# run database migrations
supabase db push

# start API server
uvicorn main:app --reload

# start Celery worker (separate terminal)
celery -A core.celery_app worker --loglevel=info

# start Celery beat scheduler (separate terminal)
celery -A core.celery_app beat --loglevel=info
```

### iOS App

```bash
cd pheme/ios
open BrighterDay.xcodeproj
```

Set `API_BASE_URL` and `SUPABASE_ANON_KEY` in `Config.swift`, then build and run on a simulator or device (iOS 18+).

<br />

## Environment Variables

```bash
# Supabase
SUPABASE_URL=https://your-project.supabase.co
SUPABASE_SERVICE_KEY=your-service-key
SUPABASE_ANON_KEY=your-anon-key

# AI
ANTHROPIC_API_KEY=sk-ant-...
OPENAI_API_KEY=sk-...

# Data sources
GITHUB_TOKEN=ghp_...
REDDIT_CLIENT_ID=...
REDDIT_CLIENT_SECRET=...
EXA_API_KEY=...

# Infrastructure
REDIS_URL=redis://localhost:6379/0
DATABASE_URL=postgresql://...

# App
ENVIRONMENT=development  # development | staging | production
SECRET_KEY=your-secret-key-for-jwt
```

<br />

## Pricing

| Tier | Price | What you get |
|---|---|---|
| **Free** | $0/mo | Morning brief, top 5 signals (24h delay), 3 Rising Minds cards/week |
| **Pro** | $19/mo | Real-time signals, full Genius Radar, relationship graph, custom alerts |
| **Teams** | $79/mo | Everything in Pro + CSV export, API access, Slack integration |

<br />

## Why this exists

The AI space moves faster than any newsletter can track. By the time something hits your inbox, it already has 10k GitHub stars and a TechCrunch article.

The people who consistently stay ahead don't read *more* — they watch *different* signals. They notice a grad student's repo before it blows up. They see a cluster of talent leaving one lab for another. They read job descriptions as strategy documents.

Brighter Day automates that intuition. It watches the signals you can't watch yourself, and surfaces the ones that matter — before they matter to everyone else.

<br />

## Contributing

Contributions are welcome. Please open an issue to discuss before submitting large PRs.

```bash
# clone and set up
git clone https://github.com/mythosic/pheme
cd pheme

# create a feature branch
git checkout -b feature/your-feature-name

# make your changes, then commit using conventional commits
git commit -m "feat: describe your change"
# or: fix: / docs: / refactor: / test: / chore:

git push origin feature/your-feature-name
# open a PR against main
```

See [CLAUDE.md](CLAUDE.md) for full architecture details, coding conventions, and a guide to adding new collectors or API routes.

<br />

## License

MIT — see [LICENSE](LICENSE)

<br />

---

<div align="center">

Built with obsession · Powered by Claude · Made for people who want to know first

</div>
