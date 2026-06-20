# schedule-prompts

Each file here is a self-contained setup prompt for a recurring scheduled task. The idea: instead of re-explaining a workflow every time, drop a prompt in here once and reuse it to spin up the schedule on demand.

## Convention for each prompt file

- A short description of what the schedule does and when you'd trigger it.
- A **copy-paste prompt block** to hand to Claude.
- The configuration questions Claude should ask before building.
- Notes / gotchas — approaches that work and ones that don't.

## Prompts

- [`keyword-gap-analysis.md`](keyword-gap-analysis.md) — weekly keyword gap analysis: trend sweep + content coverage report for a content site.
