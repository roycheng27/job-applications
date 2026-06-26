# Tailoring Preferences & Feedback (living document)

This is the **evolving** preferences log for the resume-tailoring pipeline. Edit it freely and
often — the daily agent reads it at the start of **every** run and treats it as authoritative.
Use it to record what you like, what to avoid, and corrections after reviewing a day's PRs.

> Precedence: these preferences **override** the defaults in `AGENTS.md` and `pipeline/sources.md`
> wherever they conflict. If a preference contradicts a hard rule (1-page, truthfulness), the
> hard rule still wins — but flag the conflict in the daily report.

> How to update: edit this file directly, OR just tell Claude (e.g. "from now on, lead SWE
> resumes with the Biomedical project" / "stop using the word 'leveraged'") and it will append
> the rule here. Keep entries short and specific.

---

## Active preferences

### Wording & tone
- _(none yet — e.g. "avoid the verb 'leveraged'", "prefer 'built' over 'developed'")_

### Line budgets (strict — any violation causes a 2-page resume)
Every element in the resume has a fixed rendered-line count. Never exceed these:
- Awards & Programs → **1 line**
- Certificates → **1 line**
- Relevant Coursework → **exactly 2 lines** (highest risk; see coursework rule below)
- Each Work/Leadership experience → **5 lines** (company/location header + role/date subheader + 3 x 1-line bullets)
- Each Projects/Research entry → **4 lines** (header + 3 x 1-line bullets)
- Languages / Libraries / Tools & Platforms → **1 line each**

### Relevant Coursework reorder — character-boundary check
Reordering the 10 coursework items (same text, different order) changes line-break positions and
can inflate 2 lines → 3 lines (proved in the first cloud run: all SWE resumes overflowed because
of this). Rule: promote only 1–3 relevant items to the front; keep the rest in base order. The
safe base split (line 1 ≈ 97 chars) is: **line 1** = "Machine Learning, Neural Networks & Deep
Learning, Data Analysis & Regression, Data Mining, Data Structures & Algorithms" · **line 2** =
"Probability & Statistical Inference, Databases, NLP, Monte Carlo, Optimization". Swapping 1–2
front items with their base counterparts is safe; reshuffling all 10 requires a char-count check.

### Bullet & content choices
- _(none yet — e.g. "for DS roles, always surface a quantified metric in bullet 1")_

### Experience / project selection
- _(none yet — e.g. "for SWE, swap Million Dollar Baby out for the Biomedical project")_

### Skills & coursework emphasis
- _(none yet — e.g. "always list SQL before Python for analyst roles")_

### Job selection & filtering
- _(none yet — e.g. "deprioritize finance/trading firms", "include Boston in locations")_

### Per-field notes
- **AI-MLE:** _(none yet)_
- **SWE:** _(none yet)_
- **DA-DS:** _(none yet)_

---

## Change log
Record dated decisions so the rationale isn't lost.

- **2026-06-22** — Created preferences log. Initial behavior defined in `AGENTS.md`.
- **2026-06-26** — Added per-element line budgets + coursework char-boundary check after first
  cloud run produced 10 × 2-page SWE resumes. Root cause: coursework reorder shifted long items
  to line boundaries, inflating 2 rendered lines → 3.
