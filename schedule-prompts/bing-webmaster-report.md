# Bing Webmaster Weekly Report — Setup Prompt

## What to say to Claude

> "I want a weekly Bing Webmaster Tools report for my site."

Or variations like:
> "Set up a Bing search performance digest with backlinks."
> "Build me a BWT report including AI/Copilot citations."

---

## Prompt for Claude (copy this in full)

I want a recurring automated Bing Webmaster Tools (BWT) report for my site — the Bing equivalent of a Search Console digest, plus the things Google doesn't give me: inbound links and AI/Copilot citation data. Here's how it should work at a high level:

1. On a schedule I choose, you'll pull my BWT data via the official API.
2. You'll compare the latest period against the prior equivalent period (week-on-week).
3. You'll produce a digest covering: a scorecard, daily trend, top queries and pages with deltas, movers, opportunities, a **Links** section (top linked pages, anchor text, referring domains), crawl stats/issues, sitemaps, and URL-submission quota.
4. You'll also include a **Backlinks** and an **AI Performance** (Copilot citations) section — even though those aren't in the API yet — by scraping them from the BWT portal and folding them into the same digest.
5. You'll save a timestamped report so I can track changes over time.

Before you build anything, ask me the following questions:

**About my site:**
- What's the exact verified site URL in BWT? (BWT has no `sc-domain:` abstraction — it needs the exact URL, e.g. `https://example.com`.)
- Is your site verified in BWT yet? (Fastest path: "Import from Google Search Console".)

**About the schedule and output:**
- How often, and what day/time? (weekly is typical)
- Markdown, emailed, or both? Summary in chat?

Once I have your answers, I'll set up the API key auth, write the report script, set up the portal-scrape step for backlinks + AI performance, and schedule it.

---

## Notes for Claude when setting this up

- **Auth = a single BWT API key** (BWT → Settings → API access → API Key). No expiry, no refresh — same unattended rationale as a service account. Store it locally (file or env) and gitignore it.
- Pass the **exact verified URL** as the site (no `sc-domain:` form exists in BWT).
- Run **on the user's machine** (the endpoint host and the portal scrape both behave best there, and it matches a multi-agent fleet that already runs locally).

### API quirks to handle
- Responses are wrapped in `{"d": ...}` — unwrap before parsing.
- Dates come back as ASP.NET `/Date(ms-0700)/` — write a small parser.
- Query/Page stats are bucketed **weekly (Saturday buckets)**, so week-on-week comes straight from the two latest buckets (no second call needed). Rank & Traffic is daily — use it for the scorecard/trend.
- Links methods (`GetLinkCounts`, `GetUrlLinks`) are paginated via a 0-based `page` index plus a `TotalPages` field.

### ⚠️ Backlinks and AI Performance are NOT in the API — scrape them
- The legacy link methods (`GetLinkCounts` / `GetUrlLinks`) return the **old inbound-link index**, which is often empty/stale and does **not** surface the newer **Backlinks** report.
- **AI Performance** (Copilot citations) has **no API at all** yet (Microsoft has said API access is planned later, but it isn't available now).
- So pull both **via a Chrome scrape** of the BWT portal and write them to a JSON file (e.g. `scraped/scrape_latest.json`); the report script reads that file and folds both into the one weekly digest, with a staleness warning if the scrape is more than ~10 days old.
- Scrape targets: `…/backlinks?siteUrl=…` (List By → Pages for source page + anchor + target; Domains tab for per-domain counts) and `…/aiperformance?siteUrl=…` (List By → Grounding Queries and Pages tabs, plus the daily Total Citations / Avg Cited Pages series).
- The API path (needs the machine) and the scrape path (needs Chrome) are independent but converge through that JSON file. **Always close the scrape tab when done.**

---

## ⚠️ Approaches that do NOT work — do not attempt these

### Expecting the API to return modern backlinks
`GetLinkCounts` / `GetUrlLinks` return the deprecated link index, not the current **Backlinks** report. Don't present their output as "the backlinks" — scrape the portal instead.

### Looking for an AI Performance / Copilot citations API
There is none yet. Don't burn time searching the API reference for it — scrape the portal.

### OAuth for BWT
Unnecessary. BWT uses a simple API key, which is the right unattended choice. Don't add an OAuth flow.

### What actually works
A single BWT API key for the core report (scorecard, queries/pages WoW, movers, crawl, sitemaps, quota), plus a Chrome portal scrape for Backlinks + AI Performance written to a shared JSON file, both folded into one weekly digest on the user's machine.
