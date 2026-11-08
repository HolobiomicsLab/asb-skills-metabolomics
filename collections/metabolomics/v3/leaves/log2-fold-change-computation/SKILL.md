---
name: log2-fold-change-computation
description: Use when you have an LC-MS/MS feature abundance table with group metadata
  and need per-feature effect sizes (log-2 fold changes) to overlay with globaltest
  cluster-wise P-values for feature prioritization.
when_to_use_negative:
- Your goal is only cluster-wise statistical testing (globaltest P-values) without
  per-feature effect sizes — the k-medoids/globaltest path suffices.
- Your data has more than two groups per contrast without a specified pairwise comparison
  — log-2 fold change is defined for a two-group contrast like 0% vs 80% substrate.
- You want to run msFeaST only in the browser dashboard on pre-processed example files
  — the dashboard loads ready-made dashboard_data.json and does not require the fold-change
  computation step.
edam_operation: http://edamontology.org/operation_3663
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3520
tools:
- name: msFeaST
  role: Python/R workflow that computes per-feature log-2 fold changes between the
    0% and 80% substrate groups and combines them with globaltest cluster P-values
    for interactive dashboard prioritization
  repo: https://github.com/kevinmildau/msFeaST
- name: globaltest
  role: Bioconductor package used in the msFeaST pipeline to derive feature group-wise
    P-values that are written to modified_cosine_stats_table.csv alongside the log-2
    fold changes
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btae584
    title: Combined LC-MS/MS feature grouping, statistical prioritization, and interactive
      networking in msFeaST
  source_content_hash: sha256:4e7bbf894b124dc345e04736052bac6d54cd00b726a46f2ddc32b61a3ac2ccee
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kevinmildau/msFeaST
  tools:
  - globaltest
  - msFeaST
  edam_operation: http://edamontology.org/operation_3663
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# log2-fold-change-computation

## Summary

Computes per-LC-MS/MS-feature log-2 fold changes between sample groups (e.g. Pleurotus eryngii grown on 0% vs 80% olive mill solid waste) to pair semi-quantitative abundance shifts with msFeaST's spectral-similarity cluster P-values in the interactive dashboard.

## When to use

Use when you have an LC-MS/MS feature abundance table with group metadata and need per-feature effect sizes (log-2 fold changes) to overlay with globaltest cluster-wise P-values for feature prioritization.

## When NOT to use

- Your goal is only cluster-wise statistical testing (globaltest P-values) without per-feature effect sizes — the k-medoids/globaltest path suffices.
- Your data has more than two groups per contrast without a specified pairwise comparison — log-2 fold change is defined for a two-group contrast like 0% vs 80% substrate.
- You want to run msFeaST only in the browser dashboard on pre-processed example files — the dashboard loads ready-made dashboard_data.json and does not require the fold-change computation step.

## Inputs

- LC-MS/MS feature abundance/quantification table (per-feature intensities across samples)
- Sample group metadata table (e.g. 0% vs 80% olive mill solid waste substrate labels)
- MS/MS peak lists (spectral data, for the combined msFeaST workflow)
- Cluster-wise globaltest P-values (for pairing with fold changes in modified_cosine_stats_table.csv)

## Outputs

- Per-feature log-2 fold changes between sample groups
- modified_cosine_stats_table.csv combining cluster-wise P-values and per-feature log-2 fold changes
- dashboard_data.json for interactive exploration of fold changes with spectral similarity clusters

## How to apply

Load the deposited Pleurotus eryngii LC-MS/MS feature abundance table and sample group metadata (0% vs 80% olive mill solid waste substrate) into the msFeaST pipeline notebooks (preprocessing and msfeast_pipeline .ipynb), then compute the pairwise log-2 ratio of mean feature abundance between the two substrate groups. The msFeaST workflow produces these per-feature log-2 fold changes alongside the k-medoids spectral-similarity clusters and globaltest feature group-wise P-values, so the fold changes can be written into modified_cosine_stats_table.csv together with the cluster P-values. In the msFeaST dashboard, the log-2 fold changes serve as the semi-quantitative experimental layer that is interactively integrated with mass-spectral structural similarity information to enhance prioritization of features and feature sets during exploratory data analysis. Validate that the resulting statistics are comparable to the deposited additional run files reported in Section 3 of the article.

## Related tools

- **msFeaST** (Python/R workflow that computes per-feature log-2 fold changes between the 0% and 80% substrate groups and combines them with globaltest cluster P-values for interactive dashboard prioritization) — https://github.com/kevinmildau/msFeaST
- **globaltest** (Bioconductor package used in the msFeaST pipeline to derive feature group-wise P-values that are written to modified_cosine_stats_table.csv alongside the log-2 fold changes)

## Examples

```
conda activate msfeast_environment && jupyter-notebook   # then open preprocessing_mushroom_type_comparison.ipynb and msfeast_pipeline_mushroom_type_comparison.ipynb, edit data filepaths to the 0% vs 80% Pleurotus eryngii abundance/metadata tables, and run all cells
```

## Evaluation signals

- modified_cosine_stats_table.csv contains both cluster-wise P-values and per-feature log-2 fold changes
- The combination is comparable to the files of the additional deposited run reported in Section 3 of the article
- The dashboard (msFeaST_Dashboard_bundle.html) loads dashboard_data.json and shows the fold-change layer in the dataview tab
- The R configuration check returns the expected session info: globaltest_5.56.0, survival_3.5-8, readr_2.1.5, tibble_3.2.1, dplyr_1.1.4 under R 4.3.3
- Fold changes show a pattern of many elevated log-2 fold changes with fewer corresponding statistical enrichment patterns for the 0% vs 80% contrast

## Limitations

- The msFeaST pre-processing and pipeline workflow has been tested on macOS and should work identically on Linux; Windows support is currently being worked on.
- Three samples per group (the 0% vs 80% olive mill solid waste contrast) — small n limits statistical power; many features show elevated log-2 fold changes but fewer show statistical enrichment.
- R installation via rscript can cache the wrong R path if the terminal is reused; close the terminal, reopen, and re-activate the conda msfeast_environment before installing.
- Changing to your own data requires editing the data filepath arguments in the notebooks; magenta italics text in the notebooks highlights required user input.

## Evidence

- [other] Fold changes paired with cluster P-values: [evidence span withheld — anchor status: not_found]
- [abstract] Interactive integration of fold changes with spectral similarity: "msFeaST provides interactive integration of the semi-quantitative experimental information with mass-spectral structural similarity information, enhancing the prioritization of features and"
- [other] Deposited output artifact for the run: [evidence span withheld — anchor status: not_found]
- [readme] Notebooks are the entry point for the pipeline including fold-change computation: [evidence span withheld — anchor status: external_doc]
