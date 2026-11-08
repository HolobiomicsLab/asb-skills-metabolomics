---
name: feature-set-global-testing
description: Use when analyzing LC-MS/MS untargeted metabolomics feature tables where
  per-feature tests lose power or multiplicity, and structurally similar features
  should be jointly tested as groups (clusters) against a two-level group contrast.
when_to_use_negative:
- When features have no MS/MS spectra and cannot be grouped by spectral similarity,
  so no cluster-level feature set exists to test jointly.
- When the design is not a simple two-level group contrast but a complex multi-factor
  design beyond the msFeaST k-medoids-cluster + globaltest pipeline shown here.
- When the goal is per-feature univariate testing alone rather than group-wise feature-set
  testing for prioritization.
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_3520
tools:
- name: msFeaST
  role: 'Pipeline orchestrator: pre-processes LC-MS/MS data, computes spectral similarity,
    clusters features via k-medoids, and packages feature-set testing results for
    the interactive dashboard.'
  repo: https://github.com/kevinmildau/msFeaST
- name: globaltest
  role: Bioconductor package used to fit the self-contained, GLM-based feature-set
    test on each k-medoids cluster, providing cluster-level P-values and feature-specific
    univariate P-values.
provenance:
  source_task_ids:
  - task_007
  - task_004
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btae584
    title: Combined LC-MS/MS feature grouping, statistical prioritization, and interactive
      networking in msFeaST
  source_content_hash: sha256:d9e2a41e549bfd93152e9ccea5d4641ddfa0140a32c6f8ef635cf5d91bf7b465
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kevinmildau/msFeaST
  tools:
  - globaltest
  - msFeaST
  edam_operation: http://edamontology.org/operation_3664
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# feature-set-global-testing

## Summary

Feature group-wise statistical testing in msFeaST: k-medoids clusters of spectrally similar LC-MS/MS features are jointly tested as feature sets with the globaltest package, yielding cluster-level P-values with Bonferroni correction plus per-feature univariate statistics and log-2 fold changes for prioritization.

## When to use

Use when analyzing LC-MS/MS untargeted metabolomics feature tables where per-feature tests lose power or multiplicity, and structurally similar features should be jointly tested as groups (clusters) against a two-level group contrast.

## When NOT to use

- When features have no MS/MS spectra and cannot be grouped by spectral similarity, so no cluster-level feature set exists to test jointly.
- When the design is not a simple two-level group contrast but a complex multi-factor design beyond the msFeaST k-medoids-cluster + globaltest pipeline shown here.
- When the goal is per-feature univariate testing alone rather than group-wise feature-set testing for prioritization.

## Inputs

- Preprocessed LC-MS/MS feature quantification/intensity table
- Sample group metadata with two contrast levels (e.g., 0% vs 80% olive mill solid waste substrate, or mushroom-type comparison)
- MS/MS peak lists / spectra per feature (for similarity-based clustering)
- K-medoids cluster assignments from the msFeaST spectral similarity clustering step

## Outputs

- Cluster-wise globaltest P-values with Bonferroni-adjusted values per cluster per contrast (globaltest_results.csv)
- Feature-specific univariate P-values extracted from the globaltest fits
- Per-feature log-2 fold-change columns between the contrast groups (e.g., modified_cosine_stats_table.csv, differential_abundance_mushroom_type.csv)
- Combined differential-abundance statistics table feeding the msFeaST dashboard JSON

## How to apply

First compute a pairwise spectral similarity matrix between MS/MS features (e.g., modified cosine or MS2Deepscore), then run k-medoids clustering on that similarity matrix so clusters act as proxies for structurally similar features. For each cluster, fit the self-contained globaltest (a GLM-based feature-set test from Bioconductor's globaltest package) on the restricted sample metadata for the two contrast levels (e.g., 0% vs 80% olive mill solid waste), which jointly models the features in the set and gives high power to detect small concordant effects with reduced multiplicity penalties. Apply Bonferroni correction with family size set to the number of groups multiplied by the number of contrasts. Extract feature-specific univariate P-values from the globaltest fits and combine them with cluster-wise corrected P-values and per-feature log-2 fold changes into a single differential-abundance table for interactive exploration in the msFeaST dashboard.

## Related tools

- **msFeaST** (Pipeline orchestrator: pre-processes LC-MS/MS data, computes spectral similarity, clusters features via k-medoids, and packages feature-set testing results for the interactive dashboard.) — https://github.com/kevinmildau/msFeaST
- **globaltest** (Bioconductor package used to fit the self-contained, GLM-based feature-set test on each k-medoids cluster, providing cluster-level P-values and feature-specific univariate P-values.)

## Examples

```
rscript -e "BiocManager::install('globaltest', version='3.18')"
```

## Evaluation signals

- Output CSV (e.g., globaltest_results.csv) exists and contains P-value columns for every k-medoids cluster in the contrast.
- All corrected (Bonferroni) P-values fall within [0, 1]; family size equals number of groups multiplied by number of contrasts.
- Per-feature log-2 fold-change column covers all analyzed features and, for the mushroom-type comparison, differential intensity patterns are qualitatively consistent with wide-scale feature highlighting in the deposited Section 3 run.
- R session check shows the expected package stack: globaltest_5.56.0 with survival_3.5-8, Matrix_1.6-5, readr_2.1.5, tibble_3.2.1, dplyr_1.1.4 on R 4.3.3.
- Results are qualitatively consistent with the additional deposited run reported in Section 3 of the article (many metabolites with elevated log-2 fold changes, a lower number with corresponding enrichment).

## Limitations

- The pipeline has only been tested on macOS (arm64) and is expected to work identically on Linux; Windows support is not yet complete (untested).
- R package installation must use pinned versions (Matrix 1.6-5, survival 3.5-8, listenv 0.9.1, readr 2.1.5, tibble 3.2.1, dplyr 1.1.4, globaltest 5.56.0 via Bioconductor 3.18); mismatched versions can break the workflow.
- Conda-cached rscript paths can install R packages into the wrong R version; the terminal must be closed and reopened after conda R installation before installing R dependencies.
- The k-medoids clusters serve only as a proxy for structurally similar features; cluster membership does not guarantee shared chemical identity.
- Bonferroni correction is conservative; family size is defined as number of groups multiplied by number of contrasts.

## Evidence

- [abstract] globaltest high power claim: "using the globaltest package, which provides high power to detect small concordant effects via joint modeling and reduced multiplicity adjustment penalties."
- [abstract] k-medoids cluster rationale: "spectral similarity-based feature groupings generated using k-medoids clustering, where the resulting clusters serve as a proxy for grouping structurally similar features"
- [readme] pinned R versions from README: [evidence span withheld — anchor status: external_doc]
- [readme] expected session info: [evidence span withheld — anchor status: external_doc]
- [supplementary] Bonferroni family size: [evidence span withheld — anchor status: not_found]
