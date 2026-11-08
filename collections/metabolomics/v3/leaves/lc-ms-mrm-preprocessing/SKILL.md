---
name: lc-ms-mrm-preprocessing
description: Use when raw MRM-mode LC-QQQ-MS data (e.g. mzML files from an Agilent
  6495 QQQ-MS) must be processed into quality-scored chromatographic peak reports
  without manual peak review.
when_to_use_negative:
- The data are not MRM-mode LC-MS (e.g. untargeted DDA/DIA or GC-MS data); automRm
  is specific to MRM preprocessing
- Only a quantifier transition without any qualifier is available — the article states
  optimal performance requires at least one qualifier in addition to the quantifier
- The sample batch is excessively dirty, very-low-input, or from a new LC method with
  no relevant training data — manual peak review is the recommended retreat
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: automRm
  role: Core R package providing process_batch(), train_model(), and automrm_gui()
    for ML-powered MRM data preprocessing
  repo: https://gitlab.gwdg.de/joerg.buescher/automrm
- name: mzR
  role: Dependency used by automRm for parsing mzML files
- name: caret
  role: Machine learning backend for training peak picking and peak reporting models
    (NNet, RF, SVM, XGB) with hyperparameter tuning and an 80%/20% training/validation
    split
- name: openxlsx
  role: Reading and writing xlsx files for reported peak results
- name: devtools
  role: Used to install automRm from GitLab via install_gitlab()
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.1c05224
    title: 'automRm: An R Package for Fully Automatic LC-QQQ-MS Data Preprocessing
      Powered by Machine Learning'
  source_content_hash: sha256:d2eb7eaca0991b1203f9eb204703141b94d25155031e09c7680a1836ee53b0d1
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - automRm
  - caret
  - devtools
  - mzR
  - openxlsx
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 18
---

# lc-ms-mrm-preprocessing

## Summary

Fully automatic preprocessing of LC-MS data recorded in MRM mode using machine learning for chromatographic peak detection and quality-controlled peak reporting, as implemented in the automRm R package, replacing manual peak review.

## When to use

Use when raw MRM-mode LC-QQQ-MS data (e.g. mzML files from an Agilent 6495 QQQ-MS) must be processed into quality-scored chromatographic peak reports without manual peak review.

## When NOT to use

- The data are not MRM-mode LC-MS (e.g. untargeted DDA/DIA or GC-MS data); automRm is specific to MRM preprocessing
- Only a quantifier transition without any qualifier is available — the article states optimal performance requires at least one qualifier in addition to the quantifier
- The sample batch is excessively dirty, very-low-input, or from a new LC method with no relevant training data — manual peak review is the recommended retreat

## Inputs

- Batch of raw MRM-mode LC-MS data files (mzML format, parsed via mzR)
- Metabolite MRM transition table with quantifier and at least one qualifier
- Training data set with expert ground-truth peak labels (e.g. 7817 positive / 32032 negative peak-picking and 4987 positive / 4164 negative peak-reporting examples from four LC-QQQ-MS methods)
- Trained ML models (caret algorithms such as NNet, RF, SVM, XGB) for peak picking and peak reporting

## Outputs

- Quality-scored chromatographic peak reports for every metabolite-sample pair (xlsx via openxlsx)
- Per-metabolite, per-sample RT shifts and integrated peak areas/heights
- Correct-reporting rates (fraction of reported peaks matching manually defined correct peaks) when benchmarked against ground truth

## How to apply

Install automRm via devtools::install_gitlab('joerg.buescher/automRm@master', host = 'https://gitlab.gwdg.de') in R (tested on R 3.6.0 and 4.0.2) and parse raw files with the mzR package; optionally use the automrm_gui() graphical interface. Run process_batch() on a batch of raw mzML files: it detects local maxima in a smoothed version of the quantifier trace ranked by peak height, calculates RT shifts separately for each metabolite in each sample by simultaneous cross-correlation, fits RT shifts as a quadratic function of RT, sums RT-shifted chromatograms across samples to build a prototype, and computes quality scores. Train or apply ML models for peak picking and peak reporting via train_model(); the peak reporting model reuses the same quality scores that were previously used by the peak picking model. Ensure the acquisition method includes at least one qualifier in addition to the quantifier, since this is required for optimal performance, and use openxlsx for reading/writing xlsx result files. Validate output by comparing reported peaks against manually defined ground truth (e.g. correct-reporting rates) and retreat to manual peak review only for selected metabolites in excessively dirty samples, very-low-input samples, or new LC methods.

## Related tools

- **automRm** (Core R package providing process_batch(), train_model(), and automrm_gui() for ML-powered MRM data preprocessing) — https://gitlab.gwdg.de/joerg.buescher/automrm
- **mzR** (Dependency used by automRm for parsing mzML files)
- **caret** (Machine learning backend for training peak picking and peak reporting models (NNet, RF, SVM, XGB) with hyperparameter tuning and an 80%/20% training/validation split)
- **openxlsx** (Reading and writing xlsx files for reported peak results)
- **devtools** (Used to install automRm from GitLab via install_gitlab())

## Examples

```
Rscript -e "devtools::install_gitlab('joerg.buescher/automRm@master', host='https://gitlab.gwdg.de'); library(automRm); process_batch()"
```

## Evaluation signals

- Cross-check reported correct-reporting rates: RF and XGB should nearly perfectly classify the peak-reporting task, and the combined training set (48 samples, four LC-QQQ-MS methods) should be the most generally applicable variant
- Reproduce manual peak review results on published data sets such as MTBLS429 (pyrrolizidine alkaloids) and MTBLS897 (grape secondary metabolites)
- Verify lower variation in peak integration among replicate measurements and a higher number of correctly reported peaks than alternative automatic software on a HILIC-MS data set
- Confirm each MRM metabolite has at least one qualifier transition; results without qualifiers degrade below optimal performance
- Check that quality scores feed both the peak picking and peak reporting models, so reported peaks carry consistent quality annotations

## Limitations

- At least one qualifier in addition to the quantifier is required for optimal automRm performance, and adding qualifiers reduces the number of analytes detectable simultaneously in MRM mode
- Results reach manual-review quality only with sufficient training on relevant raw data; new LC methods or lack of relevant training data require retreating to manual peak review
- Manual peak review is still needed for selected metabolites in special cases such as excessively dirty samples or very-low-input samples

## Evidence

- [abstract] ML-powered MRM preprocessing: "Here, we present the R package automRm for fully automatic preprocessing of LC-MS data recorded in MRM mode."
- [abstract] ML peak detection: "Using machine learning (ML) for detection of chromatographic peaks and quality control of reported results enables the automatic recognition of complex patterns in raw data."
- [methods] Training/validation split: "caret13 for machine learning (ML)"
- [methods] Installation command: "devtools::install_gitlab('joerg.buescher/automRm@master', host = 'https://gitlab.gwdg.de')"
- [methods] Peak picking: "local maxima are detected in a smoothed version of the quantifier trace and ranked by peak height"
- [methods] RT alignment: "calculate RT shifts separately for each metabolite in each sample by the simultaneous cross-correlation"
- [discussion] Qualifier requirement: "For optimal performance, automRm requires the use of at least one qualifier in addition to the quantifier."
- [discussion] Manual review retreat: "We only retreat to manual peak review for selected metabolites in special cases such as excessively dirty samples, very-low-input samples, or new LC methods."
