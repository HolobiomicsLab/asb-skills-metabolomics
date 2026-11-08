---
name: metabolomics-univariate-group-comparison
description: Use when you have a normalized two-group or three-group metabolomics/proteomics
  feature table (e.g. POMA SummarizedExperiment from a preprocessed ST000284-style
  study) and need per-feature significance testing with multiple-testing correction.
when_to_use_negative:
- The feature table still contains missing values, is unnormalized, or has not passed
  POMAShiny's missing-value imputation (20% default removal threshold) and normalization
  panels — run preprocessing first.
- The goal is exploratory visualization (PCA, heatmap) rather than hypothesis testing
  — use the EDA panel instead.
- Data are spectral raw files (mzML) needing processing; POMAShiny expects the two
  CSV input files, with upstream MS processing done via RforMassSpectrometry packages
  before migrating to POMA.
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_2269
tools:
- name: POMAShiny
  role: Web application hosting the statistical analysis panel that runs t-test/ANOVA
    and Mann-Whitney U/Kruskal-Wallis with FDR correction on uploaded feature and
    metadata files
  repo: https://github.com/nutrimetabolomics/POMAShiny
- name: POMA
  role: R/Bioconductor back-end package providing the univariate statistical functions
    outside the web environment, increasing reproducibility and flexibility of the
    analyses
  repo: https://github.com/nutrimetabolomics/POMA
- name: R
  role: Programming language in which the POMA univariate tests and tidyverse results-table
    assembly are executed
  repo: http://www.bioconductor.org
- name: tidyverse
  role: R package family used in the back-end to compile test statistics, raw p-values,
    and adjusted p-values into a single results table
- name: limma
  role: Linear-model framework for covariate-adjusted differential testing compared
    against plain t-test/ANOVA
provenance:
  source_task_ids:
  - task_002
  - task_006
  source_papers:
  - doi: 10.1371/journal.pcbi.1009148
    title: 'POMAShiny: A user-friendly web-based workflow for metabolomics and proteomics
      data analysis'
  source_content_hash: sha256:c0f8074bb56c790b27a12ac5191a418f8b02ff6b07b96288db67be4cc87e6389
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nutrimetabolomics/POMAShiny
  tools:
  - POMA
  - POMAShiny
  - R
  - limma
  - tidyverse
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# metabolomics-univariate-group-comparison

## Summary

Runs per-feature group-comparison testing on a preprocessed metabolomics or proteomics feature matrix — parametric t-test/ANOVA and non-parametric Mann-Whitney U/Kruskal-Wallis — with FDR multiple-testing correction, as implemented in POMAShiny's statistical analysis panel backed by the POMA R/Bioconductor package.

## When to use

Use when you have a normalized two-group or three-group metabolomics/proteomics feature table (e.g. POMA SummarizedExperiment from a preprocessed ST000284-style study) and need per-feature significance testing with multiple-testing correction.

## When NOT to use

- The feature table still contains missing values, is unnormalized, or has not passed POMAShiny's missing-value imputation (20% default removal threshold) and normalization panels — run preprocessing first.
- The goal is exploratory visualization (PCA, heatmap) rather than hypothesis testing — use the EDA panel instead.
- Data are spectral raw files (mzML) needing processing; POMAShiny expects the two CSV input files, with upstream MS processing done via RforMassSpectrometry packages before migrating to POMA.

## Inputs

- Preprocessed metabolomics/proteomics feature matrix (CSV features file) as a POMA SummarizedExperiment object
- Target (metadata) CSV file defining the two- or three-group study design
- Optional covariate columns in the metadata for ANCOVA/limma-adjusted models

## Outputs

- Per-feature results table with test statistic, raw p-value, and FDR-adjusted p-value columns (one row per feature)
- Significant feature sets at the chosen FDR threshold
- Comparison table of overlapping vs. adjusted-only significant features when covariate models are fitted

## How to apply

Load the preprocessed feature matrix and its target (metadata) file into R as a POMA SummarizedExperiment object, then for each metabolite or protein feature run the parametric test appropriate to the group design — t-test for two groups, ANOVA for three groups — and the corresponding non-parametric alternatives Mann-Whitney U and Kruskal-Wallis. POMAShiny computes FDR-adjusted p-values alongside raw p-values for every feature because a large number of tests is performed in these analyses, so always report both statistic, raw p-value, and adjusted p-value columns in a single results table (assembled with tidyverse). To improve accuracy, optionally fit covariate-adjusted models via ANCOVA or the limma linear-model framework including study covariates, and compare the significant feature sets at the chosen FDR threshold against the plain ANOVA/t-test results to identify features significant only under adjustment. Decide between parametric and non-parametric variants per design: the two-group contrast uses t-test or Mann-Whitney U, the three-group design uses ANOVA or Kruskal-Wallis. Run this panel only after the POMAShiny preprocessing (missing-value imputation, normalization, outlier removal) and exploratory data analysis panels have been completed.

## Related tools

- **POMAShiny** (Web application hosting the statistical analysis panel that runs t-test/ANOVA and Mann-Whitney U/Kruskal-Wallis with FDR correction on uploaded feature and metadata files) — https://github.com/nutrimetabolomics/POMAShiny
- **POMA** (R/Bioconductor back-end package providing the univariate statistical functions outside the web environment, increasing reproducibility and flexibility of the analyses) — https://github.com/nutrimetabolomics/POMA
- **R** (Programming language in which the POMA univariate tests and tidyverse results-table assembly are executed) — http://www.bioconductor.org
- **tidyverse** (R package family used in the back-end to compile test statistics, raw p-values, and adjusted p-values into a single results table)
- **limma** (Linear-model framework for covariate-adjusted differential testing compared against plain t-test/ANOVA)

## Examples

```
R: res <- POMA::PomaUnivariate(se, method = "ttest", paired = FALSE, adjust = "fdr")
```

## Evaluation signals

- Output table has exactly one row per input feature with non-empty statistic, raw p-value, and adjusted p-value columns.
- Adjusted p-values are FDR-corrected, not raw — verify no column equals the uncorrected p-value set.
- Three-group designs were analyzed with ANOVA/Kruskal-Wallis, two-group with t-test/Mann-Whitney U — check the test type matches the metadata group structure.
- When covariate models are run, compute the overlap of significant feature sets between unadjusted and adjusted results and confirm adjusted-only features are identifiable.
- The analysis was run on the preprocessed (imputed, normalized, outlier-cleaned) data, not the raw upload.

## Limitations

- Multiple-testing correction is mandatory in this panel — a large number of tests is performed, so raw per-feature p-values alone are not an acceptable output.
- POMAShiny was not quantitatively benchmarked against other tools in the paper; no comparison study is reported.
- POMAShiny requires exactly two CSV inputs (target and features files); raw spectral files must be preprocessed upstream, e.g. via RforMassSpectrometry, before migration to POMA.
- POMAShiny was built with bs4Dash version 0.5.0 and is not compatible with newer versions of that package.

## Evidence

- [results] Parametric and non-parametric tests both offered with FDR correction: [evidence span withheld — anchor status: not_found]
- [results] Covariate adjustment improves accuracy: [evidence span withheld — anchor status: not_found]
- [intro] Tool scope: univariate statistical methods included: "including univariate statistical methods, multivariate and dimension reduction methods, feature selection methods, regularized regression analysis approaches, machine learning classification"
- [results] Input format requirement: "POMAShiny requires two comma-separated value (CSV) files as an input: the target (or metadata) file and the features file"
- [intro] Back-end packages: [evidence span withheld — anchor status: not_found]
- [results] Preprocessing must precede statistical analysis: "it is highly recommended to perform an EDA before any statistical analysis"
- [discussion] No quantitative benchmark reported: "This intuitive and powerful web interface allows users to perform an integrated data analysis in an interactive, well-documented and extremely user-friendly web environment"
- [results] Example data set design: "The example data set ST000284 consists of a targeted metabolomics three-group study"
