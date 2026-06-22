# AGENTS.md — Daily Resume-Tailoring Pipeline

This file is the standing instruction set for the daily scheduled agent that tailors Roy
Cheng's resume to newly-posted jobs. Read this file in full at the start of every run, then
execute the pipeline below. Behavior is tuned by editing this file, `pipeline/sources.md`, and
the `knowledgebase/` — not by changing code.

**Repo:** `roycheng27/job-applications` · **Owner:** Roy (Yuanxi) Cheng · **Grad:** Mar 2027

---

## Inputs (read these every run)

- `pipeline/sources.md` — job source URLs + targeting filters + field→folder mapping. **Filters
  defined there are authoritative;** the summary in this file is a convenience copy.
- `pipeline/preferences.md` — **living preferences/feedback log. Read it every run and honor it.**
  Its entries **override** the defaults in this file and in `sources.md` wherever they conflict
  (except the hard rules: one-page and truthfulness always win — flag any conflict in the report).
  This is how Roy continuously tunes tailoring over time.
- `knowledgebase/00_INDEX.md` — the routing entrypoint for selecting which experiences to use.
- `knowledgebase/<experience>.md` — atomic experience files (load only the routed ones).
- `resume_base.tex` — the master template AND the gold-standard set of bullets. Never modify it.
- `pipeline/seen_jobs.json` — dedupe ledger of jobs already processed.

---

## Pipeline (run in order)

### 0. Load preferences
Read `pipeline/preferences.md` first. Apply its active preferences throughout this run; they
override defaults in this file and `sources.md` on conflict (hard rules excepted). If a
preference is ambiguous or conflicts with a hard rule, proceed sensibly and note it in the report.

### 1. Fetch sources
Fetch the reliable GitHub raw README URLs in `pipeline/sources.md`. Then attempt the
intern-list.com pages (best-effort; don't block the run if they return nothing). Log any
source that fails to fetch.

### 2. Parse
Extract rows into records: `{company, role, location, date, link, source}`. Parsing notes for the
GitHub tables (5 columns: Company / Role / Location / Application/Link / Date Posted):
- The **link** lives inside an `<a href="...">` wrapping an "Apply" image — extract the `href`.
  A bare `🔒` in that column means **closed → skip the row**.
- A company cell of `↳` means **same company as the row above** (continuation listing) — inherit
  the previous row's company name.
- Strip status emoji from the role: 🛂 (no sponsorship) and 🇺🇸 (citizenship) — note them on the
  record but don't auto-drop on them.

### 2b. Freshness (dedupe ledger, not a date cutoff)
Per `pipeline/sources.md`: "new" = the job key is **not** in `seen_jobs.json`. Do not apply a
hard recent-date window (sources update sparsely). `Date Posted` is used only for ranking.

### 3. Filter (per `pipeline/sources.md` targeting filters)
Drop a job unless ALL hold:
- **Fresh:** posted ≈ today / within last 1–2 days, AND its key is not in `seen_jobs.json`.
- **No required Master's:** drop if MS/PhD is a *required* qualification (a "preferred" Master's is fine).
- **Location** is in the allowlist (LA, SF Bay Area, Seattle, Chicago, DC/Arlington, NYC, Remote-US).
- **Role-relevant** to one of the four priority families.

### 4. Rank & cap
Sort survivors by role priority (DS/MLE > SWE > DA > other analyst), then by company renown
(larger / more recognized first). Take the top **15**.

### 5. Per selected job
For each job, in ranked order:

a. **Read the JD.** Fetch the application link (WebFetch; use WebSearch to fill in company/role
   context if the page is JS-gated or thin). Extract required skills, keywords, qualifications,
   and tech stack. If the JD reveals a *required* Master's, skip the job and log it.
   **Dead/inaccessible links are common** (ATS links expire or are JS-gated → 404/empty). If the
   JD can't be retrieved, fall back to WebSearch on "<company> <role>" + general knowledge of the
   role family to infer the likely skill/keyword set, and note in the report that the JD was
   inferred rather than read. Do not skip a good job solely because its link is dead.

b. **Classify** into a field folder using the Field→Folder mapping in `pipeline/sources.md`:
   `AI-MLE`, `SWE`, or `DA-DS`.

c. **Route the knowledge base.** Open `knowledgebase/00_INDEX.md`, find the role family, read its
   row in the Routing Table + the matching Quick-load set. Load **only** the 3–6 routed
   experience files (plus the INDEX). Respect the INDEX's exclusions.

d. **Tailor** a copy of `resume_base.tex` following the **Resume Editing Rules** below. Surface
   the JD's keywords/skills/tools by editing existing bullets, skills, and coursework first.

e. **Write the metadata comment block** at the very top of the tailored `.tex` (format below),
   accurately listing keywords added and every change made vs base.

f. **Save** to:
   - `applications/<field>/YuanxiCheng_<Company>_<Role>/resume.tex`
   - `applications/<field>/YuanxiCheng_<Company>_<Role>/job.md` (JD snapshot: link, source,
     location, date pulled, extracted qualifications/keywords).

   Sanitize `<Company>` and `<Role>` for filesystem safety: no spaces or `/` (use CamelCase or
   hyphens), strip punctuation. Example: `YuanxiCheng_Stripe_SoftwareEngineerIntern`.

g. **Record** the job in `seen_jobs.json` (see ledger format below).

### 6. Report
Write `pipeline/reports/daily_<YYYY-MM-DD>.md`: counts of jobs found / selected / skipped (with
skip reasons), the list of tailored resumes produced (company, role, field, link), any sources
that failed or knowledge-base gaps noticed, and — if relevant — **suggested preference updates**
for Roy to consider adding to `pipeline/preferences.md` (this closes the feedback loop).

### 7. Open one PR
Create a branch `tailor/<YYYY-MM-DD>`, commit all new files, push, and open **one PR**:
- Title: `Resume tailoring — <YYYY-MM-DD> (<N> jobs)`
- Body: the contents of the daily report.

**Do NOT compile PDFs** — Roy compiles locally with MacTeX. Never commit to `main` directly.

---

## Resume Editing Rules (hard constraints)

- **Preserve structure.** Sections stay largely intact. The ONLY allowed structural move: for
  **SWE** jobs, *Projects and Research* may be placed **above** *Work and Leadership Experience*.
  No other section reordering.
- **One page, always — safe by construction.** `resume_base.tex` is *perfectly full* (compiles to
  exactly one page with no slack), and the agent has **no LaTeX to page-check with**. So one-page
  safety must be guaranteed by the edit itself, not by compiling:
  - **Never add or remove bullets, experiences, or lines.** Keep the exact same item counts.
  - **Each edited bullet must be ≤ the character length of the bullet it replaces.** Shorter is
    safe; longer is forbidden (a single wrapped line spills to a 2nd page). Count characters.
  - Reordering coursework/skills items (same items, same text) is always length-safe.
  - Roy's **local compile is the final gate** (he compiles each PR's `.tex` with MacTeX before
    use); the agent's job is to make that compile come out at one page every time.
- **Prefer editing over replacing.** The experiences in `resume_base.tex` are the general
  strongest set. **Do not swap an experience** unless a job's qualifications make it clearly
  necessary; default to editing existing experiences/bullets to surface the right keywords.
- **Don't over-edit.** Leaving strong bullets/experiences unchanged is fine. Change only what
  improves alignment to the JD.
- **Bullet content — XYZ format.** Each bullet states *what was accomplished*, *the tool(s) used*,
  and *the result/metric*. One accomplishment per bullet, **max 2 tools**; never a laundry list
  of tools or accomplishments.
- **Bullet grammar.** Start with an **action verb**. **Past tense** everywhere except a *current*
  role (date contains "Present" — only **Capital One** qualifies). **End on a result / downstream
  implication** (to the company, or feeding the next bullet).
- **Bullet flow.** Read smoothly, minimal pauses/commas. Fewer commas = better.
- **Order by relevance.** Reorder *Relevant Coursework* and the *Technical Skills* lists so the
  items most related to the JD appear first.
- **Truthfulness.** Every bullet must trace to a `knowledgebase/` file or an existing base bullet.
  Never fabricate. If material is missing, note it in the daily report instead of inventing it.
- **Gold-standard exemplars.** The bullets already in `resume_base.tex` define what "strong"
  means here. Read them as few-shot examples and match their structure, voice, length, and
  metric density. When in doubt, mirror the existing bullets.

---

## Metadata comment block (top of every tailored `resume.tex`)

```latex
% === TAILORED RESUME METADATA ===
% Job:        <Role> @ <Company>
% Link:       <application URL>
% Source:     <vanshb03 / vanshb03-offseason / cvrve / intern-list>
% Field:      <AI-MLE | SWE | DA-DS>
% Tailored:   <YYYY-MM-DD>
% Keywords added: <comma-separated JD keywords surfaced>
% Changes vs base:
%   - Experience "<X>": bullets <2,3> rewritten to emphasize <theme>
%   - Swapped experience "<A>" -> "<B>" (only if necessary; cite knowledgebase file)
%   - Skills line reordered to lead with <skills>
%   - Coursework reordered to lead with <courses>
%   - (write "none" for any category not changed)
% ================================
```

---

## `seen_jobs.json` ledger format

Keyed by a stable job key `slug(company)|slug(role)|slug(location)`:

```json
{
  "stripe|software-engineer-intern|seattle-wa": {
    "company": "Stripe",
    "role": "Software Engineer Intern",
    "location": "Seattle, WA",
    "link": "https://...",
    "source": "vanshb03",
    "field": "SWE",
    "first_seen": "2026-06-22",
    "status": "tailored"
  }
}
```

Use `status: "skipped"` with a `reason` field for jobs evaluated but not tailored (e.g.
`"masters_required"`, `"location_out_of_region"`, `"cap_reached"`), so they aren't re-evaluated.

---

## Tuning this system over time

Behavior is entirely doc-driven — no code to change. To adjust how resumes get tailored:
- **`pipeline/preferences.md`** — day-to-day preferences & feedback (wording, bullet choices,
  experience selection, per-field notes). Read every run; overrides defaults. Edit this most.
- **`pipeline/sources.md`** — job sources, location allowlist, role priorities, daily cap.
- **`knowledgebase/`** — the experience bank. Add detail/metrics here for sharper, truthful bullets.
- **`AGENTS.md`** (this file) — the stable pipeline spec and hard rules. Change rarely.

Roy can edit these directly or just ask Claude to update them (e.g. "stop using 'leveraged'" →
Claude appends the rule to `preferences.md`). After reviewing each daily PR, fold corrections
back into `preferences.md` so quality compounds over time.
