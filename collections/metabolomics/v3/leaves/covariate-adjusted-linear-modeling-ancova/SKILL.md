---
name: covariate-adjusted-linear-modeling-ancova
description: Use when a metabolomics or proteomics feature table has study covariates
  (metadata beyond the primary group label) and you need to test whether covariate
  adjustment changes which features are statistically significant versus plain ANOVA/t-tests.
when_to_use_negative:
- The analysis goal is only plain two-group or multi-group testing with no covariates
  of interest — POMA's univariate tests alone suffice
- The data are raw spectra requiring upstream MS processing — the paper directs that
  work to the RforMassSpectrometry packages before migrating to POMA for statistical
  analysis
- The metadata file lacks covariate columns beyond the group label, so no adjusted
  model can be fit
tools:
- name: POMA
  role: R/Bioconductor package providing the univariate group-comparison statistical
    tests and the reproducible backend for analysis outside the web environment
  repo: https://github.com/nutrimetabolomics/POMA
- name: POMAShiny
  role: Reference workflow whose data upload (target + features CSV) and statistical
    analysis panel structure the comparison
  repo: https://github.com/nutrimetabolomics/POMAShiny
- name: tidyverse
  role: Backend R package family used for data manipulation and tidy set operations
    when assembling the comparison table
- name: limma
  role: Linear-model framework used to fit covariate-adjusted per-feature models with
    empirical-Bayes moderation and FDR correction
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1371/journal.pcbi.1009148
    title: 'POMAShiny: A user-friendly web-based workflow for metabolomics and proteomics
      data analysis'
  source_content_hash: sha256:ba86d1a611ef045699907e600ac8269992cd8662f93fb94d971ba874455b13c3
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
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 15
---

# covariate-adjusted-linear-modeling-ancova

## Summary

Method for reanalyzing POMAShiny/POMA univariate metabolomics results (e.g. ST000284 three-group targeted metabolomics data) with covariate-adjusted linear models (ANCOVA and the limma framework) and comparing significant feature sets against plain t-test/ANOVA results. The paper claims that covariate-combining methods such as ANCOVA or limma improve the accuracy and understanding of POMAShiny analyses.

## When to use

Use when a metabolomics or proteomics feature table has study covariates (metadata beyond the primary group label) and you need to test whether covariate adjustment changes which features are statistically significant versus plain ANOVA/t-tests.

## When NOT to use

- The analysis goal is only plain two-group or multi-group testing with no covariates of interest — POMA's univariate tests alone suffice
- The data are raw spectra requiring upstream MS processing — the paper directs that work to the RforMassSpectrometry packages before migrating to POMA for statistical analysis
- The metadata file lacks covariate columns beyond the group label, so no adjusted model can be fit

## Inputs

- Preprocessed feature table (rows = metabolites/features, columns = samples) as CSV
- Target/metadata file mapping each sample to its study group and covariates, as CSV
- Chosen FDR threshold for significance calls

## Outputs

- Significant-feature list from plain t-test/ANOVA (multiple-testing corrected)
- Significant-feature list from covariate-adjusted ANCOVA/limma models (multiple-testing corrected)
- covariate_comparison_table.csv with overlapping, unadjusted-only, and adjusted-only features

## How to apply

Load the preprocessed feature table and the target (metadata) file — the same two CSV inputs POMAShiny requires — into R, which is the backend language shared by POMAShiny and the POMA R/Bioconductor package. First run plain group-comparison univariate tests (t-test for two groups, ANOVA for three groups, as in the ST000284 three-group targeted metabolomics study) on each feature using POMA's statistical analysis functions. Then fit covariate-adjusted models for the same features with ANCOVA and the limma linear-model framework, building a design matrix that includes the study covariates alongside the group term. Apply multiple-testing correction to both result sets, define significant feature sets at the chosen FDR threshold, and use tidyverse set operations to compute the overlap, producing three categories: overlapping features, unadjusted-only features, and adjusted-only features.

## Related tools

- **POMA** (R/Bioconductor package providing the univariate group-comparison statistical tests and the reproducible backend for analysis outside the web environment) — https://github.com/nutrimetabolomics/POMA
- **POMAShiny** (Reference workflow whose data upload (target + features CSV) and statistical analysis panel structure the comparison) — https://github.com/nutrimetabolomics/POMAShiny
- **tidyverse** (Backend R package family used for data manipulation and tidy set operations when assembling the comparison table)
- **limma** (Linear-model framework used to fit covariate-adjusted per-feature models with empirical-Bayes moderation and FDR correction)

## Examples

```
Rscript -e "library(POMA); library(limma); design <- model.matrix(~ group + covariate, data = target); fit <- eBayes(lmFit(features, design))"
```

## Evaluation signals

- The plain-test and adjusted-test result sets are computed on the identical feature list, so their union/intersection is well-defined with no unmatched features
- Both feature sets have been multiple-testing corrected at the same chosen FDR threshold before comparison
- The comparison table partitions all features into exactly three disjoint categories: overlap, unadjusted-only, adjusted-only
- Adjusted-only features are biologically/study-design sensible given the covariate included in the design matrix
- The input CSVs conform to POMAShiny's two-file convention: a target (metadata) file and a features file

## Limitations

- POMAShiny's web interface was built against specific package versions (e.g. bs4Dash 0.5.0; the README notes it 'is not compatible with newer versions of bs4Dash due to substantial breaking changes in the API'), so the R-side replication must pin compatible POMA/dependency versions
- The article reports no benchmarking or quantitative evaluation of the statistical workflow against other tools
- No software version number is stated for POMAShiny or POMA in the article, complicating exact reproduction

## Evidence

- [results] POMAShiny requires two comma-separated value (CSV) files as an input: the target (or metadata) file and the features file: "POMAShiny requires two comma-separated value (CSV) files as an input: the target (or metadata) file and the features file"
- [abstract] POMA backend and reproducibility: "it is based on the POMA R/Bioconductor package, which increases the reproducibility and flexibility of analyses outside the web environment"
- [intro] Backend package stack: [evidence span withheld — anchor status: not_found]
- [readme] README dependency installation including POMA and tidyverse: "pk2 <- c('Biobase', 'POMA', 'MSnbase')"
