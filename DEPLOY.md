# ThumbnailFlow AI — Phase 5 Deployment Guide

## Architecture (production)

| Component | Recommended host |
|-----------|------------------|
| Next.js web | **Vercel** |
| PostgreSQL | **Supabase** (or Neon) |
| Redis | **Upstash Redis** (serverless-friendly) |
| Trend / Thumbnail engines | **Railway**, **Fly.io**, or Docker VPS |
| Background jobs | Vercel Cron + `/api/cron/process-queue` **or** dedicated worker container |

---

## 1. Supabase

1. Create a project at [supabase.com](https://supabase.com).
2. **Authentication → URL configuration**: add your production URL and `https://YOUR_DOMAIN/auth/callback`.
3. **Database → Connection string**: copy the **URI** (`postgresql://...`) into Vercel env as `DATABASE_URL` and `DIRECT_URL`.
4. **API keys**: use the **anon/publishable** key for `NEXT_PUBLIC_SUPABASE_ANON_KEY` (never the secret key in the browser).

Run schema once from your machine:

```bash
DATABASE_URL="your-uri" npm run db:push
```

---

## 2. Upstash Redis (queues)

1. Create a database at [upstash.com](https://upstash.com).
2. Copy the Redis URL → `REDIS_URL` in Vercel (e.g. `rediss://default:...@....upstash.io:6379`).

Cron jobs enqueue work; `/api/cron/process-queue` drains the queue every 10 minutes (configured in `vercel.json`).

For heavier load, run the dedicated worker:

```bash
npm run worker
```

Or `docker compose up -d worker` with the full stack.

---

## 3. Vercel deployment

1. Import the GitHub repo in Vercel.
2. **Root directory**: repository root (monorepo).
3. Framework preset: **Next.js** — settings are in `apps/web/vercel.json`.

### Required environment variables

| Variable | Notes |
|----------|--------|
| `NEXT_PUBLIC_SUPABASE_URL` | Supabase project URL |
| `NEXT_PUBLIC_SUPABASE_ANON_KEY` | Anon/publishable key |
| `DATABASE_URL` | `postgresql://` URI |
| `DIRECT_URL` | Same as DATABASE_URL for Prisma |
| `NEXT_PUBLIC_APP_URL` | `https://your-domain.vercel.app` |
| `CRON_SECRET` | Long random string (32+ chars) |
| `REDIS_URL` | Upstash Redis URL |
| `TREND_ENGINE_URL` | Public URL of trend-engine service |
| `THUMBNAIL_ENGINE_URL` | Public URL of thumbnail-engine service |
| `OPENAI_API_KEY` | Optional |
| `YOUTUBE_API_KEY` | Optional |
| `MAKE_WEBHOOK_URL` | Optional — real social posting |

4. Deploy. Vercel automatically registers crons from `vercel.json`.

### Cron authentication

Vercel Cron sends `Authorization: Bearer <CRON_SECRET>` when you add `CRON_SECRET` to the project. Verify in Vercel → Settings → Cron Jobs.

Manual test:

```http
GET https://YOUR_DOMAIN/api/cron/process-queue?limit=5
Authorization: Bearer YOUR_CRON_SECRET
```

---

## 4. Python microservices (Railway / Docker)

Deploy `services/trend-engine` and `services/thumbnail-engine` with public HTTPS URLs, then set:

```
TREND_ENGINE_URL=https://trend-engine-xxxx.up.railway.app
THUMBNAIL_ENGINE_URL=https://thumbnail-engine-xxxx.up.railway.app
```

Health checks:

- `GET /health` on each service

---

## 5. Full local production-like stack

```powershell
docker compose up -d
npm run db:push
# Terminal 1
npm run dev
# Terminal 2
npm run worker
```

Health: http://localhost:3000/api/health

---

## 6. GitHub Actions CI

Pushes to `main` run lint + build (see `.github/workflows/ci.yml`).

---

## Checklist before go-live

- [ ] `CRON_SECRET` is not the dev default
- [ ] `DATABASE_URL` starts with `postgresql://`
- [ ] Supabase redirect URLs include production domain
- [ ] Redis reachable from Vercel (`/api/health` shows `redis: ok`)
- [ ] Engines reachable or Node fallbacks acceptable
- [ ] `npm run build` passes locally
