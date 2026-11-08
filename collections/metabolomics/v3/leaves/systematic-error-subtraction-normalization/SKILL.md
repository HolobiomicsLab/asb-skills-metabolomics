---
name: systematic-error-subtraction-normalization
description: Use when large-scale untargeted GC-MS metabolomics batches (thousands
  of study samples plus pooled QC runs, e.g. an xlsx feature table with batch/sampleType/time
  metadata) show injection-order drift that neither internal standards nor sum-based
  normalization can remove.
when_to_use_negative:
- Sample sets are blank samples or other samples the user does not want normalized
  — the Shiny-SERRF README instructs leaving such sampleType cells empty so they are
  excluded
- Feature table lacks batch/sampleType/time/label metadata, so batches and processing
  order cannot be defined for training the autoencoder on pooled QC samples
- The analysis is a small batch where sum-based or internal-standard normalization
  already yields median RSD below ~30%; SERDA's value was demonstrated on the 563-metabolite,
  >4000-sample, 1.2-year GC-MS study
edam_operation: http://edamontology.org/operation_3665
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0203
tools:
- name: R
  role: Hosts SERDA runner.R; the runner script and reticulate-driven training/prediction
    are executed in R 3.6.3
- name: keras
  role: Trains the denoising autoencoder (elu activation, dropout, Adam mini-batch
    optimization of W, b, W', b') used to model systematic error
- name: reticulate
  role: Bridges R and the Python 3.6.10 r-reticulate environment so keras can be driven
    from SERDA runner.R
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.3390/metabo13080944
    title: Denoising Autoencoder Normalization for Large-Scale Untargeted Metabolomics
      by Gas Chromatography–Mass Spectrometry
  source_content_hash: sha256:f137f2b66649d00243bed328352f5672b8a7276745ed6a7c854fe8f88f419ba0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - keras
  - reticulate
  edam_operation: http://edamontology.org/operation_3665
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# systematic-error-subtraction-normalization

## Summary

SERDA normalization: train a denoising autoencoder on pooled quality-control samples from large-scale GC-MS metabolomics batches, then predict and subtract the systematic-error component Φ(x2) from target (study) samples to yield normalized intensities.

## When to use

Use when large-scale untargeted GC-MS metabolomics batches (thousands of study samples plus pooled QC runs, e.g. an xlsx feature table with batch/sampleType/time metadata) show injection-order drift that neither internal standards nor sum-based normalization can remove.

## When NOT to use

- Sample sets are blank samples or other samples the user does not want normalized — the Shiny-SERRF README instructs leaving such sampleType cells empty so they are excluded
- Feature table lacks batch/sampleType/time/label metadata, so batches and processing order cannot be defined for training the autoencoder on pooled QC samples
- The analysis is a small batch where sum-based or internal-standard normalization already yields median RSD below ~30%; SERDA's value was demonstrated on the 563-metabolite, >4000-sample, 1.2-year GC-MS study

## Inputs

- xlsx feature table of pooled QC (training) and target study samples from GC-MS batches, with batch, sampleType, time, label, and compound No columns (SERRF example dataset - with validate.xlsx)
- R environment with reticulate 1.19 and keras 2.7.0 configured to a Python 3.6.10 r-miniconda env (r-reticulate)

## Outputs

- serda_normalized_data.xlsx — the target (study) sample feature table after subtracting the predicted systematic error Φ(x2) and exponentiating back to raw-intensity scale

## How to apply

Download the SERRF example dataset (.xlsx) from the Shiny-SERRF repository and point SERDA runner.R's data_file variable at it. In R, glog-transform the training and target peak intensities, corrupt them with Gaussian noise scaled between sigma2 and sigma1, optionally oversample, and auto-scale features; train the denoising autoencoder in keras (via reticulate) using elu activation, dropout, and mini-batch Adam optimization of W, b, W', b' with early stopping on a 20% validation split. Predict the systematic error Φ(x2) on the target samples, subtract it (centered by its mean) from the observed intensities, optionally apply median normalization, and exponentiate back to the original intensity scale. Export the normalized feature table as serda_normalized_data.xlsx. Rationale: the trained model Φ(x) captures systematic error learned from repeated pooled QC measurements, so removing Φ(x2) reduces median RSD in target samples the way it reduced training QC RSD to 16%.

## Related tools

- **R** (Hosts SERDA runner.R; the runner script and reticulate-driven training/prediction are executed in R 3.6.3)
- **keras** (Trains the denoising autoencoder (elu activation, dropout, Adam mini-batch optimization of W, b, W', b') used to model systematic error)
- **reticulate** (Bridges R and the Python 3.6.10 r-reticulate environment so keras can be driven from SERDA runner.R)

## Examples

```
Rscript -e 'data_file <- "SERRF example dataset - with validate.xlsx"; source("SERDA runner.R")'
```

## Evaluation signals

- Median RSD of pooled QC training samples should drop to roughly 16% after SERDA normalization (reported value)
- Independent validation QC samples (e.g. BioIVT plasma pools not used for training) should approach ~19% RSD after applying the trained model
- Normalized values must be back-transformed (exponentiated) to the original intensity scale, so outputs should be non-negative intensities comparable to raw values
- Output file serda_normalized_data.xlsx should contain the same 563-metabolite feature set and sample rows as the input xlsx, minus excluded blank samples
- The keras validation split (20%) should show early stopping engaged, indicating the autoencoder fit converged without overfitting

## Limitations

- Performance depends on batch-level system drift: the paper's validation used 1.2 years of GC-MS acquisition with seven column changes; on small or stable batches the benefit may be minimal
- External validation pools can behave differently from training QC: SERRF showed 19% internal precision but 34% RSD on external BioIVT pools, so SERDA should be checked against truly independent validation QC samples
- The runner script targets a specific environment (Windows 10, R 3.6.3, Python 3.6.10 via r-miniconda r-reticulate) — version mismatches may break keras/reticulate setup

## Evidence

- [abstract] Training-based QC normalization...: "SERDA lowered the median standard deviations of the training QC samples down to 16% RSD, yielding an overall error of 19% RSD when applied to the independent BioIVT validation QC samples"
- [methods] Autoencoder target-sample normalization: "Download `SERDA runner.R`. Open it and edit `data_file = "filelocation.xlsx"` and run all the script lines."
- [readme] Input metadata contract: [evidence span withheld — anchor status: external_doc]
- [readme] Exclusion rule: [evidence span withheld — anchor status: external_doc]
