# CLAUDE.md — Brighter Day (pheme)

This file is the primary reference for AI assistants (Claude, Copilot, etc.) working in this repository. Read this before making any changes.

---

## Table of Contents

1. [Project Overview](#1-project-overview)
2. [Repository Layout](#2-repository-layout)
3. [Backend Architecture](#3-backend-architecture)
4. [iOS Architecture](#4-ios-architecture)
5. [Database Schema](#5-database-schema)
6. [API Design](#6-api-design)
7. [Scoring Algorithms](#7-scoring-algorithms)
8. [Synthesis Pipeline](#8-synthesis-pipeline)
9. [Celery Task Schedule](#9-celery-task-schedule)
10. [Coding Conventions](#10-coding-conventions)
11. [Adding a New Collector](#11-adding-a-new-collector)
12. [Adding a New API Route](#12-adding-a-new-api-route)
13. [Testing Strategy](#13-testing-strategy)
14. [Deployment](#14-deployment)
15. [Key Design Decisions](#15-key-design-decisions)

---

## 1. Project Overview

**Brighter Day** (codebase: `pheme`) is an iOS app + Python backend that surfaces early signals in the AI/ML space:

- **Signal Feed** — repos, papers, and posts gaining traction before they go viral
- **Morning Brief** — Claude-synthesized daily digest with TTS audio
- **Rising Minds** — scores researchers on a "genius radar" across paper, code, social, and network signals
- **Relationship Graph** — maps co-authorship clusters and talent migrations between labs
- **Lab Intel** — job postings, API changelogs, and patent filings from top AI organisations

The app is **iOS-only** (SwiftUI, iOS 18+). The backend is **Python 3.12 + FastAPI**, with Celery workers for async data collection, Supabase (Postgres + pgvector) for storage, and Claude for synthesis.

---

## 2. Repository Layout

```
pheme/
├── ios/                          # Swift / SwiftUI iOS app
│   ├── BrighterDay/
│   │   ├── App/
│   │   │   ├── BrighterDayApp.swift      # @main entry point
│   │   │   └── DependencyContainer.swift # Manual DI container
│   │   ├── Features/
│   │   │   ├── Feed/
│   │   │   │   ├── FeedView.swift
│   │   │   │   ├── FeedViewModel.swift
│   │   │   │   └── SignalCardView.swift
│   │   │   ├── Brief/
│   │   │   │   ├── BriefView.swift
│   │   │   │   ├── BriefViewModel.swift
│   │   │   │   └── AudioPlayerView.swift
│   │   │   ├── Minds/
│   │   │   │   ├── MindsView.swift
│   │   │   │   ├── MindsViewModel.swift
│   │   │   │   └── ResearcherCardView.swift
│   │   │   ├── Graph/
│   │   │   │   ├── GraphView.swift
│   │   │   │   └── GraphViewModel.swift
│   │   │   ├── LabIntel/
│   │   │   │   ├── LabIntelView.swift
│   │   │   │   └── LabIntelViewModel.swift
│   │   │   └── Alerts/
│   │   │       ├── AlertsView.swift
│   │   │       └── AlertsViewModel.swift
│   │   ├── Core/
│   │   │   ├── Networking/
│   │   │   │   ├── APIClient.swift       # URLSession wrapper
│   │   │   │   ├── RealtimeClient.swift  # Supabase Realtime WS
│   │   │   │   └── Endpoints.swift
│   │   │   ├── Models/                   # Shared Codable structs
│   │   │   ├── Storage/
│   │   │   │   ├── LocalStore.swift      # SwiftData container
│   │   │   │   └── UserPreferences.swift # @AppStorage helpers
│   │   │   └── Extensions/
│   │   └── Config.swift                  # API_BASE_URL, keys
│   └── BrighterDay.xcodeproj
│
├── backend/
│   ├── api/
│   │   ├── routers/
│   │   │   ├── feed.py           # GET /feed
│   │   │   ├── brief.py          # GET /brief, GET /brief/audio
│   │   │   ├── minds.py          # GET /minds, GET /minds/{id}
│   │   │   ├── graph.py          # GET /graph/nodes, /graph/edges
│   │   │   ├── labs.py           # GET /labs/jobs, /labs/changelogs
│   │   │   ├── search.py         # GET /search
│   │   │   └── auth.py           # POST /auth/token, /auth/refresh
│   │   ├── models/
│   │   │   ├── signal.py         # Signal, SignalType, SignalSource
│   │   │   ├── brief.py          # Brief, BriefItem
│   │   │   ├── researcher.py     # Researcher, GeniusScore
│   │   │   ├── graph.py          # GraphNode, GraphEdge
│   │   │   └── auth.py           # Token, UserProfile
│   │   └── dependencies.py       # current_user(), db(), rate_limit()
│   │
│   ├── collectors/               # Celery tasks (one file per source)
│   │   ├── base.py               # BaseCollector abstract class
│   │   ├── github.py
│   │   ├── arxiv.py
│   │   ├── hackernews.py
│   │   ├── reddit.py
│   │   ├── semantic_scholar.py
│   │   ├── exa.py
│   │   └── rss.py
│   │
│   ├── ranker/
│   │   ├── velocity.py           # Star/view/comment velocity
│   │   ├── genius.py             # Multi-dimension researcher scorer
│   │   └── novelty.py            # Semantic novelty (pgvector cosine)
│   │
│   ├── synthesis/
│   │   ├── brief.py              # Claude prompt builder + pipeline
│   │   ├── tts.py                # OpenAI TTS → MP3 → Supabase Storage
│   │   └── prompts/
│   │       ├── morning_brief.txt  # System + user prompt template
│   │       └── genius_score.txt   # Researcher scoring prompt
│   │
│   ├── db/
│   │   ├── migrations/           # SQL migration files (Supabase)
│   │   │   ├── 001_initial.sql
│   │   │   ├── 002_pgvector.sql
│   │   │   └── 003_realtime.sql
│   │   └── queries/
│   │       ├── signals.py
│   │       ├── researchers.py
│   │       └── briefs.py
│   │
│   ├── core/
│   │   ├── config.py             # Pydantic BaseSettings
│   │   ├── celery_app.py         # Celery factory + beat schedule
│   │   ├── supabase_client.py    # Singleton Supabase client
│   │   └── logging.py            # Structured JSON logging
│   │
│   ├── main.py                   # FastAPI app factory
│   ├── requirements.txt
│   └── .env.example
│
├── CLAUDE.md                     # ← you are here
└── README.md
```

---

## 3. Backend Architecture

### FastAPI App (`backend/main.py`)

The app is created with a factory pattern:

```python
def create_app() -> FastAPI:
    app = FastAPI(title="Brighter Day API", version="0.1.0")
    app.include_router(feed.router, prefix="/feed")
    app.include_router(brief.router, prefix="/brief")
    app.include_router(minds.router, prefix="/minds")
    app.include_router(graph.router, prefix="/graph")
    app.include_router(labs.router, prefix="/labs")
    app.include_router(search.router, prefix="/search")
    app.include_router(auth.router, prefix="/auth")
    return app
```

All routes require a valid JWT except `POST /auth/token`.

### Collectors (`backend/collectors/`)

Each collector extends `BaseCollector`:

```python
class BaseCollector:
    source: SignalSource          # enum value
    rate_limit_rps: float = 1.0   # requests per second

    async def fetch(self) -> list[RawEvent]: ...
    async def normalise(self, raw: RawEvent) -> Signal: ...
    async def run(self): ...      # fetch → normalise → upsert
```

Collectors are Celery tasks registered with `@app.task`. They are **idempotent** — upsert on `(source, source_id)` composite key.

### Ranker (`backend/ranker/`)

The ranker runs after each collector batch and updates `signals.score`:

- **`velocity.py`** — computes a Z-score for star/view/comment growth relative to the rolling 7-day baseline for that signal type.
- **`novelty.py`** — embeds the signal title+abstract (OpenAI `text-embedding-3-small`), stores in `pgvector`, and scores semantic distance from recent items already in the feed. High novelty = low cosine similarity to recent items.
- **`genius.py`** — combines paper signals, code signals, social signals, and network signals into a single 0–100 score for each researcher. Weights are configurable in `core/config.py`.

Final signal score: `score = 0.5 * velocity + 0.3 * novelty + 0.2 * social_boost`

### Synthesis Engine (`backend/synthesis/`)

The morning brief job runs daily at 05:00 UTC:

1. Pull top-50 signals from the last 24h (by score)
2. Pull top-5 Rising Minds updates
3. Build a prompt from `prompts/morning_brief.txt`
4. Call Claude API (`claude-opus-4-5`, max_tokens=2048)
5. Parse structured JSON response → persist `briefs` table
6. Send brief text to OpenAI TTS → upload MP3 to Supabase Storage
7. Trigger push notification via APNs

---

## 4. iOS Architecture

### Pattern: Feature-Sliced MVVM

Each feature (`Feed`, `Brief`, `Minds`, `Graph`, `LabIntel`, `Alerts`) is a self-contained folder with:

```
FeatureView.swift       — SwiftUI View, reads from ViewModel's @Published state
FeatureViewModel.swift  — @Observable class, owns business logic + API calls
Supporting views        — sub-views, cells, detail sheets
```

### State Management

- `@Observable` (Swift 5.9 observation) for ViewModels — no `ObservableObject`/`@StateObject`
- `@AppStorage` for user preferences (theme, notification time, alert thresholds)
- `SwiftData` for offline caching of signals, briefs, and researcher cards
- `Combine` for WebSocket events from Supabase Realtime

### Networking

`APIClient` is a thin `URLSession` wrapper:

```swift
struct APIClient {
    func get<T: Decodable>(_ endpoint: Endpoint) async throws -> T
    func post<B: Encodable, T: Decodable>(_ endpoint: Endpoint, body: B) async throws -> T
}
```

`RealtimeClient` subscribes to Supabase Realtime channels and publishes new signals to a `PassthroughSubject` consumed by `FeedViewModel`.

### Dependency Injection

`DependencyContainer` is created once in `BrighterDayApp` and passed down via SwiftUI's `.environment()`. ViewModels receive their dependencies through initialiser injection.

### Audio (Morning Brief)

`BriefViewModel` uses `AVPlayer` to stream the MP3 from a pre-signed Supabase Storage URL. Playback state (playing, paused, position) is managed inside the ViewModel and reflected in `AudioPlayerView`.

---

## 5. Database Schema

All tables live in Supabase Postgres. Row Level Security (RLS) is enabled on all tables.

```sql
-- Signals (the core feed)
CREATE TABLE signals (
  id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  source      TEXT NOT NULL,              -- 'github' | 'arxiv' | 'hn' | ...
  source_id   TEXT NOT NULL,              -- original ID in source system
  type        TEXT NOT NULL,              -- 'repo' | 'paper' | 'post' | 'thread'
  title       TEXT NOT NULL,
  url         TEXT NOT NULL,
  summary     TEXT,
  score       FLOAT DEFAULT 0,
  velocity    FLOAT DEFAULT 0,
  novelty     FLOAT DEFAULT 0,
  metadata    JSONB,
  embedding   vector(1536),              -- text-embedding-3-small
  created_at  TIMESTAMPTZ DEFAULT now(),
  updated_at  TIMESTAMPTZ DEFAULT now(),
  UNIQUE (source, source_id)
);

-- Researchers (Rising Minds)
CREATE TABLE researchers (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name            TEXT NOT NULL,
  handle          TEXT,                   -- Twitter / GitHub username
  affiliation     TEXT,
  arxiv_id        TEXT UNIQUE,
  github_username TEXT UNIQUE,
  genius_score    FLOAT DEFAULT 0,
  paper_score     FLOAT DEFAULT 0,
  code_score      FLOAT DEFAULT 0,
  social_score    FLOAT DEFAULT 0,
  network_score   FLOAT DEFAULT 0,
  metadata        JSONB,
  embedding       vector(1536),
  first_seen_at   TIMESTAMPTZ DEFAULT now(),
  updated_at      TIMESTAMPTZ DEFAULT now()
);

-- Researcher ↔ Signal edges
CREATE TABLE researcher_signals (
  researcher_id UUID REFERENCES researchers(id) ON DELETE CASCADE,
  signal_id     UUID REFERENCES signals(id) ON DELETE CASCADE,
  role          TEXT,                    -- 'author' | 'contributor' | 'cited_by'
  PRIMARY KEY (researcher_id, signal_id)
);

-- Co-authorship edges (for Relationship Graph)
CREATE TABLE collaborations (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  researcher_a    UUID REFERENCES researchers(id),
  researcher_b    UUID REFERENCES researchers(id),
  paper_count     INT DEFAULT 1,
  first_collab_at TIMESTAMPTZ,
  last_collab_at  TIMESTAMPTZ,
  UNIQUE (researcher_a, researcher_b)
);

-- Daily briefs
CREATE TABLE briefs (
  id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  date        DATE UNIQUE NOT NULL,
  content     JSONB NOT NULL,            -- structured brief from Claude
  audio_url   TEXT,                      -- Supabase Storage presigned URL
  created_at  TIMESTAMPTZ DEFAULT now()
);

-- User alert preferences
CREATE TABLE alert_configs (
  id                UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id           UUID NOT NULL,
  star_velocity     INT DEFAULT 500,     -- stars/48h threshold
  domains           TEXT[],              -- filter by domain tags
  notification_hour INT DEFAULT 7,       -- local time for Morning Brief
  created_at        TIMESTAMPTZ DEFAULT now(),
  UNIQUE (user_id)
);

-- Lab Intel: job postings
CREATE TABLE lab_jobs (
  id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  lab         TEXT NOT NULL,
  title       TEXT NOT NULL,
  url         TEXT NOT NULL UNIQUE,
  department  TEXT,
  location    TEXT,
  posted_at   TIMESTAMPTZ,
  fetched_at  TIMESTAMPTZ DEFAULT now()
);

-- Lab Intel: API changelogs
CREATE TABLE api_changelogs (
  id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  lab         TEXT NOT NULL,
  version     TEXT,
  summary     TEXT,
  url         TEXT,
  detected_at TIMESTAMPTZ DEFAULT now()
);
```

### Indexes

```sql
CREATE INDEX ON signals (score DESC);
CREATE INDEX ON signals (created_at DESC);
CREATE INDEX ON signals USING ivfflat (embedding vector_cosine_ops) WITH (lists = 100);
CREATE INDEX ON researchers (genius_score DESC);
CREATE INDEX ON researchers USING ivfflat (embedding vector_cosine_ops) WITH (lists = 100);
```

---

## 6. API Design

All endpoints return JSON. Authenticated routes require `Authorization: Bearer <jwt>`.

### Authentication

```
POST /auth/token          → { access_token, refresh_token, expires_in }
POST /auth/refresh        → { access_token, expires_in }
```

### Feed

```
GET /feed
  ?limit=20&offset=0
  &source=github,arxiv
  &domain=llms,agents
  &since=2024-01-01T00:00:00Z
  → { items: Signal[], total: int, has_more: bool }

GET /feed/{signal_id}
  → Signal (full detail)

WebSocket /feed/live
  → streams new Signal objects as they arrive via Supabase Realtime
```

### Morning Brief

```
GET /brief/latest
  → { date, items: BriefItem[], audio_url }

GET /brief/{date}         → Brief for specific date (YYYY-MM-DD)
GET /brief/audio/{date}   → Redirect to Supabase Storage audio URL
```

### Rising Minds

```
GET /minds
  ?limit=20&offset=0
  &sort=genius_score|paper_score|code_score
  → { researchers: Researcher[], total: int }

GET /minds/{researcher_id}
  → Researcher (full detail + recent signals)

GET /minds/{researcher_id}/signals
  → { signals: Signal[] }
```

### Relationship Graph

```
GET /graph/nodes
  ?affiliation=MIT,Stanford
  &min_genius_score=50
  → { nodes: GraphNode[] }

GET /graph/edges
  ?researcher_id=<uuid>
  &depth=2
  → { edges: GraphEdge[] }
```

### Lab Intel

```
GET /labs/jobs
  ?lab=anthropic,openai
  &department=research
  → { jobs: LabJob[] }

GET /labs/changelogs
  ?lab=anthropic
  &since=2024-01-01
  → { changelogs: ApiChangelog[] }
```

### Search

```
GET /search
  ?q=<query>
  &type=signal|researcher|all
  &limit=10
  → { results: SearchResult[] }
```

Semantic search is powered by `pgvector` — the query is embedded with `text-embedding-3-small` and searched via cosine similarity.

---

## 7. Scoring Algorithms

### Signal Velocity Score

```python
# For a given signal s with metric m (stars, views, comments):
# - Compute hourly growth rate: delta_m / delta_hours
# - Compare to 7-day rolling baseline for that (source, type) pair
# - Z-score: (rate - mean) / std
# - Clip to [0, 1] using sigmoid

def velocity_score(current_rate: float, baseline_mean: float, baseline_std: float) -> float:
    if baseline_std == 0:
        return 0.5
    z = (current_rate - baseline_mean) / baseline_std
    return 1 / (1 + math.exp(-z))   # sigmoid
```

### Semantic Novelty Score

```python
# Embed the signal title + abstract.
# Find the 5 most similar items already in the feed (last 7 days).
# Novelty = 1 - mean(top_5_cosine_similarities)
# High novelty means the signal is semantically unlike recent items.

def novelty_score(embedding: list[float], recent_embeddings: list[list[float]]) -> float:
    sims = [cosine_similarity(embedding, e) for e in recent_embeddings]
    top_5 = sorted(sims, reverse=True)[:5]
    return 1.0 - (sum(top_5) / len(top_5)) if top_5 else 1.0
```

### Genius Score (Rising Minds)

The genius score is a weighted sum of four sub-scores, each in [0, 100]:

| Sub-score | Weight | Signals used |
|---|---|---|
| `paper_score` | 35% | Citation growth rate, influence percentile, citing-author h-index |
| `code_score` | 25% | GitHub stars, commit frequency, fork ratio, recency |
| `social_score` | 25% | Twitter mentions by researchers with high h-index, Hacker News karma |
| `network_score` | 15% | Lab reputation, advisor h-index, co-author graph PageRank |

```python
def genius_score(p: float, c: float, s: float, n: float) -> float:
    return 0.35 * p + 0.25 * c + 0.25 * s + 0.15 * n
```

Weights are stored in `core/config.py` as `GENIUS_WEIGHTS` and can be tuned without code changes by updating environment variables.

---

## 8. Synthesis Pipeline

### Morning Brief

The brief generation job (`synthesis/brief.py`) runs daily at 05:00 UTC.

**Prompt structure** (see `prompts/morning_brief.txt`):

```
System: You are an expert AI researcher and science journalist...

User:
Here are today's top signals from the AI/ML world:

<SIGNALS JSON>

Produce a morning brief with:
1. A one-sentence "TL;DR" of today's most important development
2. Exactly 7 items, each with: headline, 2-sentence summary, why_it_matters, signal_ids[]
3. A "theme of the day" (one phrase)

Respond in valid JSON matching the Brief schema.
```

**Claude call:**

```python
response = anthropic.messages.create(
    model="claude-opus-4-5",
    max_tokens=2048,
    messages=[{"role": "user", "content": prompt}],
    system=SYSTEM_PROMPT,
)
```

### TTS (Voice Brief)

After the brief is persisted, `synthesis/tts.py` sends the brief text to OpenAI TTS:

```python
response = openai.audio.speech.create(
    model="tts-1-hd",
    voice="nova",
    input=brief_text,
)
```

The MP3 is uploaded to Supabase Storage at `briefs/{date}.mp3` and the URL is stored back in the `briefs` table.

---

## 9. Celery Task Schedule

```python
# core/celery_app.py

app.conf.beat_schedule = {
    "collect-github":          {"task": "collectors.github.run",           "schedule": crontab(minute="*/15")},
    "collect-hackernews":      {"task": "collectors.hackernews.run",        "schedule": crontab(minute="*/10")},
    "collect-arxiv":           {"task": "collectors.arxiv.run",             "schedule": crontab(minute=0)},        # hourly
    "collect-reddit":          {"task": "collectors.reddit.run",            "schedule": crontab(minute="*/30")},
    "collect-rss":             {"task": "collectors.rss.run",               "schedule": crontab(minute="*/30")},
    "collect-exa":             {"task": "collectors.exa.run",               "schedule": crontab(minute=0)},        # hourly
    "collect-semantic-scholar":{"task": "collectors.semantic_scholar.run",  "schedule": crontab(hour=2, minute=0)},# 02:00 UTC
    "collect-lab-jobs":        {"task": "collectors.lab_jobs.run",          "schedule": crontab(hour=3, minute=0)},# 03:00 UTC
    "rank-signals":            {"task": "ranker.velocity.run",              "schedule": crontab(minute="*/5")},
    "run-genius-scorer":       {"task": "ranker.genius.run",                "schedule": crontab(hour=4, minute=0)},# 04:00 UTC
    "generate-morning-brief":  {"task": "synthesis.brief.run",              "schedule": crontab(hour=5, minute=0)},# 05:00 UTC
}
```

---

## 10. Coding Conventions

### Python

- **Python 3.12+** with type hints on all functions.
- **Pydantic v2** for all data models and settings.
- **async/await** everywhere in API handlers and collectors. Use `httpx.AsyncClient` for HTTP.
- **Black** for formatting. **Ruff** for linting. `line-length = 100`.
- Imports: stdlib → third-party → local, separated by blank lines.
- No `print()` — use `structlog` logger.
- All Celery tasks must be **idempotent** (upsert, not insert).
- Errors from external APIs must be caught and logged; never crash the worker.

### Swift / iOS

- **Swift 6** strict concurrency — `@MainActor` on all ViewModels.
- `@Observable` (not `ObservableObject`) for ViewModels.
- Avoid `force unwrap` (`!`) — use `guard let` or optional chaining.
- **SwiftLint** enforced. Run before committing.
- Prefer `async/await` over completion handlers.
- Model structs conform to `Codable`, `Identifiable`, `Hashable`.
- All `Date` values stored and transmitted as ISO 8601 strings.

### Commits

Follow **Conventional Commits**:

```
feat: add Reddit collector
fix: handle GitHub rate limit 403
refactor: extract BaseCollector abstract class
docs: update CLAUDE.md with genius score weights
test: add velocity scorer unit tests
chore: bump httpx to 0.27
```

---

## 11. Adding a New Collector

1. Create `backend/collectors/<source>.py`:

```python
from collectors.base import BaseCollector
from api.models.signal import Signal, SignalSource, SignalType

class MySourceCollector(BaseCollector):
    source = SignalSource.MY_SOURCE
    rate_limit_rps = 2.0

    async def fetch(self) -> list[dict]:
        async with httpx.AsyncClient() as client:
            resp = await client.get("https://api.mysource.com/...", headers=self.headers)
            resp.raise_for_status()
            return resp.json()["items"]

    async def normalise(self, raw: dict) -> Signal:
        return Signal(
            source=self.source,
            source_id=str(raw["id"]),
            type=SignalType.POST,
            title=raw["title"],
            url=raw["url"],
            summary=raw.get("description"),
        )
```

2. Register the Celery task in the same file:

```python
from core.celery_app import app

@app.task(name="collectors.my_source.run")
def run():
    import asyncio
    asyncio.run(MySourceCollector().run())
```

3. Add to the beat schedule in `core/celery_app.py`.
4. Add `MY_SOURCE` to the `SignalSource` enum in `api/models/signal.py`.
5. Add any required API keys to `.env.example` and `core/config.py`.

---

## 12. Adding a New API Route

1. Create `backend/api/routers/<feature>.py`:

```python
from fastapi import APIRouter, Depends
from api.dependencies import current_user, get_db
from api.models.<feature> import MyResponse

router = APIRouter(tags=["feature"])

@router.get("/", response_model=MyResponse)
async def list_items(user=Depends(current_user), db=Depends(get_db)):
    ...
```

2. Register in `backend/main.py`:

```python
from api.routers import my_feature
app.include_router(my_feature.router, prefix="/my-feature")
```

3. Add corresponding Pydantic models in `api/models/<feature>.py`.
4. Add the iOS `Endpoint` case in `ios/BrighterDay/Core/Networking/Endpoints.swift`.
5. Add a corresponding ViewModel + View in the appropriate feature folder.

---

## 13. Testing Strategy

### Backend

- **Unit tests**: `pytest` in `backend/tests/unit/`
  - Ranker algorithms (velocity, novelty, genius scorer)
  - Collector normalisation functions
  - Synthesis prompt builders

- **Integration tests**: `pytest` with `httpx.AsyncClient` against a test FastAPI app
  - All API endpoints, covering auth, pagination, and filter params
  - Celery tasks run synchronously via `task.apply()`

- **Fixtures**: Use `pytest-asyncio` + `asyncio_mode = "auto"`.

Run tests:

```bash
cd backend
pytest tests/ -v --cov=. --cov-report=term-missing
```

### iOS

- **Unit tests**: `XCTest` for ViewModels using mock APIClient
- **UI tests**: `XCUITest` for critical flows (sign in, view feed, play brief)
- Use `@MainActor` in test setup to match production concurrency

---

## 14. Deployment

The backend is deployed to **Railway**:

```
pheme-api       — FastAPI (uvicorn, 2 workers)
pheme-worker    — Celery worker (concurrency=4)
pheme-beat      — Celery beat scheduler (single instance)
pheme-redis     — Redis 7 (managed)
```

Environment variables are set in Railway's dashboard. Supabase is managed separately (Supabase cloud).

**Deploy:**

```bash
# Railway deploys automatically on push to main.
# For manual deploy:
railway up
```

**Database migrations:**

```bash
supabase db push
```

---

## 15. Key Design Decisions

### Why Supabase?

- **Realtime** gives us free WebSocket push to iOS without building our own pubsub.
- **pgvector** means semantic search lives in the same DB as everything else — no separate vector DB.
- **Row Level Security** handles multi-user data isolation without application-level auth logic.
- Managed Postgres means no ops burden for a small team.

### Why Celery over cron?

- Collectors have different schedules, retry logic, and rate limits — Celery handles all of this.
- Beat scheduler provides a single source of truth for the job schedule.
- Worker concurrency can be scaled horizontally on Railway without code changes.

### Why Claude for synthesis (not GPT-4)?

- Claude handles long-context prompts better (we send 50 signals at once).
- Claude's structured output is more reliable for JSON-heavy prompts.
- The project is named `pheme` after the Greek goddess of fame/rumour — Anthropic felt thematically right.

### Why not stream the feed via polling?

- Supabase Realtime is used for the Signal Feed — the iOS app opens a WebSocket on launch.
- This avoids battery-draining background polling and gives true real-time updates.
- Free tier supports up to 200 concurrent connections, which is more than enough for early traction.

### Genius Score weight rationale

- `paper_score` (35%) is the highest weight because citation trajectory is the strongest predictor of long-term research impact.
- `social_score` (25%) captures the network effect — who is talking about the work matters.
- `code_score` (25%) rewards builders who ship, not just publish.
- `network_score` (15%) is useful signal but can be gamed (famous advisor ≠ good researcher).

---

*Last updated: 2026-04-29*
