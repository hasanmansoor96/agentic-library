# Google Analytics 4 Weekly Report — Setup Prompt

## What to say to Claude

> "I want a weekly GA4 traffic report for my site."

Or variations like:
> "Set up an automated Google Analytics digest with week-on-week changes."
> "Build me a GA4 engagement report I don't have to log in for."

---

## Prompt for Claude (copy this in full)

I want a recurring automated Google Analytics 4 report for my site, so I can see traffic and engagement trends without logging into the GA4 UI. Here's how it should work at a high level:

1. On a schedule I choose, you'll pull my GA4 data via the official Data API.
2. You'll compare the most recent period against the prior equivalent period (week-on-week by default).
3. You'll produce a digest covering: a scorecard (sessions, users, new users, page views, engagement rate, average session duration, bounce rate, events, key events/conversions), a daily trend, channels, top pages, sources/mediums, landing pages, countries, devices, and top events — with deltas on the dimensioned tables.
4. You'll save a timestamped report so I can track changes over time.

Before you build anything, ask me the following questions:

**About my property:**
- What's the GA4 **numeric Property ID** (a 9-digit number from Admin → Property Settings — NOT the `G-XXXX` measurement ID)?
- Do you already have API access set up, or do we need to grant it?

**About filtering:**
- Do you want to **exclude any countries** from the whole report (e.g. known bot-traffic regions)? If so, which?

**About the schedule and output:**
- How often, and what day/time? (weekly is typical; GA4 is near-realtime)
- Markdown, emailed, or both? Summary in chat?

Once I have your answers, I'll set up authentication, write the report script (scorecard + trend + channels + pages + sources + landing + countries + devices + events, all week-on-week), and schedule it.

---

## Notes for Claude when setting this up

- **Auth = a Google Cloud service account**, added as a **Viewer** on the GA4 property (Admin → Property Access Management). Enable the **"Google Analytics Data API"** in the Cloud project. Scope: `https://www.googleapis.com/auth/analytics.readonly`. **Reuse the same service account** as the Search Console agent if there is one — no new credentials needed.
- **Why a service account, not OAuth:** service accounts can be added directly as GA4 property users, so there's no consent/refresh — fully unattended.
- A `403` on the test call almost always means the service account wasn't added on the property — re-check that grant.
- Get deltas on dimensioned tables (channels, pages, etc.) with a current-vs-prior join on the dimension.
- If excluding countries, apply the GA4 `dimensionFilter` (a `notExpression` + `inListFilter` on `country`) to **every** query including the scorecard, so all sections are consistent — and state the exclusion in the digest header so the filter is always visible.
- Save timestamped reports (JSON + markdown); never overwrite.

### ⚠️ Where this must run (important)

The GA4 Data API host is `analyticsdata.googleapis.com` — a `*.googleapis.com` domain, **blocked inside the Cowork/sandbox network**. The report **must run on the user's own machine** (macOS `launchd`, Linux `cron`). Verify with a small test call first.

---

## ⚠️ Approaches that do NOT work — do not attempt these

### Running it inside Cowork
`googleapis.com` is blocked in the sandbox. Move execution to the user's machine.

### Using the `G-XXXX` measurement ID as the property
The Data API needs the **numeric Property ID**, not the measurement ID. Passing `G-XXXX` will fail.

### OAuth user credentials for an unattended job
Unnecessary friction — GA4 accepts service accounts as property users directly. Use the service account (and reuse the GSC one).

### What actually works
A service account with `analytics.readonly`, added as a Viewer on the property, running **on the user's machine** weekly, producing a week-on-week digest from the GA4 Data API — with an optional country-exclusion filter applied uniformly across all sections.
