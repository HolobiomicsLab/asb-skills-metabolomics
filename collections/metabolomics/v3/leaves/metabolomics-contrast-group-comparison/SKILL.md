---
name: metabolomics-contrast-group-comparison
description: Use when you have a preprocessed LC-MS/MS feature quantification table
  with k-medoids cluster assignments from spectral similarity grouping and need feature-set
  level differential testing between two treatment groups.
when_to_use_negative:
- When the feature quantification table has not yet been clustered into k-medoids
  spectral-similarity groups — the globaltest feature-set step requires prior msFeaST
  grouping.
- When raw spectral data has not been preprocessed through the msFeaST preprocessing
  notebook (quantification table, metadata table, and spectral data processing are
  prerequisites).
- When the analysis requires fully calibrated quantitative metabolomics rather than
  semi-quantitative experimental information, since msFeaST integration is built on
  semi-quantitative data.
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0219
tools:
- name: globaltest
  role: Self-contained GLM-based feature-set test used to jointly model the features
    in each k-medoids cluster and produce cluster-wise and feature-wise P-values
  repo: https://github.com/kevinmildau/msFeaST
- name: msFeaST
  role: Workflow that produces the k-medoids spectral-similarity feature groups and
    preprocessed quantification tables that this skill consumes, and that visualizes
    the resulting globaltest outputs in an interactive dashboard
  repo: https://github.com/kevinmildau/msFeaST
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btae584
    title: Combined LC-MS/MS feature grouping, statistical prioritization, and interactive
      networking in msFeaST
  source_content_hash: sha256:4c8a3708bf8de4090427ce15921c5bfbd7e1dc24e25a0819aade870314442b99
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kevinmildau/msFeaST
  tools:
  - globaltest
  - msFeaST
  edam_operation: http://edamontology.org/operation_3664
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# metabolomics-contrast-group-comparison

## Summary

A group-wise statistical testing approach for LC-MS/MS untargeted metabolomics that applies the self-contained globaltest to k-medoids spectral-similarity feature clusters, then extracts Bonferroni-corrected cluster P-values alongside per-feature univariate P-values to prioritize differential metabolite patterns across a two-level contrast (e.g. 0% vs 80% olive mill solid waste substrate).

## When to use

Use when you have a preprocessed LC-MS/MS feature quantification table with k-medoids cluster assignments from spectral similarity grouping and need feature-set level differential testing between two treatment groups.

## When NOT to use

- When the feature quantification table has not yet been clustered into k-medoids spectral-similarity groups — the globaltest feature-set step requires prior msFeaST grouping.
- When raw spectral data has not been preprocessed through the msFeaST preprocessing notebook (quantification table, metadata table, and spectral data processing are prerequisites).
- When the analysis requires fully calibrated quantitative metabolomics rather than semi-quantitative experimental information, since msFeaST integration is built on semi-quantitative data.

## Inputs

- k-medoids cluster assignments derived from spectral similarity-based feature grouping of LC-MS/MS MS/MS spectra
- preprocessed feature quantification table (metabolite/feature intensities per sample)
- sample metadata table with contrast levels (e.g. 0% vs 80% olive mill solid waste substrate)

## Outputs

- globaltest_results.csv containing cluster-wise Bonferroni-corrected P-values and per-feature univariate P-values for the contrast
- cluster-level and feature-level statistical evidence feeding the msFeaST interactive dashboard (dashboard_data.json)

## How to apply

Load the k-medoids cluster assignments and the preprocessed feature quantification table (e.g. from the msFeaST pipeline for the Kathib et al. Pleurotus eryngii dataset) into R. Restrict the sample metadata to the two contrast levels (e.g. 0% and 80% olive mill solid waste substrate). For each k-medoids cluster, fit the self-contained globaltest (GLM-based feature-set test) using the globaltest package (BiocManager version 3.18, globaltest 5.56.0 under R 4.3.3), which jointly models all features in the set to gain power for small concordant effects while avoiding the multiplicity penalty that per-feature testing would incur. Apply Bonferroni correction with family size set to the number of groups multiplied by the number of contrasts, yielding cluster-wise adjusted P-values. Extract feature-specific univariate P-values from the globaltest fits, combine the cluster-corrected and per-feature P-values into a single table, and write it to globaltest_results.csv. In the msFeaST workflow, this step sits after the feature-grouping stage and feeds the interactive dashboard where semi-quantitative experimental information and mass-spectral structural similarity information are integrated to prioritize features and feature sets.

## Related tools

- **globaltest** (Self-contained GLM-based feature-set test used to jointly model the features in each k-medoids cluster and produce cluster-wise and feature-wise P-values) — https://github.com/kevinmildau/msFeaST
- **msFeaST** (Workflow that produces the k-medoids spectral-similarity feature groups and preprocessed quantification tables that this skill consumes, and that visualizes the resulting globaltest outputs in an interactive dashboard) — https://github.com/kevinmildau/msFeaST

## Examples

```
BiocManager::install('globaltest', version='3.18'); globaltest::gt(formula = group_contrast ~ 1, data = feature_table, subsets = kmedoids_clusters)
```

## Evaluation signals

- globaltest_results.csv exists and contains P-value columns for every cluster in the 0% vs 80% contrast.
- All corrected (Bonferroni-adjusted) P-values fall within [0, 1].
- Qualitative consistency with the results reported in Section 3 of the article (e.g. elevated log-2 fold changes for many metabolites with fewer showing corresponding statistical enrichment).
- The R session reports globaltest_5.56.0 alongside survival_3.5-8 and the other pinned dependency versions, confirming the correct environment.
- Cluster-wise family size equals number of groups × number of contrasts, matching the Bonferroni correction specification.

## Limitations

- Tested on macOS and Linux; Windows support is currently being worked on.
- R package compilation steps during installation may take several minutes on some systems.
- Conda R path caching issues can cause rscript calls to install packages into the wrong R environment; must open a fresh terminal after installation to avoid this.
- The Bioconductor release 3.18 pins globaltest 5.56.0 and R 4.3.3 — deviating from these versions may break the workflow.
- msFeaST provides semi-quantitative rather than fully quantitative metabolite information, so statistical enrichment results are for prioritization rather than absolute concentration claims.

## Evidence

- [abstract] Feature group-wise statistical testing with globaltest: "using the globaltest package, which provides high power to detect small concordant effects via joint modeling and reduced multiplicity adjustment penalties."
- [readme] Workflow restriction to contrast levels: [evidence span withheld — anchor status: external_doc]
- [readme] Version pinning: [evidence span withheld — anchor status: external_doc]
- [abstract] Interactive dashboard integration: "msFeaST provides interactive integration of the semi-quantitative experimental information with mass-spectral structural similarity information"
