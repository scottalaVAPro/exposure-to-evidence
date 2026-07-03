# exposure-to-evidence
# Exposure-to-Evidence

**An AI evidence engine that maps military environmental exposures to disease mechanisms.**

Built for [Built with Claude: Life Sciences](https://cerebralvalley.ai/e/built-with-claude-life-sciences) (July 2026), using **Claude Science** and **Claude Code**.

---

## The Problem

The PACT Act era produced a flood of peer-reviewed research linking military environmental exposures — burn pits, PFAS, jet and diesel fuels, herbicides, particulate matter — to specific diseases. That evidence is real, growing, and scattered across thousands of papers, dozens of databases, and inconsistent terminology.

The people who need it most can't synthesize it fast enough:

- **Researchers** re-derive the same exposure-outcome literature reviews from scratch.
- **Clinicians** lack a fast way to check whether a patient's exposure history has documented disease associations.
- **Veteran advocates** need to connect a specific exposure to a specific condition, with citations, under time pressure.

There is no structured, queryable bridge between *exposure* and *documented disease mechanism*. This project builds one.

## The Solution

**Exposure-to-Evidence** is a pipeline that ingests biomedical literature on military exposures, extracts causal and quantitative claims into a structured evidence database, and exposes that database through a queryable interface.

Literature in → structured evidence out → queryable by exposure or by condition.

The architecture mirrors a validated Claude Science use case: multi-agent pipelines that read thousands of papers, extract central claims and quantitative findings, and store them in an evidence database. This project applies that pattern to a domain — military environmental medicine — where the domain knowledge is scarce and the human impact is high.

---

## Architecture

```
                    ┌─────────────────────────────────────────┐
                    │            SOURCE LAYER                  │
                    │  PubMed · PMC · toxicology databases ·   │
                    │  govt exposure registries                │
                    └───────────────────┬─────────────────────┘
                                        │  (search + retrieval, MCP)
                                        ▼
                    ┌─────────────────────────────────────────┐
                    │          INGESTION AGENT                 │
                    │  Query construction, dedup, full-text    │
                    │  retrieval, metadata normalization       │
                    └───────────────────┬─────────────────────┘
                                        │
                                        ▼
                    ┌─────────────────────────────────────────┐
                    │         EXTRACTION AGENTS                │
                    │  Per-paper: pull exposure, condition,    │
                    │  mechanism, effect size, study type,     │
                    │  confidence — into a typed schema        │
                    └───────────────────┬─────────────────────┘
                                        │
                                        ▼
                    ┌─────────────────────────────────────────┐
                    │        EVIDENCE DATABASE                 │
                    │  Normalized exposure→condition edges,    │
                    │  each with citations + quantitative      │
                    │  findings + provenance                   │
                    └───────────────────┬─────────────────────┘
                                        │
                                        ▼
                    ┌─────────────────────────────────────────┐
                    │           QUERY LAYER                    │
                    │  Ask by exposure or by condition;        │
                    │  returns ranked, cited evidence          │
                    └─────────────────────────────────────────┘
```

**Orchestration** is handled by Claude Code driving Claude Science agents; **retrieval** runs through MCP connections to literature sources; **extraction** uses structured-output schemas so every claim lands as typed data, not prose.

---

## Pipeline Stages

**1. Ingestion.** An agent constructs targeted queries against PubMed/PMC and toxicology sources for a given exposure class, retrieves full text where available, deduplicates, and normalizes metadata (authors, year, study design, journal).

**2. Extraction.** Per-paper agents read each source and extract into a fixed schema:

| Field | Description |
|---|---|
| `exposure` | Normalized exposure agent (e.g. "particulate matter — burn pit") |
| `condition` | Normalized disease/outcome (mapped to standard vocabulary) |
| `mechanism` | Described biological pathway, if stated |
| `effect_size` | Quantitative finding (OR, RR, HR, incidence) where reported |
| `study_type` | Cohort, case-control, meta-analysis, review, etc. |
| `confidence` | Extraction confidence + whether claim is causal or associative |
| `citation` | Full provenance back to the source sentence |

**3. Aggregation.** Extracted claims are merged into exposure→condition edges. Where multiple papers support an edge, findings are grouped so a query returns the weight of evidence, not a single hit. Conflicting findings are preserved and flagged rather than collapsed.

**4. Query.** The interface accepts queries in two directions:
- *By exposure:* "What conditions are associated with PFAS exposure?" → ranked conditions with citations and effect sizes.
- *By condition:* "What exposures are linked to bladder cancer?" → ranked exposures with the same.

Every answer is **auditable** — each edge traces back to specific source sentences.

---

## Example Queries

```
> exposures linked to bladder cancer

Bladder cancer — associated exposures (12 papers):
  1. JP-8 / jet fuel        — 4 studies, 2 cohort · OR range 1.4–2.1
  2. Diesel exhaust         — 3 studies, 1 meta-analysis · RR 1.3
  3. Aromatic amines        — 5 studies · well-established mechanism
  [each result expandable to citations + extracted findings]
```

```
> conditions associated with burn pit particulate exposure

Burn pit particulate — associated conditions (23 papers):
  1. Constrictive bronchiolitis   — mechanism documented
  2. Asthma (new-onset)           — 6 studies · incidence elevated
  3. Chronic rhinosinusitis       — 4 studies
  [ranked by evidence weight; associative vs causal flagged]
```

```
> compare evidence: PFAS → thyroid vs PFAS → kidney

Side-by-side evidence summary with study counts, effect sizes,
and strength-of-evidence notes for each pathway.
```

---

## Why This Matters

The evidence to connect exposure to disease already exists in the literature. What's missing is a structured, fast, citable way to traverse it. Exposure-to-Evidence turns a scattered body of research into a queryable evidence base — accelerating literature review for researchers, supporting clinical decision-making, and giving advocates a defensible, cited bridge from a veteran's exposure history to documented disease associations.

---

## Built With

- **Claude Science** — multi-agent research workbench for ingestion, extraction, and the evidence database
- **Claude Code** — pipeline orchestration and the query layer
- **MCP** — connections to PubMed, PMC, and toxicology data sources

## Status

Hackathon build — July 7–13, 2026. Roadmap:
- [ ] Ingestion agent + source connections
- [ ] Extraction schema + per-paper agents
- [ ] Evidence database + aggregation
- [ ] Query interface (by exposure / by condition)
- [ ] Demo: end-to-end on one exposure class

## Author

**Scott Mollette** — developer, inventor, and filmmaker. Founder of Blue Meridian Pictures. Patent holder in signal-processing technology (BlackVoid). U.S. Army veteran (101st / 82nd Airborne) with 20+ years working directly with veterans' medical evidence and disability claims.

---

*This project handles published literature only — no patient data, no PHI — keeping it clear of privacy and compliance constraints while remaining directly useful to the people who need this evidence.*
