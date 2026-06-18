---
name: tailor-resume
description: For each Interested job in new-role-postings.md without an existing resume, fetches the full JD via Chrome, identifies any experience gaps, asks before adding anything new, then generates a tailored resume and cover letter as .docx files and updates the tracker to 🟡 Resume Ready
---

Read "/Users/jdlawson/Documents/Claude/Projects/Application Tracker/new-role-postings.md" and collect every row where Status = "Interested". These are the candidate jobs.

For each candidate job, check whether a tailored resume already exists by running:
`ls "/Users/jdlawson/Documents/Claude/Projects/Resume Builder/"` and looking for a file matching `*[Company]*Resume.docx` where [Company] is the company name from the row. Skip any job that already has a matching file — only process jobs with no existing resume.

If there are no new jobs to process, say so and stop.

Read the following files as context before building anything:
- "/Users/jdlawson/Documents/Claude/Projects/Resume Builder/build_toast.js" — this is the BASE RESUME (the single source of truth for Jonathan's experience, skills, and accomplishments)
- "/Users/jdlawson/Documents/Claude/Projects/Resume Builder/resume-instructions.md" — resume writing rules to follow

Process one job at a time. For each new job:

## Step 1 — Fetch the full job description via Chrome

Use the same Chrome enrichment approach as the linkedin-job-alert-digest skill:
1. Navigate to `https://www.linkedin.com/jobs/view/<JOBID>/`
2. Wait ~2-3 seconds, then scroll down to trigger lazy-loading
3. Read the page with mcp__Claude_in_Chrome__get_page_text
4. If "About the job" is truncated behind a "…more" link, click it and re-read the full page
5. Do a second scroll/wait/read pass if the description hasn't fully loaded on the first attempt
6. If Chrome is unavailable or the page won't load after two passes, ask the user to paste the JD manually before continuing — do not proceed without it

## Step 2 — Identify gaps and ask before building

Before making any changes, analyze the JD to identify its most important requirements (typically: must-have skills, specific tools, domain experience, or role responsibilities that the JD emphasizes repeatedly or marks as required).

Compare each important requirement against the base resume content (build_toast.js). Categorize each as:
- **Covered** — clearly supported by an existing bullet, skill row, or the summary
- **Partially covered** — related experience exists but doesn't directly address the requirement
- **Missing** — no evidence of this experience anywhere in the base resume

For any **Missing** or **Partially covered** requirement that is important to the role, DO NOT assume Jonathan has it. Instead, surface these as questions BEFORE building. Ask concisely, grouped together — for example:

> "Before I build the [Company] resume, I have a few questions about experience the JD emphasizes that isn't clearly on your base resume:
> 1. The JD calls out [X] — do you have experience with this? If so, can you give me a brief example?
> 2. They list [Y] as required — is this something you've done at any of your previous roles?"

Wait for Jonathan's answers before proceeding. If he confirms experience, use his description to add or update content. If he says he doesn't have it, note it as a genuine gap — do not add it to the resume.

**If there are no gaps worth asking about, skip this step and proceed directly to Step 3.**

## Step 3 — Tailor the resume content

Starting from the exact content in build_toast.js, make only these types of changes:

**What you CAN do:**
- **Remove** bullets that are clearly irrelevant to this role (keep the resume focused)
- **Reorder** bullets within each job to put the most relevant accomplishments first
- **Reword** bullets to mirror the JD's language — the underlying facts, metrics, and responsibilities must remain identical; only phrasing may change to match JD keywords
- **Rewrite the Professional Summary** to position Jonathan specifically for this role using language from the JD
- **Reorder and adjust** the Core Skills section to emphasize what the JD prioritizes; you may add sub-skill terms to existing skill rows only if Jonathan confirmed he has them
- **Add new content** only for experience Jonathan confirmed in Step 2 — write it in the same bullet style as the rest of the resume

**What you must NEVER do:**
- Add bullets, accomplishments, metrics, or responsibilities not in the base resume (unless confirmed in Step 2)
- Imply experience in areas not covered (pre-sales, field sales, enterprise CS, quota-carrying)
- Inflate or change metrics (numbers, team sizes, revenue figures must stay exact)
- Change Jonathan's official title at Robinhood — always "Deployment Lead"
- Attribute AI work to OpenAI — always "Claude Code"

## Step 4 — Write and run the resume build script

Write a new file `/Users/jdlawson/Documents/Claude/Projects/Resume Builder/build_[Company].js` based on build_toast.js, replacing the content with the tailored version from Step 3 and updating the output path to:
`/Users/jdlawson/Documents/Claude/Projects/Resume Builder/Jonathan_Lawson_[Company]_Resume.docx`

Use the exact company name as it appears in new-role-postings.md, with spaces replaced by underscores if needed in the filename.

Check if the `docx` npm package is available:
`ls "/Users/jdlawson/Documents/Claude/Projects/Resume Builder/node_modules/docx" 2>/dev/null || echo "missing"`

If missing, run: `cd "/Users/jdlawson/Documents/Claude/Projects/Resume Builder" && npm install docx`

Then run: `node "/Users/jdlawson/Documents/Claude/Projects/Resume Builder/build_[Company].js"`

Confirm the .docx file was created. If the script errors, diagnose and fix it before moving on.

## Step 5 — Write the cover letter

Write a new file `/Users/jdlawson/Documents/Claude/Projects/Resume Builder/build_[Company]_cover.js` that generates a cover letter .docx using the same `docx` package and styling (Navy headers, Arial font, same margins).

Cover letter structure:
- Header: Jonathan's name and contact info (same as resume header)
- Date
- Hiring team salutation (generic: "Dear Hiring Team,")
- Opening paragraph: Why this role and company specifically — draw from the JD and any Company Notes in application-tracker.md
- Body (1-2 paragraphs): The 2-3 most relevant accomplishments from his background, tied directly to what the JD is asking for. Only use experience from the base resume or confirmed in Step 2.
- Closing paragraph: Brief, confident close. No fluff.
- Sign-off: "Best, Jonathan Lawson"

Output path: `/Users/jdlawson/Documents/Claude/Projects/Resume Builder/Jonathan_Lawson_[Company]_CoverLetter.docx`

Run it: `node "/Users/jdlawson/Documents/Claude/Projects/Resume Builder/build_[Company]_cover.js"`

Confirm the .docx was created.

## Step 6 — Update the tracker

In "/Users/jdlawson/Documents/Claude/Projects/Application Tracker/application-tracker.md", find the row for this company in the Active Pipeline table and:
- Update Status to "🟡 Resume Ready"
- Update the Files column to "Resume, Cover Letter"
- Remove the "⚠️ Needs application" flag if present
- Append a dated note to the Company Notes section: "- Resume & cover letter built ([date]): tailored to [role title]"

## Final summary

After processing all jobs, output a summary listing:
- Each company processed: role title, files created (Resume ✓, Cover Letter ✓), and any gaps that were surfaced and confirmed
- Any jobs skipped because files already existed
- Any genuine gaps noted (important JD requirements Jonathan confirmed he doesn't have)
