<div align="center">

<br />

# ☀️ Brighter Day

**AI intelligence platform for people who want to know first.**

Signals · News · Genius Radar

<br />

[![iOS](https://img.shields.io/badge/iOS-18%2B-black?style=flat-square&logo=apple)](https://apple.com)
[![Swift](https://img.shields.io/badge/Swift-6.0-F05138?style=flat-square&logo=swift)](https://swift.org)
[![License](https://img.shields.io/badge/license-MIT-blue?style=flat-square)](LICENSE)
[![Status](https://img.shields.io/badge/status-in%20development-yellow?style=flat-square)]()

<br />

<img src="assets/mockups.svg" alt="Brighter Day app screens — Signal Feed, Morning Brief, Rising Minds" width="100%"/>

<br />

</div>

---

## What is Brighter Day?

Most people find out about the next big thing in AI **after** it already blew up.

Brighter Day is an iOS app that watches GitHub stars accelerating, arXiv papers dropping, Hacker News threads forming — and surfaces the signal **before** it becomes noise. It also tracks the people behind breakthroughs: young researchers and builders who are quietly doing the work that everyone will talk about in 12 months.

Three layers. One app.

| Layer | What it does |
|---|---|
| 📡 **Signal Feed** | Early detection of repos, papers, and projects gaining traction |
| ⚡ **Morning Brief** | AI-synthesized news digest from 20+ sources, every morning |
| 🧠 **Rising Minds** | Discovers young researchers and builders before they're famous |

<br />

## Features

### 📡 Signal Feed — catch the wave early

- Monitors GitHub for repos gaining 500+ stars in under 48 hours
- Tracks arXiv submission velocity by topic cluster
- Watches Hacker News, Product Hunt, and Reddit r/MachineLearning for breakout threads
- Configurable alert thresholds: star velocity, fork ratio, issue activity
- Filter by domain: LLMs, agents, vision, robotics, tooling, infra

### ⚡ Morning Brief

- Aggregates Hacker News, arXiv, AI blogs, Reddit into a single morning digest
- Claude-powered synthesis: top 7 stories, no filler
- Voice mode: listen like a daily podcast on your commute
- Tap any story to go deeper — full thread, paper abstract, related signals
- Delivered as a push notification at the time you choose

### 🧠 Rising Minds — the genius radar

The most unique feature. When a student or indie researcher publishes something interesting, Brighter Day scores them across four dimensions:

- **Paper signals** — citation growth rate, quality of citing authors, topic relevance
- **Code signals** — GitHub activity, project stars, commit frequency
- **Social signals** — who is engaging with their work, are top researchers taking notice
- **Network signals** — which lab, advisor, past collaborators

The output is a card: *"Watch this person. They'll be everywhere in 2 years."*

### 🕸️ Relationship Graph

- Visualize who works with whom across labs and companies
- Track researcher migrations: 3 hires from one MIT group → that group is worth watching
- See co-authorship clusters and where ideas are flowing between institutions

### 🔍 Lab Intel

- Job postings from Anthropic, OpenAI, DeepMind, xAI, Mistral, and 20+ labs
- API changelog monitoring — what changed quietly in the docs
- Patent filings tracker

<br />

## Architecture

```
┌─────────────────────────────────────────────────────────┐
│                iOS App (SwiftUI)                         │
│     Signals · Brief · Rising Minds · Graph · Alerts      │
└────────────────────────┬────────────────────────────────┘
                         │ REST / WebSocket
┌────────────────────────▼────────────────────────────────┐
│                  API Server (FastAPI)                     │
│           Auth · Feed · Search · Profiles                │
└──────┬──────────────┬───────────────┬───────────────────┘
       │              │               │
┌──────▼──────┐ ┌─────▼──────┐ ┌─────▼──────────────────┐
│  Collector  │ │   Ranker   │ │   Synthesis Engine       │
│  (scrapers) │ │ (scoring)  │ │   Claude API + TTS       │
└──────┬──────┘ └─────┬──────┘ └─────┬──────────────────┘
       │              │               │
┌──────▼──────────────▼───────────────▼──────────────────┐
│                     Supabase                            │
│           Postgres · pgvector · Realtime                │
└─────────────────────────────────────────────────────────┘
```

### Data Sources

| Source | What we collect | Method |
|---|---|---|
| GitHub | Stars, forks, commits, topics | REST API |
| arXiv | New papers, authors, abstracts | OAI-PMH feed |
| Hacker News | Posts, scores, velocity | Algolia API |
| Reddit | r/MachineLearning, r/LocalLLaMA | Reddit API |
| RSS | 30+ AI labs and researcher blogs | Feed parser |
| Exa.ai | Twitter/X signal extraction | Exa API |
| Job boards | Lab postings across major employers | Scraper |

<br />

## Tech Stack

**iOS**
- Swift 6 / SwiftUI
- Combine + async/await
- AVFoundation — voice playback
- Swift Charts — signal graphs

**Backend**
- Python 3.12 + FastAPI
- Celery + Redis — async collectors
- Supabase — Postgres + pgvector + Realtime
- Claude API — synthesis, scoring, summarisation
- OpenAI TTS — voice briefing
- Railway — deployment

<br />

## Roadmap

- [x] Concept & architecture
- [ ] Data collectors — GitHub, HN, arXiv, RSS
- [ ] Ranking & scoring engine
- [ ] Claude synthesis pipeline
- [ ] Morning brief with TTS voice
- [ ] iOS app — Signal Feed screen
- [ ] iOS app — Morning Brief + audio player
- [ ] iOS app — Rising Minds cards
- [ ] Genius scoring model v1
- [ ] Relationship graph
- [ ] Push notifications & custom alerts
- [ ] App Store launch
- [ ] Pro subscription via RevenueCat
- [ ] Lab Intel module
- [ ] Teams & API access

<br />

## Getting Started

### Prerequisites

- Xcode 16+
- Python 3.12+
- Supabase account
- Claude API key (Anthropic)

### Backend

```bash
git clone https://github.com/yourusername/brighter-day
cd brighter-day/backend

python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt

cp .env.example .env
# add your API keys

uvicorn main:app --reload

# start collectors
celery -A tasks worker --loglevel=info
```

### iOS App

```bash
cd brighter-day/ios
open BrighterDay.xcodeproj
```

Set your API base URL in `Config.swift`, then build and run on simulator or device.

<br />

## Pricing

| Tier | Price | What you get |
|---|---|---|
| Free | $0 | Morning brief, top 5 signals (24h delay), 3 rising minds cards/week |
| Pro | $19/mo | Real-time signals, full genius radar, relationship graph, custom alerts |
| Teams | $79/mo | Everything in Pro + CSV export, API access, Slack integration |

<br />

## Why this exists

The AI space moves faster than any newsletter can track. By the time something hits your inbox, it already has 10k GitHub stars and a TechCrunch article.

The people who consistently stay ahead don't read more — they watch different signals. They notice a grad student's repo before it blows up. They see a cluster of talent leaving one lab for another. They read job descriptions as strategy documents.

Brighter Day automates that intuition.

<br />

## Contributing

Contributions welcome. Open an issue to discuss before submitting large PRs.

```bash
git checkout -b feature/your-feature
# make your changes
git commit -m "feat: describe your change"
git push origin feature/your-feature
# open a PR
```

<br />

## License

MIT — see [LICENSE](LICENSE)

<br />

---

<div align="center">

Built with obsession · Powered by Claude · Made for people who want to know first

</div>
