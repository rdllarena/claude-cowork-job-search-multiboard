# Claude Cowork - Global Startup Instructions

Mount ~/Claude-Cowork-Workspace/CLAUDE.md every time.

## Mandatory Session Startup Sequence

At the start of EVERY session, before doing anything else:

1. Read ~/Claude-Cowork-Workspace/context/about-me.md — my career history, skills, and preferences
2. Read ~/Claude-Cowork-Workspace/memory/working-memory.md — current job search status, active applications
3. Read ~/Claude-Cowork-Workspace/memory/TASKS.md — open action items
4. Read ~/Claude-Cowork-Workspace/memory/context/job-pipeline.md — full application pipeline

Then confirm startup complete: "Context loaded. [brief status summary]"

## Core Rules

- My comp floor is $[YOUR_COMP_FLOOR]. Never recommend applying to roles below this.
- My geography: [YOUR_LOCATION] — willing to commute [X] miles for on-site/hybrid.
- Remote roles: fully eligible nationwide.
- Do NOT apply to any roles on my behalf — scan, score, and generate materials only.
- Job boards: LinkedIn (via Claude in Chrome) + Indeed, Dice, ZipRecruiter (via official MCP connectors). If a connector is disconnected, tell me — don't silently skip a board.
- Do NOT modify job-pipeline.md during automated tasks — I update the pipeline manually.
- Always use my personal email: [YOUR_EMAIL]
- Never use my work email on any job document.

## Resume Building Rules

- Font: Arial, 10pt body, 12pt section headers (navy #1F3864)
- Page: US Letter, 0.75" margins
- Bullets: use Word BULLET numbering config (never unicode characters)
- Contact line: [YOUR_PHONE] | [YOUR_EMAIL] | linkedin.com/in/[YOUR_LINKEDIN]
- Output path: ~/Claude-Cowork-Workspace/Resume/[YourName]_Resume_{Company}_{RoleAbbrev}.docx
- After building, always validate with the docx validation script
- Cover letters for roles scoring 8+/10 only

## Memory Update Protocol

After each meaningful session, update these files:
- working-memory.md — any new applications, interviews scheduled, offers, decisions
- TASKS.md — complete finished tasks, add new ones
- job-pipeline.md — add new applications, update statuses
