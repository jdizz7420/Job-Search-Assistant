# claude-skills

Personal Claude Code skills for job search automation.

---

## 📬 linkedin-job-alert-digest

Processes LinkedIn job alert emails from the last 24 hours into a structured digest.

**What it does:**
- Searches Gmail for LinkedIn job alert emails
- Parses job listings (title, company, location, salary) and deduplicates by Job ID
- Enriches new listings via Chrome to extract salary ranges and job descriptions
- Cross-references against the job tracker to skip already-seen postings
- Appends new jobs to `new-role-postings.md` and outputs a digest grouped by alert

**Invoke:** `/linkedin-job-alert-digest`

---

## 📋 application-status-check

Checks Gmail for status updates on active applications and syncs them to the tracker.

**What it does:**
- Reads the active pipeline from `application-tracker.md`
- Searches Gmail for emails from each tracked company in the last 24 hours
- Detects rejections, interview invites, recruiter screens, offers, and assessments
- Updates the `Status` column and appends a dated note under Company Notes
- Flags ambiguous emails for manual review rather than guessing

**Invoke:** `/application-status-check`

---

## 🗂 Dependencies

Both skills read and write files in the Application Tracker project:

| File | Used by |
|---|---|
| `application-tracker.md` | `application-status-check` (r/w), `linkedin-job-alert-digest` (write on interest) |
| `new-role-postings.md` | `linkedin-job-alert-digest` (r/w) |

Gmail access is provided via the Claude.ai Gmail MCP connector.
