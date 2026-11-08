---
name: r-batch-workflow-execution
description: Use when a raw batch of LC-MS/MS MRM data files (mzML) must be transformed
  end-to-end into a processed quantitative report (peakinfo.xlsx) via a single scripted
  R entry point like process_batch().
when_to_use_negative:
- Excessively dirty samples, very-low-input samples, or new LC methods where the article
  reports retreating to manual peak review for selected metabolites
- The acquisition method has no qualifier transitions — automRm requires at least
  one qualifier in addition to the quantifier for optimal performance
- No relevant pretrained or trainable ML models exist for the data (quality similar
  to manual review requires sufficient training on relevant raw data)
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: automRm
  role: R package whose process_batch() function is the end-to-end entry point for
    fully automatic batch preprocessing of MRM data
  repo: https://gitlab.gwdg.de/joerg.buescher/automrm
- name: devtools
  role: Used to install automRm from the GitLab host via devtools::install_gitlab()
- name: mzR
  role: Parses mzML raw files during batch processing
- name: caret
  role: Provides the machine learning backend for peak picking and peak reporting
    models used inside process_batch()
- name: openxlsx
  role: Writes the peakinfo.xlsx xlsx batch report
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.1c05224
    title: 'automRm: An R Package for Fully Automatic LC-QQQ-MS Data Preprocessing
      Powered by Machine Learning'
  source_content_hash: sha256:c2009c93a0558bc92d1715ec2f4e589fbf1bec68c6f708857a9395752b9345a6
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
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# r-batch-workflow-execution

## Summary

Execute an end-to-end batch preprocessing workflow in R, exemplified by automRm's process_batch(), which converts raw mzML MRM files plus metadata into a processed batch report without manual intervention.

## When to use

Use when a raw batch of LC-MS/MS MRM data files (mzML) must be transformed end-to-end into a processed quantitative report (peakinfo.xlsx) via a single scripted R entry point like process_batch().

## When NOT to use

- Excessively dirty samples, very-low-input samples, or new LC methods where the article reports retreating to manual peak review for selected metabolites
- The acquisition method has no qualifier transitions — automRm requires at least one qualifier in addition to the quantifier for optimal performance
- No relevant pretrained or trainable ML models exist for the data (quality similar to manual review requires sufficient training on relevant raw data)

## Inputs

- Raw MRM LC-MS data files in mzML format (batch of samples)
- update_prm.tsv parameter file with locations of raw data and saved pretrained ML models
- metabdb.xlsx metabolite-specific settings file (quantifier/qualifier transitions, expected RTs)
- sample.info sample metadata file
- Pretrained peak-picking and peak-reporting ML models (e.g. from the demodata repository)

## Outputs

- peakinfo.xlsx processed batch report
- peakoverview.pdf chromatogram plots
- shiftplots.pdf retention-time-shift plots

## How to apply

Install the workflow package from its GitLab source with devtools::install_gitlab('joerg.buescher/automRm@master', host='https://gitlab.gwdg.de'), ensuring the R dependencies mzR (mzML parsing), caret (machine learning), and openxlsx (xlsx output) are available. Prepare the update_prm.tsv parameter file pointing to the raw data files, the pretrained peak-picking and peak-reporting ML models, and the metabolite-specific settings file metabdb.xlsx containing quantifier/qualifier transitions and expected retention times. Then call process_batch(), which parses mzML raw data and sample metadata from sample.info, detects peak candidates (default 5 per metabolite per sample) ranked by 20 quality scores, cross-correlates RT shifts against a reference sample and fits them as a quadratic function of RT, sums RT-shifted chromatograms into a prototype chromatogram for robust peak detection, and lets the peak reporting ML model filter which peaks are reported. Verify success by inspecting the generated peakinfo.xlsx report along with peakoverview.pdf chromatogram plots and shiftplots.pdf retention-time-shift plots.

## Related tools

- **automRm** (R package whose process_batch() function is the end-to-end entry point for fully automatic batch preprocessing of MRM data) — https://gitlab.gwdg.de/joerg.buescher/automrm
- **devtools** (Used to install automRm from the GitLab host via devtools::install_gitlab())
- **mzR** (Parses mzML raw files during batch processing)
- **caret** (Provides the machine learning backend for peak picking and peak reporting models used inside process_batch())
- **openxlsx** (Writes the peakinfo.xlsx xlsx batch report)

## Examples

```
devtools::install_gitlab('joerg.buescher/automRm@master', host='https://gitlab.gwdg.de'); automRm::process_batch()
```

## Evaluation signals

- peakinfo.xlsx report is generated with entries for all metabolites/samples in the batch
- peakoverview.pdf and shiftplots.pdf are produced, confirming the RT-shift fitting and chromatogram plotting steps ran
- Reported peaks match results of manual peak review on the same batch (automRm replicates manual review on published data)
- RT shifts align via cross-correlation against the reference sample and fit a quadratic function of RT without residual misalignment
- Replicate measurements show low variation in peak integration and a high number of correctly reported peaks compared with alternative software

## Limitations

- For optimal performance, at least one qualifier in addition to the quantifier is required; adding qualifiers reduces the number of analytes detectable simultaneously in MRM mode
- Manual peak review is still needed for selected metabolites in special cases such as excessively dirty samples, very-low-input samples, or new LC methods
- Quality similar to manual peak review is reached only with sufficient training on relevant raw data

## Evidence

- [methods] process_batch() end-to-end entry point: "process_batch() for preprocessing of batches of raw data"
- [abstract] fully automatic MRM preprocessing: "Here, we present the R package automRm for fully automatic preprocessing of LC-MS data recorded in MRM mode."
- [methods] installation command: "devtools::install_gitlab('joerg.buescher/automRm@master', host = 'https://gitlab.gwdg.de')"
- [methods] dependencies: "automRm depends on several packages such as mzR7 for parsing of mzML files"
- [methods] RT shift fitting: "RT shifts of all metabolites in a sample are fitted as a quadratic function of RT"
- [discussion] manual review fallback: "We only retreat to manual peak review for selected metabolites in special cases such as excessively dirty samples, very-low-input samples, or new LC methods."
