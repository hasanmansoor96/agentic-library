# schedule-prompts

Each file here is a self-contained setup prompt for a recurring scheduled task. The idea: instead of re-explaining a workflow every time, drop a prompt in here once and reuse it to spin up the schedule on demand.

## Convention for each prompt file

- A short description of what the schedule does and when you'd trigger it.
- A **copy-paste prompt block** to hand to Claude.
- The configuration questions Claude should ask before building.
- Notes / gotchas — approaches that work and ones that don't.

## Prompts

- [`keyword-gap-analysis.md`](keyword-gap-analysis.md) — weekly keyword gap analysis: Google Trends sweep + content coverage report for a content site.
- [`reddit-rumor-digest.md`](reddit-rumor-digest.md) — daily community signal digest from public subreddit feeds (must run on your own machine; private signal only). (WIP)
- [`search-console-report.md`](search-console-report.md) — weekly Google Search Console digest (clicks/impressions/CTR/position, queries, pages, movers) via a service account.
- [`bing-webmaster-report.md`](bing-webmaster-report.md) — weekly Bing Webmaster digest via API, plus Backlinks + AI/Copilot citations via a Chrome portal scrape.
- [`ga4-traffic-report.md`](ga4-traffic-report.md) — weekly Google Analytics 4 traffic/engagement digest via a service account (optional country-exclusion filter).
- [`core-web-vitals-report.md`](core-web-vitals-report.md) — weekly real-user Core Web Vitals (LCP/INP/CLS) from the CrUX History API, phone + desktop.
- [`adsense-earnings-report.md`](adsense-earnings-report.md) — weekly AdSense earnings snapshot + week-on-week scorecard via the Management API (OAuth — no service-account support).
- [`weekly-combined-email.md`](weekly-combined-email.md) — one weekly email stitching the latest digest from every reporting agent (Resend/SMTP).

## Cross-cutting lesson

Most of the Google-based agents (Search Console, GA4, CrUX, AdSense) and the Reddit agent **cannot run inside the Cowork sandbox** — its network blocks `*.googleapis.com` and Reddit hosts. They run on your own machine (macOS `launchd` / Linux `cron`) instead. Each prompt notes this where it applies.
