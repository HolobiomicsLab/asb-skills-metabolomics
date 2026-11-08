---
name: fdr-qvalue-rank-assignment
description: Use when a feature table already carries FDR-adjusted p-values (q-values)
  from feature-wise tests plus multivariate importance ranks (e.g., MUVR RF 'mid'
  model), and the goal is to pick the most robust metabolic features for downstream
  identification.
when_to_use_negative:
- Only univariate results exist (no multivariate importance ranks available), so one
  of the two ranks cannot be computed
- Data is still raw or preprocessed LC–MS output (e.g., MS-DIAL peak table) that has
  not yet passed feature-wise statistical testing
- You need a formal rank-aggregation algorithm (e.g., Borda, RRA) rather than the
  simple two-rank sum used here
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: notame
  role: R package whose feature-wise statistical results and feature metadata feed
    the q-value ranking; used to assemble the feature table for aggregation
  repo: https://github.com/hanhineva-lab/notame
- name: MUVR
  role: Supplies the multivariate (random-forest 'mid' model) feature importance ranks
    that are summed with the q-value ranks
- name: R
  role: Environment in which the rank computation, joining, sorting and CSV export
    are performed
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.3390/metabo10040135
    title: '“Notame”: Workflow for Non-Targeted LC–MS Metabolic Profiling'
  source_content_hash: sha256:d9fa1e473f9644b47af824fbfede136118c1d672c7a3e67f8c8b7297dabd60b3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hanhineva-lab/notame
  tools:
  - MUVR
  - R
  - notame
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# fdr-qvalue-rank-assignment

## Summary

A rank-aggregation step in the notame workflow that combines each molecular feature's feature-wise rank (sorted by lowest FDR q-value) with its multivariate importance rank (e.g., from a MUVR random-forest 'mid' model) into a single 'Final Ranks' column used to select the most robust LC–MS features for metabolite identification.

## When to use

Use when a feature table already carries FDR-adjusted p-values (q-values) from feature-wise tests plus multivariate importance ranks (e.g., MUVR RF 'mid' model), and the goal is to pick the most robust metabolic features for downstream identification.

## When NOT to use

- Only univariate results exist (no multivariate importance ranks available), so one of the two ranks cannot be computed
- Data is still raw or preprocessed LC–MS output (e.g., MS-DIAL peak table) that has not yet passed feature-wise statistical testing
- You need a formal rank-aggregation algorithm (e.g., Borda, RRA) rather than the simple two-rank sum used here

## Inputs

- Per-feature statistics table with FDR-adjusted p-values (q-values) from the feature-wise analysis step
- Multivariate feature importance ranks from MUVR random-forest modeling (e.g., RF 'mid' model)

## Outputs

- Selection table sorted ascending by Final Ranks, saved as results_selection.csv

## How to apply

Load the per-feature statistics table containing FDR-adjusted p-values (q-values) produced by the feature-wise statistical-analysis step (e.g., via clean_stats_results) and the MUVR multivariate importance ranks from the random-forest modeling step. For each feature, compute its rank position in the q-value-sorted ordering and its rank position in the importance-sorted ordering, then sum the two ranks per feature into a new 'Final Ranks' column. Sort the selection table ascending by Final Ranks and save it to results_selection.csv. The rationale is that a feature only reaches a top position if it is significant in both the feature-wise tests and the multivariate model, making this a simple robustness filter before metabolite identification.

## Related tools

- **notame** (R package whose feature-wise statistical results and feature metadata feed the q-value ranking; used to assemble the feature table for aggregation) — https://github.com/hanhineva-lab/notame
- **MUVR** (Supplies the multivariate (random-forest 'mid' model) feature importance ranks that are summed with the q-value ranks)
- **R** (Environment in which the rank computation, joining, sorting and CSV export are performed)

## Examples

```
sel <- join_fData(stats, importance); sel$Final_Ranks <- rank(sel$q_value) + rank(sel$RF_rank); write.csv(sel[order(sel$Final_Ranks),], "results_selection.csv")
```

## Evaluation signals

- The output table contains a 'Final Ranks' column
- Every feature row has both a q-value rank and an importance rank contributing to its Final Ranks value
- Rows in results_selection.csv are ordered by ascending Final Ranks
- Features passing the FDR threshold occupy top ranks in the final ordering
- Final Ranks value equals the sum of the two component ranks for each feature (recomputable invariant)

## Limitations

- The notame package API is still quite experimental and breaking changes are possible
- The original repository (antonvsdata/notame) is archived and unmigrated; active development has moved to hanhineva-lab/notame
- The aggregation depends on the choice of multivariate model (e.g., MUVR RF 'mid'); a different model or rank basis changes the selection

## Evidence

- [readme] A bunch of statistical analyses, both feature-wise tests and multivariate models: [evidence span withheld — anchor status: external_doc]
- [abstract] we introduce notame, an analytical workflow for non-targeted metabolic profiling approaches: "we introduce notame, an analytical workflow for non-targeted metabolic profiling approaches"
- [intro] This package can be used to analyze preprocessed LC-MS data in non-targeted metabolomics: "This package can be used to analyze preprocessed LC-MS data in non-targeted metabolomics"
