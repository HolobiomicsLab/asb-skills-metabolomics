---
name: chromatographic-peak-quality-scoring
description: Use when raw MRM-mode LC-MS data (mzML with quantifier plus qualifier
  transitions and expected RTs in metabdb.xlsx) must be converted into a trustworthy
  set of reported peaks, i.e. deciding which of several peak candidates per metabolite/sample
  is the correct one.
when_to_use_negative:
- The acquisition method has no qualifier transition besides the quantifier — automRm
  requires at least one qualifier for optimal performance, so reported-peak quality
  will degrade.
- Samples are excessively dirty, very-low-input, or derive from a brand-new LC method
  with no relevant training data — the article retreats to manual peak review in these
  cases.
- Input is already a processed feature/peak table rather than raw mzML MRM chromatograms
  — process_batch() operates on raw files.
edam_operation: http://edamontology.org/operation_3633
tools:
- name: automRm
  role: R package implementing ML-based peak picking, 20 quality scores, RT-shift
    fitting, prototype-chromogram summing, and the peak-reporting filter
  repo: https://gitlab.gwdg.de/joerg.buescher/automrm
- name: mzR
  role: Parses mzML raw files during process_batch()
- name: caret
  role: Trains the peak-picking and peak-reporting ML models (NNet, RF, SVM, XGB)
    with hyperparameter tuning and an 80%/20% train/validation split
- name: openxlsx
  role: Reads metabdb.xlsx transition/RT definitions and writes the peakinfo.xlsx
    report
merged_aliases:
- chromatographic-peak-quality-assessment
merged_alias_records:
- alias: chromatographic-peak-quality-assessment
  slug: chromatographic-peak-quality-assessment
  jaccard_score: 0.6
  method: token-set-jaccard
  decision: needs_review
provenance:
  source_task_ids:
  - task_001
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.1c05224
    title: 'automRm: An R Package for Fully Automatic LC-QQQ-MS Data Preprocessing
      Powered by Machine Learning'
  source_content_hash: sha256:1cb7944dcbbeca40e56729c5ceba54607f2bbf3c188d8f020c82f387ecaff395
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - Shiny
  - automRm
  - caret
  - chromatographR
  - devtools
  - metScribeR
  - mzR
  - openxlsx
  edam_operation: http://edamontology.org/operation_3633
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 21
---

# chromatographic-peak-quality-scoring

## Summary

Scores candidate chromatographic peaks from MRM LC-QQQ-MS raw data with 20 quality scores and a trained peak-reporting ML model so that reported peaks can replace manual peak review. Implemented as the peak picking/reporting step of automRm's process_batch() pipeline.

## When to use

Use when raw MRM-mode LC-MS data (mzML with quantifier plus qualifier transitions and expected RTs in metabdb.xlsx) must be converted into a trustworthy set of reported peaks, i.e. deciding which of several peak candidates per metabolite/sample is the correct one.

## When NOT to use

- The acquisition method has no qualifier transition besides the quantifier — automRm requires at least one qualifier for optimal performance, so reported-peak quality will degrade.
- Samples are excessively dirty, very-low-input, or derive from a brand-new LC method with no relevant training data — the article retreats to manual peak review in these cases.
- Input is already a processed feature/peak table rather than raw mzML MRM chromatograms — process_batch() operates on raw files.

## Inputs

- mzML raw LC-QQQ-MS MRM files (quantifier + qualifier transitions per metabolite)
- metabdb.xlsx with metabolite-specific quantifier/qualifier transitions and expected RTs
- sample.info sample metadata
- update_prm.tsv configuration pointing to raw data, saved pretrained ML models, and metabdb.xlsx
- Pretrained peak-picking and peak-reporting ML models (from gitlab.gwdg.de/joerg.buescher/demodata or self-trained via train_model())

## Outputs

- peakinfo.xlsx processed batch report with reported peaks and quality scores
- peakoverview.pdf chromatogram plots
- shiftplots.pdf retention-time-shift plots
- Correct-reporting-rate table (fraction of reported peaks matching manually defined correct peaks) for benchmarking against manual review

## How to apply

Run automRm's process_batch() after configuring update_prm.tsv with raw-data locations, pretrained ML models, and metabdb.xlsx; it parses mzML files, metabolite-specific quantifier/qualifier transitions and expected RTs from metabdb.xlsx, and sample metadata from sample.info. Peak candidates (default 5 per metabolite per sample) are obtained by detecting local maxima in a smoothed version of the quantifier trace, ranked by peak height, and each candidate is described by 20 quality scores. RT shifts are calculated per metabolite per sample by simultaneous cross-correlation, fitted as a quadratic function of RT, and the RT-shifted chromatograms of a metabolite across all samples are summed into a prototype chromatogram for robust peak detection. The peak reporting ML model consumes the same quality scores previously used by the peak picking ML model and filters which candidates are reported; train it with train_model() (random forest or XGB performed near-perfectly on the peak-reporting task; a combined multi-method training set was the most generally applicable). Inspect peakinfo.xlsx, plus peakoverview.pdf chromatogram plots and shiftplots.pdf RT-shift plots, and check reported peaks against any manually curated ground truth.

## Related tools

- **automRm** (R package implementing ML-based peak picking, 20 quality scores, RT-shift fitting, prototype-chromogram summing, and the peak-reporting filter) — https://gitlab.gwdg.de/joerg.buescher/automrm
- **mzR** (Parses mzML raw files during process_batch())
- **caret** (Trains the peak-picking and peak-reporting ML models (NNet, RF, SVM, XGB) with hyperparameter tuning and an 80%/20% train/validation split)
- **openxlsx** (Reads metabdb.xlsx transition/RT definitions and writes the peakinfo.xlsx report)

## Examples

```
devtools::install_gitlab('joerg.buescher/automRm@master', host='https://gitlab.gwdg.de'); library(automRm); process_batch()
```

## Evaluation signals

- peakinfo.xlsx, peakoverview.pdf, and shiftplots.pdf are generated for the full batch, confirming end-to-end preprocessing completed.
- Compute the correct-reporting rate: for every metabolite-sample pair, check whether the reported peak matches the manually defined correct peak on a held-out batch.
- RF- and XGB-trained peak-reporting models should nearly perfectly classify the peak-reporting task; large drops vs. manual ground truth indicate a bad training-data-set/algorithm choice.
- On a HILIC-MS data set, replicate-measurement variation in peak integration and number of correctly reported peaks should match or beat alternative automatic software.
- Each reported candidate should carry the 20 quality scores, and RT shifts of metabolites in each sample should follow the fitted quadratic function of RT (inspect shiftplots.pdf).

## Limitations

- At least one qualifier in addition to the quantifier is required for optimal automRm performance, and adding qualifiers reduces the number of analytes detectable simultaneously in MRM mode.
- Quality similar to manual peak review is only reached with sufficient training on relevant raw data; training-data-set choice and ML algorithm measurably affect correct peak reporting.
- Manual peak review remains necessary for selected metabolites in special cases such as excessively dirty samples, very-low-input samples, or new LC methods.

## Evidence

- [abstract] Using machine learning (ML) for detection of chromatographic peaks and quality control of reported results enables the automatic recognition of complex patterns in raw data.: "Using machine learning (ML) for detection of chromatographic peaks and quality control of reported results enables the automatic recognition of complex patterns in raw data."
- [methods] the peak reporting ML model uses the same quality scores that were previously used by the peak picking ML model: "the peak reporting ML model uses the same quality scores that were previously used by the peak picking ML model"
- [methods] local maxima are detected in a smoothed version of the quantifier trace and ranked by peak height: "local maxima are detected in a smoothed version of the quantifier trace and ranked by peak height"
- [methods] RT shifts of all metabolites in a sample are fitted as a quadratic function of RT: "RT shifts of all metabolites in a sample are fitted as a quadratic function of RT"
- [discussion] With sufficient training on relevant raw data, results from automRm can reach a similar level of quality as manual peak review.: "With sufficient training on relevant raw data, results from automRm can reach a similar level of quality as manual peak review."
