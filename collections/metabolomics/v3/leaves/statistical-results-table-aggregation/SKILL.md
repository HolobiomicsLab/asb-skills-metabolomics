---
name: statistical-results-table-aggregation
description: Use when both per-feature statistics with FDR q-values and a multivariate
  importance ranking (e.g., MUVR RF 'mid' model ranks) exist for the same features,
  and a single combined ordering is needed to prioritize features for metabolite identification.
when_to_use_negative:
- Only one ranking source exists (e.g., feature-wise q-values without any multivariate
  model) — a summed rank cannot be computed.
- The downstream task requires strict statistical significance cutoffs alone rather
  than a combined robustness ordering.
- Input is still raw or preprocessed LC-MS data that has not yet gone through feature-wise
  testing and MUVR modeling.
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_2269
tools:
- name: notame
  role: R package whose statistical-analysis output tables feed the aggregation; provides
    helpers for joining feature data (e.g., join_fData, clean_stats_results)
  repo: https://github.com/hanhineva-lab/notame
- name: MUVR
  role: Supplies the multivariate importance ranks that are summed with the feature-wise
    q-value ranks
- name: R
  role: Implementation language for computing per-feature ranks, summing them into
    Final Ranks, and writing results_selection.csv
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.3390/metabo10040135
    title: '“Notame”: Workflow for Non-Targeted LC–MS Metabolic Profiling'
  source_content_hash: sha256:d0b07e742aa9c5916a7475636e1161f0f134985ee363e84797a72fa498748471
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hanhineva-lab/notame
  tools:
  - MUVR
  - R
  - clean_stats_results
  - notame
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# statistical-results-table-aggregation

## Summary

Combines feature-wise statistical results (ranked by FDR-adjusted q-values) with multivariate model importance ranks (e.g., MUVR RF) by summing the two ranks per feature into a single 'Final Ranks' column, so the most robust LC-MS metabolic features can be selected for identification.

## When to use

Use when both per-feature statistics with FDR q-values and a multivariate importance ranking (e.g., MUVR RF 'mid' model ranks) exist for the same features, and a single combined ordering is needed to prioritize features for metabolite identification.

## When NOT to use

- Only one ranking source exists (e.g., feature-wise q-values without any multivariate model) — a summed rank cannot be computed.
- The downstream task requires strict statistical significance cutoffs alone rather than a combined robustness ordering.
- Input is still raw or preprocessed LC-MS data that has not yet gone through feature-wise testing and MUVR modeling.

## Inputs

- per-feature statistics table with FDR-adjusted p-values (q-values) from feature-wise tests
- MUVR multivariate importance ranks (e.g., RF 'mid' model) for the same features

## Outputs

- selection table with a Final Ranks column, sorted ascending by Final Ranks
- results_selection.csv used to select features for identification

## How to apply

Load the per-feature statistics table containing FDR-adjusted p-values (q-values) produced in the statistical-analysis step and the MUVR multivariate importance ranks from the modeling step, joining them per feature (e.g., with notame helpers such as clean_stats_results and join_fData in R). For each feature, compute its rank position in the q-value-sorted ordering and its rank position in the importance-sorted ordering, then sum the two ranks into a 'Final Ranks' column. Sort the selection table ascending by Final Ranks — lower summed rank means the feature is robust across both the feature-wise and multivariate evidence. Save the sorted table as results_selection.csv and use it to drive the metabolite identification step, prioritizing features that also pass the FDR threshold.

## Related tools

- **notame** (R package whose statistical-analysis output tables feed the aggregation; provides helpers for joining feature data (e.g., join_fData, clean_stats_results)) — https://github.com/hanhineva-lab/notame
- **MUVR** (Supplies the multivariate importance ranks that are summed with the feature-wise q-value ranks)
- **R** (Implementation language for computing per-feature ranks, summing them into Final Ranks, and writing results_selection.csv)

## Evaluation signals

- The output table contains a Final Ranks column and every feature row has both a q-value rank and an importance rank contributing to it.
- Rows are ordered by ascending Final Ranks.
- Features passing the FDR threshold occupy top ranks in the selection table.
- Final Ranks values equal the sum of each feature's q-value rank and importance rank (spot-checkable).
- The sorted table is persisted as results_selection.csv.

## Limitations

- Requires both feature-wise and multivariate results for the same feature set; one-sided inputs break the aggregation.
- Ties in q-values or importance ranks are not addressed by the described procedure, which may affect rank stability.
- The notame package API is experimental with possible breaking changes, and the original antonvsdata/notame repository is archived (development moved to hanhineva-lab/notame).

## Evidence

- [readme] feature-wise and multivariate analysis are combined: [evidence span withheld — anchor status: external_doc]
- [readme] results feed metabolite identification: [evidence span withheld — anchor status: external_doc]
- [abstract] notame as analytical workflow for metabolic profiling: "we introduce notame, an analytical workflow for non-targeted metabolic profiling approaches"
