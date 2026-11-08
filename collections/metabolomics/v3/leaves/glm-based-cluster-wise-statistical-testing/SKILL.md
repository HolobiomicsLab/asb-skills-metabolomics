---
name: glm-based-cluster-wise-statistical-testing
description: Use when you have k-medoids clusters of spectrally similar LC-MS/MS features
  and a two-level treatment contrast, and you need cluster-wise enrichment testing
  with reduced multiplicity penalties rather than per-feature univariate tests.
when_to_use_negative:
- When the goal is per-feature univariate differential abundance testing without any
  feature-set/cluster grouping
- When the input features have not been grouped into spectral-similarity clusters
  (k-medoids clustering must be done first)
- When the contrast involves more than two treatment levels without a clear pairwise
  comparison definition
edam_operation: http://edamontology.org/operation_3656
edam_topics:
- http://edamontology.org/topic_0153
- http://edamontology.org/topic_3172
tools:
- name: globaltest
  role: GLM-based self-contained feature-set testing applied cluster-wise
- name: msFeaST
  role: Pipeline that generates k-medoids clusters and orchestrates the globaltest
    workflow
  repo: https://github.com/kevinmildau/msFeaST
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btae584
    title: Combined LC-MS/MS feature grouping, statistical prioritization, and interactive
      networking in msFeaST
  source_content_hash: sha256:b817a1eb2189682b4de8a5b7be384644f13e133122d3961e2b044e7f3755d3b5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kevinmildau/msFeaST
  tools:
  - globaltest
  - msFeaST
  edam_operation: http://edamontology.org/operation_3656
  edam_topics:
  - http://edamontology.org/topic_0153
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# glm-based-cluster-wise-statistical-testing

## Summary

Self-contained feature-set testing that jointly models structurally similar LC-MS/MS features grouped by k-medoids spectral clustering, using the globaltest GLM-based framework with Bonferroni correction to identify enriched metabolite clusters.

## When to use

Use when you have k-medoids clusters of spectrally similar LC-MS/MS features and a two-level treatment contrast, and you need cluster-wise enrichment testing with reduced multiplicity penalties rather than per-feature univariate tests.

## When NOT to use

- When the goal is per-feature univariate differential abundance testing without any feature-set/cluster grouping
- When the input features have not been grouped into spectral-similarity clusters (k-medoids clustering must be done first)
- When the contrast involves more than two treatment levels without a clear pairwise comparison definition

## Inputs

- k-medoids cluster assignments for LC-MS/MS features (structurally similar features grouped by spectral similarity)
- Preprocessed feature quantification table
- Sample metadata table with two-level treatment contrast (e.g. 0% and 80% olive mill solid waste)

## Outputs

- globaltest_results.csv containing cluster-wise Bonferroni-corrected P-values for every group across the contrast
- Per-feature univariate P-values extracted from the globaltest fits, combined with cluster-wise corrected P-values in the same table

## How to apply

Load the k-medoids cluster assignments and the preprocessed feature quantification table into R. Restrict the sample metadata to the two contrast levels of interest (e.g. 0% vs 80% olive mill solid waste). For each cluster, fit a self-contained GLM-based globaltest that jointly models the features in the set — this joint modeling provides high power to detect small concordant effects while reducing the multiplicity adjustment penalty relative to testing features individually. Apply Bonferroni correction with family size set to the number of groups multiplied by the number of contrasts to obtain cluster-wise adjusted P-values. Extract per-feature univariate P-values from the same globaltest fits and combine the corrected cluster-wise P-values with the feature-specific P-values into a single results table.

## Related tools

- **globaltest** (GLM-based self-contained feature-set testing applied cluster-wise)
- **msFeaST** (Pipeline that generates k-medoids clusters and orchestrates the globaltest workflow) — https://github.com/kevinmildau/msFeaST

## Examples

```
rscript -e "BiocManager::install('globaltest', version='3.18'); library(globaltest); gt <- globaltest(response ~ features, data=cluster_data); p.adjust(pvalue(gt), method='bonferroni', n=n_clusters*n_contrasts)"
```

## Evaluation signals

- globaltest_results.csv exists and contains P-value columns for every cluster in the target contrast
- All corrected P-values fall within [0, 1]
- Cluster-wise Bonferroni-corrected P-values align with the results reported in Section 3 of the article
- Every feature has a corresponding univariate P-value alongside its cluster's corrected P-value

## Limitations

- Bonferroni family size is set to number of groups × number of contrasts, which may be conservative
- Self-contained testing detects concordant effects within a cluster but does not identify which individual features drive the signal without consulting the extracted univariate P-values
- Requires R version 4.3.3 and globaltest 5.56.0 (Bioconductor 3.18) with specific pinned dependency versions (Matrix 1.6-5, survival 3.5-8)

## Evidence

- [abstract] Spectral clustering done in this way allows for feature group-wise statistical testing using the globaltest package, which provides high power to detect small concordant effects via joint modeling and reduced multiplicity adjustment penalties.: "Spectral clustering done in this way allows for feature group-wise statistical testing using the globaltest package, which provides high power to detect small concordant effects via joint modeling"
- [other] For each k-medoids cluster, fit the self-contained globaltest (GLM-based feature-set test) using the globaltest package, which jointly models the features in the set. 4. Apply Bonferroni correction with family size set to the number of groups multiplied by the number of contrasts: [evidence span withheld — anchor status: not_found]
- [readme] Command install globaltest dependency at development version using Bioconductor release version. Note that the Bioconductor version of 3.18 implies globaltest 5.56.0 & R version 4.3.3: [evidence span withheld — anchor status: external_doc]
