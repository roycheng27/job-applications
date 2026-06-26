# CLAUDE.md — job-applications repo

## What this repo does

Automated daily pipeline that scrapes newly-posted internship/new-grad jobs, tailors a copy of
Roy's LaTeX resume to each job, compiles a PDF, and opens a single daily PR for review.

**Owner:** Roy (Yuanxi) Cheng · UCLA Statistics & Data Science · Grad: Mar 2027  
**Routine:** scheduled cloud agent, runs daily at 5 PM ET · ID: `trig_01LwQBRnBNGQV9wsxLXZDkcY`  
**Manage routine:** https://claude.ai/code/routines/trig_01LwQBRnBNGQV9wsxLXZDkcY

---

## Key files

| File | Role |
|---|---|
| `resume_base.tex` | Master template — **never modify**; also the gold-standard for bullet quality |
| `AGENTS.md` | Full pipeline spec + hard editing rules — the cloud agent reads this every run |
| `pipeline/preferences.md` | **Edit this to tune behavior.** Read every run; overrides AGENTS.md defaults |
| `pipeline/sources.md` | Job source URLs, location allowlist, role priorities, daily cap (currently 7) |
| `pipeline/seen_jobs.json` | Dedupe ledger — a job is "new" only if its key is absent here |
| `pipeline/reports/` | Daily run summaries written by the agent |
| `knowledgebase/00_INDEX.md` | Router for the experience bank — start here when the agent selects bullets |
| `knowledgebase/<name>.md` | Atomic experience files (16 total); agent loads only the routed subset |
| `applications/AI-MLE/` | Tailored outputs for DS/ML/AI roles |
| `applications/SWE/` | Tailored outputs for software engineering roles |
| `applications/DA-DS/` | Tailored outputs for data/business analyst roles |

Each job output folder contains: `resume.tex` (tailored copy + metadata comment block at top),
`resume.pdf` (compiled by the agent), and `job.md` (JD snapshot + qualifications).

---

## How to tune behavior

The system is **entirely doc-driven** — no code to change.

- **Day-to-day preferences / feedback** → edit `pipeline/preferences.md`. This is the right place
  for wording rules, bullet choices, experience selection notes, per-field instructions.
- **Sources, locations, cap, role priorities** → edit `pipeline/sources.md`.
- **Experience bank** → edit files in `knowledgebase/`. Richer files = better tailoring.
- **Pipeline spec / hard rules** → edit `AGENTS.md`. Change rarely.

After editing any of these, push to `main` — the cloud agent pulls from `main` on each run.  
The routine's baked-in prompt also encodes critical rules; update it via the RemoteTrigger API
if the AGENTS.md changes are structural (cap, PDF step, report format, etc.).

---

## Critical one-page rules (easiest thing to get wrong)

`resume_base.tex` compiles to **exactly one page with zero slack**. The cloud agent has no LaTeX,
so safety is by construction. These are the allowed rendered-line counts per element:

| Element | Lines |
|---|---|
| Awards & Programs | 1 |
| Certificates | 1 |
| Relevant Coursework | **exactly 2** ← highest risk |
| Each Work/Leadership entry | 5 (company+loc header · role+date subheader · 3 bullets) |
| Each Projects/Research entry | 4 (header · 3 bullets) |
| Languages / Libraries / Tools rows | 1 each |

**Bullet edits:** each edited bullet must be ≤ the character count of the bullet it replaces.

**Coursework reorder:** reordering the same 10 items changes where line breaks fall — 2 lines can
become 3. Safe approach: promote only 1–3 items to the front; keep the rest in base order. The
confirmed base split at ≈97 chars/line is:  
- Line 1: `Machine Learning, Neural Networks & Deep Learning, Data Analysis & Regression, Data Mining, Data Structures & Algorithms`  
- Line 2: `Probability & Statistical Inference, Databases, NLP, Monte Carlo, Optimization`

---

## PR / output workflow

- Agent branches `tailor/<YYYY-MM-DD>`, commits all new files, opens **one PR per day**.
- Each PR body is the daily report: summary table (Company / Role / Location / Type / Source /
  Field / Link / PDF ✅❌), counts, source health, KB gaps, suggested preference updates.
- **Roy compiles nothing** — agent runs `latexmk` in the cloud and commits `resume.pdf`.
  Build artifacts (`.aux`, `.log`, etc.) are cleaned before commit.
- `resume_base.tex` and `.pdf` (root level) are gitignored. `applications/**/resume.pdf` is tracked.
- Review PRs, merge what you want to apply, close the rest.

---

## Job sources

| Source | Type assigned | URL |
|---|---|---|
| vanshb03 Summer2027-Internships README | `summer internship` | raw GitHub |
| vanshb03 OFFSEASON_README | `off season internship` | raw GitHub |
| cvrve/New-Grad README | `new grad` | raw GitHub |
| intern-list.com (aiml/da/swe) | best-effort (JS-rendered) | — |

Freshness = not in `seen_jobs.json` (no date cutoff — sources update sparsely).  
Filters: no required Master's; locations LA / SF Bay / Seattle / Chicago / DC-Arlington / NYC / Remote-US.  
Role priority: DS/MLE > SWE > DA > other analyst. Cap: 7/day.

---

## Common tasks

**Add a preference** (e.g. "avoid the word leveraged"):
→ Append to `pipeline/preferences.md` under the relevant section, push to main.

**Add a location or change the cap**:
→ Edit `pipeline/sources.md`, push to main, then update the routine prompt via RemoteTrigger
  if the cap is referenced there.

**Expand an experience for richer tailoring**:
→ Edit the relevant `knowledgebase/<name>.md` file (fill in metrics, extra bullets).

**Trigger a run now**:
→ Use `RemoteTrigger {action: "run", trigger_id: "trig_01LwQBRnBNGQV9wsxLXZDkcY"}`.

**Routine auto-disabled itself** (ended_reason: auto_disabled_repo_access):
→ Re-enable: `RemoteTrigger {action: "update", trigger_id: "...", body: {enabled: true}}`.
  Then check GitHub App repo access at https://claude.ai/code/onboarding?magic=github-app-setup.

**Compile a tailored resume locally** (verify page count after editing):
→ `cd applications/<field>/<folder> && latexmk -pdf resume.tex`  
  MacTeX is installed at `/Library/TeX/texbin/latexmk`.
