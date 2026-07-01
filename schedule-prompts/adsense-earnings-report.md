# Google AdSense Weekly Report — Setup Prompt

## What to say to Claude

> "I want a weekly AdSense earnings report for my site."

Or variations like:
> "Set up an automated AdSense digest with week-on-week changes."
> "Extract my AdSense home-page numbers on a schedule."

---

## Prompt for Claude (copy this in full)

I want a recurring automated AdSense report for my site, so I get my key earnings numbers without logging into the AdSense dashboard. Here's how it should work at a high level:

1. On a schedule I choose, you'll pull my AdSense data via the official **AdSense Management API v2**.
2. You'll produce an **earnings snapshot** mirroring the AdSense home page (today, yesterday, this month, last month, last 7 days, last 30 days), plus a **week-on-week scorecard** (earnings, page views, impressions, clicks, ad requests, RPM, CTR, cost per click).
3. You'll add breakdowns: a daily trend, top countries (with deltas), top sites/domains, top ad units, and platforms.
4. You'll save a timestamped report so I can track changes over time.

Before you build anything, ask me the following questions:

**About my account:**
- What's your AdSense publisher ID? (the `pub-XXXXXXXXXXXXXXXX` from your AdSense URL)
- What currency should the report use? (the API converts on the fly)

**About the schedule and output:**
- How often, and what day/time? (weekly is typical)
- Markdown, emailed, or both? Summary in chat?

Once I have your answers, I'll walk you through the one-time OAuth setup, write the report script, and schedule it.

---

## Notes for Claude when setting this up

### ⚠️ AdSense does NOT support service accounts — this agent must use OAuth

Unlike Search Console and GA4 (which accept service accounts as users), **AdSense has no way to add a service account** — the Management API only accepts **OAuth 2.0**. So this is the one reporting agent that needs a one-time interactive consent:

1. In a Google Cloud project, enable the **"AdSense Management API"**.
2. Configure the **OAuth consent screen** (External is fine) and add the user's Google account as a **Test user**. (Test mode is fine; a desktop-app refresh token for a test user keeps working — no need to publish/verify.)
3. Create an **OAuth client ID → Application type: Desktop app**, download it as `client_secret.json`.
4. Run a one-time `authorize.py` that does the installed-app flow (scope `https://www.googleapis.com/auth/adsense.readonly`, `access_type=offline`, `prompt=consent`) and writes a **refresh token** to `token.json` (gitignored).
5. After that, the scheduled run loads `token.json` and **refreshes access tokens silently forever** — fully unattended, same end state as the rest of the fleet.

> The Google account that consents must be the one that owns the AdSense account. If consent fails with **"Access blocked … Error 403: access_denied / app is being tested"**, the account isn't on the **Test users** list yet — add it on the OAuth consent screen (Audience → Test users) and retry. An "unverified app" warning afterward is normal for a private single-user app (Advanced → continue).

### API quirks to handle
- `accounts.reports.generate` uses dotted query params (`startDate.year`, etc.). In the Python client these become **underscore kwargs** (`startDate_year`, `startDate_month`, `startDate_day`, and the same for `endDate`).
- **CTR** comes back as a **0–1 fraction** — render it as a percentage.
- **Ratio metrics** (RPM, CTR, CPC) must be read from the API's **`totals`** row, not summed across rows.
- Set `reportingTimeZone=ACCOUNT_TIME_ZONE` so numbers match the UI.
- Preset date ranges (`TODAY`, `YESTERDAY`, `MONTH_TO_DATE`, `LAST_7_DAYS`, `LAST_30_DAYS`) cover the home cards; there's no "last month" preset, so compute that as a custom range.
- **End the week-on-week window yesterday:** earnings up to yesterday are final; today's are still estimated (AdSense recomputes for spam / exchange rates).

### ⚠️ Where this must run (important)

The host is `adsense.googleapis.com` — a `*.googleapis.com` domain, **blocked inside the Cowork/sandbox network**. The report **must run on the user's own machine** (macOS `launchd`, Linux `cron`). Because the API returns the same numbers as the home page, **no screen-scraping is needed**.

---

## ⚠️ Approaches that do NOT work — do not attempt these

### Service-account auth
AdSense does not support service accounts at all — you cannot add one as an AdSense user. Don't try to reuse the GSC/GA4 service account here; it will never authorize. OAuth is mandatory.

### Running it inside Cowork
`googleapis.com` is blocked in the sandbox. Move execution to the user's machine.

### Scraping the AdSense home page
Tempting, but unnecessary and fragile — the Management API returns the same figures (and more). Only fall back to a logged-in Chrome scrape if the user explicitly refuses the one-time OAuth setup.

### Summing RPM/CTR/CPC across rows
These are ratios; summing them is wrong. Read them from the API `totals` row.

### What actually works
A one-time OAuth consent (Desktop app, `adsense.readonly`) → a long-lived refresh token → silent refresh on every scheduled run, executed **on the user's machine**, producing an earnings snapshot + week-on-week digest (window ending yesterday) from the AdSense Management API v2.
