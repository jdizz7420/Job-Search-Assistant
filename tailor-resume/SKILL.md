---
name: tailor-resume
description: For each Interested job in new-role-postings.md without an existing resume, fetches the full JD via Chrome, generates a tailored resume and cover letter as .docx files, and updates the tracker to 🟡 Resume Ready
---

Read "/Users/jdlawson/Documents/Claude/Projects/Application Tracker/new-role-postings.md" and collect every row where Status = "Interested". These are the candidate jobs.

For each candidate job, check whether a tailored resume already exists by running:
`ls "/Users/jdlawson/Documents/Claude/Projects/Resume Builder/"` and looking for a file matching `*[Company]*Resume.docx` where [Company] is the company name from the row. Skip any job that already has a matching file — only process jobs with no existing resume.

If there are no new jobs to process, say so and stop.

Read the following files as context before building anything:
- "/Users/jdlawson/Documents/Claude/Projects/Resume Builder/build_toast.js" — this is the master resume template (structure, formatting, full content)
- "/Users/jdlawson/Documents/Claude/Projects/Resume Builder/resume-instructions.md" — resume writing rules to follow

For each new job, work through these steps:

## Step 1 — Fetch the full job description via Chrome

Use the same Chrome enrichment approach as the linkedin-job-alert-digest skill:
1. Navigate to `https://www.linkedin.com/jobs/view/<JOBID>/`
2. Wait ~2-3 seconds, then scroll down to trigger lazy-loading
3. Read the page with mcp__Claude_in_Chrome__get_page_text
4. If "About the job" is truncated behind a "…more" link, click it and re-read the full page
5. Do a second scroll/wait/read pass if the description hasn't fully loaded on the first attempt
6. If Chrome is unavailable or the page won't load after two passes, skip enrichment and flag this job in the summary — do not fail the whole run

## Step 2 — Tailor the resume content

Using the master template from build_toast.js and the full JD, produce tailored resume content:

**Professional Summary:** Rewrite to position Jonathan specifically for this role. Mirror the language and priorities from the JD. Keep it 2-3 sentences. Do not use clichés. Do not claim experience he doesn't have.

**Experience bullets:** Reorder and lightly reword bullets to surface the most relevant accomplishments first. Mirror keywords from the JD naturally — never keyword-stuff. Keep all facts accurate; do not fabricate metrics, titles, or responsibilities.

**Core Skills:** Reorder skill rows and adjust the listed sub-skills to emphasize what the JD prioritizes. Do not add skills he doesn't have.

**Constraints (always enforce):**
- Official title at Robinhood: "Deployment Lead" (never "Growth Marketing Operations & Strategy")
- AI tool attribution: "Claude Code" — never OpenAI
- Never claim pre-sales, field sales, enterprise CS, or quota-carrying experience
- Font minimum 10pt (already enforced in the template)
- Honest framing of gaps; no fabricated experience

## Step 3 — Write and run the resume build script

Write a new file `/Users/jdlawson/Documents/Claude/Projects/Resume Builder/build_[Company].js` based on build_toast.js, replacing the content with the tailored version and updating the output path to:
`/Users/jdlawson/Documents/Claude/Projects/Resume Builder/Jonathan_Lawson_[Company]_Resume.docx`

Use the exact company name as it appears in new-role-postings.md, with spaces replaced by underscores if needed in the filename.

Check if the `docx` npm package is available:
`ls "/Users/jdlawson/Documents/Claude/Projects/Resume Builder/node_modules/docx" 2>/dev/null || echo "missing"`

If missing, run: `cd "/Users/jdlawson/Documents/Claude/Projects/Resume Builder" && npm install docx`

Then run: `node "/Users/jdlawson/Documents/Claude/Projects/Resume Builder/build_[Company].js"`

Confirm the .docx file was created. If the script errors, diagnose and fix it before moving on.

## Step 4 — Write the cover letter

Write a new file `/Users/jdlawson/Documents/Claude/Projects/Resume Builder/build_[Company]_cover.js` that generates a cover letter .docx using the same `docx` package and styling (Navy headers, Arial font, same margins).

Cover letter structure:
- Header: Jonathan's name and contact info (same as resume header)
- Date
- Hiring team salutation (generic: "Dear Hiring Team,")
- Opening paragraph: Why this role and company specifically — draw from the JD and any Company Notes in application-tracker.md
- Body (1-2 paragraphs): The 2-3 most relevant accomplishments from his background, tied directly to what the JD is asking for. Be specific and honest.
- Closing paragraph: Brief, confident close. No fluff.
- Sign-off: "Best, Jonathan Lawson"

Output path: `/Users/jdlawson/Documents/Claude/Projects/Resume Builder/Jonathan_Lawson_[Company]_CoverLetter.docx`

Run it: `node "/Users/jdlawson/Documents/Claude/Projects/Resume Builder/build_[Company]_cover.js"`

Confirm the .docx was created.

## Step 5 — Update the tracker

In "/Users/jdlawson/Documents/Claude/Projects/Application Tracker/application-tracker.md", find the row for this company in the Active Pipeline table and:
- Update Status to "🟡 Resume Ready"
- Update the Files column to "Resume, Cover Letter"
- Remove the "⚠️ Needs application" flag if present
- Append a dated note to the Company Notes section: "- Resume & cover letter built ([date]): tailored to [role title]"

## Final summary

After processing all jobs, output a summary listing:
- Each company processed: role title, files created (Resume ✓, Cover Letter ✓)
- Any jobs skipped because files already existed
- Any jobs where Chrome enrichment failed (flagged for manual JD paste)

If a job's JD couldn't be fetched, tell the user: "Paste the job description for [Company] and say 'tailor my resume to this role' to build it manually."
