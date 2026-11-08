---
name: qc-sample-frequency-resampling-in-metabolomics
description: Decision support for choosing how often cohort pool QC samples should
  be injected in large-scale untargeted metabolomics batches, by training the normalization
  model at several QC frequencies and comparing errors on independent validation QC
  pools.
when_to_use_negative:
- The goal is to evaluate a different axis of QC design (e.g. QC pooling composition
  or number of metabolites) rather than the frequency of QC placement relative to
  biological samples
- The dataset is small or lacks enough pooled QC injections to form configurations
  at multiple sampling intervals (10/20/40/80), so resampling cannot distinguish frequencies
- The normalization approach is not trained on QC samples (e.g. sum-based or internal-standard
  normalization), since the paper reports these never reached <30% median precision
  across 563 annotations and the QC-frequency effect was established for the SERDA
  autoencoder
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: SERDA runner.R
  role: R script used to train the SERDA denoising autoencoder on each resampled QC
    configuration after editing the data_file path
  repo: https://github.com/slfan2013/SERDA
- name: reticulate
  role: Bridges R 3.6.3 to the Python 3.6.10 environment running keras 2.7.0 for SERDA
    training
- name: keras
  role: Implements the denoising autoencoder model trained at each QC frequency
- name: SERRF example dataset - with validate.xlsx
  role: Example ordered acquisition table containing qc, sample, and validate sampleType
    rows used as the base dataset for resampling configurations
  repo: https://github.com/slfan2013/Shiny-SERRF/raw/master/SERRF%20example%20dataset%20-%20with%20validate.xlsx
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.3390/metabo13080944
    title: Denoising Autoencoder Normalization for Large-Scale Untargeted Metabolomics
      by Gas Chromatography–Mass Spectrometry
  source_content_hash: sha256:16ea40ab049b43780856a96a98b77c9d19ebcd4be82db33e17539759205cff27
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/slfan2013/SERDA
  tools:
  - R
  - SERDA runner.R
  - SERRF example dataset - with validate.xlsx
  - keras
  - reticulate
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# QC-sample frequency resampling in metabolomics

## Summary

Subsample or duplicate pooled QC injections in a large GC-MS metabolomics run to simulate different QC-placement densities (one pool QC per 10, 20, 40, or 80 biological samples) before normalization-model training, in order to quantify how QC sampling frequency drives post-normalization precision.

## When to use

Decision support for choosing how often cohort pool QC samples should be injected in large-scale untargeted metabolomics batches, by training the normalization model at several QC frequencies and comparing errors on independent validation QC pools.

## When NOT to use

- The goal is to evaluate a different axis of QC design (e.g. QC pooling composition or number of metabolites) rather than the frequency of QC placement relative to biological samples
- The dataset is small or lacks enough pooled QC injections to form configurations at multiple sampling intervals (10/20/40/80), so resampling cannot distinguish frequencies
- The normalization approach is not trained on QC samples (e.g. sum-based or internal-standard normalization), since the paper reports these never reached <30% median precision across 563 annotations and the QC-frequency effect was established for the SERDA autoencoder

## Inputs

- Ordered metabolomics acquisition table (e.g. .xlsx) with columns _batch_, _sampleType_ (qc/sample/validate), _time_, _label_, and _No_ for compounds
- Independent external validation QC pool set (e.g. BioIVT pooled plasma QC samples) excluded from training
- SERDA runner.R script and R/Python environment (R 3.6.3, reticulate 1.19, keras 2.7.0, Python 3.6.10)

## Outputs

- Per-QC-frequency training configurations (datasets with one pool QC after every 10, 20, 40, 80 biological samples)
- Median RSD across all metabolite annotations on BioIVT validation QC pools for each frequency (expected: 17%, 20%, 22%, 25% RSD)
- Tabulated result such as figure6_qc_frequency_median_rsd.csv supporting the one-QC-per-10-samples recommendation

## How to apply

Take a single ordered acquisition table (e.g. the SERRF example dataset 'SERRF example dataset - with validate.xlsx') and create four training configurations in which one pooled QC row follows every 10, 20, 40, and 80 biological samples respectively, by subsampling or duplicating pool QC rows while preserving _batch_, _sampleType_, _time_, and _label_ metadata. Train the SERDA denoising autoencoder for each configuration in R 3.6.3 via reticulate 1.19 bridging to Python 3.6.10 with keras 2.7.0, using the same `SERDA runner.R` workflow with `data_file` pointing at each resampled file. After training, apply each model unchanged to the independent BioIVT external validation QC pools that were excluded from training, and compute the median relative standard deviation across all 563 metabolite annotations for each QC frequency. The rationale is that fewer training QC samples starve the autoencoder of drift information, so errors should rise monotonically as the sampling interval grows; the paper's recomputed medians were 17%, 20%, 22%, and 25% RSD for 10/20/40/80 sampling, identifying one pool QC per 10 biological samples as optimal.

## Related tools

- **SERDA runner.R** (R script used to train the SERDA denoising autoencoder on each resampled QC configuration after editing the data_file path) — https://github.com/slfan2013/SERDA
- **reticulate** (Bridges R 3.6.3 to the Python 3.6.10 environment running keras 2.7.0 for SERDA training)
- **keras** (Implements the denoising autoencoder model trained at each QC frequency)
- **SERRF example dataset - with validate.xlsx** (Example ordered acquisition table containing qc, sample, and validate sampleType rows used as the base dataset for resampling configurations) — https://github.com/slfan2013/Shiny-SERRF/raw/master/SERRF%20example%20dataset%20-%20with%20validate.xlsx

## Examples

```
Rscript -e 'data_file <- "SERRF example dataset - with validate.xlsx"; source("SERDA runner.R")'
```

## Evaluation signals

- Median RSD on the independent BioIVT validation QC pools increases monotonically with decreasing QC frequency: approximately 17% (10), 20% (20), 22% (40), 25% (80)
- Each recomputed median RSD matches the reported BioIVT errors within reasonable tolerance, confirming one pool QC per 10 biological samples as optimal
- The same trained SERDA model is applied unchanged to the external BioIVT validation QC set for every frequency, so differences reflect training data density, not model changes
- Metadata columns (_batch_, _sampleType_, _time_, _label_) are preserved when subsampling or duplicating pool QC rows
- Training QC median precision after SERDA normalization is around 16% RSD, consistent with the paper's reported training-set performance

## Limitations

- Only one pool QC per 10 biological samples was found optimal for this GC-MS setup; the exact intervals are data-set-specific and were derived from a 1.2-year, >4000-sample type 2 diabetes cohort study with seven column changes
- The effect was characterized for the SERDA denoising autoencoder; other normalization strategies (internal standards, sum-based) are not the subject of this frequency comparison
- External validation relies on independent BioIVT pooled plasma QC samples, so conclusions depend on availability of a truly independent validation QC set

## Evidence

- [abstract] monotonic degradation of median RSD with sparser QC training data: "external cross-validation with BioIVT plasma pools yielded a median 34% relative standard deviation (RSD)"
- [methods] SERDA runner.R workflow for training configurations: "oad `SERDA runner.R`. Open it and edit `data_file = "filelocation.xlsx"` and run all the script lines."
- [readme] SERRF/SERDA input metadata requirements for resampling: [evidence span withheld — anchor status: external_doc]
- [readme] software stack for SERDA training: "version:        3.6.10 |Anaconda, Inc.| (default, May  7 2020, 19:46:08) [MSC v.1916 64 bit (AMD64)]"
