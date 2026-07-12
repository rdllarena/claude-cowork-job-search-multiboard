---
name: daily-job-scan
description: Daily multi-board job scan — find new postings on LinkedIn, Indeed, Dice, and ZipRecruiter, score them, build custom resumes for roles scoring 6+
---

You are running an automated daily job scan and resume generation task for [YOUR FULL NAME] ([YOUR EMAIL]), a [SENIORITY LEVEL] [YOUR FIELD] professional actively searching for a new position.

Each run: find new job postings across FOUR job boards (LinkedIn, Indeed, Dice, ZipRecruiter), de-duplicate across boards, score each unique role against fit criteria, generate a custom resume for every role scoring 6+, and present a full report including both passing and failing roles.

---

## STEP 1: LOAD CONTEXT

Read these files before doing anything else:
- ~/Claude-Cowork-Workspace/context/about-me.md — full career history, certifications, skills, search keywords
- ~/Claude-Cowork-Workspace/CLAUDE.md — resume building rules (mandatory)
- ~/Claude-Cowork-Workspace/memory/context/job-pipeline.md — current applications (for de-duplication)
- ~/Claude-Cowork-Workspace/memory/working-memory.md — compensation context, geography rules

---

## STEP 2: SEARCH FOR NEW ROLES — FOUR BOARDS

Search all four boards for every keyword set defined in about-me.md. Three boards use official MCP connectors (fast, structured); LinkedIn uses Claude in Chrome.

**Run the three connector searches in parallel where possible.** If any connector is unavailable or returns an auth error, note it in the report and continue with the remaining boards — never abort the whole scan because one board failed.

### 2a. Indeed (MCP connector)

Tool: `search_jobs` on the Indeed connector.

For each keyword set, run BOTH a local and a remote search:

```
search_jobs(search="[KEYWORDS]", location="[YOUR CITY, ST]", country_code="US", job_type="fulltime")
search_jobs(search="[KEYWORDS]", location="remote", country_code="US", job_type="fulltime")
```

Indeed connector behavior (verified):
- Returns markdown with Job Id, company, location, posted date, comp, and a `to.indeed.com` apply link
- **No posted-date filter exists** — results include older postings. Check the "Posted on" date and discard anything older than 3 days (it was either caught by a prior scan or is stale)
- Narrow multi-word queries often return zero results — if a keyword set returns nothing, retry once with a broader 2–3 word version
- Use `get_job_details(job_id)` to pull the full JD for any role that passes scoring

### 2b. Dice (MCP connector) — strongest for tech roles

Tool: `search_jobs` on the Dice connector.

```
search_jobs(keyword="[KEYWORDS]", location="[YOUR CITY, ST]", radius=[MILES], radius_unit="mi",
            posted_date="ONE", employment_types=["FULLTIME"], jobs_per_page=25)
search_jobs(keyword="[KEYWORDS]", workplace_types=["Remote"],
            posted_date="ONE", employment_types=["FULLTIME"], jobs_per_page=25)
```

Dice connector behavior (verified):
- `posted_date="ONE"` = last 24 hours — this is a daily scan, keep it
- Returns structured JSON: title, companyName, salary, employmentType, workplaceTypes, `detailsPageUrl` (the apply link), postedDate
- Dice is contract-heavy. `employment_types=["FULLTIME"]` filters most of it, but still check `employmentType` on each result — Contract/C2H/Third Party roles hit the pre-disqualifiers unless your about-me.md says otherwise
- Location searches also return remote roles from other states — verify `jobLocation` against your geography rules

### 2c. ZipRecruiter (MCP connector)

Tool: `search_jobs` on the ZipRecruiter connector.

```
search_jobs(job_role="[KEYWORDS]", location="[YOUR CITY, ST]", country_admin_code="US",
            radius_miles=[MILES], max_posted_minutes_ago=1440,
            employment_types=["FULL_TIME"], salary_min=[YOUR_COMP_FLOOR])
search_jobs(job_role="[KEYWORDS]", location_types=["REMOTE"], country_admin_code="US",
            max_posted_minutes_ago=1440, employment_types=["FULL_TIME"], salary_min=[YOUR_COMP_FLOOR])
```

ZipRecruiter connector behavior (verified):
- `max_posted_minutes_ago=1440` = last 24 hours (1440 is the minimum the API accepts)
- `salary_min` pre-filters below-floor roles at the source (note: API caps salary values at $300,000 — if your floor is higher, omit the filter and rely on scoring)
- Returns title, company, location, salary range, `job_redirect_url` (the apply link), days_ago
- US and Canada only
- Matching is loose — off-lane titles show up. Apply your exclusion list aggressively

### 2d. LinkedIn (Claude in Chrome — no MCP connector exists)

Use Claude in Chrome (`tabs_context_mcp` to get or create a tab). Run each search URL and collect every unique result.

**LOCAL — within [X] miles of [YOUR CITY], posted last 24 hours:**
```
https://www.linkedin.com/jobs/search/?keywords=[KEYWORDS_ENCODED]&location=[YOUR_CITY_ENCODED]&distance=[MILES]&f_TPR=r86400&sortBy=R
```

**NATIONAL REMOTE — remote only (f_WT=2), posted last 24 hours:**
```
https://www.linkedin.com/jobs/search/?keywords=[KEYWORDS_ENCODED]&f_WT=2&f_TPR=r86400&sortBy=R
```

URL-encoding: spaces → %20, quotes → %22. Example: "security architect" → %22security%20architect%22

If Chrome is unavailable, fall back to WebSearch for LinkedIn queries and note the limitation in the report.

### 2e. ATS direct sweep (WebSearch)

```
WebSearch: "[YOUR KEY TITLE KEYWORDS]" site:boards.greenhouse.io OR site:jobs.lever.co OR site:myworkdayjobs.com remote [CURRENT YEAR]
```

### 2f. De-duplicate — across boards AND against your pipeline

The same role frequently appears on 2–4 boards (aggregators cross-post). De-dupe by normalized **company + role title** (ignore punctuation, seniority prefixes like "Sr."/"Senior", and requisition numbers):

1. Merge duplicates into ONE entry
2. Record every board it appeared on in a `Sources` field (useful signal — widely cross-posted roles are often recruiter-farmed)
3. Prefer the **direct/ATS apply link** in this priority: company ATS URL > Indeed/Dice/ZipRecruiter link > LinkedIn link
4. Prefer the richest salary data across the copies (boards often show different ranges — note the discrepancy if material)

Then remove clearly off-lane titles (define your own exclusions):
- [EXCLUSION 1 — e.g., "Analyst (non-senior, non-architect)"]
- [EXCLUSION 2 — e.g., "Recruiter / HR roles"]
- [EXCLUSION 3 — e.g., "Sales / pre-sales / solutions engineer"]

Check ~/Claude-Cowork-Workspace/Resume/Hunt/Applied Companies/ — if a company subfolder already exists, note "ALREADY APPLIED" but still include the role in scoring.

---

## STEP 3: SCORE EACH ROLE (1–10 scale)

**Start at 5. Adjust up or down:**

Role type:
+3 = [YOUR TOP TARGET ROLE — e.g., VP Engineering, Head of Product, Principal Architect]
+3 = [SECOND TOP TARGET — e.g., Director at growth-stage company]
+2 = [GOOD FIT ROLE — e.g., Staff-level IC, Senior Director at enterprise]
+1 = [DECENT FIT — e.g., Senior Manager with path to Director]
-1 = [SLIGHT MISMATCH — e.g., Consulting/advisory role]
-2 = [SIGNIFICANT MISMATCH — e.g., Manager without Director path]
-3 = [WRONG LEVEL — e.g., IC contributor, no leadership scope]

Fit signals:
+1 = [PREFERRED INDUSTRY 1 — e.g., FinTech, AI/ML, SaaS]
+1 = [PREFERRED INDUSTRY 2]
+1 = [LOCAL BONUS — within [Y] miles of home]
+1 = [YOUR KEY CREDENTIAL explicitly listed as preferred]

Red flags (reduce score):
-2 = [HARD SKILL GAP 1 — e.g., requires specific tool you don't have]
-2 = [INDUSTRY MISMATCH — e.g., automotive, hardware]
-2 = Posted comp range tops out below $[YOUR_COMP_FLOOR] base (flag as COMP MISS)
-1 = Recruiter/staffing-agency posting cross-posted on 3+ boards with no named end client

Cap at 10, floor at 1.

Record score AND primary reason for score (one phrase) for every role, regardless of score.

---

## STEP 4: FOR EACH ROLE SCORING 6+, GENERATE A CUSTOM RESUME

Do this for every 6+ role, in sequence.

### 4a. Get the full job description, verify the posting is active, and capture the application URL

- **Indeed roles:** call `get_job_details(job_id)` on the Indeed connector for the full JD
- **Dice roles:** navigate to `detailsPageUrl` in Chrome and use get_page_text
- **ZipRecruiter roles:** navigate to `job_redirect_url` in Chrome and use get_page_text (it redirects to the source posting)
- **LinkedIn roles:** navigate to the posting URL in Chrome and use get_page_text

**Verify the posting is still active before building a resume.** A posting is NOT active if:
- The page returns a 404 or "job no longer exists" message
- The page says "No longer accepting applications"
- The ATS page is empty or redirects to a generic jobs page

If the posting is inactive, skip resume generation and note "POSTING EXPIRED" in the report.

**Capture the direct application URL** — required in the final report. Priority:
1. The "Apply" button destination (Greenhouse, Lever, Workday, company ATS)
2. The board's job posting URL (Indeed/Dice/ZipRecruiter link)
3. LinkedIn job view URL if company is confidential

### 4b. Read base resumes

```bash
# Use pandoc to extract text from each base resume
pandoc ~/Claude-Cowork-Workspace/Resume/[YourName]_Resume_[Variant1].docx -t plain
pandoc ~/Claude-Cowork-Workspace/Resume/[YourName]_Resume_[Variant2].docx -t plain
```

### 4c. Synthesize and build resume

Select the most appropriate base resume for the role type. Tailor it to the JD by:
- Adjusting the summary to match the role's language and focus
- Reordering bullet points to lead with most relevant accomplishments
- Mirroring key terms from the JD (for ATS matching)
- Adjusting the title line to match the role's language

Build the resume as a Node.js docx-js script:
- NODE_PATH=/usr/local/lib/node_modules_global/lib/node_modules
- US Letter: width 12240, height 15840 DXA; margins 1080 DXA all sides
- Font: Arial, 20pt body (= 10pt — docx size is in half-points)
- NEVER use Unicode bullets — use LevelFormat.BULLET with Word numbering config, text: "-"
- No tables, no headers, no footers
- Tab stops for right-aligned dates: position 10080
- Section headers: bold 22pt (= 11pt) navy (#1F3864) with bottom border
- Title line matches JD role title language
- Contact line: [YOUR_PHONE] | [YOUR_EMAIL] | linkedin.com/in/[YOUR_LINKEDIN]
- Use "--" instead of em dashes
- Output: ~/Claude-Cowork-Workspace/Resume/[YourName]_Resume_{CompanyName}_{RoleAbbrev}.docx

Run and validate:
```bash
cd ~/Claude-Cowork-Workspace/outputs && NODE_PATH=/usr/local/lib/node_modules_global/lib/node_modules node build_resume_{CompanyName}.js
VALIDATE_SCRIPT=$(find /sessions/*/mnt/.claude/skills/docx/scripts/office/validate.py 2>/dev/null | head -1)
python3 "$VALIDATE_SCRIPT" ~/Claude-Cowork-Workspace/Resume/[YourName]_Resume_{CompanyName}_{RoleAbbrev}.docx
```

### 4d. Cover letter

For roles scoring 8+, also generate a brief cover letter (.docx). Lead with the strongest fit narrative for that role type. Save as: [YourName]_CoverLetter_{CompanyName}_{RoleAbbrev}.docx

---

## STEP 5: PRESENT RESULTS

Use mcp__cowork__present_files to present each generated resume and cover letter.

Then output the full report in this exact format:

---
### Daily Job Scan -- {YYYY-MM-DD}
**Boards scanned:** LinkedIn, Indeed, Dice, ZipRecruiter {note any board that failed}
**Scanned:** {N} unique roles ({N} before cross-board de-dupe) | **Scoring 6+:** {N} | **Below threshold:** {N} | **Resumes generated:** {N}

---

#### ROLES SCORING 6+ (sorted highest to lowest)

**[Score]/10 -- [Role Title] | [Company] | [City or Remote]**
Sources: {LinkedIn, Indeed, Dice, and/or ZipRecruiter}
Comp: {posted range or "not listed"} | {On-site/Hybrid/Remote}
Apply: {direct URL}
Resume: {file path}
{Cover letter path if generated}
Fit note: {2-3 sentences on fit, any flags}
{ALREADY APPLIED if applicable}

---

#### ROLES BELOW THRESHOLD (sorted by score, highest to lowest)

| Score | Role | Company | Location | Sources | Why it didn't make it |
|-------|------|---------|----------|---------|-----------------------|

(Include ALL roles scanned that scored 1-5, with one-liner reason for each)

---

If no roles score 6+:
> No new 6+ roles found today. See below-threshold list for all {N} roles scanned.

---

## IMPORTANT CONSTRAINTS
- Do NOT apply to any roles — scan, score, and generate only
- Do NOT modify job-pipeline.md during this task — update the pipeline manually
- If a job board connector fails, continue with the remaining boards and note the failure in the report
- If Chrome is unavailable, fall back to WebSearch for LinkedIn and note the limitation
- If docx generation fails for a role, note the error and continue with remaining roles
- Each run is independent — do not reference prior scan results
- 24-hour posting windows (f_TPR=r86400, posted_date="ONE", max_posted_minutes_ago=1440) are intentional — this is a daily scan, not a backfill. Indeed has no date filter, so discard results older than 3 days manually
