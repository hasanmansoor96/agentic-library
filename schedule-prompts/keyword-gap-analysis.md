# Keyword Gap Analysis Schedule — Setup Prompt

## What to say to Claude

> "I want to set up a weekly keyword gap analysis schedule for my content site."

Or variations like:
> "Help me build a keyword analysis agent for my blog."
> "Set up an automated keyword gap report for my site."

---

## Prompt for Claude (copy this in full)

I want to set up a recurring automated keyword gap analysis for my content site, similar to how SEO teams track which trending search terms their published content is missing. Here's how it should work at a high level:

1. On a schedule I choose, you'll pull a rotating batch of seed keywords and fetch their related/trending queries from Google Trends.
2. You'll then pull my published content from wherever I store it (a CMS, a website, a blog, etc.) and check which of those trending terms are covered — or not covered — across my titles, meta descriptions, body text, and any FAQ sections.
3. You'll generate a keyword gap report showing me which terms are rising in my niche that my content isn't targeting yet.
4. You'll present a summary of the top gaps and any changes vs. the previous week.

Before you build anything, ask me the following questions so you can configure this for my specific setup:

**About my content topic:**
- What is the main topic or niche of my site? (e.g. gaming, travel, finance, fitness)
- What are 20–40 seed keywords that represent my niche? These are the starting terms you'll use to discover related trending queries on Google Trends. (I can give you a list, or you can suggest some based on my topic.)

**About where my content lives:**
- Where are my published posts/articles stored? Options include:
  - A Sanity CMS project (provide project ID, dataset name, and the content type/schema)
  - A WordPress site (provide the site URL; you'll use the WP REST API)
  - A plain website or blog (provide the URL; you'll scrape or use a sitemap)
  - A CSV or JSON export of my posts (I'll upload the file)
  - Another CMS or API (describe it)
- Which fields should be checked for keyword coverage? (e.g. title, meta description, body, FAQs, tags)

**About the schedule:**
- How often do you want this to run? (e.g. weekly, every Monday, twice a week)
- What time of day? (e.g. 9am, or just pick a sensible default)

**About the output:**
- Do you want the gap report saved as a markdown file, emailed to you, or both?
- Do you want a summary presented directly in chat each time it runs?

Once I have your answers, I'll:
1. Set up the rotation state and seed list for your topic
2. Configure the content-fetching logic for your specific source
3. Write the gap analysis script
4. Schedule it to run automatically and report back to you each time

---

## Notes for Claude when setting this up

- Adapt the content-fetching step to the user's source (Sanity GROQ query, WP REST API, sitemap scrape, file parse, etc.)
- Google Trends scraping requires Chrome to be connected and signed in; warn the user if it isn't available at runtime
- Space Trends requests 60–90 seconds apart — never burst, or Google will CAPTCHA-block the session
- Use a rotation_state.json to cycle through seeds across runs rather than hitting all seeds every time
- Save timestamped reports; never overwrite prior ones so the user can track changes week over week
- If Chrome isn't available during a run, fall back to the existing cached related_raw.json and note this in the summary

---

## ⚠️ Approaches that do NOT work — do not attempt these

These were all tried and failed before the browser-scrape method was settled on. Don't go down these paths again, even if they look promising.

### pytrends (Python library)
`pytrends` (`pip install pytrends`, `TrendReq`, `interest_over_time()`, `related_queries()`) is the most obvious solution but **does not work reliably**. Google actively blocks automated requests from it — you get 429s, empty responses, or responses with no data within a few calls. It worked years ago; it no longer does at any meaningful scale. Do not suggest it.

### Direct Google Trends API calls (requests / httpx / curl)
Google Trends has no official public API. Calls made directly to `trends.google.com` endpoints (e.g. `/trends/api/explore`, `/trends/api/widgetdata/multiline`) without a real browser session return 429s or malformed/empty JSON almost immediately. Adding headers, rotating user agents, or adding delays does not reliably fix this — Google checks for a valid browser fingerprint and cookie session.

### Other Python/npm scraping libraries targeting Trends
Libraries like `google-trends-api` (npm), `pyGTrends`, `serpapi`'s Trends module (unless you have a paid SerpAPI key), and similar packages all either wrap the same blocked endpoints or are unmaintained. None of them worked consistently enough to build a scheduled task on.

### The Google Keyword Planner / Search Console APIs
These provide search volume data but **not** the rising/related-query signal that makes this workflow useful. They also require OAuth credentials, a verified property, or ad account access — too much friction for a general setup.

### What actually works
The only reliable method is rendering the Google Trends explore page inside a **real, signed-in Chrome browser** and reading the data from the rendered shadow DOM. This requires:
- Chrome connected to Cowork/Claude
- The user signed into a Google account in that browser
- Requests spaced 60–90 seconds apart to avoid throttling
- A shadow-DOM scraper script (like `scrape_related.js` in this project) rather than any HTTP-level approach

If Chrome is unavailable, the run should fall back to cached data from the previous sweep rather than attempting any of the above alternatives.
