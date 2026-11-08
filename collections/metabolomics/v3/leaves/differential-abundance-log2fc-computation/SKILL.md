---
name: differential-abundance-log2fc-computation
description: Use when preprocessed LC-MS/MS feature intensity tables and MS/MS spectra
  must be statistically compared across sample groups (e.g. mushroom type), with per-feature
  log-2 fold changes extracted after spectral-similarity clustering.
when_to_use_negative:
- Input is raw vendor MS data or spectra that have not yet been preprocessed into
  a feature intensity table plus metadata table
- The goal is structural annotation or spectral library matching rather than group-wise
  differential testing of pre-clustered features
- Clusters of structurally similar features have not yet been formed — globaltest
  group-wise testing in msFeaST expects k-medoids spectral-similarity clusters as
  its feature sets
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: msFeaST
  role: 'Orchestrates the pipeline: preprocessing notebooks, k-medoids clustering,
    R-based group-wise testing, and dashboard visualization of differential intensity
    patterns'
  repo: https://github.com/kevinmildau/msFeaST
- name: globaltest
  role: Performs the feature group-wise statistical testing on spectral-similarity
    clusters and yields the group-wise test statistics combined with per-feature log-2
    fold changes
  repo: https://github.com/kevinmildau/msFeaST
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btae584
    title: Combined LC-MS/MS feature grouping, statistical prioritization, and interactive
      networking in msFeaST
  source_content_hash: sha256:0f294743a816fc216fd4ea66adaeb9d395ff69d7e49b5acfd6146835a07cd949
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kevinmildau/msFeaST
  tools:
  - globaltest
  - msFeaST
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# differential-abundance-log2fc-computation

## Summary

Group-wise differential-abundance testing of LC-MS/MS feature clusters using the R globaltest package inside the msFeaST workflow, extracting group-wise test statistics together with per-feature log-2 fold changes into a differential-abundance statistics table for prioritization.

## When to use

Use when preprocessed LC-MS/MS feature intensity tables and MS/MS spectra must be statistically compared across sample groups (e.g. mushroom type), with per-feature log-2 fold changes extracted after spectral-similarity clustering.

## When NOT to use

- Input is raw vendor MS data or spectra that have not yet been preprocessed into a feature intensity table plus metadata table
- The goal is structural annotation or spectral library matching rather than group-wise differential testing of pre-clustered features
- Clusters of structurally similar features have not yet been formed — globaltest group-wise testing in msFeaST expects k-medoids spectral-similarity clusters as its feature sets

## Inputs

- Preprocessed LC-MS/MS feature intensity (quantification) table
- Associated MS/MS spectra per feature
- Sample group metadata table (e.g. mushroom-type contrast)
- Spectral similarity matrix and k-medoids feature clusters computed upstream in msFeaST

## Outputs

- Differential-abundance statistics table (e.g. differential_abundance_mushroom_type.csv) with group-wise test statistics and a per-feature log-2 fold-change column
- dashboard_data.json for the msFeaST interactive dashboard, showing differential intensity patterns and feature highlighting

## How to apply

Load the preprocessed LC-MS/MS feature intensity table, associated MS/MS spectra, and sample group metadata into the msFaST workflow (preprocessing_mushroom_type_comparison.ipynb then msfeast_pipeline_mushroom_type_comparison.ipynb). First compute the pairwise spectral similarity matrix (e.g. modified cosine or MS2Deepscore) and run k-medoids clustering so clusters proxy structurally similar features. Then run feature group-wise statistical testing on those clusters with the globaltest package (pinned via Bioconductor release 3.18, i.e. globaltest 5.56.0 under R 4.3.3), which is chosen because it provides high power to detect small concordant effects via joint modeling with reduced multiplicity adjustment penalties. Finally extract the group-wise test statistics together with per-feature log-2 fold changes into a differential-abundance statistics table (e.g. differential_abundance_mushroom_type.csv) and feed it to the interactive dashboard for feature prioritization.

## Related tools

- **msFeaST** (Orchestrates the pipeline: preprocessing notebooks, k-medoids clustering, R-based group-wise testing, and dashboard visualization of differential intensity patterns) — https://github.com/kevinmildau/msFeaST
- **globaltest** (Performs the feature group-wise statistical testing on spectral-similarity clusters and yields the group-wise test statistics combined with per-feature log-2 fold changes) — https://github.com/kevinmildau/msFeaST

## Examples

```
rscript -e "catch <- lapply(c('dplyr', 'tibble', 'readr', 'listenv', 'BiocManager', 'globaltest'), library, character.only = TRUE); sessionInfo()"
```

## Evaluation signals

- The output table (e.g. differential_abundance_mushroom_type.csv) exists and contains a per-feature log-2 fold-change column covering the analyzed features
- The differential-intensity pattern is qualitatively consistent with the deposited Section 3 mushroom-type run, which shows large differential intensity patterns with wide-scale feature highlighting
- The R session check reports globaltest_5.56.0, survival_3.5-8, readr_2.1.5, tibble_3.2.1, and dplyr_1.1.4 attached under R 4.3.3, confirming the pinned testing stack loaded
- Feature sets passed to globaltest correspond to k-medoids clusters of the spectral similarity matrix, not individual ungrouped features

## Limitations

- globaltest group-wise testing depends on upstream clustering quality: clusters are only a proxy for structurally similar features
- msFeaST preprocessing and pipeline were tested on macOS and Linux; Windows support is being worked on (dashboard itself works in desktop browsers)
- R packages are not installed via conda due to unreliable conda R environments; versions are pinned via remotes/BiocManager, and a cached rscript path can silently install packages into the wrong R installation
- The skill prioritizes features for exploratory analysis; differential intensity patterns are visual and qualitative (e.g. wide-scale feature highlighting) rather than per-feature significance calls

## Evidence

- [abstract] Spectral clustering done in this way allows for feature group-wise statistical testing using the globaltest package: "Spectral clustering done in this way allows for feature group-wise statistical testing using the globaltest package"
- [abstract] globaltest power rationale: "which provides high power to detect small concordant effects via joint modeling and reduced multiplicity adjustment penalties."
- [other] deposited mushroom-type result: [evidence span withheld — anchor status: not_found]
- [readme] notebook inputs: [evidence span withheld — anchor status: external_doc]
- [readme] version pinning: [evidence span withheld — anchor status: external_doc]
