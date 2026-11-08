---
name: multi-tissue-metabolomics-group-comparison
description: Use when a metabolomics feature table has many biological groups (e.g.
  14 tissues) and the goal is to identify features dramatically enriched in one group
  versus all others at a strict FDR threshold (e.g. P<0.001, >1000-fold change).
when_to_use_negative:
- The dataset has only two groups or no single tissue/group expected to dominate abundance
  — a standard pairwise comparison without the >1000-fold filter is more appropriate
- The data are positive-ion-mode or already normalized/summarized to identified metabolites
  rather than raw mass features — the thresholds and row-count validation are specific
  to negative-ion-mode mass-feature tables
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3520
tools:
- name: MetaboReport
  role: R Shiny app providing the statistical analysis, data preprocessing, visualization,
    and reporting functions used to run the between-group abundance test and generate
    the HTML report
  repo: https://github.com/YonghuiDong/MetaboReport
- name: R
  role: Runtime environment in which MetaboReport is installed as an R package and
    its analysis helpers (R/fct_*.R) are executed
  repo: https://github.com/YonghuiDong/MetaboReport
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btae373
    title: 'MetaboReport: from metabolomics data analysis to comprehensive reporting'
  source_content_hash: sha256:a4850c15a5058c28e2f5932ae6740b9d3d4f23ca962aae10e7e666037da14833
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YonghuiDong/MetaboReport
  tools:
  - MetaboReport
  - R
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# multi-tissue-metabolomics-group-comparison

## Summary

A statistical differential-abundance workflow for multi-tissue LC-MS metabolomics feature tables, exemplified by comparing one tissue group (buccal gland) against the pooled remaining 13 tissue groups in negative-ion-mode data, using MetaboReport's analysis functions with Benjamini-Hochberg FDR correction and fold-change filtering.

## When to use

Use when a metabolomics feature table has many biological groups (e.g. 14 tissues) and the goal is to identify features dramatically enriched in one group versus all others at a strict FDR threshold (e.g. P<0.001, >1000-fold change).

## When NOT to use

- The dataset has only two groups or no single tissue/group expected to dominate abundance — a standard pairwise comparison without the >1000-fold filter is more appropriate
- The data are positive-ion-mode or already normalized/summarized to identified metabolites rather than raw mass features — the thresholds and row-count validation are specific to negative-ion-mode mass-feature tables

## Inputs

- MetaboReport Dataset 1 feature table (cancerCell example data), negative-ion-mode LC-MS conditions
- Sample metadata mapping 94 samples to 14 lamprey tissue groups

## Outputs

- Filtered differential abundance table (differential_abundance_buccal_gland.csv) containing features with fold change >1000 and FDR-adjusted P<0.001
- Per-feature between-group test P values with Benjamini-Hochberg FDR adjustment
- MetaboReport comprehensive HTML report with statistical results, interactive plots and tables

## How to apply

Load the feature table (e.g. the cancerCell example dataset shipped with MetaboReport, under negative-ion-mode conditions) into R, then run MetaboReport's between-group statistical analysis helpers (R/fct_*.R functions) comparing the target group — buccal gland — against the pooled other 13 tissue groups. Apply Benjamini-Hochberg FDR adjustment to the resulting per-feature P values. Compute per-feature abundance fold change (target group vs. others) and filter to features more than 1000-times higher in the target group while retaining FDR-adjusted P<0.001. Export the filtered differential abundance table (e.g. differential_abundance_buccal_gland.csv) and validate the row count against the expected metric (182 features).

## Related tools

- **MetaboReport** (R Shiny app providing the statistical analysis, data preprocessing, visualization, and reporting functions used to run the between-group abundance test and generate the HTML report) — https://github.com/YonghuiDong/MetaboReport
- **R** (Runtime environment in which MetaboReport is installed as an R package and its analysis helpers (R/fct_*.R) are executed) — https://github.com/YonghuiDong/MetaboReport

## Examples

```
library(MetaboReport); run_app()
```

## Evaluation signals

- The exported differential_abundance_buccal_gland.csv contains exactly 182 rows under buccal-gland-tissue / negative-ion-mode conditions
- Every retained feature has FDR-adjusted P<0.001 after Benjamini-Hochberg correction
- Every retained feature shows more than 1000-times higher abundance in buccal gland than in each of the other 13 tissues
- The analysis ran via MetaboReport's internal analysis helpers (R/fct_*.R) rather than ad-hoc code, and package checks pass (testthat::test_local(), rcmdcheck::rcmdcheck())
- The generated HTML report includes the statistical results, interactive plots and tables consistent with the filtered table

## Limitations

- The public package API is intentionally limited to run_app(); analysis helpers are internal, so their implementation can evolve and direct programmatic use is not a compatibility contract
- The validated result (182 features, FDR P<0.001) is specific to the lamprey multi-tissue dataset (94 samples, 14 tissues, negative ion mode); other datasets will yield different counts
- Report generation historically had bugs fixed only in v0.3.6, so older versions may produce incomplete reports

## Evidence

- [intro] Shiny app for metabolomics data preprocessing, statistical analysis, visualisation, and self-contained HTML reporting.: "Shiny app for metabolomics data preprocessing, statistical analysis, visualisation, and self-contained HTML reporting."
- [readme] R/fct_*.R contains pure analysis, input/output, plotting, and report functions.: "`R/fct_*.R` contains pure analysis, input/output, plotting, and report
  functions."
- [readme] you can use `library(MetaboReport); run_app()` or `MetaboReport::run_app()` to start the app: "you can use `library(MetaboReport); run_app()` or `MetaboReport::run_app()` to start the app."
- [abstract] simplifies metabolomics experiment design, data preprocessing, exploration, statistical analysis, visualization, and reporting: "we introduced MetaboReport, a versatile and interactive web app that simplifies metabolomics experiment design, data preprocessing, exploration, statistical analysis, visualization, and reporting"
- [readme] The supported public package API is intentionally small: `run_app()`.: "The supported public package API is intentionally small: `run_app()`."
