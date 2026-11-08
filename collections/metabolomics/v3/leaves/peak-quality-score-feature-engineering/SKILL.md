---
name: peak-quality-score-feature-engineering
description: Use when building or retraining ML models for peak picking/peak reporting
  on LC-QQQ-MS MRM data (mzML) and you need the quality-score features that automRm
  derives from quantifier and qualifier traces.
when_to_use_negative:
- Excessively dirty samples, very-low-input samples, or newly established LC methods,
  where the article advises retreating to manual peak review
- Acquisition methods lacking any qualifier transition, since automRm requires at
  least one qualifier in addition to the quantifier for optimal performance
- Rule-based (non-ML) peak reporting is desired, since the quality-score feature engineering
  exists specifically to power ML-based decisions that surpass classical rule-based
  approaches
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: automRm
  role: R package that computes the peak quality scores (process_batch) and trains
    the reporting models from them (train_model)
  repo: https://gitlab.gwdg.de/joerg.buescher/automrm
- name: mzR
  role: Parses mzML raw files from which quality-score features are derived
- name: caret
  role: Performs hyperparameter tuning and random forest training on the quality-score
    features
- name: openxlsx
  role: Reads and writes the manual_peakcheck.xlsx label files and template
- name: R
  role: Implementation platform (developed/tested in R 3.6.0 and 4.0.2)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.1c05224
    title: 'automRm: An R Package for Fully Automatic LC-QQQ-MS Data Preprocessing
      Powered by Machine Learning'
  source_content_hash: sha256:dd0482825798ec17191d5011c38e7fea26713703e07f138177c3c6a600663025
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - automRm
  - caret
  - mzR
  - openxlsx
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# peak-quality-score-feature-engineering

## Summary

Engineering of ML feature vectors (quality scores) from LC-QQQ MRM chromatograms in automRm: local maxima in smoothed quantifier traces, cross-correlation-based RT shifts fitted as a quadratic function of RT, and prototype chromatogram construction. These same quality scores feed both the peak-picking and peak-reporting random forest models trained with caret.

## When to use

Use when building or retraining ML models for peak picking/peak reporting on LC-QQQ-MS MRM data (mzML) and you need the quality-score features that automRm derives from quantifier and qualifier traces.

## When NOT to use

- Excessively dirty samples, very-low-input samples, or newly established LC methods, where the article advises retreating to manual peak review
- Acquisition methods lacking any qualifier transition, since automRm requires at least one qualifier in addition to the quantifier for optimal performance
- Rule-based (non-ML) peak reporting is desired, since the quality-score feature engineering exists specifically to power ML-based decisions that surpass classical rule-based approaches

## Inputs

- Raw LC-QQQ MRM data in mzML format (e.g. from an Agilent 1290 Infinity II UHPLC in line with an Agilent 6495 QQQ-MS)
- Acquisition method containing at least one qualifier in addition to the quantifier
- Manual peak-review labels (manual_peakcheck.xlsx with scores 0 = do not report, 1 = exclude from training, 2 = report) when (re)training the reporting model

## Outputs

- qslog.tsv containing the peak-picking classification (quality) scores for every metabolite in every sample
- manual_peakcheck_template.xlsx for expert annotation
- Trained random forest peak-reporting model serialized to disk, plus training-summary metrics (e.g. F1 score on the 20% validation subset)

## How to apply

Parse raw MRM data from mzML files with mzR, then detect local maxima in a smoothed version of each metabolite's quantifier trace and rank candidate peaks by peak height. Calculate RT shifts separately for each metabolite in each sample via simultaneous cross-correlation, fit the RT shifts of all metabolites in a sample as a quadratic function of RT, and sum the RT-shifted chromatograms of a metabolite across all samples into a prototype chromatogram. From these quantities compute the quality scores that serve as ML features; by design, the peak reporting ML model uses the same quality scores that were previously used by the peak picking ML model. Run automRm's process_batch() to emit these scores to qslog.tsv (one score set per metabolite-sample pair) and generate the manual_peakcheck_template.xlsx; for training, combine qslog.tsv scores with 0/1/2 labels from manual_peakcheck.xlsx, split 80% training / 20% validation, and tune a random forest with caret via train_model(). Require at least one qualifier in addition to the quantifier for optimal performance.

## Related tools

- **automRm** (R package that computes the peak quality scores (process_batch) and trains the reporting models from them (train_model)) — https://gitlab.gwdg.de/joerg.buescher/automrm
- **mzR** (Parses mzML raw files from which quality-score features are derived)
- **caret** (Performs hyperparameter tuning and random forest training on the quality-score features)
- **openxlsx** (Reads and writes the manual_peakcheck.xlsx label files and template)
- **R** (Implementation platform (developed/tested in R 3.6.0 and 4.0.2))

## Examples

```
devtools::install_gitlab('joerg.buescher/automRm@master', host = 'https://gitlab.gwdg.de')
```

## Evaluation signals

- qslog.tsv contains a quality-score entry for every metabolite in every sample of the batch
- Trained peak-reporting model reaches F1 > 0.98 on the 20% validation subset for combined models trained on combined data sets
- RT shifts fitted as a quadratic function of RT show no systematic misalignment of prototype chromatograms across samples
- The same quality scores are reused consistently by both the peak-picking and peak-reporting ML models (no divergent feature definitions)
- automRm-reported peaks replicate results obtained by manual peak review on published data sets (e.g. MTBLS429, MTBLS897, HILIC 1a/1b)

## Limitations

- At least one qualifier in addition to the quantifier is required for optimal automRm performance; adding qualifiers also reduces the number of analytes detectable simultaneously in MRM mode
- Sufficient training on relevant raw data is required before results reach quality similar to manual peak review
- Manual peak review is still needed for selected metabolites in special cases such as excessively dirty samples, very-low-input samples, or new LC methods

## Evidence

- [abstract] ML features from peak traces: "Using machine learning (ML) for detection of chromatographic peaks and quality control of reported results enables the automatic recognition of complex patterns in raw data."
- [methods] Peak candidate detection: "local maxima are detected in a smoothed version of the quantifier trace and ranked by peak height"
- [methods] RT alignment: "calculate RT shifts separately for each metabolite in each sample by the simultaneous cross-correlation"
- [methods] RT shift fitting: "RT shifts of all metabolites in a sample are fitted as a quadratic function of RT"
- [methods] Shared features between models: "the peak reporting ML model uses the same quality scores that were previously used by the peak picking ML model"
- [discussion] Qualifier requirement: "For optimal performance, automRm requires the use of at least one qualifier in addition to the quantifier."
