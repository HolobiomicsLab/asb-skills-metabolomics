---
name: machine-learning-peak-classification
description: Use when preprocessing batches of MRM-mode LC-MS raw data (mzML) where
  chromatographic peaks must be detected, scored, and automatically accepted/rejected
  without manual peak review.
when_to_use_negative:
- Data lacks any qualifier transition in addition to the quantifier — automRm requires
  at least one qualifier for optimal performance
- Excessively dirty samples, very-low-input samples, or brand-new LC methods without
  relevant training data — the paper states these cases require retreat to manual
  peak review
- Non-MRM acquisition modes (e.g. DDA/DIA) — the method is built for data recorded
  in MRM mode
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_0091
tools:
- name: automRm
  role: R package whose peak picking and peak reporting ML models score and filter
    MRM chromatographic peaks; invoked via process_batch() and train_model()
  repo: https://gitlab.gwdg.de/joerg.buescher/automrm
- name: caret
  role: R package used for training the machine learning (ML) models
- name: mzR
  role: Parses mzML files into chromatogram traces for peak detection
- name: openxlsx
  role: Writes the peakinfo.xlsx peak report
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.1c05224
    title: 'automRm: An R Package for Fully Automatic LC-QQQ-MS Data Preprocessing
      Powered by Machine Learning'
  source_content_hash: sha256:6f18c0b167776baa598da18bd2d59d73eaf586ad65668ebe4ac1763d5d8b05ab
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
  edam_operation: http://edamontology.org/operation_3216
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# machine-learning-peak-classification

## Summary

Classifies and reports chromatographic peaks from LC-QQQ-MS MRM data using random-forest ML models (via the automRm R package) that consume 20 quality scores per peak candidate, replacing manual peak review for automated quality control.

## When to use

Use when preprocessing batches of MRM-mode LC-MS raw data (mzML) where chromatographic peaks must be detected, scored, and automatically accepted/rejected without manual peak review.

## When NOT to use

- Data lacks any qualifier transition in addition to the quantifier — automRm requires at least one qualifier for optimal performance
- Excessively dirty samples, very-low-input samples, or brand-new LC methods without relevant training data — the paper states these cases require retreat to manual peak review
- Non-MRM acquisition modes (e.g. DDA/DIA) — the method is built for data recorded in MRM mode

## Inputs

- mzML raw MRM data files (LC-QQQ-MS, e.g. Agilent 1290 Infinity II UHPLC + Agilent 6495 QQQ-MS)
- update_prm.tsv configuration file pointing to raw data, ML models, and metabolite settings
- metabdb.xlsx metabolite database with quantifier/qualifier transitions and expected RTs
- sample metadata (sample.info)
- pretrained or user-trained random-forest peak picking and peak reporting models (train via train_model())

## Outputs

- peakinfo.xlsx report of accepted peak intensities per metabolite per sample
- peakoverview.pdf chromatogram plots
- shiftplots.pdf retention-time-shift plots

## How to apply

Configure update_prm.tsv to point to the mzML raw files, pretrained peak-picking/peak-reporting ML models, and a metabolite database (metabdb.xlsx) listing quantifier/qualifier MRM transitions and expected retention times. Run process_batch(), which detects local maxima in a smoothed quantifier trace and ranks up to 5 peak candidates per metabolite per sample, computes 20 quality scores per candidate, aligns RT shifts by cross-correlation against a reference sample, and fits those shifts as a quadratic function of RT. The peak reporting ML model then accepts or rejects each peak using the same 20 quality scores plus the peak-picking classification score and its 0th, 25th, 50th, 75th, and 100th percentile across samples. Rationale: ML-based decisions surpass classical rule-based filtering, and with sufficient training on relevant raw data the reported peaks reach quality similar to manual peak review. Models can be retrained on user data with train_model(); include at least one qualifier transition per quantifier for optimal performance.

## Related tools

- **automRm** (R package whose peak picking and peak reporting ML models score and filter MRM chromatographic peaks; invoked via process_batch() and train_model()) — https://gitlab.gwdg.de/joerg.buescher/automrm
- **caret** (R package used for training the machine learning (ML) models)
- **mzR** (Parses mzML files into chromatogram traces for peak detection)
- **openxlsx** (Writes the peakinfo.xlsx peak report)

## Examples

```
devtools::install_gitlab('joerg.buescher/automRm@master', host = 'https://gitlab.gwdg.de'); automRm::process_batch()
```

## Evaluation signals

- Pearson correlation between automRm peak areas and originally reported (manually reviewed) signal intensities exceeds 0.95 for the majority of metabolites — the paper reports >0.95 for 22/34 metabolites in MTBLS429 and 20/30 in MTBLS897
- Variation in peak integration among replicate measurements is lower than alternative software solutions
- Number of correctly reported peaks on a HILIC-MS data set is not degraded relative to manual review
- peakinfo.xlsx, peakoverview.pdf, and shiftplots.pdf are produced for the batch, confirming the end-to-end process_batch() run completed
- RT shifts fit well as a quadratic function of RT across metabolites, and peak boundaries propagate correctly from the prototype chromatogram back to each sample

## Limitations

- Requires at least one qualifier transition in addition to the quantifier for optimal performance; adding qualifiers reduces the number of analytes detectable simultaneously in MRM mode
- Results only reach quality similar to manual peak review given sufficient training on relevant raw data
- Manual peak review is still needed for selected metabolites in special cases such as excessively dirty samples, very-low-input samples, or new LC methods

## Evidence

- [abstract] Using machine learning (ML) for detection of chromatographic peaks and quality control of reported results enables the automatic recognition of complex patterns in raw data.: "Using machine learning (ML) for detection of chromatographic peaks and quality control of reported results enables the automatic recognition of complex patterns in raw data."
- [methods] the peak reporting ML model uses the same quality scores that were previously used by the peak picking ML model: "the peak reporting ML model uses the same quality scores that were previously used by the peak picking ML model"
- [methods] local maxima are detected in a smoothed version of the quantifier trace and ranked by peak height: "local maxima are detected in a smoothed version of the quantifier trace and ranked by peak height"
- [discussion] The use of machine learning enables complex automatic decisions that surpass classical rule-based approaches.: "The use of machine learning enables complex automatic decisions that surpass classical rule-based approaches."
- [discussion] With sufficient training on relevant raw data, results from automRm can reach a similar level of quality as manual peak review.: "With sufficient training on relevant raw data, results from automRm can reach a similar level of quality as manual peak review."
