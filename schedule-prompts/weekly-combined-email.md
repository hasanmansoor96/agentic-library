# Weekly Combined Email — Setup Prompt

## What to say to Claude

> "Stitch all my agent reports into one weekly email."

Or variations like:
> "Send me a single weekly digest combining all my reporting agents."
> "Email me the latest report from each agent every Monday."

---

## Prompt for Claude (copy this in full)

I have several reporting agents that each write a markdown digest to their own folder. I want one combined email per week that pulls the latest digest from each and sends them to me as a single, nicely formatted message. Here's how it should work at a high level:

1. On a schedule I choose (after the individual agents have run), you'll find the **newest** markdown digest in each agent's reports folder.
2. You'll stitch them into one HTML email with a table of contents and a section per agent.
3. Each section gets a **freshness badge**; digests older than a threshold are flagged "stale", and any agent that produced nothing is **called out, not silently dropped**.
4. You'll send it via email, and optionally archive the built HTML locally.

Before you build anything, ask me the following questions:

**About my agents:**
- Which agents/reports should be included, and where does each write its markdown? (e.g. `gsc-agent/reports/gsc_*.md`, `ga4-agent/reports/ga4_*.md`, …)
- What order do you want the sections in?

**About delivery:**
- What email service should I use? (e.g. Resend, SendGrid, SMTP) Do you already have an API key / verified sender domain?
- What's the recipient address? What from-address/sender name?

**About the schedule:**
- What day/time? (it must run **after** the individual agents finish)

Once I have your answers, I'll write the gather-and-send script (newest-file glob per agent, markdown→HTML, freshness badges, ToC) and schedule it.

---

## Notes for Claude when setting this up

- **Discover reports by glob, newest match wins.** Keep a simple list of `(section title, emoji, [glob patterns])` so adding a new agent later is a one-line change. Try multiple patterns per section if a source has fallback filenames.
- **Markdown → HTML:** use `python-markdown` if available, else a small built-in fallback so the dependency stays optional.
- **Freshness:** badge each section; flag digests older than ~8 days as "stale" and explicitly list any missing sections (so a silently-broken agent is visible).
- **Sending:** use the user's email provider. For Resend, the **from-domain must be verified** in the provider; reuse an existing API key (e.g. from the site's `.env.local`) rather than minting a new one. Allow recipient/sender overrides via env.
- Offer a **`--dry-run`** that builds and archives the HTML locally without sending — useful for testing.

### ⚠️ Where this must run (important)

This agent reads the **other agents' locally-generated report files**, so it must run **on the same machine** where those agents write their reports (macOS `launchd`, Linux `cron`). Schedule it **after** all the individual agents (e.g. if they run at 09:10–09:30, send at 09:40). The email provider's API (e.g. `api.resend.com`) is reachable from the machine; it is **not** a `googleapis.com` host, so it isn't subject to that sandbox block — but it still needs the local report files, which only exist on the machine.

---

## ⚠️ Approaches that do NOT work — do not attempt these

### Running it inside Cowork
The combined email depends on the individual agents' `reports/` files, which are generated on the user's machine. Cowork can't see them. Run it on the machine, after the agents.

### Scheduling it before/with the agents
If it runs before the agents finish, it emails stale or missing sections. Always schedule it **after** the last contributing agent.

### Silently dropping missing agents
If an agent didn't run, the email should **say so** rather than omit the section — a missing section that looks like "no news" hides a broken agent.

### Hardcoding the agent list inline
Keep the sources in one small, glob-based config list so new agents are trivial to add. Don't scatter per-agent logic through the script.

### What actually works
A glob-based "newest digest per agent" gather step, markdown→HTML with freshness badges and a table of contents, sent via the user's email provider (verified from-domain) **on the machine**, scheduled **after** the contributing agents.
