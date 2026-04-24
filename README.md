# metalsmith-prices

Static gold/silver spot prices for the MetalSmith mobile app, refreshed 3×/day
by a GitHub Action.

## What this is

- `prices.json` — the single source of truth the app reads. Locked shape:
  ```json
  { "gold": 3287.40, "silver": 33.12, "lastUpdated": "2026-04-24T14:00:00Z", "source": "stooq" }
  ```
  `gold` and `silver` are **USD per troy ounce**.
- `.github/workflows/fetch-prices.yml` — cron at 06/14/22 UTC. Fetches from
  `stooq.com`, runs plausibility bounds checks, writes + commits `prices.json`,
  pings a healthchecks.io dead-man's switch.

## Why this exists

The app caches prices 30 min per device. Rather than have every user call a
metered API, one GitHub Action writes a static JSON, GitHub's CDN serves it
free, and every user reads the same file. Cost: $0. Secrets in git: none.

## Failure alerting

Three layers, all email lilpillstudios@gmail.com:

1. **GitHub's default workflow-failure email** — catches hard failures (network,
   stooq down, bounds check rejection).
2. **Sanity bounds in the workflow** — gold $500–$20,000, silver $5–$500. Any
   out-of-range value fails the run loud enough that the email is unmissable.
3. **healthchecks.io dead-man's switch** — alerts if the workflow silently
   stops running (e.g., GH auto-disables scheduled workflows after 60 days of
   repo inactivity; our commits-on-change pattern should prevent this but
   belt-and-suspenders is cheap).

### Enabling healthchecks.io (optional, 5 min)

1. Sign up at https://healthchecks.io (free tier is fine forever).
2. Create a check: schedule "every 8 hours", grace 2 hours.
3. Copy the ping URL (e.g. `https://hc-ping.com/abc123...`).
4. In this repo's GitHub settings: Secrets and variables → Actions → New
   repository secret → name `HEALTHCHECKS_URL`, paste the URL.
5. The workflow picks it up automatically next run. If the secret is absent,
   the ping steps silently skip — everything still works.

## Pivot path (when stooq eventually dies)

See `../MetalSmithApp/scripts/fetch-prices.local.mjs.template` for the
keyed-API fallback (goldapi.io, run locally via Windows Task Scheduler).
Same `prices.json` shape, zero app changes required.
