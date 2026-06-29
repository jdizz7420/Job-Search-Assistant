# Job Search Assistant

A production Claude Code system that automates my job search — built to run daily, not as a demo.

Three skills, one shared state file, real MCP integrations. Every job in my tracker went through this pipeline.

---

## What it does

### `/linkedin-job-alert-digest`
Runs each morning against my Gmail inbox. Pulls LinkedIn job alert emails from the last 24 hours, parses each listing, deduplicates by Job ID against a running log, applies a fit-scoring rubric (title match, seniority, salary, company recognition), and auto-declines roles that fail hard filters (salary below floor, engineering titles, non-remote outside Phoenix). Outputs a ranked digest and appends new jobs to the tracker.

### `/tailor-resume`
For each job marked Interested without an existing resume: fetches the full JD from LinkedIn (saved to disk so it's never fetched twice), analyzes it against a base resume to categorize every requirement as covered, partially covered, or missing, surfaces gaps as questions before building anything, then writes a tailored `.docx` resume and cover letter. The gap-check step is load-bearing — it exists specifically to prevent the model from inventing experience that isn't there.

### `/application-status-check`
Reads the active pipeline, searches Gmail for emails from each company in the last 24 hours, classifies them (rejection, interview invite, recruiter screen, offer, assessment), and updates the tracker. Flags ambiguous emails for manual review rather than guessing.

---

## Architecture

```
Gmail MCP ──────────────────────────────────────────────┐
Chrome MCP (LinkedIn scraping) ──────────────────────── Claude Code
Google Drive MCP ───────────────────────────────────────┘
                                        │
                              ┌─────────▼──────────┐
                              │  new-role-postings  │  ← dedup log + interest tracker
                              │  application-tracker│  ← pipeline state
                              │  job-descriptions/  │  ← cached JDs
                              │  Resume Builder/    │  ← .docx outputs
                              └────────────────────┘
```

The tracker file is the shared state machine. Each skill reads from it, writes to it, and leaves a dated audit trail. Skills are designed to be re-runnable without creating duplicates.

---

## Built with

- [Claude Code](https://claude.ai/code) — skill authoring and execution
- Claude.ai MCP connectors — Gmail, Chrome, Google Drive
- `docx` (npm) — programmatic resume generation
- Node.js — resume build scripts

---

## Why I built it

I was laid off in June 2026 as part of a Robinhood restructuring. This is the first thing I built after — partly to run the search efficiently, partly because building real systems is how I stay sharp.

The resume tailor alone has processed 15+ applications. The digest runs every morning. The status checker caught two rejections and one interview invite before I checked my email.
