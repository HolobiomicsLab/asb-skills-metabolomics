---
name: internal-standard-based-sample-correction
description: Use when a metabolomics feature table contains samples spiked with one
  or more internal standards and unwanted sample-to-sample variation (e.g., from extraction
  or injection variability) must be corrected before statistical analysis or reporting.
when_to_use_negative:
- The dataset contains no internal standards, making per-sample scaling by a standard
  signal impossible — use probabilistic quotient or quantile normalization instead.
- The input is a raw (unimputed) feature table that still requires missing-value handling
  before normalization.
- The goal is to force identical empirical intensity distributions across samples,
  which quantile normalization — not internal-standard correction — provides.
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: MetaboReport
  role: R Shiny web app whose preprocessing module implements internal standard-based
    normalization alongside four other normalization methods for metabolomics feature
    tables
  repo: https://github.com/YonghuiDong/MetaboReport
- name: R
  role: Language in which MetaboReport is installed as an R package and where the
    normalization routine can be reimplemented for the demo dataset
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btae373
    title: 'MetaboReport: from metabolomics data analysis to comprehensive reporting'
  source_content_hash: sha256:9f6fc4b072fe9216a1c5b5fbd2178252022aa438b16baba3ab44043dd68dd181
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YonghuiDong/MetaboReport
  tools:
  - MetaboReport
  - R
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# internal-standard-based-sample-correction

## Summary

Normalizes metabolomics feature tables by dividing each metabolite feature by the internal standard signal measured in the same sample, reducing sample-to-sample variation from injection or extraction differences. This is one of five normalization methods implemented in MetaboReport for data preprocessing.

## When to use

Use when a metabolomics feature table contains samples spiked with one or more internal standards and unwanted sample-to-sample variation (e.g., from extraction or injection variability) must be corrected before statistical analysis or reporting.

## When NOT to use

- The dataset contains no internal standards, making per-sample scaling by a standard signal impossible — use probabilistic quotient or quantile normalization instead.
- The input is a raw (unimputed) feature table that still requires missing-value handling before normalization.
- The goal is to force identical empirical intensity distributions across samples, which quantile normalization — not internal-standard correction — provides.

## Inputs

- Imputed metabolomics feature table (samples x features) with internal standard signals present, e.g., the cancerCell demo dataset from MetaboReport

## Outputs

- Normalized feature table with the same dimensions (same samples and same features) as the input, suitable for export as CSV

## How to apply

Load the imputed feature table (e.g., the cancerCell demo dataset shipped with MetaboReport) as a samples-by-features matrix. For each sample, identify the internal standard signal measured within that same sample, then divide every metabolite feature value in that sample by the internal standard signal, so that each sample is scaled relative to its own standard. This method is one of five normalization options MetaboReport implements to reduce sample-to-sample variation, alongside linear baseline normalization (mean or median), probabilistic quotient normalization, and quantile normalization. Apply the correction after imputation of missing values and before downstream statistical analysis or HTML report generation. Export the normalized table (e.g., as CSV) and verify it preserves the input dimensions — the same samples and the same features.

## Related tools

- **MetaboReport** (R Shiny web app whose preprocessing module implements internal standard-based normalization alongside four other normalization methods for metabolomics feature tables) — https://github.com/YonghuiDong/MetaboReport
- **R** (Language in which MetaboReport is installed as an R package and where the normalization routine can be reimplemented for the demo dataset)

## Examples

```
library(MetaboReport); run_app()  # then load the imputed cancerCell feature table and select internal standard-based normalization in the preprocessing module
```

## Evaluation signals

- The normalized table has exactly the same dimensions (same samples and same features) as the imputed input table.
- Every feature value in each sample is divided by that sample's own internal standard signal — spot-check that ratios reproduce the original-over-standard computation.
- Normalized CSV output exists and is well-formed for export alongside the other four MetaboReport normalization methods.
- Sample-to-sample variation (e.g., spread of per-sample totals or internal-standard-anchored scaling factors) is reduced relative to the imputed input.
- No missing values are reintroduced where the input was imputed and the internal standard signal was nonzero.

## Limitations

- Requires a measurable internal standard signal in every sample; samples missing or lacking the standard cannot be corrected by this method.
- Corrects only per-sample scaling differences attributable to the internal standard; it does not equalize feature-wise distributions across samples the way quantile normalization does.
- The public MetaboReport package API is intentionally small (run_app()), so analysis helpers implementing normalization are internal and their implementation can evolve without a compatibility contract.

## Evidence

- [full_text] internal standard-based normalization: [evidence span withheld — anchor status: not_found]
- [abstract] Shiny app scope: "we introduced MetaboReport, a versatile and interactive web app that simplifies metabolomics experiment design, data preprocessing, exploration, statistical analysis, visualization, and reporting"
- [methods] demo dataset: "cancerCell (MetaboReport)"
- [readme] installation: "remotes::install_github("YonghuiDong/MetaboReport")"
- [readme] public API: "The supported public package API is intentionally small: `run_app()`. Analysis helpers are internal so their implementation can evolve without creating an accidental compatibility contract."
