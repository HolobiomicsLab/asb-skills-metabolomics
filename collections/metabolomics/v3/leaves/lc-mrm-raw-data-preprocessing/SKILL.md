---
name: lc-mrm-raw-data-preprocessing
description: Use when a batch of raw MRM-mode LC-MS mzML files (with quantifier and
  qualifier transitions and expected RTs per metabolite) must be peak-picked, RT-aligned,
  scored, and reported automatically.
when_to_use_negative:
- Excessively dirty samples, very-low-input samples, or brand-new LC methods where
  the article says one should retreat to manual peak review for selected metabolites
- Data lacking any qualifier transition besides the quantifier, since automRm requires
  at least one qualifier for optimal performance
- Raw data not in MRM mode or not parseable as mzML (the pipeline is built specifically
  on mzR parsing of mzML files)
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: automRm
  role: Core R package providing process_batch(), the end-to-end MRM preprocessing
    entry point with ML peak picking and reporting
  repo: https://gitlab.gwdg.de/joerg.buescher/automrm
- name: mzR
  role: Dependency used by automRm for parsing mzML raw files
- name: caret
  role: Dependency providing the machine learning backend for peak picking and peak
    reporting models
- name: openxlsx
  role: Dependency for reading metabdb.xlsx settings and writing the peakinfo.xlsx
    report
- name: devtools
  role: Used to install automRm from the GWDG GitLab host
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.1c05224
    title: 'automRm: An R Package for Fully Automatic LC-QQQ-MS Data Preprocessing
      Powered by Machine Learning'
  source_content_hash: sha256:0420abb9f5dde5027a62fd26867e34436c22cac7a7614c7e7ab7baa8b2fba33e
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
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# lc-mrm-raw-data-preprocessing

## Summary

Fully automatic, machine learning-powered preprocessing of LC-QQQ-MS MRM raw data in R via the automRm package, converting batches of mzML files into a quality-controlled quantification report without manual peak review.

## When to use

Use when a batch of raw MRM-mode LC-MS mzML files (with quantifier and qualifier transitions and expected RTs per metabolite) must be peak-picked, RT-aligned, scored, and reported automatically.

## When NOT to use

- Excessively dirty samples, very-low-input samples, or brand-new LC methods where the article says one should retreat to manual peak review for selected metabolites
- Data lacking any qualifier transition besides the quantifier, since automRm requires at least one qualifier for optimal performance
- Raw data not in MRM mode or not parseable as mzML (the pipeline is built specifically on mzR parsing of mzML files)

## Inputs

- Batch of raw LC-QQQ-MS mzML files recorded in MRM mode
- metabdb.xlsx metabolite settings file with quantifier/qualifier transitions and expected RTs
- sample.info file with sample metadata
- update_prm.tsv parameter file pointing to raw data, saved ML models, and metabolite settings
- Pretrained peak-picking and peak-reporting ML models (from the demodata repository or train_model())

## Outputs

- peakinfo.xlsx processed batch report with peak quantities and quality scores
- peakoverview.pdf chromatogram plots per metabolite/sample
- shiftplots.pdf retention-time-shift plots

## How to apply

Install automRm from the R console with devtools::install_gitlab('joerg.buescher/automRm@master', host='https://gitlab.gwdg.de'), which pulls dependencies mzR (mzML parsing), caret (machine learning), and openxlsx (xlsx I/O). Configure update_prm.tsv with raw-data locations, saved pretrained peak-picking/reporting ML models, and the metabolite settings file metabdb.xlsx, alongside sample.info metadata and quantifier/qualifier transitions with expected RTs. Run process_batch(), which detects up to 5 default peak candidates per metabolite per sample as local maxima in a smoothed quantifier trace ranked by peak height, computes 20 quality scores per candidate, estimates RT shifts by simultaneous cross-correlation against a reference sample fitted as a quadratic function of RT, sums RT-shifted chromatograms into a prototype chromatogram for robust peak detection, and lets the peak reporting ML model decide which peaks are reported. Verify completion by inspecting the emitted peakinfo.xlsx report plus peakoverview.pdf chromatogram plots and shiftplots.pdf RT-shift plots.

## Related tools

- **automRm** (Core R package providing process_batch(), the end-to-end MRM preprocessing entry point with ML peak picking and reporting) — https://gitlab.gwdg.de/joerg.buescher/automrm
- **mzR** (Dependency used by automRm for parsing mzML raw files)
- **caret** (Dependency providing the machine learning backend for peak picking and peak reporting models)
- **openxlsx** (Dependency for reading metabdb.xlsx settings and writing the peakinfo.xlsx report)
- **devtools** (Used to install automRm from the GWDG GitLab host)

## Examples

```
devtools::install_gitlab('joerg.buescher/automRm@master', host = 'https://gitlab.gwdg.de'); automRm::process_batch(getmd = 'update_prm.tsv')
```

## Evaluation signals

- peakinfo.xlsx is produced and populated for every metabolite/sample combination in the batch (end-to-end completion)
- Reported peaks replicate results obtained by manual peak review on published data (e.g., MTBLS429, MTBLS897 benchmarks)
- Variation in peak integration among replicate measurements is lower than alternative software on the HILIC-MS benchmark data set
- shiftplots.pdf shows RT shifts well fitted by the quadratic function of RT across metabolites
- Number of correctly reported peaks (judged by the ML reporting model on 20 quality scores) matches or exceeds alternative tools

## Limitations

- Requires at least one qualifier transition in addition to the quantifier; adding qualifiers reduces the number of analytes detectable simultaneously in MRM mode
- Quality similar to manual peak review is only reached with sufficient training on relevant raw data; manual review is still needed for excessively dirty samples, very-low-input samples, or new LC methods
- ML training-data choice, algorithm, and individual peak characteristics affect correct peak reporting

## Evidence

- [abstract] automRm performs fully automatic preprocessing: "Here, we present the R package automRm for fully automatic preprocessing of LC-MS data recorded in MRM mode."
- [methods] process_batch entry point: "process_batch() for preprocessing of batches of raw data"
- [methods] dependencies: "automRm depends on several packages such as mzR7 for parsing of mzML files"
- [methods] RT shift fitting: "RT shifts of all metabolites in a sample are fitted as a quadratic function of RT"
- [discussion] qualifier requirement: "For optimal performance, automRm requires the use of at least one qualifier in addition to the quantifier."
- [discussion] manual review fallback: "We only retreat to manual peak review for selected metabolites in special cases such as excessively dirty samples, very-low-input samples, or new LC methods."
