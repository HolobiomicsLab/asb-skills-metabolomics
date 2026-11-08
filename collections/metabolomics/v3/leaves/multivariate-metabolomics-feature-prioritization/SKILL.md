---
name: multivariate-metabolomics-feature-prioritization
description: Use when both feature-wise tests (with FDR-adjusted q-values) and a multivariate
  importance model (e.g., MUVR RF ranks) have already been produced for the same feature
  table, and the goal is to prioritize features for metabolite identification.
when_to_use_negative:
- Input data has not yet passed notame preprocessing/QC (drift correction, flagging
  of low-quality features, random forest imputation, batch effect correction) — rank
  aggregation assumes a cleaned, analyzed feature table
- No multivariate model importance ranks exist yet (e.g., MUVR was not run); summing
  ranks is then impossible and univariate FDR ranking alone should be used instead
- The goal is exploratory interpretation or visualization of statistical results rather
  than producing a prioritized selection list for metabolite identification
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: notame
  role: R package providing the feature-wise statistical analyses whose q-value rankings
    feed the Final Ranks aggregation
  repo: https://github.com/hanhineva-lab/notame
- name: MUVR
  role: Supplies the multivariate (random forest 'mid' model) feature importance ranks
    that are summed with the q-value ranks
- name: R
  role: Runtime environment in which the rank computation, joining and sorting of
    the selection table are performed
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.3390/metabo10040135
    title: '“Notame”: Workflow for Non-Targeted LC–MS Metabolic Profiling'
  source_content_hash: sha256:27c85043fb1b940c79c82a467099300287491bb0e74e339f904712a6f10a7fef
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hanhineva-lab/notame
  tools:
  - MUVR
  - R
  - notame
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# multivariate-metabolomics-feature-prioritization

## Summary

Combines per-feature univariate statistical evidence (FDR-adjusted p-values/q-values) with multivariate model importance ranks (e.g., MUVR random forest 'mid' model) into a single 'Final Ranks' ordering used to select the most robust metabolic features for downstream identification.

## When to use

Use when both feature-wise tests (with FDR-adjusted q-values) and a multivariate importance model (e.g., MUVR RF ranks) have already been produced for the same feature table, and the goal is to prioritize features for metabolite identification.

## When NOT to use

- Input data has not yet passed notame preprocessing/QC (drift correction, flagging of low-quality features, random forest imputation, batch effect correction) — rank aggregation assumes a cleaned, analyzed feature table
- No multivariate model importance ranks exist yet (e.g., MUVR was not run); summing ranks is then impossible and univariate FDR ranking alone should be used instead
- The goal is exploratory interpretation or visualization of statistical results rather than producing a prioritized selection list for metabolite identification

## Inputs

- Per-feature statistics table with FDR-adjusted p-values (q-values) from the notame feature-wise statistical analysis step
- Multivariate feature importance ranks from the MUVR modeling step (e.g., RF 'mid' model rankings)

## Outputs

- results_selection.csv: feature selection table with a 'Final Ranks' column (sum of q-value rank and importance rank), sorted ascending by Final Ranks

## How to apply

Load the per-feature statistics table from the statistical-analysis step, which contains FDR-adjusted p-values (q-values), together with the multivariate feature importance ranks from the MUVR modeling step. For each feature, compute its rank position in the q-value-sorted ordering and its rank position in the importance-sorted ordering (e.g., from the RF 'mid' model). Sum the two ranks per feature into a new 'Final Ranks' column, so that features must perform well on both univariate significance and multivariate importance to rise to the top. Sort the selection table ascending by Final Ranks and save it as results_selection.csv. The rationale, per the notame workflow, is that features used in metabolite identification should be those the statistical analysis is most confident about under both lenses, since the results of this step feed directly into identification of the actual metabolites.

## Related tools

- **notame** (R package providing the feature-wise statistical analyses whose q-value rankings feed the Final Ranks aggregation) — https://github.com/hanhineva-lab/notame
- **MUVR** (Supplies the multivariate (random forest 'mid' model) feature importance ranks that are summed with the q-value ranks)
- **R** (Runtime environment in which the rank computation, joining and sorting of the selection table are performed)

## Evaluation signals

- The output table contains a 'Final Ranks' column
- Every feature row has both a q-value rank and an importance rank contributing to its Final Ranks value (no missing component ranks)
- Rows are ordered by ascending Final Ranks
- Features passing the FDR threshold occupy the top ranks of the sorted table
- Final Ranks value equals the sum of the feature's q-value rank and its importance rank (spot-check a few features)

## Limitations

- The notame package API is 'still quite experimental, and breaking changes are possible', so function names used in this step may change between versions
- The original repository (antonvsdata/notame) is archived and no longer maintained; active development moved to hanhineva-lab/notame
- Requires both upstream result sets (univariate q-values and multivariate importance ranks) for the same feature identifiers; joining failures silently drop features from the selection table

## Evidence

- [readme] A bunch of statistical analyses, both feature-wise tests and multivariate models: [evidence span withheld — anchor status: external_doc]
- [full_text] After feature-wise and multivariate analysis, each feature's multivariate (e.g., RF 'mid' model) rank is added to its feature-wise rank (sorted by lowest FDR q-value) to create a new Final Ranks column used to sort and select features.: [evidence span withheld — anchor status: not_found]
- [readme] We then use the obtained results in identification of the actual metabolites.: [evidence span withheld — anchor status: external_doc]
- [abstract] We provide an overview of lab protocols and statistical methods that we commonly practice for the analysis of nutritional metabolomics data.: "We provide an overview of lab protocols and statistical methods that we commonly practice for the analysis of nutritional metabolomics data."
