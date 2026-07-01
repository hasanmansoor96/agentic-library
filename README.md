# schedule-prompts

Each file here is a self-contained setup prompt for a recurring scheduled task. The idea: instead of re-explaining a workflow every time, drop a prompt in here once and reuse it to spin up the schedule on demand.

## Convention for each prompt file

- A short description of what the schedule does and when you'd trigger it.
- A **copy-paste prompt block** to hand to Claude.
- The configuration questions Claude should ask before building.
- Notes / gotchas — approaches that work and ones that don't.

## Prompts

- [`keyword-gap-analysis.md`](keyword-gap-analysis.md) — weekly keyword gap analysis: Google Trends sweep + content coverage report for a content site.
- (Work in Progress) [`reddit-rumor-digest.md`](reddit-rumor-digest.md) — daily community signal digest from public subreddit feeds (must run on your own machine; private signal only).
- [`search-console-report.md`](search-console-report.md) — weekly Google Search Console digest (clicks/impressions/CTR/position, queries, pages, movers) via a service account.
- [`bing-webmaster-report.md`](bing-webmaster-report.md) — weekly Bing Webmaster digest via API, plus Backlinks + AI/Copilot citations via a Chrome portal scrape.
- [`ga4-traffic-report.md`](ga4-traffic-report.md) — weekly Google Analytics 4 traffic/engagement digest via a service account (optional country-exclusion filter).
- [`core-web-vitals-report.md`](core-web-vitals-report.md) — weekly real-user Core Web Vitals (LCP/INP/CLS) from the CrUX History API, phone + desktop.
- [`adsense-earnings-report.md`](adsense-earnings-report.md) — weekly AdSense earnings snapshot + week-on-week scorecard via the Management API (OAuth — no service-account support).
- [`weekly-combined-email.md`](weekly-combined-email.md) — one weekly email stitching the latest digest from every reporting agent (Resend/SMTP).

## Cross-cutting lesson

Most of the Google-based agents (Search Console, GA4, CrUX, AdSense) and the Reddit agent **cannot run inside the Cowork sandbox** — its network blocks `*.googleapis.com` and Reddit hosts. They run on your own machine (macOS `launchd` / Linux `cron`) instead. Each prompt notes this where it applies.

---

## Prerequisites

### 1. Install Python dependencies

From the root of this repo, run:

```bash
pip install -r requirements.txt
```

Or manually:

```bash
pip install \
  google-api-python-client \
  google-auth \
  google-auth-httplib2 \
  google-auth-oauthlib \
  google-analytics-data \
  requests \
  markdown
```

| Package | Used by |
|---|---|
| `google-api-python-client` | Search Console, AdSense |
| `google-auth` + `google-auth-httplib2` | Search Console, GA4, AdSense |
| `google-auth-oauthlib` | AdSense only (OAuth — the one agent that can't use a service account) |
| `google-analytics-data` | GA4 |
| `requests` | Bing Webmaster API, Reddit feeds, general HTTP |
| `markdown` | Weekly combined email (optional — has a stdlib fallback) |

> **CrUX / Core Web Vitals** uses stdlib only — no extra packages needed.

### 2. Claude in Chrome (two agents)

The **keyword gap analysis** agent scrapes Google Trends via a live browser session — it's the only reliable method (`pytrends` and direct API calls are blocked by Google). The **Bing Webmaster** agent scrapes the BWT portal for Backlinks and AI/Copilot Performance data, which have no API yet. Both require the [Claude in Chrome extension](https://code.claude.com/docs/en/chrome) connected and signed into the relevant Google / Microsoft account at runtime.

### 3. Credentials

| Agent | What you need |
|---|---|
| Search Console | Google Cloud service account JSON key; service account added as a user on the GSC property; **"Search Console API"** enabled in the Cloud project |
| GA4 | Same service account as GSC (reuse it); **"Google Analytics Data API"** enabled; service account added as a Viewer on the GA4 property |
| AdSense | OAuth client (`client_secret.json`, Desktop app type); **"AdSense Management API"** enabled; one-time browser consent to generate a `token.json` refresh token |
| CrUX / Core Web Vitals | Google API key with **"Chrome UX Report API"** enabled |
| Bing Webmaster | BWT API key (BWT → Settings → API access) |
| Reddit Rumor Digest | None — uses public unauthenticated `.json` feeds |
| Weekly Combined Email | Resend / SendGrid / SMTP account with a verified sender domain |
