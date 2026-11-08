---
name: rank-sum-selection-scoring
description: Use when you have both feature-wise test results with FDR-adjusted p-values
  and multivariate model (e.g., RF 'mid' model) importance ranks, and you need one
  ordered shortlist of robust features for metabolite identification.
when_to_use_negative:
- Only one line of evidence exists (e.g., feature-wise FDR results but no multivariate
  importance ranks) — a single-rank sort, not rank-sum scoring, is appropriate
- Input is raw LC-MS vendor data or a peak table not yet preprocessed and statistically
  analyzed; notame expects preprocessed LC-MS data before statistical selection
edam_operation: http://edamontology.org/operation_0336
edam_topics:
- http://edamontology.org/topic_3517
tools:
- name: notame
  role: Provides the feature-wise statistical tests and the MetaboSet feature data
    (fData) that supply the q-value half of the rank sum
  repo: https://github.com/hanhineva-lab/notame
- name: MUVR
  role: Supplies multivariate importance ranks (e.g., from the RF 'mid' model) that
    form the second half of the rank sum
- name: R
  role: Implementation language for computing ranks, summing them into Final Ranks,
    sorting, and writing results_selection.csv
  repo: https://github.com/antonvsdata/notame
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.3390/metabo10040135
    title: '“Notame”: Workflow for Non-Targeted LC–MS Metabolic Profiling'
  source_content_hash: sha256:977e239bbb1dd925261555b7a0b89882f50c4fb138a76890ed1cf66d26b4157c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hanhineva-lab/notame
  tools:
  - MUVR
  - R
  - notame
  edam_operation: http://edamontology.org/operation_0336
  edam_topics:
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# rank-sum-selection-scoring

## Summary

Combines per-feature statistical significance (FDR q-value rank) with multivariate model importance rank (e.g., MUVR random forest) by summing the two ranks into a single 'Final Ranks' column, so the most robust LC-MS metabolic features can be prioritized for identification.

## When to use

Use when you have both feature-wise test results with FDR-adjusted p-values and multivariate model (e.g., RF 'mid' model) importance ranks, and you need one ordered shortlist of robust features for metabolite identification.

## When NOT to use

- Only one line of evidence exists (e.g., feature-wise FDR results but no multivariate importance ranks) — a single-rank sort, not rank-sum scoring, is appropriate
- Input is raw LC-MS vendor data or a peak table not yet preprocessed and statistically analyzed; notame expects preprocessed LC-MS data before statistical selection

## Inputs

- Per-feature statistics table with FDR-adjusted p-values (q-values) from the notame statistical-analysis step (e.g., feature-wise test results)
- Multivariate importance ranks from MUVR modeling (e.g., RF 'mid' model variable importance per feature)

## Outputs

- Selection table with a 'Final Ranks' column (sum of q-value rank and importance rank), sorted ascending by Final Ranks
- results_selection.csv with ranked features for downstream identification

## How to apply

Load the per-feature statistics table from the notame statistical-analysis step and the MUVR multivariate importance ranks, joining them per molecular feature (e.g., via join_fData on the MetaboSet feature data). For each feature, compute its rank position in the q-value-sorted ordering (lowest FDR q-value first) and its rank position in the importance-sorted ordering, then sum the two ranks into a 'Final Ranks' column. Sort the selection table ascending by Final Ranks and write it to results_selection.csv. The rationale is that features robust across both independent lines of evidence — univariate testing and multivariate modeling — occupy low summed ranks, so the selection is not driven by a single statistical lens.

## Related tools

- **notame** (Provides the feature-wise statistical tests and the MetaboSet feature data (fData) that supply the q-value half of the rank sum) — https://github.com/hanhineva-lab/notame
- **MUVR** (Supplies multivariate importance ranks (e.g., from the RF 'mid' model) that form the second half of the rank sum)
- **R** (Implementation language for computing ranks, summing them into Final Ranks, sorting, and writing results_selection.csv) — https://github.com/antonvsdata/notame

## Examples

```
sel$Final_Ranks <- rank(sel$P_FDR) + sel$importance_rank; sel <- sel[order(sel$Final_Ranks),]; write.csv(sel, "results_selection.csv")
```

## Evaluation signals

- The output table contains a 'Final Ranks' column populated for every feature row
- Every feature has both a q-value rank (from FDR-adjusted p-values) and an importance rank (from the multivariate model) contributing to its Final Ranks value
- Rows are ordered ascending by Final Ranks
- Features passing the FDR threshold occupy top Final Ranks positions
- results_selection.csv is saved and matches the sorted in-memory selection table

## Limitations

- Requires both univariate and multivariate result tables to be available and joinable per feature; missing importance ranks for any feature break the scoring
- The notame package API is experimental with possible breaking changes (the antonvsdata/notame repository is archived and migrated to hanhineva-lab/notame)
- Equal weighting of the two ranks is implicit in the sum; no per-source weighting scheme is described in the workflow

## Evidence

- [readme] A bunch of statistical analyses, both feature-wise tests and multivariate models: [evidence span withheld — anchor status: external_doc]
- [abstract] we introduce notame, an analytical workflow for non-targeted metabolic profiling approaches: "we introduce notame, an analytical workflow for non-targeted metabolic profiling approaches"
- [intro] This package can be used to analyze preprocessed LC-MS data in non-targeted metabolomics: "This package can be used to analyze preprocessed LC-MS data in non-targeted metabolomics"
- [readme] We then use the obtained results in identification of the actual metabolites.: [evidence span withheld — anchor status: external_doc]
