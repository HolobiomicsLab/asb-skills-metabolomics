---
name: mrm-chromatogram-parsing-from-mzml
description: Use when raw LC-QQQ-MS data are provided as mzML files and per-metabolite
  quantifier/qualifier MRM chromatograms must be extracted per sample before peak
  picking and quality scoring.
when_to_use_negative:
- Data are not MRM-mode LC-MS (e.g. DDA or DIA full-scan data) — automRm targets 'fully
  automatic preprocessing of LC-MS data recorded in MRM mode'
- The acquisition method contains no qualifier transitions — 'For optimal performance,
  automRm requires the use of at least one qualifier in addition to the quantifier'
- A preprocessed peak/quality-score table already exists (e.g. qslog_initial.tsv)
  — parsing mzML again is redundant
edam_operation: http://edamontology.org/operation_3231
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: automRm
  role: R package whose preprocessing pipeline (process_batch()) orchestrates mzML
    parsing and subsequent peak picking and ML scoring
  repo: https://gitlab.gwdg.de/joerg.buescher/automrm
- name: mzR
  role: Dependency package used by automRm for parsing mzML files to extract MRM chromatograms
  repo: https://gitlab.gwdg.de/joerg.buescher/automrm
- name: openxlsx
  role: Used for reading and writing of xlsx files, including the metabolite database
    that defines the MRM transitions to extract
  repo: https://gitlab.gwdg.de/joerg.buescher/automrm
- name: devtools
  role: Used to install automRm from its GitLab host (devtools::install_gitlab)
  repo: https://gitlab.gwdg.de/joerg.buescher/automrm
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.1c05224
    title: 'automRm: An R Package for Fully Automatic LC-QQQ-MS Data Preprocessing
      Powered by Machine Learning'
  source_content_hash: sha256:eec5ef763b671839fdcfe11cc7f0f64f21477075d57a2cde5c3d325be654b924
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - automRm
  - devtools
  - mzR
  - openxlsx
  edam_operation: http://edamontology.org/operation_3231
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# mrm-chromatogram-parsing-from-mzml

## Summary

Parse raw LC-QQQ-MS MRM data by extracting quantifier and qualifier chromatograms for every metabolite in every sample from mzML files using mzR, as the first step of automRm's preprocessing pipeline.

## When to use

Use when raw LC-QQQ-MS data are provided as mzML files and per-metabolite quantifier/qualifier MRM chromatograms must be extracted per sample before peak picking and quality scoring.

## When NOT to use

- Data are not MRM-mode LC-MS (e.g. DDA or DIA full-scan data) — automRm targets 'fully automatic preprocessing of LC-MS data recorded in MRM mode'
- The acquisition method contains no qualifier transitions — 'For optimal performance, automRm requires the use of at least one qualifier in addition to the quantifier'
- A preprocessed peak/quality-score table already exists (e.g. qslog_initial.tsv) — parsing mzML again is redundant

## Inputs

- Batch of LC-QQQ-MS raw data files in mzML format (e.g. the automRm demodata batch from gitlab.gwdg.de/joerg.buescher/demodata)
- Metabolite database file (e.g. metabdb.xlsx) defining metabolites, quantifier and qualifier MRM transitions, and expected retention times
- Parameter file (update_prm.tsv) with smoothing degree and candidate count settings

## Outputs

- Per-metabolite, per-sample extracted quantifier MRM chromatograms (raw traces ready for smoothing and local-maxima peak picking)
- Per-metabolite, per-sample extracted qualifier MRM chromatograms for quality-score computation (e.g. QS_To2, QS_T.h, QS_T.l, QS_cor123)

## How to apply

Read each mzML file in the batch with mzR, which automRm depends on for parsing of mzML files, extracting both the quantifier and qualifier MRM chromatograms for each metabolite listed in the metabolite database (e.g. metabdb.xlsx) in each sample. Parameters controlling downstream smoothing and the number of candidate peaks per metabolite per sample come from the parameter file (update_prm.tsv). Rationale: machine-learning peak detection and quality control in automRm operate on these extracted chromatogram traces, so complete and correctly paired quantifier/qualifier extraction per metabolite per sample is a prerequisite for the subsequent peak-picking and ML scoring steps.

## Related tools

- **automRm** (R package whose preprocessing pipeline (process_batch()) orchestrates mzML parsing and subsequent peak picking and ML scoring) — https://gitlab.gwdg.de/joerg.buescher/automrm
- **mzR** (Dependency package used by automRm for parsing mzML files to extract MRM chromatograms) — https://gitlab.gwdg.de/joerg.buescher/automrm
- **openxlsx** (Used for reading and writing of xlsx files, including the metabolite database that defines the MRM transitions to extract) — https://gitlab.gwdg.de/joerg.buescher/automrm
- **devtools** (Used to install automRm from its GitLab host (devtools::install_gitlab)) — https://gitlab.gwdg.de/joerg.buescher/automrm

## Examples

```
devtools::install_gitlab('joerg.buescher/automRm@master', host = 'https://gitlab.gwdg.de')
```

## Evaluation signals

- Extracted chromatograms exist for every metabolite in the metabolite database in every sample of the batch
- Both quantifier and qualifier traces are present per transition — parsing that yields only quantifiers will break downstream quality scores such as QS_To2, QS_T.h/QS_T.l and QS_cor123
- Trace lengths and RT ranges are consistent across samples in the batch, enabling later RT alignment via cross-correlation
- Feeding the parsed traces into the next pipeline stage (smoothing and local-maxima detection) produces ranked candidate peaks per metabolite per sample as configured in update_prm.tsv

## Limitations

- automRm requires at least one qualifier in addition to the quantifier for optimal performance, so acquisitions with quantifier-only transitions cannot be fully scored
- Samples that are excessively dirty, very-low-input, or run with new LC methods may still require retreat to manual peak review after parsing and scoring
- automRm was developed and tested in R 3.6.0 (64-bit Linux) and tested in R 4.0.2 (macOS 10.14); other R environments are unverified

## Evidence

- [methods] automRm depends on several packages such as mzR7 for parsing of mzML files: "automRm depends on several packages such as mzR7 for parsing of mzML files"
- [methods] Peak picking with quality scores: "local maxima are detected in a smoothed version of the quantifier trace and ranked by peak height"
- [abstract] Here, we present the R package automRm for fully automatic preprocessing of LC-MS data recorded in MRM mode.: "Here, we present the R package automRm for fully automatic preprocessing of LC-MS data recorded in MRM mode."
- [methods] Devtools install invocation: "devtools::install_gitlab('joerg.buescher/automRm@master', host = 'https://gitlab.gwdg.de')"
