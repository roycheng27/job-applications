# Job Sources & Targeting Filters

This file is the editable configuration for the daily tailoring pipeline. The scheduled agent
re-reads it every run, so tune behavior by editing here — no code changes needed.

---

## 1. Job Sources

### Reliable (markdown tables — fetch raw, always parseable)

| Source | Raw URL | Covers |
|---|---|---|
| vanshb03 — Summer 2027 Internships | `https://raw.githubusercontent.com/vanshb03/Summer2027-Internships/main/README.md` | Summer internships |
| vanshb03 — Off-season Internships | `https://raw.githubusercontent.com/vanshb03/Summer2027-Internships/main/OFFSEASON_README.md` | Winter / off-cycle internships |
| cvrve — New Grad | `https://raw.githubusercontent.com/cvrve/New-Grad/main/README.md` | New-grad full-time roles |

Table columns: Company / Role / Location / Date (Age). Markers: 🔒 = closed (skip), 🛂 = no
sponsorship (note but do not auto-skip), 🇺🇸 = citizenship required (note).

> If a raw URL 404s (repo renamed default branch or moved files), try `master` instead of
> `main`, then fall back to the repo's GitHub web README. Log any source that fails to fetch.

### Best-effort (JavaScript-rendered — static fetch returns only a shell)

- `https://www.intern-list.com/?k=aiml` (AI / ML)
- `https://www.intern-list.com/?k=da` (Data Analysis)
- `https://www.intern-list.com/?k=swe` (SWE)

Attempt these and use whatever rows come back, but **do not block the run** if they yield
nothing — the GitHub tables are the dependable backbone.

---

## 2. Targeting Filters (applied during selection)

**Freshness — the dedupe ledger is the real signal.** "Newly posted" = **not already in
`seen_jobs.json`**. The first run treats everything as new (process up to the daily cap, newest
`Date Posted` first); each later run only processes jobs that have appeared since. Do NOT use a
hard "posted in the last 1–2 days" cutoff — these source repos update sparsely (e.g. the newest
batch may be weeks old, especially early in a season), so a tight date window often yields zero
jobs. Use `Date Posted` only for **ranking/tie-breaking** (newer first), not as a filter.

**Hard excludes — drop the job if any apply:**
- Master's / MS / PhD listed as a **required** qualification (a "preferred"/"plus" Master's is OK).
- Listing marked closed (🔒).
- Location not in the allowlist below.

**Location allowlist (US only):**
- Los Angeles, CA (+ greater LA / Orange County)
- SF Bay Area, CA (San Francisco, South Bay, Peninsula, Oakland)
- Seattle, WA
- Chicago, IL
- Washington, DC / Arlington, VA (DMV)
- New York City, NY
- Remote (US)

**Company preference:** Prefer larger / more renowned companies when ranking.

**Role priority order (highest first):**
1. Data Scientist / Machine Learning Engineer (→ `AI-MLE`)
2. Software Engineer (→ `SWE`)
3. Data Analyst (→ `DA-DS`)
4. Other analyst roles (Business / Product / Quant analyst, etc.)

**Timeline match:** new-grad roles (Roy graduates Mar 2027), summer internships, and winter /
off-cycle internships.

**Daily cap:** up to **15** newly-posted matching jobs per run (ranked by role priority, then
company renown).

---

## 3. Field → Folder Mapping

| Field folder | Role families routed here |
|---|---|
| `AI-MLE` | Data Scientist, ML Engineer, AI Engineer, Applied Scientist, Research |
| `SWE`    | Software Engineer, Data Engineer, full-stack/backend/ML-infra |
| `DA-DS`  | Data Analyst, Business Analyst, Product Analyst, Quant/other analyst |
