# HR-PSA-SKILLS-MAPPING-MVP — Comprehensive Project Summary

**Repository:** `cof-sandbox/HR-PSA-SKILLS-MAPPING-MVP`
**Status:** Phase 1 complete (BA pilot); Phases 2-3 planned

---

## What It Does

A Capital One HR pipeline that maps workforce performance management (PM) reviews to **O*NET Detailed Work Activities (DWAs)** — a standardized U.S. labor taxonomy — to produce a ranked, empirically-grounded skills menu per job family.

**Central question:** *What work activities actually show up in the performance reviews of a given job family?*

The output is a recruiter-facing ranked CSV of DWAs that can:
- Populate preferred-qualifications lists
- Inform org design decisions.
- Surface AI-specific skills gaps

**BA Pilot Results:** 1,459 reviews processed → 31 ranked DWAs produced. The pipeline is designed to generalize to any Capital One job family.

---

## Tools & Libraries

### Open-Source Python Stack

| Library | Role |
|---|---|
| `sentence-transformers` | Local embedding model loader (`BAAI/bge-base-en`) |
| `faiss-cpu` | Vector similarity search (cosine) over O*NET tasks via normalized inner product |
| `pandas` / `numpy` | Data manipulation and array math |
| `spacy` + `spacy-transformers` + `en_core_web_sm` | Post-hoc citation validation — lemmatization, sentence splitting, NER |
| `tenacity` | Retry logic with exponential backoff for LLM calls |
| `PyYAML` | Config and prompt file loading |
| `snowflake-connector-python[pandas]` | Snowflake data ingestion |
| `requests` | HTTP client for anonymizer API |
| `torch` | CUDA device detection for embedding model |
| `jupyter` | Notebook-based prompt tuning, evaluation, and debugging |
| `pytest` | Unit and integration testing |

### Capital One Internal Packages (Runtime-Provided)

| Package | Role |
|---|---|
| `c1.genai.integrations.langchain.inference` | Enterprise Model Platform LangChain wrapper (`create_chat_openai_model`) |
| `c1.aiml.huggingface` | Internal HuggingFace model downloader |

---

## AI / LLM Stack

### Primary LLM

**Model:** `gpt-oss-120b` via Capital One's Enterprise Model Platform.

Used in two distinct pipeline stages:

#### Stage 4a — Denoiser (`src/preprocessing/denoiser.py`)
Compresses raw PM reviews (~1,000 words / ~1,300 tokens) down to 150-200 tokens of dense task evidence.

- Strips values language, behavioral feedback, and work attributed to others
- The "anti-shadowing" constraint prevents inflating a reviewer's implied skill set
- System prompt defined in `src/config/prompts.yaml`

#### Stage 4b — DWA Matcher (`src/matching/dwa_matcher.py`)
Given a denoised review and FAISS-retrieved candidate DWAs, the LLM:

- Assigns evidence scores (1-5 scale) per DWA
- Returns verbatim snippets from the review as citations
- Provides confidence levels (`"High"` / `"Medium"` / `"Low"`) and reasoning
- Returns `"NO_MATCH"` or `"Custom DWA"` rather than forcing a match (anti-hallucination guardrail)
- Every returned verbatim snippet is validated post-hoc via substring matching or spaCy-backed semantic fuzzy matching

### Rate Limiting
Custom `EnterpriseLLMClient` class with:
- Sliding-window token throttling: 320k input / 80k output tokens per 10-minute window
- Per-reservation token accounting
- Retry-after-aware backoff

### Embedding Model
**`BAAI/bge-base-en`** (~130MB, local, offline). Loaded via `sentence-transformers` for both FAISS index construction and semantic search. Queries use the required BAAI prefix: `"Represent this sentence for searching relevant passages: "`.

### Anonymization
All review text is passed through Capital One's internal PII anonymizer API (`api.cloud.capitalone.com/internal-operation/people-data-platform/anomymize-employee-data`) via OAuth2 client credentials **before** any LLM processing. Raw PII review text is never sent to the LLM.

---

## Architecture — 6-Stage Pipeline

```
Stage 0   Anchor Derivation                                          (~5 sec, cached)
          Job family prose description
          → BGE embeddings
          → cosine score against all 923 O*NET occupations
          → adaptive top-K anchors (≥ 0.65 cosine threshold, max 8)
          Output: derived_anchors.yaml

Stage 1   SOC Major-Group Filter                                     (~5 sec, cached)
          ~923 occupations → ~140
          Hardcoded SOC major-group allowlist per job family (e.g., "11","13","15")

Stage 2   Hierarchical SOC Filter                                    (~5 sec, cached)
          ~140 occupations → ~57
          Widens automatically if below min_survivors threshold

Stage 3   Task→DWA Roll-up                                           (~5 sec, cached)
          ~57 occupations → ~200-305 candidate DWAs
          Via O*NET Tasks-to-DWAs join table

Stage 4   Empirical Grounding                                        (~2-3 hrs for BA)
          Reviews loaded from Snowflake (or CSV fallback)
          → Anonymized via internal PII API
          → Denoised by LLM (1,300 → 200 tokens)
          → FAISS top-K DWA candidates retrieved
          → LLM DWA matcher assigns scores + verbatim citations
          → Per-DWA prevalence + average evidence score computed
          → DWAs below 1% prevalence floor dropped

Stage 5   Composite Ranking                                          (instant)
          score = prevalence × avg_evidence_score
          Output: ranked_dwas.csv
```

**Stages 0-3** complete in ~5 seconds total (embeddings cached after first run).
**Stage 4** is the computational bottleneck (~2-3 hours for the 1,459-review BA pilot).

---

## Data Sources

| Source | Contents |
|---|---|
| `HRDW_DB.PHDP_HR_API_PERFNC_REVW_TXT.PL_WORKR_PERFNC_REVW_CMNT` | Raw PM review text (Snowflake) — filtered to `2025 Year-end Performance Review` |
| `hrdw_db_proc.prd_util.lp_prod_eom_vw` | Employee metadata (job family, level) |
| `PL_WORKR_BC` / `PL_JOB_PRFL_BC` | Job family and level classification |
| `data/raw/onet.csv` | O*NET 27.2 Tasks export (local) |
| `data/raw/tasks_to_dwas.csv` | O*NET Tasks-to-DWAs join table (local) |

---

## Key Files

| File | Role |
|---|---|
| `src/job_family_pruning/run.py` | Main CLI orchestrator — wires all stages, handles DI for testing |
| `src/clients/llm_client.py` | `EnterpriseLLMClient` — LLM invocation, rate limiting, retry |
| `src/preprocessing/denoiser.py` | LLM-backed review compression (Stage 4a) |
| `src/matching/dwa_matcher.py` | LLM-backed DWA evidence matcher + citation validator (Stage 4b) |
| `src/vector_store/index_builder.py` | FAISS index construction and persistence over O*NET tasks |
| `src/vector_store/search_engine.py` | Semantic search against FAISS index |
| `src/job_family_pruning/anchor_derivation.py` | Stage 0: BGE-based O*NET occupation ranking |
| `src/job_family_pruning/empirical_grounding.py` | Stage 4: full review-grounding loop with no-match capture |
| `src/job_family_pruning/ranker.py` | Stage 5: composite scoring + output writers |
| `src/ingestion/snowflake_reviews_loader.py` | Snowflake query, dynamic filter injection, anonymization |
| `src/ingestion/anonymizer_client.py` | OAuth2 client for Capital One PII anonymizer API |
| `src/config/prompts.yaml` | System + user prompts for denoiser and DWA matcher |
| `src/config/llm_config.yaml` | LLM platform, model, rate limits, embedding model config |
| `src/config/app_config.json` | Snowflake connection + query parameters (job family, level, rating filters) |
| `sql/load_reviews.sql` | Snowflake query with `{{DYNAMIC_FILTER_CLAUSE}}` injection |
| `src/job_family_pruning/config/business_analyst.yaml` | Per-family config: SOC groups, anchor thresholds, prevalence floor |
| `data/raw/business_analyst_description.txt` | 5-paragraph prose description of the BA job family (pipeline input) |
| `ARCHITECTURE_DECISIONS.MD` | 23-entry Architecture Decision Record (ADR) document |
| `docs/augmentation-avenues.md` | Product roadmap for Phases 2 & 3 |
| `scripts/sample_gold_set.py` | Stratified gold-set sampling script for prompt evaluation |
| `notebooks/` | 5 notebooks: prompt tuning, RAG evaluation, final eval, gold set labeling, Snowflake debug |

---

## Three Planned Product Phases

### Phase 1 — Job-Family DWA Skills Menu (Complete)
Bottom-up, review-grounded skills menu per job family. BA pilot: 31 DWAs ranked from 1,459 reviews. Generalizes to any Capital One job family by swapping the prose description and SOC allowlist.

### Phase 2 — Conditional Probability Skills Menu (Planned)
Compute `P(skill_j | skill_i)` co-occurrence matrix across the DWA output to separate:
- **Baseline skills** — table-stakes present across nearly all reviews
- **Specialization-path differentiators** — skills that cluster with specific other skills

### Phase 3 — AI Task Augmentation / GenAI Skills Taxonomy (Planned)
Bottom-up induction of a GenAI skills taxonomy from the `NO_MATCH` and `Custom DWA` records — cases where O*NET has no existing vocabulary. These are precisely the emerging AI work activities that the standard taxonomy hasn't yet captured.

---

## Design Principles (from ARCHITECTURE_DECISIONS.MD)

- **Empirical grounding over expert elicitation** — skills menus derived from actual review evidence, not SME opinion
- **Anti-hallucination by design** — `NO_MATCH` is a valid and valued output; citation validation enforced post-hoc
- **PII-first architecture** — anonymization is a hard gate before any LLM call, not an optional step
- **O*NET as vocabulary, not oracle** — the taxonomy scopes the search space; the reviews are the evidence
- **Stages 0-3 as cheap filters** — semantic and hierarchical pruning reduces the LLM's candidate space from 923 occupations to ~200-305 DWAs before any token is spent
