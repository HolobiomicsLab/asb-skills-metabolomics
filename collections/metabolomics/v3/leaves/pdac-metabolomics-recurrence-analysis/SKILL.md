---
name: pdac-metabolomics-recurrence-analysis
description: Use when input data are per-study aggregate statistics (p-values and
  fold-changes) for metabolites across multiple PDAC studies, and the goal is to rank
  compounds by recurrence with consistent up/down trend directions rather than pooled
  effect sizes.
when_to_use_negative:
- Input is raw metabolomics spectra or a feature table rather than already-published
  aggregate statistics (p-values and fold-changes) — amanida is designed for aggregate
  data meta-analysis
- A quantitative pooled effect-size meta-analysis is required; vote-counting is a
  qualitative method that discards magnitude information beyond trend direction
- The input dataset is not metabolomics compound-level data (e.g., genomics expression
  studies where compound naming harmonization via webchem does not apply, despite
  aggregate-statistics support)
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: amanida
  role: R package implementing the qualitative vote-counting meta-analysis (+1 up
    / -1 down per study) on aggregate p-value and fold-change data
  repo: https://github.com/mariallr/easy-amanida
- name: Easy-Amanida
  role: Shiny web app wrapping amanida and webchem, providing the workflow to optimize
    naming combination and run the meta-analysis
  repo: https://github.com/mariallr/easy-amanida
- name: webchem
  role: R package used for compounds naming harmonization (synonym resolution) before
    vote aggregation
- name: Shiny
  role: R package add-on for interactive web apps in which Easy-Amanida is implemented
- name: R
  role: Language in which the vote-counting pipeline is run to produce vote_counting_results.csv
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/jrsm.1713
    title: 'Easy‐Amanida: An R Shiny application for the meta‐analysis of aggregate
      results in clinical metabolomics using Amanida and Webchem'
  source_content_hash: sha256:b7d84c6d737126eea4aee22ec6456d764d7d9e874ec1d7bfbf1d64140ce4346e
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/mariallr/easy-amanida
  repo_url: https://github.com/mariallr/easy-amanida
  tools:
  - Easy-Amanida
  - R
  - Shiny
  - amanida
  - webchem
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# pdac-metabolomics-recurrence-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Qualitative vote-counting meta-analysis of pancreatic cancer (PDAC) metabolomics aggregate statistics (p-values and fold-changes) using the amanida R package / Easy-Amanida Shiny app, to identify metabolites recurrently and consistently dysregulated across studies.

## When to use

Use when input data are per-study aggregate statistics (p-values and fold-changes) for metabolites across multiple PDAC studies, and the goal is to rank compounds by recurrence with consistent up/down trend directions rather than pooled effect sizes.

## When NOT to use

- Input is raw metabolomics spectra or a feature table rather than already-published aggregate statistics (p-values and fold-changes) — amanida is designed for aggregate data meta-analysis
- A quantitative pooled effect-size meta-analysis is required; vote-counting is a qualitative method that discards magnitude information beyond trend direction
- The input dataset is not metabolomics compound-level data (e.g., genomics expression studies where compound naming harmonization via webchem does not apply, despite aggregate-statistics support)

## Inputs

- PDAC metabolomics dataset of aggregate statistics per study (p-values and fold-changes per compound), loadable from the easy-amanida GitHub repository
- Compound name lists requiring harmonization (synonym resolution via webchem)

## Outputs

- vote_counting_results.csv — summarized vote-counting table with per-compound recurrence counts and trend directions
- Tally of consistent hits: 8 up-regulated and 12 down-regulated significant compounds after removal of 15 inconsistent-trend metabolites

## How to apply

Load the PDAC dataset of aggregate statistics (p-values and fold-changes per study) from the easy-amanida GitHub repository into R, harmonizing compound names to resolve synonyms before analysis. Apply the amanida qualitative vote-counting method, assigning +1 to up-regulated and -1 to down-regulated compounds in each study, then aggregate votes across studies to compute the number of studies in which each metabolite is recurrently identified with a consistent trend. Flag the 15 metabolites with inconsistent regulation trends across studies and exclude them from the significant-compound tally. Summarize the consistent hits — glutamine (7 studies, down-regulated), histidine (5 studies, down-regulated), and the three recurrent up-regulated bile acids tauroursodeoxycholic, taurocholic, and glycocholic acid (3 studies each) — totaling 8 up-regulated and 12 down-regulated significant compounds, and write the table to vote_counting_results.csv.

## Related tools

- **amanida** (R package implementing the qualitative vote-counting meta-analysis (+1 up / -1 down per study) on aggregate p-value and fold-change data) — https://github.com/mariallr/easy-amanida
- **Easy-Amanida** (Shiny web app wrapping amanida and webchem, providing the workflow to optimize naming combination and run the meta-analysis) — https://github.com/mariallr/easy-amanida
- **webchem** (R package used for compounds naming harmonization (synonym resolution) before vote aggregation)
- **Shiny** (R package add-on for interactive web apps in which Easy-Amanida is implemented)
- **R** (Language in which the vote-counting pipeline is run to produce vote_counting_results.csv)

## Examples

```
library(amanida); results <- vote_counts(pdac_data); write.csv(results, "vote_counting_results.csv")
```

## Evaluation signals

- Glutamine appears in 7 studies, consistently down-regulated
- Histidine appears in 5 studies, consistently down-regulated
- Tauroursodeoxycholic, taurocholic, and glycocholic acid each recur in 3 studies as up-regulated
- Exactly 8 up-regulated and 12 down-regulated significant compounds remain after excluding the 15 inconsistent-trend metabolites
- vote_counting_results.csv exists and its per-compound study counts match the reported tallies

## Limitations

- Metabolites with inconsistent regulation trends across studies (15 in the PDAC dataset) must be excluded, which discards potentially real effect heterogeneity
- Meta-analysis in metabolomics is hindered by the wide variety of compounds, differing experimental methods, lack of consensus on statistical reporting, and compound naming synonyms
- Vote-counting uses only trend direction per study, not effect magnitudes or within-study uncertainty

## Evidence
