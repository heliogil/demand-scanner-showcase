# Demand Signal Scanner

Multi-source demand signal scanner that identifies freelance and B2B opportunities matching a configured portfolio of services.

## What it does

Continuously scrapes 8 job/demand sources, classifies signals with MiniMax M3, scores them against a service portfolio, and delivers high-score matches via Discord DM and the Nerve Center UI.

## Sources

| Source | Type | Volume |
|---|---|---|
| Hacker News (monthly) | Job posts "Who is hiring?" | Monthly thread |
| Hacker News (daily) | Show HN / Ask HN | Daily |
| RemoteOK | Remote job board | Continuous |
| Work With Remotely (WWR) | Remote job board | Continuous |
| Trampos.co | Brazilian job board | Continuous |
| Product Hunt | Launch signals | Daily |
| Algora | Open source bounties | Continuous |
| Reddit | Subreddit demand signals | Continuous |

## Architecture

```
Scheduler (cron)
    └─ Orchestrator
           ├─ Scraper × 8 (async, rate-limited)
           ├─ Classifier (MiniMax M3 — relevance + budget + urgency)
           ├─ DB writer (PostgreSQL)
           └─ Notifier (Discord webhook + Nerve Center queue)
```

## Scoring

Each signal is scored 1–10 across three dimensions:
- **Relevance** — match against service portfolio keywords
- **Budget signal** — explicit budget mentioned or inferred from context
- **Urgency** — deadline / hiring-now language detected

Thresholds: notify ≥ 6 | pitch draft ≥ 7 | urgent alert ≥ 8

## Stack

| Layer | Tech |
|---|---|
| Scrapers | httpx + BeautifulSoup (async) |
| Classification | MiniMax M3 (flat-rate) |
| Storage | PostgreSQL + Redis (dedup cache) |
| Notifications | Discord webhook |
| UI | Nerve Center integration |

## Key design decisions

- **No Scrapy** — lightweight httpx scrapers per source, simpler to maintain
- **Redis dedup** — 7-day TTL prevents re-alerting on the same signal
- **Portfolio-driven scoring** — `data/portfolio.json` defines services + keywords without touching code
- **Human-in-the-loop** — score ≥ 7 generates a pitch draft for human review, not auto-send

## Configuration

```bash
POSTGRES_HOST=localhost
POSTGRES_PORT=5432
POSTGRES_USER=<user>
POSTGRES_PASSWORD=<password>
POSTGRES_DB=<db>
MINIMAX_API_KEY=<key>
DISCORD_WEBHOOK_DEMAND=<webhook-url>
REDIS_URL=redis://localhost:6380/2
```

---
Part of the [Venture Studio](https://github.com/heliogil) AI infrastructure stack.
