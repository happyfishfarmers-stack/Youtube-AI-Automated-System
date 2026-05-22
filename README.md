# ThumbnailFlow AI

## Phase 1 ✓
- Supabase auth, onboarding, SaaS dashboard

## Phase 3 ✓
- **AI Thumbnail Generator** — 6 style presets, 3–10 variants, CTR ranking
- OpenAI for copy (+ optional DALL·E backgrounds); Sharp render fallback without API key
- **Dashboard → Thumbnail Generator**

## Phase 2 ✓
- **Trend engine** (Python/FastAPI) — Reddit, Google Trends, YouTube (optional API key)
- **Thumbnail engine** (Python/OpenCV) — CTR & viral scoring
- Live **Trends** and **Viral Analyzer** pages
- Cron endpoint for 6-hour trend scans

---

## Run everything (local)

### Terminal 1 — Database
```powershell
docker compose up -d postgres
npm run db:push
```

### Terminal 2 — Trend engine (port 8001)
```powershell
cd services/trend-engine
pip install -r requirements.txt
cd ../..
npm run dev:trends
```

### Terminal 3 — Thumbnail engine (port 8002)
```powershell
cd services/thumbnail-engine
pip install -r requirements.txt
cd ../..
npm run dev:thumbnails
```

### Terminal 4 — Web app (port 3000)
```powershell
npm run dev
```

Open **http://localhost:3000** → Dashboard → **Trends** → **Scan now**

---

## Environment (`apps/web/.env.local`)

| Variable | Purpose |
|----------|---------|
| `NEXT_PUBLIC_SUPABASE_URL` | Supabase project URL |
| `NEXT_PUBLIC_SUPABASE_ANON_KEY` | Publishable/anon key only |
| `DATABASE_URL` | PostgreSQL |
| `TREND_ENGINE_URL` | Default `http://localhost:8001` |
| `THUMBNAIL_ENGINE_URL` | Default `http://localhost:8002` |
| `YOUTUBE_API_KEY` | Optional — enables YouTube trending |
| `CRON_SECRET` | Protects `/api/cron/*` |
| `REDIS_URL` | Default `redis://localhost:6379` (Phase 5 queues) |

---

## Analyze a thumbnail

1. Go to **Viral Analyzer**
2. Paste a URL like: `https://i.ytimg.com/vi/VIDEO_ID/maxresdefault.jpg`
3. Click **Analyze CTR**

---

## Phase 5 — Job queue & worker

```powershell
docker compose up -d redis          # or full stack: docker compose up -d
npm run worker                      # process queued jobs (local)
```

Health check: **http://localhost:3000/api/health**

Production deploy: see **[DEPLOY.md](./DEPLOY.md)**

## Automated crons

| Endpoint | Schedule (Vercel) |
|----------|---------------------|
| `/api/cron/trends` | Every 6 hours |
| `/api/cron/social` | Daily 17:00 UTC |
| `/api/cron/process-queue` | Every 10 minutes |

```http
GET http://localhost:3000/api/cron/trends
Authorization: Bearer YOUR_CRON_SECRET
```

---

## Docker (all services)

```powershell
docker compose up -d
```

Includes postgres, redis, trend-engine, thumbnail-engine.

---

## Phase 4 ✓
- **Social automation** — daily posts, AI captions, platform rotation
- **Lead discovery** — YouTube API or Reddit, audit scores, outreach emails
- **Fiverr gig generator** — title, description, packages, FAQs

## Phase 5 ✓
- **Redis + BullMQ** job queue (`@thumbnailflow/queue`)
- **Background worker** — `npm run worker` or Docker `worker` service
- **Vercel crons** — trends (6h), social (daily), queue drain (10m)
- **`/api/health`** — database, Redis, engine status
- **GitHub Actions CI** — lint + build
- **DEPLOY.md** — Vercel + Supabase + Upstash production guide

## Phase 3 — Generator
1. **Dashboard → Thumbnail Generator**
2. Enter video title, pick style (MrBeast, Finance, etc.)
3. Click **Generate** — variants saved and ranked by CTR
4. Optional: set `OPENAI_API_KEY` in `.env.local` for GPT copy + DALL·E backgrounds
