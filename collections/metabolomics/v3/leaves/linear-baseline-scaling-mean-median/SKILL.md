---
name: linear-baseline-scaling-mean-median
description: Use when a metabolomics feature table (samples × features, e.g. the imputed
  cancerCell demo table from MetaboReport) shows sample-to-sample variation that should
  be reduced by per-sample scaling, and a simple mean- or median-intensity baseline
  is acceptable.
when_to_use_negative:
- The table is raw and not yet imputed — MetaboReport applies normalization during
  preprocessing after imputation
- A distribution-based correction is needed rather than a per-sample scalar scaling
  (use quantile or probabilistic quotient normalization instead)
- Normalization should be driven by specific reference signals (use internal standard-based
  normalization instead)
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: MetaboReport
  role: R Shiny app whose data preprocessing module implements linear baseline normalization
    (mean- and median-based) among its five normalization methods; can be run via
    run_app()
  repo: https://github.com/YonghuiDong/MetaboReport
- name: R
  role: Language in which MetaboReport is written; used to install the package and
    run the normalization workflow on the demo feature table
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btae373
    title: 'MetaboReport: from metabolomics data analysis to comprehensive reporting'
  source_content_hash: sha256:44b09200f37b2d838e67e479b4789f7908abcec62af0ff5bd0a6d39b5b25f0b2
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
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# linear-baseline-scaling-mean-median

## Summary

A sample-wise normalization method in MetaboReport's preprocessing suite that reduces sample-to-sample variation by linearly scaling each sample's feature intensities against a per-sample baseline computed as either the mean or the median intensity across all features in that sample.

## When to use

Use when a metabolomics feature table (samples × features, e.g. the imputed cancerCell demo table from MetaboReport) shows sample-to-sample variation that should be reduced by per-sample scaling, and a simple mean- or median-intensity baseline is acceptable.

## When NOT to use

- The table is raw and not yet imputed — MetaboReport applies normalization during preprocessing after imputation
- A distribution-based correction is needed rather than a per-sample scalar scaling (use quantile or probabilistic quotient normalization instead)
- Normalization should be driven by specific reference signals (use internal standard-based normalization instead)

## Inputs

- Imputed metabolomics feature table (samples × features), e.g. the cancerCell demo dataset shipped with the MetaboReport R package
- Choice of baseline statistic: per-sample mean intensity or per-sample median intensity

## Outputs

- CSV table of linear baseline-normalized feature intensities (one per baseline variant: mean-based and median-based)
- Each output preserves the input table's dimensions (same samples, same features)

## How to apply

Load the imputed feature table and compute, for each sample, a baseline scaling factor from the intensities of all its metabolite features — once using the per-sample mean and once using the per-sample median. Divide each feature intensity in a sample by that sample's baseline to produce the linear baseline-normalized table. Run the method twice, once per baseline choice, and export one CSV per variant. Validate that each output CSV preserves the dimensions of the input table (same samples and same features). This is one of the five normalization methods MetaboReport implements to reduce unwanted sample-to-sample variation, alongside internal standard-based, probabilistic quotient, and quantile normalization.

## Related tools

- **MetaboReport** (R Shiny app whose data preprocessing module implements linear baseline normalization (mean- and median-based) among its five normalization methods; can be run via run_app()) — https://github.com/YonghuiDong/MetaboReport
- **R** (Language in which MetaboReport is written; used to install the package and run the normalization workflow on the demo feature table)

## Examples

```
library(MetaboReport); run_app()
```

## Evaluation signals

- All normalized CSV outputs exist, one per method variant (mean baseline and median baseline)
- Each normalized table has the same dimensions as the input imputed feature table (same samples and same features)
- Per-sample total (or baseline) intensity is equalized across samples after division by the mean/median scaling factor
- No missing values are introduced — the input table was imputed before normalization
- Median-based and mean-based variants differ only by the choice of scaling factor, not by table structure

## Limitations

- Linear baseline scaling only corrects multiplicative per-sample offset; it cannot correct feature-specific or distributional distortions the way quantile or probabilistic quotient normalization can
- Analysis helpers in MetaboReport are internal and may evolve — the README notes implementation details can change without creating a compatibility contract, so normalization behavior should be re-verified per version
- Median- and mean-based baselines can diverge for samples with skewed feature-intensity distributions; the choice between them is a decision point with no automatic selection rule given in the article

## Evidence

- [other] Five normalization methods in MetaboReport preprocessing: [evidence span withheld — anchor status: not_found]
- [intro] Tool scope: preprocessing of metabolomics data: "Shiny app for metabolomics data preprocessing, statistical analysis, visualisation, and self-contained HTML reporting."
- [readme] Installation and launch of the tool: "After installation, you can use `library(MetaboReport); run_app()` or `MetaboReport::run_app()` to start the app."
- [methods] Demo dataset available in the package: "- `cancerCell (MetaboReport)`"
