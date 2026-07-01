# Core Web Vitals Weekly Report — Setup Prompt

## What to say to Claude

> "I want a weekly Core Web Vitals report for my site."

Or variations like:
> "Track my real-user LCP / INP / CLS over time."
> "Set up a CrUX field-data digest for my domain."

---

## Prompt for Claude (copy this in full)

I want a recurring automated Core Web Vitals report for my site using **real-user (field) data**, so I can see how my actual visitors experience page speed and whether it's improving or regressing. Here's how it should work at a high level:

1. On a schedule I choose, you'll pull real-user Core Web Vitals from the free **Chrome UX Report (CrUX) History API** — the same dataset behind Search Console's Core Web Vitals report.
2. You'll report the origin overall plus a few key URLs I specify.
3. You'll produce a digest with, for **phone and desktop** separately: a PASS/FAIL scorecard (pass when LCP, INP, and CLS are all "good"), each metric's p75 value, rating, week-on-week change, "good" percentage, and a trend sparkline.
4. You'll save a timestamped report so I can track changes over time.

Before you build anything, ask me the following questions:

**About my site:**
- What's the origin? (e.g. `https://example.com`)
- Any specific key URLs you want broken out individually? (e.g. homepage, a top landing page)

**About the schedule and output:**
- How often, and what day/time? (weekly is typical; CrUX updates on a rolling basis)
- Markdown, emailed, or both? Summary in chat?

Once I have your answers, I'll set up the API key, write the report script (origin + key-URL scorecards, per-metric p75/rating/WoW/good%/trend, phone + desktop), and schedule it.

---

## Notes for Claude when setting this up

- **Auth = a single Google API key** with the **"Chrome UX Report API"** enabled in a Google Cloud project. No expiry/refresh — same unattended rationale as other key-based agents. Store it locally and gitignore it.
- Core metrics: **LCP, INP, CLS**. Useful diagnostics: **FCP, TTFB**. Each value is CrUX's standard **28-day rolling p75**.
- **Use `queryHistoryRecord`, not `queryRecord`:** the history endpoint returns ~25 weekly collection periods in one call, so you get a trend AND a week-on-week delta from the two latest periods. `queryRecord` only gives a single current snapshot — not enough for a trend.
- Report **phone and desktop** separately (`formFactor`).
- Pages with too little traffic return **"insufficient field data"** — that's expected, not an error. Handle it and say so in the digest. Pair with lab data (e.g. Lighthouse) if the user wants lab + field.
- Stdlib-only is fine (plain HTTPS calls); no heavy SDK needed.

### ⚠️ Where this must run (important)

The CrUX host is `chromeuxreport.googleapis.com` — a `*.googleapis.com` domain, which is **blocked inside the Cowork/sandbox network**. The report **must run on the user's own machine** (macOS `launchd`, Linux `cron`). Verify with a small test call before scheduling.

---

## ⚠️ Approaches that do NOT work — do not attempt these

### Running it inside Cowork
`googleapis.com` is blocked in the sandbox. Move execution to the user's machine.

### Using `queryRecord` for a trend
The single-snapshot endpoint can't give week-on-week movement. Use `queryHistoryRecord`, which returns the full weekly series in one request.

### Treating "insufficient field data" as a failure
Low-traffic URLs legitimately have no field data — that's CrUX's traffic threshold, not a bug. Don't retry or error out; report it as a known state.

### Lab tools as a substitute for field data
Lighthouse / PageSpeed lab scores are synthetic and don't reflect real users. They complement CrUX but don't replace it. This agent is specifically the **field-data** source.

### What actually works
A single API key with the Chrome UX Report API enabled, calling `queryHistoryRecord` for origin + key URLs across phone and desktop, **on the user's machine**, weekly.
