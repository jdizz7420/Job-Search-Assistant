# claude-skills
   2
   3 Personal Claude Code skills for job search automation.
   4
   5 ## Skills
   6
   7 ### `linkedin-job-alert-digest`
   8 Processes LinkedIn job alert emails from the last 24 hours into a structured digest.
   9
  10 - Searches Gmail for LinkedIn job alert emails
  11 - Parses job listings (title, company, location, salary) and deduplicates by Job ID
  12 - Enriches new listings by visiting each LinkedIn job page via Chrome to extract salary ranges and job descriptio
     ns
  13 - Cross-references against the job tracker to skip already-seen postings
  14 - Appends new jobs to `new-role-postings.md` and outputs a formatted digest grouped by job alert
  15
  16 **Invoke:** `/linkedin-job-alert-digest`
  17
  18 ---
  19
  20 ### `application-status-check`
  21 Checks Gmail for status updates on active job applications and syncs them to the tracker.
  22
  23 - Reads the active pipeline from `application-tracker.md`
  24 - Searches Gmail for emails from or about each tracked company in the last 24 hours
  25 - Detects status changes: rejections, interview invites, recruiter screens, offers, and assessments
  26 - Updates the `Status` column and appends a dated note to Company Notes for each change
  27 - Flags ambiguous emails for manual review rather than guessing
  28
  29 **Invoke:** `/application-status-check`
  30
  31 ---
  32
  33 ## Dependencies
  34
  35 Both skills read and write markdown files in the Application Tracker project:
  36
  37 | File | Used by |
  38 |---|---|
  39 | `application-tracker.md` | `application-status-check` (read/write), `linkedin-job-alert-digest` (write on inter
     est) |
  40 | `new-role-postings.md` | `linkedin-job-alert-digest` (read/write) |
  41
  42 Gmail access is provided via the Claude.ai Gmail MCP connector.
