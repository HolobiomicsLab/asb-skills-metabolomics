---
name: bonferroni-multiple-testing-correction
description: Use when performing self-contained globaltest feature-set testing over
  spectral-similarity-derived k-medoids clusters in LC-MS/MS untargeted metabolomics
  data, and cluster-wise corrected P-values across multiple contrasts are required.
when_to_use_negative:
- When the goal is per-feature univariate differential testing rather than joint feature-set
  testing across spectrally similar feature groups — globaltest was chosen specifically
  for its 'reduced multiplicity adjustment penalties' at the set level
- When no feature grouping (e.g. k-medoids clustering of structurally similar features)
  has been performed and sets are not yet defined
- When the analysis is not applied to LC-MS/MS untargeted metabolomics quantification
  data with defined contrast levels (e.g. substrate treatments), since the family
  size calculation depends on the number of contrasts
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3520
tools:
- name: globaltest
  role: Provides the self-contained GLM-based feature-set test whose per-cluster P-values
    are Bonferroni-adjusted
- name: msFeaST
  role: Hosts the pipeline that performs k-medoids clustering and globaltest fitting,
    then applies Bonferroni correction to produce the combined results table
  repo: https://github.com/kevinmildau/msFeaST
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btae584
    title: Combined LC-MS/MS feature grouping, statistical prioritization, and interactive
      networking in msFeaST
  source_content_hash: sha256:b3e4a91d6f45c72f5926fa0d77247920a9e8f8810202f5232ea6f279978eb96d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kevinmildau/msFeaST
  tools:
  - globaltest
  - msFeaST
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# bonferroni-multiple-testing-correction

## Summary

Applies Bonferroni correction with a family size defined as the number of k-medoids feature clusters multiplied by the number of contrasts, to obtain cluster-wise adjusted P-values from globaltest feature-set fits in the msFeaST LC-MS/MS metabolomics workflow.

## When to use

Use when performing self-contained globaltest feature-set testing over spectral-similarity-derived k-medoids clusters in LC-MS/MS untargeted metabolomics data, and cluster-wise corrected P-values across multiple contrasts are required.

## When NOT to use

- When the goal is per-feature univariate differential testing rather than joint feature-set testing across spectrally similar feature groups — globaltest was chosen specifically for its 'reduced multiplicity adjustment penalties' at the set level
- When no feature grouping (e.g. k-medoids clustering of structurally similar features) has been performed and sets are not yet defined
- When the analysis is not applied to LC-MS/MS untargeted metabolomics quantification data with defined contrast levels (e.g. substrate treatments), since the family size calculation depends on the number of contrasts

## Inputs

- Globaltest GLM-based feature-set test fits, one per k-medoids cluster per contrast (e.g. 0% vs 80% olive mill solid waste substrate contrast)
- Number of k-medoids feature clusters (groups)
- Number of contrasts being tested

## Outputs

- Cluster-wise Bonferroni-adjusted P-values for every k-medoids cluster in each contrast
- Combined table (globaltest_results.csv) containing cluster-wise corrected P-values and per-feature univariate P-values from the globaltest fits

## How to apply

After fitting the self-contained globaltest (GLM-based feature-set test) for each k-medoids cluster in a contrast (e.g. 0% vs 80% olive mill solid waste substrate), compute the family size as the number of groups (clusters) multiplied by the number of contrasts, then multiply each cluster's globaltest P-value by that family size to obtain the Bonferroni-adjusted cluster-wise P-value, capping values at 1. The rationale, per the article, is that globaltest jointly models the features within a set, providing 'high power to detect small concordant effects via joint modeling and reduced multiplicity adjustment penalties' — so correction is applied at the cluster level rather than the individual-feature level. Extract the feature-specific univariate P-values from the same globaltest fits separately, and combine both cluster-wise corrected and per-feature univariate P-values into a single results table (globaltest_results.csv) so that strong log-2 fold-change signals with weaker statistical enrichment can be interpreted together. Validate that corrected P-values fall within [0, 1] and that every cluster in the contrast has a corresponding P-value column.

## Related tools

- **globaltest** (Provides the self-contained GLM-based feature-set test whose per-cluster P-values are Bonferroni-adjusted)
- **msFeaST** (Hosts the pipeline that performs k-medoids clustering and globaltest fitting, then applies Bonferroni correction to produce the combined results table) — https://github.com/kevinmildau/msFeaST

## Examples

```
rscript -e "BiocManager::install('globaltest', version='3.18')"
```

## Evaluation signals

- globaltest_results.csv exists and contains P-value columns for every cluster in the 0% vs 80% contrast
- All Bonferroni-corrected P-values fall within [0, 1]
- The family size used equals (number of k-medoids clusters) × (number of contrasts)
- The results are qualitatively consistent with Section 3 of the paper: many metabolites with elevated log-2 fold changes and a lower number showing corresponding statistical enrichment patterns

## Limitations

- R installation of globaltest and its indirect dependencies (Matrix, survival, listenv, readr, tibble, dplyr) must use specific pinned versions; conda R package management is 'not working reliably for the required packages yet' and remotes/BiocManager are used instead
- The Bioconductor version of 3.18 implies globaltest 5.56.0 & R version 4.3.3; other versions may behave differently
- In rare cases where rscript is run from a terminal prior to the conda installation of R, a cached faulty R path may cause packages to install into the wrong location — the terminal must be reopened after conda R installation
- Windows support for the msFeaST pipeline is untested and currently being worked on

## Evidence

- [methods] Bonferroni correction with family size set to the number of groups multiplied by the number of contrasts: [evidence span withheld — anchor status: not_found]
- [abstract] globaltest provides high power to detect small concordant effects via joint modeling and reduced multiplicity adjustment penalties.: "using the globaltest package, which provides high power to detect small concordant effects via joint modeling and reduced multiplicity adjustment penalties."
- [abstract] Spectral clustering done in this way allows for feature group-wise statistical testing using the globaltest package: "Spectral clustering done in this way allows for feature group-wise statistical testing using the globaltest package"
- [readme] Note that the Bioconductor version of 3.18 implies globaltest 5.56.0 & R version 4.3.3: [evidence span withheld — anchor status: external_doc]
- [methods] verify that globaltest_results.csv exists, contains P-value columns for every cluster in the 0% vs 80% contrast, and that all corrected P-values fall within [0, 1]: [evidence span withheld — anchor status: not_found]
