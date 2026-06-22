---
name: application-status-check
description: Daily check of Gmail for status updates from companies in the application tracker, auto-updates the    Status column
---

Read "/Users/jdlawson/Documents/Claude/Projects/Application Tracker/application-tracker.md" (use Read). It has an "Active Pipeline" markdown table with columns # | Company | Role | Status | Fit | Files, followed by a "Status Key" legend and a "Company Notes" section with per-company detail.

Build a list of company names from the Active Pipeline table, excluding any row whose Status is "❌ Closed" (no need to keep checking dead leads) and excluding generic/ambiguous names that would produce false-positive search results (use judgment — e.g. very short or common words).

Search the user's Gmail using the Gmail connector (search_threads tool) for messages from or about each of those companies received in the last 24 hours. Use queries like: from:<company domain or name> newer_than:1d, or a broader "<company name>" newer_than:1d search if you don't know the domain. Also run a generic sweep: newer_than:1d with keywords like "interview", "application", "offer", "next steps", "unfortunately", "regret to inform", "assessment", "schedule a call" to catch recruiter emails from domains not obviously matching the company name (e.g. Greenhouse, Lever, Workday, a recruiter's personal-sounding email).

For each relevant email found, fetch the full thread (get_thread) and determine if it signals a pipeline status change for one of the tracked companies:
- Rejection / "unfortunately" / "moving forward with other candidates" / "not selected" → Status = "❌ Closed — Rejected <date>"
- Interview invite / scheduling a call / "next round" → Status = "🎯 Interviewing"
- Recruiter screen scheduled or completed → Status = "📞 Screening"
- Offer extended → Status = "🎉 Offer <date>" (add this status value to the Status Key legend if it's not already there)
- Assessment / take-home task requested → keep Status as-is but add a Company Notes line describing what's due and by when
- General acknowledgment / "we received your application" with no real signal → do not change Status

Only update a row if the email content clearly indicates a real status change for that specific company/role — do not guess or change Status on ambiguous content. If a thread is ambiguous, leave Status untouched and just flag it in your summary message for the user to interpret themselves.

For every detected change, update the Status cell in the Active Pipeline table (use Edit, preserve all other rows and formatting exactly) and append a one-line dated entry to that company's section under Company Notes (e.g. "- Status update (6/18/26): Received rejection email — 'unfortunately we have decided to move forward with other candidates.'"). Do not alter the Fit or Files columns. Do not touch rows for companies with no matching email today. Never delete or reorder existing rows.

After making any edits, produce a concise chat message titled "Application Status Updates — [today's date]" listing each company that changed, old status → new status, and a one-line reason/quote from the email. If nothing changed today, say so plainly: "No status changes detected today." If you found ambiguous emails you chose not to act on, list them separately under a "Needs your review" heading with a short description and let the user decide.

This task runs independently of the LinkedIn job alert digest task and the new-role-postings.md file — it only reads/writes application-tracker.md.