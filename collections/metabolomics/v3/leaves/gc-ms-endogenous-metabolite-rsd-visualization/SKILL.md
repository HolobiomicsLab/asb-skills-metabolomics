---
name: gc-ms-endogenous-metabolite-rsd-visualization
description: Use when GC-MS metabolomics data with pooled QC samples must be compared
  across normalization methods (one-to-one ISTD, SERDA, SERRF, one-to-class) and summarized
  as a median-RSD bar/figure with a CSV table.
when_to_use_negative:
- The dataset lacks pooled QC samples (sampleType = qc) or batch/time/label annotations
  required by the SERRF input format, so per-method QC RSDs cannot be computed.
- The goal is SERDA model training or the full 563-metabolite precision assessment
  rather than the focused 23-compound internal-standard comparison and its visualization.
- The metabolites of interest have no one-to-one deuterated internal standard and
  are not in the sugar/sugar-alcohol class testable via sorbitol-d8 one-to-class.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: R
  role: Runs SERDA runner.R end-to-end and computes per-compound pooled-QC RSDs plus
    the Figure 7 comparison plot and CSV
- name: reticulate
  role: Bridges R to the Python backend required by SERDA's denoising autoencoder
    normalization
- name: keras
  role: Provides the denoising-autoengine backend used to generate SERDA-normalized
    peak areas before RSD computation
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.3390/metabo13080944
    title: Denoising Autoencoder Normalization for Large-Scale Untargeted Metabolomics
      by Gas Chromatography–Mass Spectrometry
  source_content_hash: sha256:e48f8daf269ebe9372064d8333627063ea58bdb703ad8679a81192a4e16d9ee8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - keras
  - reticulate
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# gc-ms-endogenous-metabolite-rsd-visualization

## Summary

Replicates the Figure 7 / Supplementary Figure S2 comparison of median pooled-QC relative standard deviations (RSDs) across normalization strategies — one-to-one deuterated internal standards, SERDA, and SERRF — for 23 endogenous GC-MS plasma metabolites, plus the sorbitol-d8 one-to-class counterexample on 20 sugars/sugar alcohols.

## When to use

Use when GC-MS metabolomics data with pooled QC samples must be compared across normalization methods (one-to-one ISTD, SERDA, SERRF, one-to-class) and summarized as a median-RSD bar/figure with a CSV table.

## When NOT to use

- The dataset lacks pooled QC samples (sampleType = qc) or batch/time/label annotations required by the SERRF input format, so per-method QC RSDs cannot be computed.
- The goal is SERDA model training or the full 563-metabolite precision assessment rather than the focused 23-compound internal-standard comparison and its visualization.
- The metabolites of interest have no one-to-one deuterated internal standard and are not in the sugar/sugar-alcohol class testable via sorbitol-d8 one-to-class.

## Inputs

- 'SERRF example dataset - with validate.xlsx' (peak-area matrix with _batch_, _sampleType_ (qc/sample/validate), _time_, _label_, and _No_ columns)
- SERDA runner.R script with editable data_file = "filelocation.xlsx" path
- List of 23 endogenous compounds with matched deuterated internal standards
- Mapping of the 20 sugars/sugar alcohols for the sorbitol-d8 one-to-class test

## Outputs

- rsd_comparison.csv (per-compound and median RSDs by normalization method)
- figure7_rsd_comparison.png (Figure 7 / Supplementary Figure S2 style median-RSD comparison plot)
- SERDA-normalized peak areas from the runner.R script

## How to apply

First generate normalized peak areas by running the SERDA runner.R script (edited with data_file = "filelocation.xlsx") through R with the reticulate/keras Python backend, and ensure the input xlsx carries the SERRF-required columns _batch_, _sampleType_, _time_, _label_, and _No_. Restrict to the 23 endogenous compounds that each have a matching deuterated internal standard and compute per-compound RSDs across pooled QC samples under one-to-one ISTD, SERDA, and SERRF normalization, checking medians against the reported targets of 19%, 3.8%, and 5% RSD respectively. Then compute RSDs for sorbitol-d8 one-to-class normalization restricted to the 20 sugars/sugar alcohols to demonstrate its failure relative to SERDA and SERRF. Finally, replicate the Figure 7 / Supplementary Figure S2 plot of median RSDs by normalization method and write rsd_comparison.csv and figure7_rsd_comparison.png.

## Related tools

- **R** (Runs SERDA runner.R end-to-end and computes per-compound pooled-QC RSDs plus the Figure 7 comparison plot and CSV)
- **reticulate** (Bridges R to the Python backend required by SERDA's denoising autoencoder normalization)
- **keras** (Provides the denoising-autoengine backend used to generate SERDA-normalized peak areas before RSD computation)

## Examples

```
Rscript -e 'data_file <- "filelocation.xlsx"; source("SERDA runner.R")'
```

## Evaluation signals

- Median RSD across the 23 compounds under one-to-one ISTD normalization is ~19%, matching the reported SERRF pooled-QC median.
- Median RSD under SERDA is ~3.8% and under SERRF is ~5%, i.e. SERDA < SERRF < one-to-one ISTD for this subset.
- Sorbitol-d8 one-to-class RSD on the 20 sugars/sugar alcohols is clearly worse than both SERDA and SERRF, reproducing the Figure 7 counterexample.
- rsd_comparison.csv contains one RSD per compound per method (ISTD, SERDA, SERRF, one-to-class) with medians consistent with the figure.
- As a global sanity check, neither ISTD nor sum-based normalization should push the median precision of all 563 metabolite annotations below 30%.

## Limitations

- SERRF's 19% median precision on pooled QC samples did not transfer externally: BioIVT validation pools gave a median 34% RSD, so in-cohort RSDs overestimate external performance.
- The SERDA runtime environment is version-sensitive: R 3.6.3, reticulate 1.19, keras 2.7.0, Python 3.6.10 with numpy 1.18.1 under Windows; other versions may not reproduce the numbers.
- Samples that should not be normalized (e.g. blanks) must be left with empty cells in the input xlsx, per the SERRF README.

## Evidence

- [results] For the same 23 endogenous compounds, SERDA normalization achieved a much lower technical error of 3.8% RSD, and even SERRF outperformed the one-to-one use of internal standards, with 5% RSD (Figure 7a).: [evidence span withheld — anchor status: not_found]
- [abstract] SERRF gave 19% median precision on pooled QC samples but 34% median RSD on external BioIVT validation pools: "SERRF algorithm gave 19% median precision based on the pooled quality control samples, external cross-validation with BioIVT plasma pools yielded a median 34%"
- [methods] Open SERDA runner.R, edit data_file path, and run all script lines: "oad `SERDA runner.R`. Open it and edit `data_file = "filelocation.xlsx"` and run all the script lines."
- [readme] SERRF input file format requires batch, sampleType, time, label for samples, and No for compounds: [evidence span withheld — anchor status: external_doc]
- [abstract] Neither internal standards nor sum-based normalizations achieved median precision below 30% across all 563 metabolite annotations: "neither internal standards nor sum-based normalizations yielded median precision of less than 30% across all 563 metabolite annotations"
