# Claude Job Search Framework — Multi-Board Edition

An AI-powered job search system built on [Claude Cowork](https://claude.ai) that automates daily scanning across **four job boards** — LinkedIn, Indeed, Dice, and ZipRecruiter — plus resume building, pipeline tracking, and interview preparation, while keeping you in full control of every application decision.

> **Credit:** This is an extended version of [davidwayland-alt/claude-cowork-assisted-job-search](https://github.com/davidwayland-alt/claude-cowork-assisted-job-search) (MIT), which was built and battle-tested over a real 6-week job search. The original scans LinkedIn only; this edition adds Indeed, Dice, and ZipRecruiter via their official MCP connectors, with cross-board de-duplication.

---

## What's Different in This Edition

| | Original | Multi-Board Edition |
|---|---|---|
| Job boards | LinkedIn only | LinkedIn + Indeed + Dice + ZipRecruiter |
| Search method | Browser (Claude in Chrome) | Official MCP connectors for Indeed/Dice/ZipRecruiter; browser for LinkedIn |
| De-duplication | Against your pipeline | Against your pipeline AND across boards (same role posted on multiple boards is merged, with sources tracked) |
| Recruiter-farm detection | — | Roles cross-posted on 3+ boards with no named end client get a scoring penalty |
| Salary pre-filtering | Scoring only | ZipRecruiter filters below-floor roles at the source |

Everything else — the memory system, fit-scoring rubric, ATS-compliant resume builds, cohort tracking, posture switching — works exactly as in the original.

---

## What It Does

**Every morning, automatically:**

- Searches LinkedIn, Indeed, Dice, and ZipRecruiter for new postings matching your target roles (local + national remote)
- Merges duplicate postings across boards and tracks which boards each role appeared on
- Scores each unique role 1–10 against your personal fit rubric
- Builds a custom ATS-compliant Word resume for every role that meets your threshold
- Generates a cover letter for high-scoring roles
- Reports everything in a structured daily scan summary

**Across every session:**

- Remembers your full application pipeline
- Tracks interview stages, contacts, and next steps
- Logs rejections and analyzes patterns
- Maintains consistent resume formatting across every build

**You decide:**

- Which roles to apply to (Claude scans and builds — never applies on your behalf)
- How selective to be (adjust your threshold from "actively searching" to "employed and passive")
- When to negotiate, when to decline, when to move on

---

## What's Included

```
/
├── Bootstrap Questionnaire.md          # Fill this out first — becomes your about-me.md
├── SETUP.md                            # Full setup walkthrough
├── templates/
│   ├── CLAUDE.md                       # Global Claude config (startup rules, resume rules)
│   ├── about-me-template.md            # Your career profile — Claude reads this every session
│   ├── working-memory-template.md      # Live job search state (applications, interviews, offers)
│   ├── job-pipeline-template.md        # Full application tracker with cohort analysis
│   └── TASKS-template.md               # Session task list with carry-forward context
└── scheduled-task/
    └── SKILL.md                        # The daily automated multi-board scan skill
```

---

## How the Multi-Board Scan Works

### Board coverage

| Board | Method | Date filter | Notes |
|-------|--------|-------------|-------|
| **Indeed** | Official MCP connector | None — skill discards results older than 3 days | Broadest general coverage; `get_job_details` pulls full JDs |
| **Dice** | Official MCP connector | `posted_date="ONE"` (24h) | Best for tech roles; contract-heavy, so full-time filtering matters |
| **ZipRecruiter** | Official MCP connector | `max_posted_minutes_ago=1440` (24h) | Supports salary-floor pre-filtering at the source; US/Canada only |
| **LinkedIn** | Claude in Chrome (no MCP exists) | `f_TPR=r86400` (24h) | Same URL-based searches as the original framework |

### Cross-board de-duplication

Aggregators cross-post aggressively — the same role routinely appears on 2–4 boards. The scan merges duplicates by normalized company + title, keeps the best apply link (company ATS > board link > LinkedIn), keeps the richest salary data, and records every source board. Wide cross-posting with no named end client is itself a signal (usually a staffing agency) and costs a point in scoring.

### Memory system

Claude reads four files at the start of every session:

| File                | Purpose                                                    |
| ------------------- | ---------------------------------------------------------- |
| `about-me.md`       | Your career history, skills, comp targets, geography rules, search keywords |
| `working-memory.md` | Current interview tracks, offers, decisions in flight      |
| `job-pipeline.md`   | Every application ever submitted, with status and source board |
| `TASKS.md`          | Open action items and session context                      |

This means Claude always knows exactly where you stand — no re-explaining your situation every day.

### Daily scan flow

1. Runs all configured searches on all four boards (24-hour posting window)
2. De-duplicates across boards and against your existing applications
3. Scores each unique role against your rubric
4. Builds a custom ATS-compliant resume for every role above your threshold
5. Generates a cover letter for top-scoring roles
6. Presents a full report with sources and direct application links

You review the report, decide what to apply to, and apply manually. Claude never submits on your behalf.

### Resume building

Resumes are built as ATS-compliant Word documents (.docx) using code, not templates:

- No Unicode bullet characters (ATS systems choke on them)
- No tables, headers, or footers
- Proper Word numbering system
- Role-targeted language pulled from the job description
- Validated automatically after each build

You maintain a small set of "base persona" resumes (Leadership, IC, Director, etc.). Claude reads all of them, identifies the best fit for each role, and synthesizes a tailored version from across all your bases.

### Fit scoring rubric

The rubric starts at 5 and adjusts based on role type, industry fit, location, credentials, and red flags. You configure the scoring weights in setup.

**Pre-disqualifiers** are evaluated before scoring and discard roles automatically:

- Comp below your floor
- Geography outside your range
- Contract / C2H / 1099 work types (especially relevant on Dice)
- Travel exceeding your threshold

---

## Setup

### Requirements

- [Claude](https://claude.ai) desktop app with Cowork mode enabled
- [Claude in Chrome](https://claude.ai/chrome) extension (for LinkedIn searches)
- **Three job board connectors** — connect each from the Claude connector directory (Settings → Connectors, or just ask Claude to suggest them):
  - **Indeed** (`mcp.indeed.com`)
  - **Dice** (`mcp.dice.com`)
  - **ZipRecruiter** (`api.ziprecruiter.com/mcp`)
- A workspace folder on your computer

### Steps

1. **Connect the three job board connectors** listed above and verify each works by asking Claude to run a quick test search.

2. **Fill out the Bootstrap Questionnaire** (`Bootstrap Questionnaire.md`). This is the one-time setup — your answers become your `about-me.md`.

3. **Open Claude Cowork** and start a new session with your workspace folder mounted.

4. **Share your completed questionnaire** and say:
   > *"Please use my answers to create my about-me.md file and set up my workspace using the multi-board job search framework."*

   Claude will build all memory files, configure your search parameters for all four boards, and set up the folder structure.

5. **Read SETUP.md** for the full walkthrough including base resume creation and scan configuration.

6. **Schedule the daily scan** by telling Claude:
   > *"Please set up the daily job scan to run at 8:00am."*

---

## Workspace Structure

After setup, your workspace looks like this:

```
~/Claude-Cowork-Workspace/workspace/
├── context/
│   ├── CLAUDE.md                   # Global config and startup rules
│   └── about-me.md                 # Your career profile + search keywords
├── memory/
│   ├── working-memory.md           # Live search state
│   ├── job-pipeline.md             # Full application tracker
│   └── TASKS.md                    # Task list
├── Resume/
│   ├── [Your base persona resumes]
│   └── Hunt/
│       ├── Applied Companies/      # Per-company submission folders
│       └── Versions/               # All custom builds (auto-named)
├── Interview Prep/                 # Interview research and prep docs
└── outputs/                        # Scan reports and other deliverables
```

---

## Customization

Everything is configurable in your `about-me.md` and `CLAUDE.md`:

- **Search keywords:** One keyword list drives all four boards — no per-board configuration needed
- **Board selection:** Don't want a board? Delete its section from the skill. Want fewer? The skill degrades gracefully when a connector is disconnected
- **Scoring weights:** Adjust what matters most for your target roles
- **Pre-disqualifiers:** Set your comp floor, geography radius, travel hard limit
- **Resume rules:** Font, formatting, contact line, output location
- **Session behavior:** What Claude checks at startup, what gets saved at close

---

## Posture Switching

The framework supports two operating modes:

**Active search** — Lower threshold (e.g., 6+), broader searches, apply aggressively.

**Passive/employed** — Higher threshold (e.g., 9+), tighter pre-disqualifiers, apply only to materially better opportunities.

Switch postures by updating your threshold and parameters in `about-me.md`.

---

## What This Is Not

- This is not a service. There is no backend, no SaaS, no subscription.
- Claude never applies to jobs on your behalf.
- Your data stays local — in your workspace folder on your computer. (Job searches go through each board's official connector under their terms.)
- This requires a Claude subscription with Cowork mode.

---

## License

MIT, same as the [original framework](https://github.com/davidwayland-alt/claude-cowork-assisted-job-search) by David Wayland. Use it, adapt it, share it.

---

*Built to solve a real problem. If it helps you land something great, that's the point.*
