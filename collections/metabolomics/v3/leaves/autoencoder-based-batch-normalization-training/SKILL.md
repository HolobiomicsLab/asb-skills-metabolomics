---
name: autoencoder-based-batch-normalization-training
description: Use when untargeted GC-MS metabolomics peak tables (e.g. xlsx with batch/sampleType/time/label
  columns and a No compound index) span long acquisition periods and pooled QC samples
  should be used to train a data-driven normalization model.
when_to_use_negative:
- 'When the goal is to normalize blank-injected or otherwise excluded samples: the
  README states blank samples should be left with empty sampleType cells and any sample
  not to be normalized should be left empty, so SERDA will not transform them.'
- When only a handful of pooled QC samples exist relative to the number of biological
  samples — QC sampling density directly controls accuracy, with median RSD on BioIVT
  validation degrading from 17% (one QC per 10 samples) to 25% (one QC per 80 samples).
- 'When the dataset is LC-MS, CE-MS, or derivatization-free MS data: SERDA was developed
  and validated on derivatization-based GC-MS metabolomics.'
edam_operation: http://edamontology.org/operation_3675
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0219
tools:
- name: R 3.6.3
  role: Host environment for `SERDA runner.R`, the driver script that loads the xlsx
    and trains the autoencoder normalization.
  repo: https://github.com/slfan2013/SERDA
- name: reticulate 1.19
  role: Bridges R to the Python 3.6.10 / numpy 1.18.1 environment needed by keras
    for autoencoder training.
  repo: https://github.com/slfan2013/SERDA
- name: keras 2.7.0
  role: Implements the denoising autoencoder architecture trained on pooled QC samples.
  repo: https://github.com/slfan2013/SERDA
- name: caret 6.0.86
  role: Supports model fitting/evaluation utilities within the SERDA R pipeline.
  repo: https://github.com/slfan2013/SERDA
- name: pROC 1.17.0.1
  role: Available in the SERDA R environment for ROC-based diagnostics.
  repo: https://github.com/slfan2013/SERDA
- name: Python 3.6.10
  role: Anaconda-managed Python backend invoked via reticulate for keras training.
  repo: https://github.com/slfan2013/SERDA
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.3390/metabo13080944
    title: Denoising Autoencoder Normalization for Large-Scale Untargeted Metabolomics
      by Gas Chromatography–Mass Spectrometry
  source_content_hash: sha256:d2a6d4e1fd812ef798adcee93fc3d7aa0d436fe17637a6803070e114dcba0847
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/slfan2013/SERDA
  tools:
  - Python 3.6.10
  - R
  - R 3.6.3
  - caret 6.0.86
  - keras 2.7.0
  - pROC 1.17.0.1
  - reticulate 1.19
  edam_operation: http://edamontology.org/operation_3675
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0219
  provenance_tier: literature
  techniques:
  - CE-MS
license: CC-BY-4.0
emails_redacted: true
triage_status: critique
yes_rate: 0.5625
n_claims: 16
---

# autoencoder-based-batch-normalization-training

## Summary

SERDA is a denoising autoencoder normalization workflow that trains on cohort pool QC samples interleaved with biological samples in large-scale GC-MS metabolomics batches, correcting batch and time-varying drift that sum-based or internal-standard normalizations cannot remove.

## When to use

Use when untargeted GC-MS metabolomics peak tables (e.g. xlsx with batch/sampleType/time/label columns and a No compound index) span long acquisition periods and pooled QC samples should be used to train a data-driven normalization model.

## When NOT to use

- When the goal is to normalize blank-injected or otherwise excluded samples: the README states blank samples should be left with empty sampleType cells and any sample not to be normalized should be left empty, so SERDA will not transform them.
- When only a handful of pooled QC samples exist relative to the number of biological samples — QC sampling density directly controls accuracy, with median RSD on BioIVT validation degrading from 17% (one QC per 10 samples) to 25% (one QC per 80 samples).
- When the dataset is LC-MS, CE-MS, or derivatization-free MS data: SERDA was developed and validated on derivatization-based GC-MS metabolomics.

## Inputs

- xlsx peak-area table following the SERRF input format: _batch_, _sampleType_ (_qc_/_sample_/_validate_, blanks empty), _time_, _label_ (row #4 samples, column B compounds), _No_ (compound index)
- Example dataset 'SERRF example dataset - with validate.xlsx' from the Shiny-SERRF GitHub repo for testing

## Outputs

- Normalized peak-intensity matrix for all samples and validate QC rows (blank sampleType cells left untouched)
- Median relative standard deviation (RSD) across metabolite annotations for training pool QC samples and for external validate QC samples (e.g. 16% and 19% in the article)

## How to apply

Download `SERDA runner.R` from the SERDA repository, open it in R 3.6.3 under Windows 10 with reticulate 1.19 bridging to Python 3.6.10 (Anaconda r-reticulate env, numpy 1.18.1) and keras 2.7.0, edit `data_file = "filelocation.xlsx"` to point at your input file, and run all the script lines. The input file must follow the Shiny-SERRF format: columns for _batch_, _sampleType_ (values qc, sample, or validate; leave blanks empty if you do not want those samples normalized), _time_ (processing order, real or integer), _label_ (sample labels in row #4, compound labels in column B), and _No_ (compound index). SERDA trains a denoising autoencoder on the pooled QC samples interleaved among biological samples, then applies the learned transformation to the samples and any _validate_ QC rows. Rationale: on a 1.2-year GC-MS study of >4000 type 2 diabetes plasma samples, neither internal standards nor sum-based normalizations reached 30% median precision across 563 metabolite annotations, whereas SERDA-trained QC samples reached 16% RSD and 19% RSD on independent BioIVT validation pools.

## Related tools

- **R 3.6.3** (Host environment for `SERDA runner.R`, the driver script that loads the xlsx and trains the autoencoder normalization.) — https://github.com/slfan2013/SERDA
- **reticulate 1.19** (Bridges R to the Python 3.6.10 / numpy 1.18.1 environment needed by keras for autoencoder training.) — https://github.com/slfan2013/SERDA
- **keras 2.7.0** (Implements the denoising autoencoder architecture trained on pooled QC samples.) — https://github.com/slfan2013/SERDA
- **caret 6.0.86** (Supports model fitting/evaluation utilities within the SERDA R pipeline.) — https://github.com/slfan2013/SERDA
- **pROC 1.17.0.1** (Available in the SERDA R environment for ROC-based diagnostics.) — https://github.com/slfan2013/SERDA
- **Python 3.6.10** (Anaconda-managed Python backend invoked via reticulate for keras training.) — https://github.com/slfan2013/SERDA

## Examples

```
Rscript -e 'data_file <- "filelocation.xlsx"; source("SERDA runner.R")'
```

## Evaluation signals

- Median RSD across all 563 metabolite annotations on the pooled training QC samples should drop below ~17% (article reports 16% RSD after SERDA).
- Median RSD on an independent external validation QC set (e.g. BioIVT plasma pools) should fall to roughly 19% RSD, well below the 34% RSD that SERRF produced on the same external pools.
- When testing QC sampling density, median validation RSD should degrade monotonically with decreasing QC frequency (expected ~17%, 20%, 22%, 25% RSD for one QC per 10, 20, 40, 80 biological samples).
- Sum-based and internal-standard normalizations applied to the same dataset should yield median precision worse than 30%, confirming the batch effect remains and SERDA's improvement is meaningful.
- Blank sampleType rows and other excluded samples must remain unchanged in the output matrix, matching the README's instruction to leave those cells empty.

## Limitations

- Performance depends on QC sampling density: external validation median RSD rises monotonically as the number of cohort pool QC samples per biological sample decreases (17% → 25% RSD for 1 QC per 10 vs 1 per 80).
- Requires Windows 10 x64, R 3.6.3, reticulate 1.19, keras 2.7.0, Python 3.6.10 (Anaconda r-reticulate env, numpy 1.18.1) — the README only documents this configuration.
- The published validation was performed on derivatization-based GC-MS human plasma datasets; transfer to other chromatography or matrix types is not documented in the article or README.
- Both the SERDA and Shiny-SERRF READMEs note contact via <maintainer e-mail, see the repository README> for questions, and the article has no discussion section text available for further caveats.

## Evidence

- [methods] Runner script usage: "oad `SERDA runner.R`. Open it and edit `data_file = "filelocation.xlsx"` and run all the script lines."
- [methods] Example dataset link: "An example dataset can be downloaded [here](https://github.com/slfan2013/Shiny-SERRF/raw/master/SERRF%20example%20dataset%20-%20with%20validate.xlsx)."
- [readme] SampleType / blank handling: [evidence span withheld — anchor status: external_doc]
- [readme] Required columns: [evidence span withheld — anchor status: external_doc]
- [intro] Soft-/hardware environment: "> packageVersion("reticulate")
[1] ‘1.19’"
- [abstract] External validation RSD: "SERDA lowered the median standard deviations of the training QC samples down to 16% RSD, yielding an overall error of 19% RSD when applied to the independent BioIVT validation QC samples"
- [abstract] Baseline normalization failure: "neither internal standards nor sum-based normalizations yielded median precision of less than 30% across all 563 metabolite annotations"
- [results] QC sampling frequency sensitivity: [evidence span withheld — anchor status: not_found]
- [abstract] Dataset size: "a study with more than 4000 human plasma samples involved in a type 2 diabetes cohort study"
- [readme] Python configuration: "version:        3.6.10 |Anaconda, Inc.| (default, May  7 2020, 19:46:08) [MSC v.1916 64 bit (AMD64)]"
