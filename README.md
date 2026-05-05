# sdr-daily-snapshot

Auto-refreshed daily snapshot of ecoplanet SDR performance metrics for the **Daily SDR Performance Briefing** Claude Code routine.

## Why this repo exists

The Claude Code routine that posts the daily SDR brief to Slack runs on Anthropic Cloud and cannot reach `qualification-dashboard-nine.vercel.app` directly — Vercel's edge WAF returns HTTP 403 for Anthropic's IP range. GitHub raw-content URLs are reachable from both Anthropic Cloud and the GitHub Actions runner that updates this file.

## Architecture

```
   GitHub Actions cron (Mo–Fr 07:45 Berlin)
       │
       │ curl /api/sdr-daily-summary  (Bearer key)
       ▼
   qualification-dashboard (Vercel)
       │
       │ aggregates AirCall + HubSpot
       ▼
   data/snapshot.json  ◀──────  this repo
       │
       │ raw.githubusercontent.com
       ▼
   Claude Code routine (Mo–Fr 08:00 Berlin)
       │
       │ format → Slack DM
       ▼
   Sebastian's Slack
```

## What's in `data/snapshot.json`

Aggregated counts only — Calls / AP Reached / Discos Booked per SDR for the previous working day plus week-to-date, plus held/accepted counts per AE. **No PII, no individual deal details, no transcripts.**

Source-of-truth for the underlying logic: Till's `analytics-bi.md` Section 3 (private).

## Operations

- **Workflow:** `.github/workflows/snapshot.yml` — cron `45 5 * * 1-5` UTC, also `workflow_dispatch` for manual runs
- **Secret:** `DAILY_SUMMARY_API_KEY` (GitHub repo secret) — Bearer token for the Vercel endpoint
- **Manual refresh:** Actions tab → "Daily SDR Snapshot" → Run workflow

If the workflow fails, the routine reads the *previous* snapshot — Sebastian gets stale data with a `_⚠️ Daten-Issues_` footer instead of nothing.
