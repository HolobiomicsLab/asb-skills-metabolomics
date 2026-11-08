---
name: metabolite-feature-table-normalization
description: Use when an imputed metabolomics feature table (samples x features, e.g.
  the cancerCell MetaboReport demo dataset) still shows sample-to-sample variation
  that must be reduced before statistical analysis or reporting.
when_to_use_negative:
- Input is raw instrument data (e.g. mzML/vendor files) rather than an already peak-picked
  and imputed feature table — MetaboReport normalization operates on feature tables
- The feature table still contains missing values that have not been imputed — normalization
  methods such as PQN and quantile require a complete matrix
- The goal is statistical analysis, visualization, or report generation rather than
  correcting sample-to-sample variation in intensity
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: MetaboReport
  role: R Shiny app whose data preprocessing modules implement the five normalization
    methods applied to the feature table
  repo: https://github.com/YonghuiDong/MetaboReport
- name: R
  role: Runtime environment; MetaboReport can be installed as an R package and launched
    with run_app()
- name: Shiny
  role: Framework hosting the MetaboReport preprocessing modules used for normalization
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btae373
    title: 'MetaboReport: from metabolomics data analysis to comprehensive reporting'
  source_content_hash: sha256:315ff909d45f0f96eaba76d1b8c69a7314031e21345e672482791a87b91d6454
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YonghuiDong/MetaboReport
  tools:
  - MetaboReport
  - R
  - Shiny
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# metabolite-feature-table-normalization

## Summary

Reduce unwanted sample-to-sample variation in metabolomics feature tables by applying the five normalization methods implemented in MetaboReport (internal standard-based, linear baseline by mean, linear baseline by median, probabilistic quotient, and quantile normalization).

## When to use

Use when an imputed metabolomics feature table (samples x features, e.g. the cancerCell MetaboReport demo dataset) still shows sample-to-sample variation that must be reduced before statistical analysis or reporting.

## When NOT to use

- Input is raw instrument data (e.g. mzML/vendor files) rather than an already peak-picked and imputed feature table — MetaboReport normalization operates on feature tables
- The feature table still contains missing values that have not been imputed — normalization methods such as PQN and quantile require a complete matrix
- The goal is statistical analysis, visualization, or report generation rather than correcting sample-to-sample variation in intensity

## Inputs

- Imputed metabolomics feature table (samples x metabolite features), e.g. the cancerCell demo dataset shipped with MetaboReport
- Internal standard signal per sample (for internal standard-based normalization)
- Reference sample or reference feature profile (for probabilistic quotient normalization)

## Outputs

- One normalized CSV feature table per method (internal standard-based, linear baseline mean, linear baseline median, PQN, quantile), each preserving the input dimensions

## How to apply

Load an imputed feature table such as the cancerCell demo dataset shipped with the MetaboReport R package and run it through MetaboReport's preprocessing modules, which implement five normalization methods: internal standard-based normalization (divide each metabolite feature by the internal standard signal in the same sample), linear baseline normalization using either each sample's mean or its median intensity as the scaling factor, probabilistic quotient normalization (scale each sample by the median of its feature-wise quotient ratios relative to a reference sample), and quantile normalization (force all sample intensity distributions to a shared empirical distribution). Choose among the five methods based on whether internal standards are available and whether the goal is to correct overall signal intensity (baseline/PQN) or full distribution shape (quantile). Export one CSV table per normalization method. MetaboReport is an R Shiny app built with golem, so the normalization is run inside the app's preprocessing workflow rather than as isolated scripts.

## Related tools

- **MetaboReport** (R Shiny app whose data preprocessing modules implement the five normalization methods applied to the feature table) — https://github.com/YonghuiDong/MetaboReport
- **R** (Runtime environment; MetaboReport can be installed as an R package and launched with run_app())
- **Shiny** (Framework hosting the MetaboReport preprocessing modules used for normalization)

## Examples

```
remotes::install_github("YonghuiDong/MetaboReport"); library(MetaboReport); run_app()
```

## Evaluation signals

- All five normalized CSV files are produced (internal standard-based, linear baseline mean, linear baseline median, PQN, quantile)
- Each normalized table preserves the dimensions of the imputed input table (same samples and same features)
- After internal standard-based normalization, each feature is scaled by the internal standard signal measured in the same sample
- After quantile normalization, the feature intensity distributions of all samples follow the same empirical distribution
- PQN scaling factors equal the median of feature-wise quotient ratios of each sample relative to the reference sample

## Limitations

- Normalization reduces sample-to-sample variation but is applied only during data preprocessing; downstream statistics and reporting are separate MetaboReport modules
- The five methods make different assumptions (availability of internal standards, need for a reference sample for PQN), so method choice is a decision point rather than a default
- The public package API is intentionally limited to run_app(); the normalization implementations are internal helpers that can evolve without a compatibility contract

## Evidence

- [results] MetaboReport implements five normalization methods to reduce sample-to-sample variation: [evidence span withheld — anchor status: not_found]
- [abstract] abstract quote on MetaboReport scope: "we introduced MetaboReport, a versatile and interactive web app that simplifies metabolomics experiment design, data preprocessing, exploration, statistical analysis, visualization, and reporting"
- [readme] readme quote on Shiny app scope: "Shiny app for metabolomics data preprocessing, statistical analysis, visualisation, and self-contained HTML reporting."
- [readme] readme install command: "remotes::install_github("YonghuiDong/MetaboReport")"
- [methods] methods quote on modular structure: "`R/mod_*.R` contains Shiny modules."
