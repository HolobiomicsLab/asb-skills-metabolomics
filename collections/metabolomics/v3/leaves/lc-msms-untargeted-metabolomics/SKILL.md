---
name: lc-msms-untargeted-metabolomics
description: Use when analyzing LC-MS/MS untargeted metabolomics feature tables with
  MS/MS spectra, where the goal is to group features by spectral similarity and statistically
  prioritize feature groups across sample conditions.
when_to_use_negative:
- Input is already a curated feature-cluster assignment or a completed differential-expression
  table with no MS/MS spectra available to recompute spectral similarity.
- The analysis requires targeted metabolite identification via spectral library matching
  rather than exploratory prioritization of structurally similar feature groups.
- Working on Windows, where the msFeaST pre-processing pipeline is still untested
  ('UNTESTED' per the README) and only the browser-based dashboard bundle is usable.
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: msFeaST
  role: Python-based workflow module that orchestrates preprocessing, k-medoids spectral
    clustering, globaltest feature-group testing, and dashboard JSON export for LC-MS/MS
    untargeted metabolomics.
  repo: https://github.com/kevinmildau/msFeaST
- name: globaltest
  role: R package (Bioconductor 3.18 / v5.56.0) used for feature group-wise statistical
    testing of the k-medoids feature clusters, detecting small concordant effects
    via joint modeling with reduced multiplicity adjustment penalties.
  repo: https://github.com/kevinmildau/msFeaST
provenance:
  source_task_ids:
  - task_002
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btae584
    title: Combined LC-MS/MS feature grouping, statistical prioritization, and interactive
      networking in msFeaST
  source_content_hash: sha256:779959e81357e1851dff059c50d97245a091020731aeafff950fd8549aa41a82
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kevinmildau/msFeaST
  tools:
  - globaltest
  - msFeaST
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# lc-msms-untargeted-metabolomics

## Summary

An LC-MS/MS untargeted metabolomics analysis workflow (msFeaST) that groups features into structurally similar clusters via k-medoids on spectral similarity matrices, prioritizes features and feature sets with group-wise globaltest statistics, and visualizes results in an interactive dashboard.

## When to use

Use when analyzing LC-MS/MS untargeted metabolomics feature tables with MS/MS spectra, where the goal is to group features by spectral similarity and statistically prioritize feature groups across sample conditions.

## When NOT to use

- Input is already a curated feature-cluster assignment or a completed differential-expression table with no MS/MS spectra available to recompute spectral similarity.
- The analysis requires targeted metabolite identification via spectral library matching rather than exploratory prioritization of structurally similar feature groups.
- Working on Windows, where the msFeaST pre-processing pipeline is still untested ('UNTESTED' per the README) and only the browser-based dashboard bundle is usable.

## Inputs

- LC-MS/MS quantification table of feature intensities across samples
- Sample group metadata table mapping samples to biological conditions (e.g. mushroom type or olive mill substrate contrast)
- MS/MS spectra associated with each LC-MS/MS feature (for spectral similarity computation)

## Outputs

- Pairwise spectral similarity matrix between features (e.g. modified cosine or MS2DeepScore scores)
- Per-feature k-medoids cluster assignments at the K maximizing mean Silhouette score, plus the Silhouette-score grid over K
- Differential-abundance statistics table with per-feature log-2 fold changes and group-wise globaltest statistics
- JSON file (dashboard_data.json) loadable in the msFeaST_Dashboard_bundle.html interactive visualization

## How to apply

First, load a quantification (feature intensity) table, sample group metadata, and associated MS/MS spectra into the msFeaST Python module (installed via pip from the kevinmildau/msFeaST GitHub repo into a conda environment with Python 3.10 and R 4.3.3). Compute a pairwise spectral similarity matrix between features — e.g. modified cosine score or MS2DeepScore — and run k-medoids clustering on that (dis)similarity matrix, selecting the number of clusters K by evaluating mean Silhouette scores over a grid of candidate K values and taking the maximum. Next, run feature group-wise statistical testing on the resulting feature clusters using the R globaltest package (version 5.56.0, installed via BiocManager from Bioconductor 3.18), which jointly models concordant effects across features in a group with reduced multiplicity adjustment penalties to boost power for small effects. Extract per-feature log-2 fold changes and group-wise test statistics into a differential-abundance table, and export a JSON file that the msFeaST_Dashboard_bundle.html interactive dashboard loads for exploratory visualization of experimental versus structural-similarity information.

## Related tools

- **msFeaST** (Python-based workflow module that orchestrates preprocessing, k-medoids spectral clustering, globaltest feature-group testing, and dashboard JSON export for LC-MS/MS untargeted metabolomics.) — https://github.com/kevinmildau/msFeaST
- **globaltest** (R package (Bioconductor 3.18 / v5.56.0) used for feature group-wise statistical testing of the k-medoids feature clusters, detecting small concordant effects via joint modeling with reduced multiplicity adjustment penalties.) — https://github.com/kevinmildau/msFeaST

## Examples

```
conda activate msfeast_environment && jupyter-notebook notebook/msfeast_pipeline_mushroom_type_comparison.ipynb
```

## Evaluation signals

- The Silhouette-score grid table contains one mean Silhouette score per evaluated K, and the selected K is the grid value with the highest mean Silhouette score.
- The final differential-abundance table has a per-feature log-2 fold-change column covering all analyzed features and a group-wise statistic per k-medoids cluster.
- The dashboard_data.json output loads successfully into msFeaST_Dashboard_bundle.html and the data-view tab populates with the loaded feature/cluster/statistics data.
- The R configuration check `rscript -e "catch <- lapply(c('dplyr', 'tibble', 'readr', 'listenv', 'BiocManager', 'globaltest'), library, character.only = TRUE); sessionInfo()"` returns globaltest_5.56.0, survival_3.5-8, and R version 4.3.3 without errors.
- Differential-intensity patterns in the output are qualitatively consistent with the deposited run — e.g. large differential intensity patterns producing wide-scale feature highlighting for the modified-cosine mushroom-type comparison.

## Limitations

- The pre-processing and pipeline workflow has been tested on macOS and should work identically on Linux; Windows support is currently being worked on and is untested.
- R installation via rscript may suffer from a cached faulty R path when run before the conda R installation — close the terminal, reopen, re-activate the msfeast_environment, and re-run rscript installs to avoid installing packages into the wrong R path.
- No additional numeric threshold is described in the paper for selecting the optimal k-medoids cluster count beyond maximizing the mean Silhouette score over the K grid.
- The Bioconductor 3.18 release pins globaltest to 5.56.0 and R to 4.3.3; older Matrix (1.6-5) and survival (3.5-8) versions must be installed explicitly as indirect requirements.

## Evidence

- [methods] msFeaST performs k-medoids clustering on the pairwise spectral distance matrix, and the user selects the number of clusters k based on Silhouette scores computed over a comprehensive grid of K values.: [evidence span withheld — anchor status: not_found]
- [abstract] spectral similarity-based feature groupings generated using k-medoids clustering, where the resulting clusters serve as a proxy for grouping structurally similar features: "spectral similarity-based feature groupings generated using k-medoids clustering, where the resulting clusters serve as a proxy for grouping structurally similar features"
- [readme] The current msFeaST pre-processing and pipeline workflow has been tested on macos and should work identically on linux operating systems. Windows support is currently being worked on.: [evidence span withheld — anchor status: external_doc]
- [methods] Run feature group-wise statistical testing on the feature clusters with the globaltest package, which provides high power to detect small concordant effects: [evidence span withheld — anchor status: not_found]
- [abstract] Spectral clustering done in this way allows for feature group-wise statistical testing using the globaltest package: "Spectral clustering done in this way allows for feature group-wise statistical testing using the globaltest package"
