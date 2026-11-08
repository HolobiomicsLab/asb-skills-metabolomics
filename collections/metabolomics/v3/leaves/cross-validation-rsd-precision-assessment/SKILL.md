---
name: cross-validation-rsd-precision-assessment
description: Use when a GC-MS (or analogous LC-MS) metabolomics study provides a pooled-QC
  training block plus an external validation QC set, and you need to measure how well
  a normalization method controls batch drift via RSD precision.
when_to_use_negative:
- The dataset contains only pooled QC samples with no independent external validation
  QC pools — RSDs on training QCs alone (e.g., SERRF's 19% median RSD) overstate real
  precision, as shown by the 34% median RSD on BioIVT validation.
- Only a small number of QC samples are available that cannot support five-fold cross-validation
  splits.
- The aim is absolute quantification rather than precision assessment of a drift-correction/normalization
  method.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3520
tools:
- name: R
  role: Primary environment (v3.6.3) in which the normalization methods are run and
    RSDs are computed via 'SERDA runner.R'
- name: reticulate
  role: R-Python bridge (v1.19) used to invoke the keras denoising autoencoder from
    R for SERDA normalization
- name: keras
  role: Implements the denoising autoencoder used for SERDA normalization (v2.7.0)
  repo: https://github.com/slfan2013/SERDA
- name: caret
  role: Used for five-fold cross-validation machinery in the R workflow (v6.0.86)
- name: pROC
  role: ROC analysis package included in the R session for downstream evaluation (v1.17.0.1)
- name: Shiny-SERRF
  role: Random-forest-based SERRF normalization baseline app; also provides the required
    xlsx input format specification
  repo: https://github.com/slfan2013/Shiny-SERRF
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.3390/metabo13080944
    title: Denoising Autoencoder Normalization for Large-Scale Untargeted Metabolomics
      by Gas Chromatography–Mass Spectrometry
  source_content_hash: sha256:17acf155ceb960bb04c35cdae532546c8714180f1c04cabafb6a52ef12174b25
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/slfan2013/SERDA
  tools:
  - R
  - Shiny-SERRF
  - caret
  - keras
  - pROC
  - reticulate
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# cross-validation-rsd-precision-assessment

## Summary

Quantifies normalization precision in large-scale untargeted GC-MS metabolomics by computing per-metabolite relative standard deviations (RSDs) under five-fold cross-validation on training pooled QC samples and on an independent external validation QC block, then benchmarking coverage of compounds below a 30% RSD threshold.

## When to use

Use when a GC-MS (or analogous LC-MS) metabolomics study provides a pooled-QC training block plus an external validation QC set, and you need to measure how well a normalization method controls batch drift via RSD precision.

## When NOT to use

- The dataset contains only pooled QC samples with no independent external validation QC pools — RSDs on training QCs alone (e.g., SERRF's 19% median RSD) overstate real precision, as shown by the 34% median RSD on BioIVT validation.
- Only a small number of QC samples are available that cannot support five-fold cross-validation splits.
- The aim is absolute quantification rather than precision assessment of a drift-correction/normalization method.

## Inputs

- GC-MS feature-intensity matrix in xlsx format with training pooled QC samples and an external validation QC block (e.g., BioIVT plasma pools)
- Sample metadata columns: _batch_, _sampleType_ (qc/sample/validate), _time_ (processing order), _label_; compound column _No_
- Compound annotations (e.g., 563 metabolite annotations) with per-sample peak intensities

## Outputs

- Per-metabolite normalized RSDs from five-fold cross-validation on the training QC block for each method
- Per-metabolite RSDs on the independent external validation QC samples for each method
- Wilcoxon signed-rank comparison p-values between methods
- Fraction (coverage %) of compounds meeting the <30% RSD precision threshold per method
- benchmark_results.csv with per-method CV/validation RSDs and coverage summaries

## How to apply

Load the feature-intensity xlsx containing both the training pooled QC samples and the external validation QC pools (e.g., BioIVT plasma pools) into R, keeping the '_batch_', '_sampleType_', '_time_' and '_label_' metadata columns required by the SERDA/SERRF input format. Run five-fold cross-validation on the training QC block for each normalization method (SERDA, batchwise-LOESS, SERRF) to obtain normalized RSDs, then apply each fitted normalization unchanged to the held-out validation QC pools and compute RSDs there as well. Judge a method by the per-metabolite RSD distribution: compare SERDA against baselines with a Wilcoxon signed-rank test, and compute the fraction of metabolites reaching the '<30% RSD' precision threshold, which ranged from 72-98% coverage for SERDA versus 50-81% for SERRF. Rationale: pooled-QC-only RSDs can be optimistically low (SERRF reached 19% median RSD on QCs but 34% on external BioIVT validation), so external validation coverage is the honest precision metric.

## Related tools

- **R** (Primary environment (v3.6.3) in which the normalization methods are run and RSDs are computed via 'SERDA runner.R')
- **reticulate** (R-Python bridge (v1.19) used to invoke the keras denoising autoencoder from R for SERDA normalization)
- **keras** (Implements the denoising autoencoder used for SERDA normalization (v2.7.0)) — https://github.com/slfan2013/SERDA
- **caret** (Used for five-fold cross-validation machinery in the R workflow (v6.0.86))
- **pROC** (ROC analysis package included in the R session for downstream evaluation (v1.17.0.1))
- **Shiny-SERRF** (Random-forest-based SERRF normalization baseline app; also provides the required xlsx input format specification) — https://github.com/slfan2013/Shiny-SERRF

## Examples

```
Rscript SERDA\ runner.R  # after editing data_file = "filelocation.xlsx" to point at the xlsx containing training QCs and validate samples
```

## Evaluation signals

- Per-metabolite RSDs are computed both on five-fold cross-validated training QCs and on the independent external validation QC pools; both sets must be reported.
- A Wilcoxon signed-rank test comparing per-metabolite RSDs between methods (e.g., SERDA vs SERRF) is performed; a significant p-value supports a real precision difference.
- Coverage of compounds below the 30% RSD threshold is calculated per method; SERDA should reach the 72-98% range on the three GC-MS datasets rather than the 50-81% SERRF baseline.
- Median RSD values should be reported separately for training QCs and external validation (e.g., SERDA 16% vs 19%; SERRF 19% vs 34%).
- The 'benchmark_results.csv' output must contain per-method CV RSDs, validation RSDs, Wilcoxon p-value, and <30% RSD coverage percentages.

## Limitations

- Pooled-QC RSDs alone are optimistic: SERRF showed 19% median RSD on training QCs but 34% on external BioIVT validation pools, so external validation is mandatory for honest precision claims.
- Neither internal standards nor sum-based normalizations achieved median precision below 30% across all 563 metabolite annotations in this study.
- The denoising-autoencoder (SERDA) path requires a Python 3.6.10 / keras 2.7.0 environment configured through reticulate (r-miniconda r-reticulate env), which adds setup complexity over pure-R baselines.

## Evidence

- [abstract] SERRF gave 19% median precision on pooled QC samples but 34% median RSD on external BioIVT validation pools: "SERRF algorithm gave 19% median precision based on the pooled quality control samples, external cross-validation with BioIVT plasma pools yielded a median 34% relative standard deviation (RSD)"
- [abstract] SERDA reduced median RSD of training QC samples to 16% and achieved 19% RSD on independent BioIVT validation QC samples: "SERDA lowered the median standard deviations of the training QC samples down to 16% RSD, yielding an overall error of 19% RSD when applied to the independent BioIVT validation QC samples"
- [abstract] Neither internal standards nor sum-based normalizations achieved median precision below 30% across all 563 metabolite annotations: "neither internal standards nor sum-based normalizations yielded median precision of less than 30% across all 563 metabolite annotations"
- [readme] Required metadata columns for the input file format: [evidence span withheld — anchor status: external_doc]
- [readme] SampleType may carry a validate class for external validation samples: [evidence span withheld — anchor status: external_doc]
