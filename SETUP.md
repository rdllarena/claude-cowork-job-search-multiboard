# Setup Guide — Claude Job Search Framework (Multi-Board Edition)

This walks you from zero to a running daily scan across LinkedIn, Indeed, Dice, and ZipRecruiter. Budget about an hour for first-time setup; after that the system runs itself.

---

## 1. Prerequisites

1. **Claude desktop app** with Cowork mode enabled (requires a paid Claude plan).
2. **Claude in Chrome extension** — install from [claude.ai/chrome](https://claude.ai/chrome) and sign in to LinkedIn in that Chrome profile. This is only used for LinkedIn; the other three boards use connectors.
3. **Job board connectors.** In the Claude app, open Settings → Connectors (or just tell Claude: *"suggest connectors for Indeed, Dice, and ZipRecruiter"*) and connect:
   - **Indeed** — official connector at `mcp.indeed.com`
   - **Dice** — official connector at `mcp.dice.com`
   - **ZipRecruiter** — official connector at `api.ziprecruiter.com/mcp`

   Verify each one: ask Claude *"search Dice for [your target title] near [your city]"* and confirm you get results. Repeat for Indeed and ZipRecruiter.

4. **A workspace folder.** Create `~/Claude-Cowork-Workspace` (or wherever you like — just be consistent, and update paths in the templates if you deviate).

---

## 2. Fill Out the Bootstrap Questionnaire

Open `Bootstrap Questionnaire.md` and answer everything. Two sections matter most:

- **Section 7 (search keywords):** These 4–6 keyword sets drive ALL FOUR boards. Pick titles the way a recruiter would post them ("GRC Analyst", "Director of Engineering"), not the way you'd describe yourself. Each keyword set is searched locally AND remote on every board, so 5 keywords = up to 40 searches per day.
- **Section 2 (comp floor):** ZipRecruiter uses this to pre-filter at the source; the other boards use it in scoring.

---

## 3. First Cowork Session — Build the Workspace

1. Start a new Cowork session with your workspace folder mounted.
2. Share the completed questionnaire and the `templates/` folder.
3. Say: *"Please use my answers to create my about-me.md file and set up my workspace using the multi-board job search framework."*

Claude will create:

```
~/Claude-Cowork-Workspace/
├── CLAUDE.md                       (from templates/CLAUDE.md, personalized)
├── context/about-me.md             (from your questionnaire)
├── memory/working-memory.md
├── memory/TASKS.md
├── memory/context/job-pipeline.md
├── Resume/Hunt/Applied Companies/
├── Resume/Hunt/Versions/
├── Interview Prep/
└── outputs/
```

---

## 4. Base Resumes

The scan synthesizes a tailored resume from your "base persona" resumes — one per role archetype you're targeting. In your first session, ask Claude to build 1–3 base resumes from your about-me.md, for example:

- `[YourName]_Resume_Leadership.docx` — Director/VP-level framing
- `[YourName]_Resume_IC.docx` — senior individual-contributor framing

Review them carefully — these are the source of truth for every daily build. Every base resume must follow the ATS rules in CLAUDE.md (no Unicode bullets, no tables, no headers/footers).

---

## 5. Configure and Schedule the Daily Scan

1. Open `scheduled-task/SKILL.md` and replace every `[BRACKETED]` placeholder: your name, email, city, radius, keywords, scoring rubric, exclusions, comp floor.
2. Save it where your Cowork skills live, or paste it into a scheduled task prompt.
3. Tell Claude: *"Please set up the daily job scan to run at 8:00am."*

Claude creates a scheduled task that runs the skill every morning. The first run makes a good test — review the report and tighten your keywords or rubric based on what came back.

### Tuning tips (from real use)

- **Too much noise?** Tighten the exclusion list first, then the rubric. Dice especially needs the contract filter; ZipRecruiter matches loosely.
- **Too few results?** Broaden Indeed keywords (it returns zero on narrow multi-word queries), widen your radius, or add a keyword set.
- **Same role appearing everywhere?** That's the cross-board dedup working — check the `Sources` line. A role on 3+ boards with a staffing agency name is usually recruiter farming.
- **A connector broke?** The scan continues with remaining boards and flags the failure. Reconnect via Settings → Connectors.

---

## 6. Daily Rhythm

1. **Morning:** Read the scan report. For each 6+ role you like, open the resume Claude built, review it, and apply manually via the direct link.
2. **After applying:** Tell Claude — it updates `job-pipeline.md` and creates the company folder under `Applied Companies/`.
3. **Interview scheduled?** Tell Claude — it updates working-memory and can build an interview prep doc (company research, likely questions, your STAR stories).
4. **Rejection?** Log it. Cohort tracking will tell you if your materials need work.

---

## 7. Do Not Skip

- Claude **never applies for you** — every application is your click.
- **Verify each connector monthly** — board APIs change.
- **Update your comp floor and threshold** when your situation changes; posture switching (active ↔ passive) is one edit in about-me.md.
