# Google Search Console Weekly Report — Setup Prompt

## What to say to Claude

> "I want a weekly Search Console report for my site."

Or variations like:
> "Set up an automated GSC report with week-on-week changes."
> "Build me a search performance digest for my domain."

---

## Prompt for Claude (copy this in full)

I want a recurring automated Search Console report for my site, so I can see how my organic search performance is trending without logging into the GSC UI every week. Here's how it should work at a high level:

1. On a schedule I choose, you'll pull my Search Console data via the official API.
2. You'll compare the most recent period against the prior equivalent period (week-on-week by default).
3. You'll produce a digest covering: a top-line scorecard (clicks, impressions, CTR, average position), a daily trend, top queries and top pages with deltas, biggest movers up and down, "opportunity" queries (high impressions but low CTR or position), countries, devices, search appearance, and sitemap status.
4. You'll save a timestamped report so I can track changes over time.

Before you build anything, ask me the following questions:

**About my property:**
- What's the Search Console property? Is it a **Domain property** (`sc-domain:example.com`) or a **URL-prefix** property (`https://example.com/`)?
- Do you already have access set up, or do we need to grant it?

**About the schedule:**
- How often? (weekly is typical)
- What day/time? (or pick a sensible default)

**About the output:**
- Markdown file, emailed, or both?
- A summary in chat each run?

Once I have your answers, I'll set up authentication, write the report script (scorecard + trend + queries/pages with deltas + movers + opportunities + countries + devices + search appearance + sitemaps), and schedule it.

---

## Notes for Claude when setting this up

- **Auth = a Google Cloud service account**, added directly as a user on the GSC property (Settings → Users and permissions → Add user → the service account's email, role "Full" or "Restricted"). Scope: `https://www.googleapis.com/auth/webmasters.readonly`. Store the key JSON locally and gitignore it.
- **Why a service account, not OAuth:** no token refresh/expiry, fully unattended. Service accounts can be added directly as GSC users, so there's no consent flow. (Reuse this same service account for other Google agents like GA4.)
- For a Domain property, pass the property exactly as `sc-domain:example.com`. For a URL-prefix property, pass the exact verified URL.
- Compute deltas with a current-vs-prior join on the dimension (query/page), so each row shows its change.
- Save timestamped reports (JSON + markdown); never overwrite.

### ⚠️ Where this must run (important)

`*.googleapis.com` is **blocked inside the Cowork/sandbox network** (connection refused). The report **must run on the user's own machine** (macOS `launchd`, Linux `cron`), which can reach Google's APIs. Verify connectivity with a small test script before scheduling.

---

## ⚠️ Approaches that do NOT work — do not attempt these

### Running it inside Cowork
The sandbox can't reach `googleapis.com`. Don't try to work around the network block — move execution to the user's machine.

### OAuth user credentials for an unattended job
OAuth works but adds token-refresh fragility and a consent screen for no benefit here — GSC accepts service accounts directly as property users, which is strictly simpler for a scheduled task. Prefer the service account. (OAuth is only unavoidable for products that don't support service accounts, e.g. AdSense.)

### Bulk index-coverage / "Pages" report via API
There is **no API** for the full index-coverage (Pages) report. Getting per-URL index status requires the URL Inspection API in a per-URL loop, which is slow and quota-limited. Skip it for the standard digest (or sample a handful of key URLs) rather than trying to reproduce the full Pages report.

### What actually works
A service account with `webmasters.readonly`, added as a user on the property, running **on the user's machine** on a weekly schedule, producing a week-on-week digest from the Search Analytics API.
