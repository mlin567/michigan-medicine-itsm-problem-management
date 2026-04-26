# A Repeatable Pipeline for Identifying and Explaining High-Impact IT Problem Candidates

A three-stage analysis pipeline that separates expected IT service desk demand from signals requiring intervention, then uses NLP to explain what structured metadata alone cannot capture.

## Overview

Michigan Medicine's Health IT Services (HITS) processes ~5,000 tickets per week across 1,000+ services. This pipeline analyzes one year of ServiceNow incident records (~274K tickets) to identify where operational burden concentrates, detect temporal disruptions, and recover hidden issue types from free-text work notes when structured fields are too generic to explain a pattern.

## Pipeline Stages

**Stage 1 — Baseline & Workflow Noise.** Establishes what normal demand looks like: weekly aggregation per service, caller repetition analysis, and workflow cluster detection (same caller/service/subcategory within 72 hours). Outputs a cleaned ticket dataframe and canonical weekly time series.

**Stage 2 — Candidate Identification.** Produces a ranked candidate list from two independent arms (see Problem Candidates below). Candidates are annotated with structured field analysis comparing spike-window vs baseline distributions, and flagged with workflow cluster share.

**Stage 3 — NLP Explanation.** For candidates where structured fields are too generic ("Other," "Move/Change/Add"), extracts agent work notes, embeds with a sentence transformer (all-MiniLM-L6-v2), reduces with UMAP, clusters with HDBSCAN, and labels clusters with TF-IDF. NLP only runs on candidates from Stage 2 — no service-week pair outside the candidate list is clustered.

## Key Concepts

**Workflow clusters** group tickets from the same caller, service, and subcategory within 72 hours. High workflow cluster share in a candidate suggests the surge is driven by repeat submissions rather than independent user-reported problems. *(Stage 1)*

**Problem candidates** are identified from two independent perspectives: *persistent burden* (service–subcategory pairs that consume the most total resolution effort year-round) and *episodic disruptions* (service-weeks where volume or MTTR deviates significantly from that service's rolling baseline). A service can appear in both. *(Stage 2)*

**Impact score** = volume × median MTTR. Captures aggregate operational cost per service–subcategory pair, not incident frequency alone. *(Stage 2, persistent burden arm)*

**Spike detection** flags service-weeks where volume or MTTR exceeds a rolling median baseline by ≥1.5×, with absolute and relative excess floors to filter noise. *(Stage 2, episodic arm)*

**Lift** compares how a structured field value (e.g., a subcategory or assignment group) is distributed in a spike window versus baseline weeks. Lift > 1.5 means the value is overrepresented during the spike. *(Stage 3, structured analysis)*

**Structured sufficiency** — the pipeline checks whether structured fields already explain a candidate before running NLP. If a single category or assignment group dominates ≥70% of spike tickets with lift ≥1.5, NLP is unnecessary for that candidate. *(Stage 3, pre-NLP gate)*

## Outputs

All outputs are written to `pipeline_outputs/`:

| File | Description |
|------|-------------|
| `summary.html` | Self-contained dashboard — open in any browser |
| `candidate_list.csv` | All candidates from Stage 2 with type, severity, and metadata |
| `nlp_cluster_summaries.csv` | Per-cluster statistics for candidates where NLP ran |
| `structured_analysis.csv` | Structured field analysis results per candidate |
| `case_studies.json` | Full case study dicts for programmatic access |

## How to Run

**Prerequisites:** Python 3.9+, Jupyter. Key packages: `pandas`, `numpy`, `altair`, `statsmodels`, `sentence-transformers`, `umap-learn`, `hdbscan`, `scikit-learn`.

**Data:** Place the four quarterly `.xlsx` files in a `data/` directory alongside the notebook.

**Execution:** Run `chunk_abc_pipeline.ipynb` top to bottom. The final cell generates `pipeline_outputs/summary.html`.
