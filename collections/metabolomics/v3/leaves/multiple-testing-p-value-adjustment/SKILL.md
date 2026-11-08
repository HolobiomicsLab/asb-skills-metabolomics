---
name: multiple-testing-p-value-adjustment
description: Use when you run univariate statistical tests (t-test, ANOVA, Mann-Whitney
  U, Kruskal-Wallis) over each metabolite/protein feature of a preprocessed feature
  matrix and need p-values adjusted for the large number of simultaneous tests.
when_to_use_negative:
- The analysis performs a single hypothesis test rather than one test per feature,
  so no multiple-testing correction is needed.
- Only unadjusted p-values are requested as a diagnostic step before correction (e.g.
  inspecting the raw p-value distribution), not as final significance calls.
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0203
tools:
- name: POMAShiny
  role: Web interface whose univariate analysis panel computes FDR-adjusted p-values
    over the per-feature test battery
  repo: https://github.com/nutrimetabolomics/POMAShiny
- name: POMA
  role: R/Bioconductor package providing the univariate test functions and SummarizedExperiment-based
    data handling behind the adjustment workflow
  repo: https://github.com/nutrimetabolomics/POMA
- name: tidyverse
  role: Used to compile the per-feature statistics, raw p-values, and adjusted p-values
    into a single tidy results table
  repo: https://github.com/nutrimetabolomics/POMAShiny
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1371/journal.pcbi.1009148
    title: 'POMAShiny: A user-friendly web-based workflow for metabolomics and proteomics
      data analysis'
  source_content_hash: sha256:9e64634233c90fc295e120034cf513793562de432a73ea11c33867e8820d847d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nutrimetabolomics/POMAShiny
  tools:
  - POMA
  - POMAShiny
  - R
  - tidyverse
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 15
---

# multiple-testing-p-value-adjustment

## Summary

Apply multiple-testing correction (FDR-adjusted p-values) to the battery of per-feature univariate tests (t-test/ANOVA and Mann-Whitney/Kruskal-Wallis) that POMAShiny runs over metabolomics or proteomics feature tables, so per-metabolite significance claims remain valid across the many simultaneous hypotheses.

## When to use

Use when you run univariate statistical tests (t-test, ANOVA, Mann-Whitney U, Kruskal-Wallis) over each metabolite/protein feature of a preprocessed feature matrix and need p-values adjusted for the large number of simultaneous tests.

## When NOT to use

- The analysis performs a single hypothesis test rather than one test per feature, so no multiple-testing correction is needed.
- Only unadjusted p-values are requested as a diagnostic step before correction (e.g. inspecting the raw p-value distribution), not as final significance calls.

## Inputs

- Preprocessed feature matrix (samples x features) loaded as a POMA SummarizedExperiment object
- Target (metadata) CSV file defining the two-group or three-group contrast
- Raw per-feature p-values from t-test/ANOVA and Mann-Whitney/Kruskal-Wallis runs

## Outputs

- Per-feature results table (one row per metabolite/protein) with test statistic, raw p-value, and FDR-adjusted p-value
- FDR-adjusted p-value vector covering all features tested

## How to apply

After loading the preprocessed feature matrix and its target (metadata) file into R as a POMA SummarizedExperiment object, run the univariate tests POMAShiny exposes: t-test for the two-group contrast and ANOVA for the three-group design, plus the non-parametric alternatives Mann-Whitney U and Kruskal-Wallis. Because these analyses test each feature separately and thus perform a large number of tests, compile raw p-values for every feature and compute FDR-adjusted p-values across the full feature set, as POMAShiny's statistical analysis panel does. Collect the test statistics, raw p-values, and adjusted p-values into a single per-feature results table using tidyverse, mirroring the POMA univariate workflow outside the web environment. Do not interpret unadjusted p-values from these panel outputs, since per-feature testing without correction inflates false positives.

## Related tools

- **POMAShiny** (Web interface whose univariate analysis panel computes FDR-adjusted p-values over the per-feature test battery) — https://github.com/nutrimetabolomics/POMAShiny
- **POMA** (R/Bioconductor package providing the univariate test functions and SummarizedExperiment-based data handling behind the adjustment workflow) — https://github.com/nutrimetabolomics/POMA
- **tidyverse** (Used to compile the per-feature statistics, raw p-values, and adjusted p-values into a single tidy results table) — https://github.com/nutrimetabolomics/POMAShiny

## Examples

```
res <- data.frame(statistic = sapply(seq_along(feat), function(i) t.test(x[i,] ~ grp)$statistic), p = apply(x, 1, function(r) t.test(r ~ grp)$p.value)); res$p_adj <- p.adjust(res$p, method = 'BH')
```

## Evaluation signals

- Results table has exactly one row per feature with non-empty statistic, raw p-value, and adjusted p-value columns
- Adjusted p-value column is populated for every feature, reflecting correction across the full test battery rather than per-test adjustment
- The two-group runs (t-test, Mann-Whitney) and three-group runs (ANOVA, Kruskal-Wallis) each produced p-values consistent with the study-group structure of the input
- Adjusted p-values are >= their corresponding raw p-values and fall within [0, 1]

## Limitations

- Adjustment must be applied jointly across all per-feature tests; applying it to a single test alone yields no correction benefit.
- The p-value distribution depends on prior preprocessing (e.g. imputation, normalization, outlier removal); heavily imputed features can distort test statistics and hence adjusted p-values.
- POMAShiny is designed for metabolomics and proteomics data with two- or three-group study designs; very unbalanced or multi-factor designs fall outside the tested univariate panels.

## Evidence

- [results] FDR correction grounds per-feature test interpretation: [evidence span withheld — anchor status: not_found]
- [abstract] POMA enables reproducible univariate work outside the web UI: "it is based on the POMA R/Bioconductor package, which increases the reproducibility and flexibility of analyses outside the web environment"
- [intro] POMAShiny test battery covers parametric and non-parametric univariate tests: "including univariate statistical methods, multivariate and dimension reduction methods, feature selection methods, regularized regression analysis approaches, machine learning classification"
- [intro] tidyverse back-end supports result table assembly: [evidence span withheld — anchor status: not_found]
- [results] Example three-group dataset used to exercise the test battery: "The example data set ST000284 consists of a targeted metabolomics three-group study"
