---
name: chromatographic-peak-detection
description: Use when MRM-mode LC-QQQ-MS raw data (mzML) must be preprocessed into
  peak intensity tables without manual review, e.g. batches from instruments like
  an Agilent 6495 QQQ-MS needing automatic peak picking and quality control.
when_to_use_negative:
- Excessively dirty samples, very-low-input samples, or brand-new LC methods — the
  article recommends retreating to manual peak review for these special cases
- Acquisition methods without at least one qualifier transition in addition to the
  quantifier — automRm performance is suboptimal without qualifiers
- Data not recorded in MRM mode, or cases lacking sufficient relevant training data
  — quality similar to manual peak review requires training on relevant raw data
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: automRm
  role: Core R package executing peak picking, RT alignment, prototype chromatogram
    construction, and ML-based peak reporting via process_batch()
  repo: https://gitlab.gwdg.de/joerg.buescher/automrm
- name: mzR
  role: Parses mzML files to extract MRM chromatogram traces for peak picking
- name: caret
  role: Machine learning framework used for training the random-forest peak picking
    and peak reporting models (train_model())
- name: openxlsx
  role: Reads the metabdb.xlsx metabolite database and writes the peakinfo.xlsx peak/QC
    table
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.1c05224
    title: 'automRm: An R Package for Fully Automatic LC-QQQ-MS Data Preprocessing
      Powered by Machine Learning'
  source_content_hash: sha256:6c52f5e19f2ce01bc623f16817202dc0cf7a9e9919a102cadd7ebdf02458d14f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AutoTuner
  - R
  - automRm
  - caret
  - devtools
  - mzR
  - openxlsx
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# chromatographic-peak-detection

## Summary

Fully automatic machine-learning-based detection and accept/reject reporting of chromatographic peaks in LC-QQQ-MS data recorded in MRM mode, replacing manual peak review with the automRm R package.

## When to use

Use when MRM-mode LC-QQQ-MS raw data (mzML) must be preprocessed into peak intensity tables without manual review, e.g. batches from instruments like an Agilent 6495 QQQ-MS needing automatic peak picking and quality control.

## When NOT to use

- Excessively dirty samples, very-low-input samples, or brand-new LC methods — the article recommends retreating to manual peak review for these special cases
- Acquisition methods without at least one qualifier transition in addition to the quantifier — automRm performance is suboptimal without qualifiers
- Data not recorded in MRM mode, or cases lacking sufficient relevant training data — quality similar to manual peak review requires training on relevant raw data

## Inputs

- mzML files of MRM-mode LC-QQQ-MS raw data batches (e.g. from MetaboLights accessions MTBLS429, MTBLS897)
- metabdb.xlsx metabolite database listing quantifier/qualifier MRM transitions and expected retention times per analyte
- update_prm.tsv configuration file pointing to raw data, pretrained models, and metabolite database
- pretrained combined random-forest peak picking and peak reporting models

## Outputs

- peakinfo.xlsx peak table with accepted peak intensities per metabolite per sample, per data set
- per-peak quality scores (20 scores plus peak-picking classification score and its across-sample percentiles) with accept/reject decisions

## How to apply

Install automRm in R via devtools::install_gitlab('joerg.buescher/automRm@master', host = 'https://gitlab.gwdg.de') and load the pretrained random-forest peak picking and peak reporting models. Configure update_prm.tsv to point to the mzML raw data, the models, and a metabolite database (metabdb.xlsx) listing quantifier/qualifier MRM transitions and expected retention times; at least one qualifier per analyte is required for optimal performance. Run process_batch(), which parses mzML with mzR, detects local maxima in a smoothed version of the quantifier trace and ranks candidate peaks by height (up to 5 candidates), then calculates RT shifts for each metabolite in each sample by simultaneous cross-correlation against a reference sample. It sums RT-shifted chromatograms into a prototype, fits RT shifts as a quadratic function of RT, propagates peak boundaries back to individual samples, and the peak reporting ML model accepts or rejects each peak using the 20 quality scores plus the peak-picking classification score and its 0th/25th/50th/75th/100th percentiles across samples. Intensities are written to peakinfo.xlsx; validate by computing Pearson correlation between automRm peak areas and originally reported intensities (success: >0.95 for the majority of metabolites, e.g. 22/34 in MTBLS429 and 20/30 in MTBLS897).

## Related tools

- **automRm** (Core R package executing peak picking, RT alignment, prototype chromatogram construction, and ML-based peak reporting via process_batch()) — https://gitlab.gwdg.de/joerg.buescher/automrm
- **mzR** (Parses mzML files to extract MRM chromatogram traces for peak picking)
- **caret** (Machine learning framework used for training the random-forest peak picking and peak reporting models (train_model()))
- **openxlsx** (Reads the metabdb.xlsx metabolite database and writes the peakinfo.xlsx peak/QC table)

## Examples

```
devtools::install_gitlab('joerg.buescher/automRm@master', host = 'https://gitlab.gwdg.de'); automRm::process_batch()
```

## Evaluation signals

- Pearson correlation between automRm peak areas and originally reported signal intensities exceeds 0.95 for the majority of metabolites (paper: 22/34 for MTBLS429, 20/30 for MTBLS897)
- peakinfo.xlsx is produced for each data set with intensity values for accepted peaks only
- Number of correctly reported peaks and variation in peak integration among replicate measurements match or exceed alternative software on the HILIC-MS benchmark
- Each accepted metabolite peak has a quantifier plus at least one qualifier transition consistent with metabdb.xlsx
- RT shifts across samples fit the expected quadratic function of RT and peak boundaries propagate consistently from the prototype chromatogram

## Limitations

- For optimal performance, automRm requires at least one qualifier in addition to the quantifier, and adding qualifiers reduces the number of analytes detectable simultaneously in MRM mode
- Manual peak review is still needed for selected metabolites in special cases such as excessively dirty samples, very-low-input samples, or new LC methods
- Quality similar to manual peak review is only reached with sufficient training on relevant raw data; the impact of training data set choice and ML algorithm on correct reporting is significant

## Evidence

- [abstract] Here, we present the R package automRm for fully automatic preprocessing of LC-MS data recorded in MRM mode.: "Here, we present the R package automRm for fully automatic preprocessing of LC-MS data recorded in MRM mode."
- [methods] local maxima are detected in a smoothed version of the quantifier trace and ranked by peak height: "local maxima are detected in a smoothed version of the quantifier trace and ranked by peak height"
- [methods] calculate RT shifts separately for each metabolite in each sample by the simultaneous cross-correlation: "calculate RT shifts separately for each metabolite in each sample by the simultaneous cross-correlation"
- [discussion] For optimal performance, automRm requires the use of at least one qualifier in addition to the quantifier.: "For optimal performance, automRm requires the use of at least one qualifier in addition to the quantifier."
- [discussion] We only retreat to manual peak review for selected metabolites in special cases such as excessively dirty samples, very-low-input samples, or new LC methods.: "We only retreat to manual peak review for selected metabolites in special cases such as excessively dirty samples, very-low-input samples, or new LC methods."
