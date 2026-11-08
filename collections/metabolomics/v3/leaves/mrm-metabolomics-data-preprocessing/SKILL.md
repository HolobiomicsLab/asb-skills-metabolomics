---
name: mrm-metabolomics-data-preprocessing
description: Use when preprocessing batches of MRM-mode LC-QQQ-MS chromatograms (mzML
  raw data) into per-analyte peak intensity tables with automatic accept/reject peak
  decisions, especially when manual peak review is too slow or is to be benchmarked.
when_to_use_negative:
- Data were acquired without at least one qualifier transition in addition to the
  quantifier, since automRm requires qualifiers for optimal performance
- Samples are excessively dirty, very-low-input, or derive from a new LC method, where
  manual peak review is still needed for selected metabolites
- The data are not MRM-mode LC-QQQ-MS raw data (e.g. untargeted full-scan MS or already
  preprocessed feature tables)
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: automRm
  role: Core R package providing process_batch() for fully automatic MRM preprocessing
    and ML-based peak reporting
  repo: https://gitlab.gwdg.de/joerg.buescher/automrm
- name: mzR
  role: Parses mzML chromatogram files during process_batch()
- name: openxlsx
  role: Writes peak intensities to peakinfo.xlsx and reads the metabdb.xlsx metabolite
    database
- name: caret
  role: Machine learning framework used for the random-forest peak picking and peak
    reporting models
- name: devtools
  role: Installs automRm from the GitLab repository
  repo: https://gitlab.gwdg.de/joerg.buescher/automrm
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.1c05224
    title: 'automRm: An R Package for Fully Automatic LC-QQQ-MS Data Preprocessing
      Powered by Machine Learning'
  source_content_hash: sha256:7e4188defc0193b18e787e527a88cb5596e97533efd05d9220493acf0a47e917
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
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 16
---

# mrm-metabolomics-data-preprocessing

## Summary

Fully automatic preprocessing of LC-QQQ-MS data recorded in MRM mode using the automRm R package, which combines peak picking, RT alignment, and random-forest-based peak quality control to produce peak/QC tables that can replicate manual peak review.

## When to use

Use when preprocessing batches of MRM-mode LC-QQQ-MS chromatograms (mzML raw data) into per-analyte peak intensity tables with automatic accept/reject peak decisions, especially when manual peak review is too slow or is to be benchmarked.

## When NOT to use

- Data were acquired without at least one qualifier transition in addition to the quantifier, since automRm requires qualifiers for optimal performance
- Samples are excessively dirty, very-low-input, or derive from a new LC method, where manual peak review is still needed for selected metabolites
- The data are not MRM-mode LC-QQQ-MS raw data (e.g. untargeted full-scan MS or already preprocessed feature tables)

## Inputs

- mzML raw data files from MRM-mode LC-QQQ-MS acquisitions (e.g. Agilent 6495 QQQ-MS batches)
- metabdb.xlsx metabolite database with quantifier/qualifier MRM transitions and expected retention times
- update_prm.tsv configuration file pointing to raw data, models, and metabolite database
- pretrained combined random-forest peak picking and peak reporting models

## Outputs

- peakinfo.xlsx per data set with accepted peak intensity values per analyte per sample
- accepted/rejected peak decisions from the random-forest peak reporting model
- RT-shift alignments and prototype chromatograms per metabolite

## How to apply

Install automRm in R via devtools::install_gitlab('joerg.buescher/automRm@master', host='https://gitlab.gwdg.de') and load the pretrained combined random-forest peak picking and peak reporting models. Configure update_prm.tsv to point to the mzML raw data, the pretrained models, and a metabolite database (metabdb.xlsx) listing quantifier/qualifier MRM transitions and expected retention times. Run process_batch(), which parses mzML with mzR, detects local maxima on smoothed quantifier traces (ranked by peak height, up to 5 candidates), aligns RT shifts by cross-correlation against a reference sample, sums RT-shifted chromatograms into a prototype, and propagates peak boundaries back to each sample. The peak reporting model then uses the 20 quality scores plus the peak-picking classification score and its 0th/25th/50th/75th/100th percentiles across samples to accept or reject each peak, writing intensities to peakinfo.xlsx. Success is judged by Pearson correlation between automRm peak areas and originally reported intensities (paper: >0.95 for 22/34 metabolites in MTBLS429 and 20/30 in MTBLS897).

## Related tools

- **automRm** (Core R package providing process_batch() for fully automatic MRM preprocessing and ML-based peak reporting) — https://gitlab.gwdg.de/joerg.buescher/automrm
- **mzR** (Parses mzML chromatogram files during process_batch())
- **openxlsx** (Writes peak intensities to peakinfo.xlsx and reads the metabdb.xlsx metabolite database)
- **caret** (Machine learning framework used for the random-forest peak picking and peak reporting models)
- **devtools** (Installs automRm from the GitLab repository) — https://gitlab.gwdg.de/joerg.buescher/automrm

## Examples

```
devtools::install_gitlab('joerg.buescher/automRm@master', host = 'https://gitlab.gwdg.de'); automRm::process_batch()
```

## Evaluation signals

- Pearson correlation between automRm peak areas and originally reported signal intensities exceeds 0.95 for the majority of metabolites (paper: 22/34 in MTBLS429, 20/30 in MTBLS897)
- peakinfo.xlsx is produced for each data set with one intensity value per analyte per sample and per-peak accept/reject flags
- Peak integration variation among replicate measurements and the number of correctly reported peaks should match or exceed alternative software solutions as reported for the HILIC-MS data set
- Each reported analyte has at least one qualifier transition evaluated alongside its quantifier in metabdb.xlsx
- RT alignment succeeds without manual adjustment: RT shifts are fitted as a quadratic function of RT across metabolites per sample

## Limitations

- For optimal performance, automRm requires the use of at least one qualifier in addition to the quantifier; adding qualifiers reduces the number of analytes detectable simultaneously in MRM mode
- Manual peak review is still needed for selected metabolites in special cases such as excessively dirty samples, very-low-input samples, or new LC methods
- Reaching quality similar to manual peak review requires sufficient training on relevant raw data
- Developed/tested in R 3.6.0 on 64-bit Linux and R 4.0.2 on macOS 10.14; other environments may be untested

## Evidence

- [abstract] Here, we present the R package automRm for fully automatic preprocessing of LC-MS data recorded in MRM mode.: "Here, we present the R package automRm for fully automatic preprocessing of LC-MS data recorded in MRM mode."
- [methods] process_batch() for preprocessing of batches of raw data: "process_batch() for preprocessing of batches of raw data"
- [methods] local maxima are detected in a smoothed version of the quantifier trace and ranked by peak height: "local maxima are detected in a smoothed version of the quantifier trace and ranked by peak height"
- [methods] RT shifts of all metabolites in a sample are fitted as a quadratic function of RT: "RT shifts of all metabolites in a sample are fitted as a quadratic function of RT"
- [methods] the peak reporting ML model uses the same quality scores that were previously used by the peak picking ML model: "the peak reporting ML model uses the same quality scores that were previously used by the peak picking ML model"
- [abstract] Next, we show that automRm can replicate results obtained by manual peak review on published data.: "Next, we show that automRm can replicate results obtained by manual peak review on published data."
- [discussion] For optimal performance, automRm requires the use of at least one qualifier in addition to the quantifier.: "For optimal performance, automRm requires the use of at least one qualifier in addition to the quantifier."
- [discussion] With sufficient training on relevant raw data, results from automRm can reach a similar level of quality as manual peak review.: "With sufficient training on relevant raw data, results from automRm can reach a similar level of quality as manual peak review."
