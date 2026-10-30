# Trafforge careers page: developer notes

`careers.html` is a self-contained page (HTML, CSS and JS in one file, no build step, no dependencies beyond Google Fonts). It lists open roles, shows each job, and posts applications to an endpoint you provide. This document covers the small backend piece it needs and how applications reach the recruiting team's candidate tracker.

## 1. Drop in the page

1. Serve `careers.html` at `/careers` (or wherever suits the site). It can be used as-is or split into your site's template; the header and footer are simple to swap for your existing ones.
2. Edit the `CONFIG` block at the top of the script:
   - `endpoint`: the URL the form POSTs to (default `/api/careers/applications`).
   - `privacyUrl`: link to the privacy policy shown next to the consent checkbox.
   - `contactEmail`: a monitored careers inbox, shown for accommodation requests and as a fallback if submission fails.
   - `maxResumeMB`: keep in sync with the backend limit.
3. Edit the `JOBS` array to manage roles. Each job's `slug` becomes its link, for example `/careers#senior-devops-cloud-engineer`. The DevOps role is taken from Trafforge's existing posting; confirm it's still open and accurate before launch.

## 2. Link LinkedIn job posts to the page

When posting a job on LinkedIn, choose to receive applicants on an external website and use the job's direct link with a source tag:

```
https://<your-domain>/careers?source=linkedin#senior-devops-cloud-engineer
```

The query string must come before the `#`. The form pre-selects "LinkedIn" as the source and sends the raw value as `source_param`, so the team can see which channel works. Use `?source=referral`, `?source=indeed` and so on for other channels.

## 3. Backend endpoint

`POST {CONFIG.endpoint}` as `multipart/form-data`. Return any `2xx` on success; any other status shows the candidate an error and lets them retry.

| Field | Required | Notes |
| --- | --- | --- |
| `first_name`, `last_name` | yes | Plain text |
| `email` | yes | Validate format server-side too |
| `phone` | no | |
| `linkedin_url` | no | Must start with http(s):// |
| `location` | yes | Free text, e.g. "Louisville, KY" |
| `work_authorization` | yes | One of the two select options |
| `resume` | yes | PDF, DOC or DOCX, max 5 MB |
| `cover_note` | no | Up to 3,000 characters |
| `source` | yes | LinkedIn, Trafforge website, Referral, Job board, Other |
| `source_param` | no | Raw `?source=` / `utm_source` value |
| `consent` | yes | Value `yes`; reject without it and store a timestamp when present |
| `job_slug`, `job_title` | yes | Added by the page |
| `submitted_at` | yes | ISO timestamp from the browser (use server time as the record of truth) |
| `company_website` | must be empty | Honeypot; if filled, return `200` and discard silently |

On each valid submission the backend should do three things:

1. **Store the application.** Save the fields to your database and the resume in private storage (not a public bucket). Generate an access-controlled link for the team to open it.
2. **Send the candidate an acknowledgement.** Email the "Application received" message (template A in the recruiting playbook) from a monitored address, not a no-reply one. This covers the team's 1-business-day standard automatically.
3. **Notify the team.** A short email or Slack message to the recruiting inbox: name, role, source, link to the record.

Also worth adding: rate limiting per IP, server-side validation of every field, virus scanning of uploads if your stack supports it, and a CAPTCHA only if spam gets past the honeypot.

## 4. Getting applications into the candidate tracker

The team works in a shared candidate tracker on claude.ai. Its **Import** button accepts a CSV file or rows pasted from a spreadsheet, skips anyone whose email is already in the tracker, and sets new candidates to the Applied stage with a follow-up date.

Provide an admin-only export (for example `GET /api/careers/applications.csv?since=2026-11-01`, behind your staff login) with these headers:

```csv
first_name,last_name,email,phone,linkedin_url,location,job_title,source,resume_url,cover_note,submitted_at
```

The importer also recognizes common alternatives (`name`, `role`, `position`, `linkedin`, `notes`), so an export from a form tool or ATS will usually work without changes. A daily or twice-weekly export and import fits the team's weekly pipeline review.

If you'd rather push applications into the tracker automatically, that's possible later; the tracker's data can be written by a signed-in Claude session, but there's no public write API, so a CSV export is the dependable route for now.

## 5. Data and compliance checklist

- Keep a retention period for unsuccessful applicants (many companies use 12–24 months) and delete resumes after it; state it in the privacy policy.
- Have someone with legal or HR responsibility review the EEO statement and consent wording in the page footer and form.
- Restrict resume and export access to the recruiting team.
- Never put candidate data in URLs, query strings or analytics events.
