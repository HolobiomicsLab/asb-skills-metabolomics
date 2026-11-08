---
name: limma-differential-abundance-analysis
description: Use when a metabolomics/proteomics feature table (e.g. ST000284) has
  study covariates (batch, sex, age, etc.) that may confound group comparisons, and
  you need covariate-adjusted differential abundance results to compare against unadjusted
  t-test/ANOVA calls.
when_to_use_negative:
- The design has no meaningful covariates beyond the group variable — a plain t-test/ANOVA
  already captures the model, and adding an aliased covariate term is redundant
- The data have not passed through preprocessing (missing-value imputation, e.g. 20%
  per-group feature removal threshold, normalization) — the linear models assume a
  cleaned, imputed feature table
- The covariate is perfectly confounded with the group label, making the design matrix
  rank-deficient
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: POMA
  role: R/Bioconductor package providing the statistical analysis functions (univariate
    tests) used as the baseline and enabling covariate-adjusted modeling outside the
    web environment
  repo: https://github.com/nutrimetabolomics/POMA
- name: limma
  role: Linear-model framework used to fit covariate-adjusted per-feature differential
    abundance models
- name: tidyverse
  role: Back-end package used for set operations and table assembly when comparing
    significant feature sets
- name: POMAShiny
  role: Web-based workflow whose statistical analysis panel motivates the covariate-adjusted
    vs. unadjusted comparison; based on POMA
  repo: https://github.com/nutrimetabolomics/POMAShiny
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1371/journal.pcbi.1009148
    title: 'POMAShiny: A user-friendly web-based workflow for metabolomics and proteomics
      data analysis'
  source_content_hash: sha256:2935244c4e5303b1d2c55df1f4c7a6964d633f7d4da955b2c4942b1b903dc1bc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nutrimetabolomics/POMA
  tools:
  - POMA
  - POMAShiny
  - R
  - limma
  - tidyverse
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# limma-differential-abundance-analysis

## Summary

Fit covariate-adjusted linear models (limma/ANCOVA) for each feature of a metabolomics or proteomics feature table, as an alternative to plain t-test/ANOVA group comparisons, and compare the resulting significant feature sets. In the POMAShiny/POMA context, this tests whether covariate adjustment changes which features are called significant.

## When to use

Use when a metabolomics/proteomics feature table (e.g. ST000284) has study covariates (batch, sex, age, etc.) that may confound group comparisons, and you need covariate-adjusted differential abundance results to compare against unadjusted t-test/ANOVA calls.

## When NOT to use

- The design has no meaningful covariates beyond the group variable — a plain t-test/ANOVA already captures the model, and adding an aliased covariate term is redundant
- The data have not passed through preprocessing (missing-value imputation, e.g. 20% per-group feature removal threshold, normalization) — the linear models assume a cleaned, imputed feature table
- The covariate is perfectly confounded with the group label, making the design matrix rank-deficient

## Inputs

- Preprocessed features CSV table (features as rows/columns, one row per sample after imputation and normalization, e.g. ST000284 targeted metabolomics three-group study)
- Target (metadata) CSV file with group labels and study covariates per sample
- Chosen FDR threshold for significance calling

## Outputs

- Significant feature set from plain t-test/ANOVA with FDR-corrected p-values
- Significant feature set from covariate-adjusted limma/ANCOVA models with FDR-corrected p-values
- covariate_comparison_table.csv with overlapping, unadjusted-only, and adjusted-only features

## How to apply

Load the preprocessed feature table and the target (metadata) file — POMAShiny requires two CSV inputs, the target/metadata file and the features file. First run the plain group-comparison univariate tests (t-test/ANOVA) per feature, then fit a linear model per feature using the limma framework (e.g. model.matrix(~ group + covariate) followed by lmFit and eBayes) or an ANCOVA, including the study covariates in the design matrix. Apply the same multiple-testing correction (e.g. Benjamini–Hochberg FDR) to both result sets and define significant feature sets at the chosen FDR threshold. Compute the overlap between the two significant sets with tidyverse set operations, splitting features into overlapping, unadjusted-only, and adjusted-only categories. The rationale is the paper's claim that covariate-combining methods such as ANCOVA or limma improve the accuracy and understanding of POMAShiny results, and that the POMA R/Bioconductor package supports running these analyses outside the web environment.

## Related tools

- **POMA** (R/Bioconductor package providing the statistical analysis functions (univariate tests) used as the baseline and enabling covariate-adjusted modeling outside the web environment) — https://github.com/nutrimetabolomics/POMA
- **limma** (Linear-model framework used to fit covariate-adjusted per-feature differential abundance models)
- **tidyverse** (Back-end package used for set operations and table assembly when comparing significant feature sets)
- **POMAShiny** (Web-based workflow whose statistical analysis panel motivates the covariate-adjusted vs. unadjusted comparison; based on POMA) — https://github.com/nutrimetabolomics/POMAShiny

## Examples

```
design <- model.matrix(~ group + covariate); fit <- eBayes(lmFit(feature_matrix, design)); topTable(fit, number = Inf, adjust = "BH")
```

## Evaluation signals

- Both result sets are FDR-corrected and significance is called at the same pre-declared threshold, so sets are directly comparable
- covariate_comparison_table.csv partitions every feature into exactly one of three disjoint categories: overlapping, unadjusted-only, adjusted-only
- Overlapping + unadjusted-only + adjusted-only feature counts sum to the total number of tested features
- Adjusted-model effect estimates remain consistent in direction with group means from the exploratory data analysis for the overlapping features
- Any covariate term in the design matrix is estimable (non-aliased with group) — the design matrix is full rank

## Limitations

- The paper claims covariate-combining methods improve accuracy and understanding of results, but no benchmarking or quantitative evaluation against other tools is reported
- POMAShiny input must be two CSV files (target/metadata and features); other formats require conversion before analysis
- Feature removal threshold (default 20% missing per study group) and imputation choices upstream can shift which features enter the differential models

## Evidence

- [results] POMAShiny requires two comma-separated value (CSV) files as an input: the target (or metadata) file and the features file: "POMAShiny requires two comma-separated value (CSV) files as an input: the target (or metadata) file and the features file"
- [results] Example dataset ST000284 is a targeted metabolomics three-group study: "The example data set ST000284 consists of a targeted metabolomics three-group study"
- [intro] POMAShiny back-end uses POMA, MSnbase and tidyverse R packages: [evidence span withheld — anchor status: not_found]
- [readme] POMA package enables reproducible, flexible analysis outside the web environment: "This tool is based on the POMA R/Bioconductor package, which increases the reproducibility and flexibility of the analysis outside the web environment."
- [readme] POMAShiny workflow panel structure including statistical analysis: "POMAShiny's workflow is structured in four sequential and well-defined panels: 1) data upload, 2) pre-processing, 3) EDA and 4) statistical analysis panels."
