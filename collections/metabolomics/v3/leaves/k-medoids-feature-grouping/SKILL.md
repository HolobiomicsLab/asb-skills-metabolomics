---
name: k-medoids-feature-grouping
description: Use when you have an LC-MS/MS untargeted metabolomics feature table with
  associated MS/MS spectra and need to aggregate features into structurally similar
  groups before feature-set (group-wise) statistical testing.
when_to_use_negative:
- Input data lack MS/MS spectra, so no spectral similarity matrix can be computed
  for clustering.
- The goal is single-feature differential analysis rather than structurally informed
  feature-set testing — per-feature testing does not need k-medoids grouping.
- You only want to inspect pre-computed example dashboards; the Dashboard bundle plus
  dashboard_data.json suffices without running the pipeline.
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: msFeaST
  role: Workflow that performs spectral similarity-based k-medoids feature grouping,
    feature-set testing, and interactive visualization for LC-MS/MS data
  repo: https://github.com/kevinmildau/msFeaST
- name: globaltest
  role: R package used downstream of the k-medoids clusters for feature group-wise
    statistical testing
  repo: https://github.com/kevinmildau/msFeaST
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btae584
    title: Combined LC-MS/MS feature grouping, statistical prioritization, and interactive
      networking in msFeaST
  source_content_hash: sha256:01eefbe26df42ce935e539a0b06990a4cfadf0243b3a5c46f1b9d6bbf69a66bb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kevinmildau/msFeaST
  tools:
  - globaltest
  - msFeaST
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# k-medoids-feature-grouping

## Summary

Groups LC-MS/MS features into structurally similar clusters by applying k-medoids clustering to a pairwise spectral similarity matrix (e.g. modified cosine or MS2Deepscore), so that each cluster acts as a proxy feature set for downstream group-wise statistical testing in msFeaST.

## When to use

Use when you have an LC-MS/MS untargeted metabolomics feature table with associated MS/MS spectra and need to aggregate features into structurally similar groups before feature-set (group-wise) statistical testing.

## When NOT to use

- Input data lack MS/MS spectra, so no spectral similarity matrix can be computed for clustering.
- The goal is single-feature differential analysis rather than structurally informed feature-set testing — per-feature testing does not need k-medoids grouping.
- You only want to inspect pre-computed example dashboards; the Dashboard bundle plus dashboard_data.json suffices without running the pipeline.

## Inputs

- Preprocessed LC-MS/MS feature intensity (quantification) table
- Associated MS/MS spectra per feature
- Sample group metadata table (e.g. mushroom-type contrast labels)
- Pairwise spectral similarity matrix (e.g. modified cosine scores or MS2Deepscore embeddings)

## Outputs

- K-medoids feature clusters serving as proxies for structurally similar feature groups
- Cluster-level group-wise test statistics from globaltest
- dashboard_data.json for interactive exploration in msFeaST_Dashboard_bundle.html

## How to apply

Compute the pairwise spectral similarity matrix between MS/MS spectra of all features using a spectral similarity score such as the modified cosine score (or MS2Deepscore, as in the deposited omsw_pleurotus_ms2deepscore example). Run k-medoids clustering on this similarity matrix; the resulting clusters serve as a proxy for grouping structurally similar features. In the msFeaST workflow this grouping is done in the preprocessing/pipeline Jupyter notebooks (e.g. preprocessing_mushroom_type_comparison.ipynb followed by msfeast_pipeline_mushroom_type_comparison.ipynb), which consume a quantification table, a metadata table, and the spectral data. Feed the clusters into feature group-wise statistical testing with the globaltest R package (version 5.56.0 / Bioconductor 3.18, R 4.3.3 in the tested setup), which exploits the joint structure of clusters to detect small concordant effects with reduced multiplicity adjustment penalties. The dashboard bundle (msFeaST_Dashboard_bundle.html) then visualizes clusters together with per-feature differential statistics.

## Related tools

- **msFeaST** (Workflow that performs spectral similarity-based k-medoids feature grouping, feature-set testing, and interactive visualization for LC-MS/MS data) — https://github.com/kevinmildau/msFeaST
- **globaltest** (R package used downstream of the k-medoids clusters for feature group-wise statistical testing) — https://github.com/kevinmildau/msFeaST

## Examples

```
BiocManager::install('globaltest', version='3.18'); rscript -e "catch <- lapply(c('dplyr', 'tibble', 'readr', 'listenv', 'BiocManager', 'globaltest'), library, character.only = TRUE); sessionInfo()"
```

## Evaluation signals

- The pipeline notebook produces a dashboard_data.json that loads successfully into msFeaST_Dashboard_bundle.html.
- Each resulting cluster contains features with high pairwise spectral similarity (e.g. modified cosine / MS2Deepscore), acting as a proxy for structural similarity.
- R session check returns globaltest_5.56.0 with survival_3.5-8, readr_2.1.5, tibble_3.2.1, dplyr_1.1.4 attached, confirming the tested dependency versions.
- Downstream globaltest statistics and per-feature log2 fold changes reproduce qualitative patterns (e.g. wide-scale feature highlighting for the mushroom-type comparison) seen in the deposited Section 3 run.

## Limitations

- Pipeline tested on macOS (macos-arm64) and expected to work on Linux; Windows support is untested/being worked on, though the dashboard itself runs in any desktop browser.
- rscript calls can install R packages into the wrong R path if the terminal was opened before conda R installation; close and reopen the terminal and re-activate the msfeast_environment.
- VS Code and other terminals that auto-activate conda environments can cause R/rscript path misalignment; verify with 'type R' and 'type rscript'.
- Conda R package environments were unreliable for the required packages, so R dependencies are version-pinned via remotes/BiocManager instead.

## Evidence

- [abstract] spectral similarity-based feature groupings generated using k-medoids clustering, where the resulting clusters serve as a proxy for grouping structurally similar features: "spectral similarity-based feature groupings generated using k-medoids clustering, where the resulting clusters serve as a proxy for grouping structurally similar features"
- [abstract] Spectral clustering done in this way allows for feature group-wise statistical testing using the globaltest package, which provides high power to detect small concordant effects via joint modeling and reduced multiplicity adjustment penalties.: "Spectral clustering done in this way allows for feature group-wise statistical testing using the globaltest package, which provides high power to detect small concordant effects via joint modeling"
- [readme] These notebooks contain a complete example of quantification table, metadata table, and spectral data processing required for msFeaST, as well as a complete use-case example.: [evidence span withheld — anchor status: external_doc]
- [readme] Note that the Bioconductor version of 3.18 implies globaltest 5.56.0 & R version 4.3.3: [evidence span withheld — anchor status: external_doc]
