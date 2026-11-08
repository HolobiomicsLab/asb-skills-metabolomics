---
name: feature-specific-univariate-p-value-extraction
description: Use when running msFeaST group-wise globaltest on k-medoids spectral-similarity
  clusters and you need feature-level univariate P-values alongside the Bonferroni-corrected
  cluster-wise P-values in a single results table.
when_to_use_negative:
- When input data has not yet been preprocessed through msFeaST's quantification table
  and spectral clustering pipeline — this skill operates on fitted globaltest objects,
  not raw LC-MS/MS data
- When the goal is purely cluster-level feature-set significance without needing feature-resolved
  univariate P-values
- When contrast metadata has not been restricted to the two treatment levels being
  tested (e.g., full dataset with all substrate conditions)
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0203
tools:
- name: globaltest
  role: Provides the self-contained GLM-based feature-set test from which feature-specific
    univariate P-values are extracted after fitting per-cluster models
  repo: https://github.com/kevinmildau/msFeaST
- name: msFeaST
  role: Orchestrates the k-medoids clustering, globaltest fitting, and results combination
    into a single CSV output
  repo: https://github.com/kevinmildau/msFeaST
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btae584
    title: Combined LC-MS/MS feature grouping, statistical prioritization, and interactive
      networking in msFeaST
  source_content_hash: sha256:46a065c662ef98fe64a43f65efaaf080a82018a7975205bc9b1ae185d730509c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kevinmildau/msFeaST
  tools:
  - globaltest
  - msFeaST
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# feature-specific-univariate-p-value-extraction

## Summary

Extracts per-feature univariate P-values from globaltest fits in the msFeaST workflow, complementing cluster-level corrected P-values with feature-level significance estimates for LC-MS/MS untargeted metabolomics contrasts.

## When to use

Use when running msFeaST group-wise globaltest on k-medoids spectral-similarity clusters and you need feature-level univariate P-values alongside the Bonferroni-corrected cluster-wise P-values in a single results table.

## When NOT to use

- When input data has not yet been preprocessed through msFeaST's quantification table and spectral clustering pipeline — this skill operates on fitted globaltest objects, not raw LC-MS/MS data
- When the goal is purely cluster-level feature-set significance without needing feature-resolved univariate P-values
- When contrast metadata has not been restricted to the two treatment levels being tested (e.g., full dataset with all substrate conditions)

## Inputs

- k-medoids cluster assignments from spectral similarity-based grouping of LC-MS/MS features
- preprocessed feature quantification table (e.g., from mushroom type comparison notebook pipeline)
- restricted sample metadata for the contrast levels (e.g., 0% and 80% olive mill solid waste)

## Outputs

- globaltest_results.csv containing cluster-wise Bonferroni-corrected P-values and per-feature univariate P-values for the tested contrast

## How to apply

After fitting each k-medoids cluster's self-contained globaltest GLM-based feature-set test (e.g., for the 0% vs 80% olive mill solid waste contrast in the Pleurotus eryngii dataset), extract the feature-specific univariate P-values from the fitted globaltest objects. Combine these with the cluster-wise Bonferoni-corrected P-values (family size = number of groups × number of contrasts) into a single table. Write the combined output as globaltest_results.csv, containing corrected P-value columns for every cluster and per-feature univariate P-values for the contrast being tested.

## Related tools

- **globaltest** (Provides the self-contained GLM-based feature-set test from which feature-specific univariate P-values are extracted after fitting per-cluster models) — https://github.com/kevinmildau/msFeaST
- **msFeaST** (Orchestrates the k-medoids clustering, globaltest fitting, and results combination into a single CSV output) — https://github.com/kevinmildau/msFeaST

## Examples

```
rscript -e [evidence span withheld — anchor status: external_doc]; then in R: extract univariate P-values from fitted globaltest objects and combine with corrected P-values into globaltest_results.csv
```

## Evaluation signals

- globaltest_results.csv exists and is parseable
- CSV contains P-value columns for every cluster in the tested contrast (e.g., 0% vs 80% olive mill solid waste)
- All corrected P-values fall within [0, 1]
- Results are consistent with those reported in Section 3 of the paper
- Per-feature univariate P-values are present for all features within each tested cluster

## Limitations

- globaltest requires specific R package versions (globaltest 5.56.0, R 4.3.3, Bioconductor 3.18) — version mismatches can cause pipeline failure
- R package installation within conda is unreliable; remotes and BiocManager are used instead of conda R packages
- Windows support is untested for the pipeline execution portion
- Bonferroni correction family size depends on number of groups and contrasts, so results are contrast-specific

## Evidence

- [full_text] Extract the feature-specific univariate P-values from the globaltest fits.: [evidence span withheld — anchor status: not_found]
- [abstract] Spectral clustering done in this way allows for feature group-wise statistical testing using the globaltest package: "Spectral clustering done in this way allows for feature group-wise statistical testing using the globaltest package"
- [readme] BiocManager::install('globaltest', version='3.18'): [evidence span withheld — anchor status: external_doc]
- [readme] R packages are currently not installed using conda since the conda R package environment is not working reliably for the required packages yet: [evidence span withheld — anchor status: external_doc]
- [abstract] globaltest provides high power to detect small concordant effects via joint modeling and reduced multiplicity adjustment penalties: [evidence span withheld — anchor status: not_found]
