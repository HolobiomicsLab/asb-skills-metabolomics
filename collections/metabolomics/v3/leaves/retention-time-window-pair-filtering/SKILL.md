---
name: retention-time-window-pair-filtering
description: Use when a non-targeted LC-MS peak table (e.g. MS-DIAL export) needs
  same-metabolite feature grouping, and you must constrain which feature pairs are
  considered correlated by a retention-time window before graph clustering.
when_to_use_negative:
- If the input is a cleaned, drift-corrected but already statistically-analyzed feature
  table and no same-metabolite feature grouping is desired — this skill only affects
  feature clustering, not downstream statistics.
- If the peak table lacks per-feature retention times (mz-only feature list), because
  the RT constraint cannot be evaluated.
- If you want clusters of features that may come from different metabolites with similar
  m/z (e.g. isobaric interferences) — the RT window is calibrated for same-metabolite
  co-elution, not for cross-metabolite separation.
edam_operation: http://edamontology.org/operation_3465
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3520
tools:
- name: notame
  role: Provides find_connections (correlation + RT pair filtering) and find_clusters
    (graph construction and connected-component extraction) as part of the same-metabolite
    clustering workflow.
  repo: https://github.com/antonvsdata/notame
- name: R
  role: Execution environment for the notame pipeline; peak tables are read with read_from_excel
    and processed with construct_metabosets.
- name: MS-DIAL
  role: Produces the raw peak table (Excel spreadsheet) that is the input to the clustering
    workflow.
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.3390/metabo10040135
    title: '“Notame”: Workflow for Non-Targeted LC–MS Metabolic Profiling'
  source_content_hash: sha256:3e9353d73d9d009e56a5ad9d42c42ee327388e7e52f2fb9776949e3fa31aa86a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/antonvsdata/notame
  tools:
  - MS-DIAL
  - R
  - liquid chromatography–mass spectrometry
  - notame
  edam_operation: http://edamontology.org/operation_3465
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# retention-time-window-pair-filtering

## Summary

Pins down the retention-time + correlation double filter that notame applies before clustering adduct/isotope/adduct-of-same-metabolite features into 'Cluster_x' groups, so that only features plausibly originating from the same metabolite are linked.

## When to use

Use when a non-targeted LC-MS peak table (e.g. MS-DIAL export) needs same-metabolite feature grouping, and you must constrain which feature pairs are considered correlated by a retention-time window before graph clustering.

## When NOT to use

- If the input is a cleaned, drift-corrected but already statistically-analyzed feature table and no same-metabolite feature grouping is desired — this skill only affects feature clustering, not downstream statistics.
- If the peak table lacks per-feature retention times (mz-only feature list), because the RT constraint cannot be evaluated.
- If you want clusters of features that may come from different metabolites with similar m/z (e.g. isobaric interferences) — the RT window is calibrated for same-metabolite co-elution, not for cross-metabolite separation.

## Inputs

- MetaboSet object built from an MS-DIAL-generated Excel peak table (e.g. sample_data_whole.xlsx), including feature abundance matrix and feature metadata (mz, RT)
- Pearson correlation threshold (article value: 0.9)
- Retention-time window for candidate pairs (article value: ~1 second)

## Outputs

- Undirected graph whose nodes are molecular features and whose edge weights are the retained Pearson correlations
- Connected components / candidate feature clusters ready for degree-based pruning (find_clusters) and Cluster_ID assignment (assign_cluster_id)

## How to apply

Read the preprocessed peak table into a MetaboSet via construct_metabosets, then run find_connections on the feature abundance matrix. Compute pairwise Pearson correlations between all features and retain only pairs whose Pearson coefficient exceeds the 0.9 threshold AND whose retention times differ by at most ~1 second, so each edge corresponds to a same-metabolite candidate rather than a coincidental cross-run correlation. Feed the retained pairs to find_clusters, which builds an undirected graph (nodes = features, edge weights = retained Pearson r values) and decomposes it into connected components. The rationale is that features originating from one metabolite co-elute (or nearly so) and track each other in intensity across samples; looser RT or correlation windows would merge unrelated co-detecting features and inflate cluster sizes. If the RT window must be adapted to a different chromatographic gradient, recompute find_connections with the new RT delta rather than post-filtering the graph.

## Related tools

- **notame** (Provides find_connections (correlation + RT pair filtering) and find_clusters (graph construction and connected-component extraction) as part of the same-metabolite clustering workflow.) — https://github.com/antonvsdata/notame
- **R** (Execution environment for the notame pipeline; peak tables are read with read_from_excel and processed with construct_metabosets.)
- **MS-DIAL** (Produces the raw peak table (Excel spreadsheet) that is the input to the clustering workflow.)

## Examples

```
Rscript -e 'library(notame); ms <- construct_metabosets(read_from_excel("sample_data_whole.xlsx"), split_by="pol", mode="neg"); conn <- find_connections(ms, correlation_cutoff = 0.9, rt_window = 1); clus <- find_clusters(conn)'
```

## Evaluation signals

- Every feature in the resulting cluster-membership table must carry a non-NA Cluster_ID of the form 'Cluster_x', including singletons.
- Within each 'Cluster_x', the representative (Cluster_ID) feature must be the member with the largest median peak area; verify by re-computing medians on the feature abundance matrix.
- All feature pairs linked inside a cluster must satisfy both constraints — Pearson r > 0.9 (or the chosen threshold) and RT delta ≤ the chosen window; any violating pair indicates find_connections parameters were loosened or the graph was built outside the intended pipeline.
- After degree-based pruning, every node that survives must meet the degree threshold (e.g. 0.8 of the maximum possible degree); discarded nodes should appear either in a new cluster or as singletons, not silently disappear.

## Limitations

- The clustering algorithm is based on MATLAB code by David Broadhurst and treats the graph as undirected, so edge direction (which feature is the adduct vs. the monoisotopic form) is not preserved; representative selection is decided by median peak area instead.
- The package API is still experimental and breaking changes are possible; find_connections/find_clusters signatures may change across releases.
- The RT window (±1 s) and correlation cutoff (0.9) were tuned for the authors' LC-MS setup; chromatographic gradients or MS-DIAL RT tolerances elsewhere may require re-tuning.
- Clustering only pairs features that are both detected and non-flagged in the abundance matrix — features removed earlier by flag_detection/flag_contaminants/flag_quality steps cannot be grouped.

## Evidence

- [other] The clustering mechanism constructs an undirected graph whose nodes are molecular features and whose edges are the correlation coefficients between features, subject to a retention-time constraint: [evidence span withheld — anchor status: not_found]
- [other] Pearson correlation with a correlation coefficient threshold of 0.9 and a retention-time window of +/-1 second: [evidence span withheld — anchor status: not_found]
- [readme] A novel method for clustering similar molecular features: [evidence span withheld — anchor status: external_doc]
- [readme] Data is stored in a custom object that holds all the information about the features and samples along with the feature abundance matrix: [evidence span withheld — anchor status: external_doc]
