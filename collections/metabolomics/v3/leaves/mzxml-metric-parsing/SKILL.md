---
name: mzxml-metric-parsing
description: Use when raw DIA files (.raw/.d/.wiff from Orbitrap, TripleTOF, or timsTOF
  Pro instruments) must be converted to mzXML and mined for the iDIA-QC QC metrics
  per file.
when_to_use_negative:
- The goal is DDA or SRM QC — the article notes existing DDA/SRM QC pipelines 'cannot
  be directly applied to large scale quantitative proteomics experiments' and that
  this skill addresses the previously absent systematic DIA QC investigation
- Input is already an mzXML, MGF, or DIA-NN result file — skip msConvert and parse
  the relevant artifact directly
- Instruments outside the Orbitrap, TripleTOF, and timsTOF Pro series (especially
  newly emerging ones), which the study explicitly could not cover
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_0092
tools:
- name: msConvert
  role: Converts vendor raw files (.raw/.d/.wiff) to mzXML and supplies metrics F3
    and F5 extracted from the converted mzXML
- name: DIA-NN
  role: v1.7.15 analysis of converted DIA files to derive peptide and protein quantification
    results from which the remaining QC metrics are parsed
- name: iDIA-QC
  role: Python GUI wrapper that orchestrates conversion, metric extraction, ML prediction,
    HTML report and matrix outputs
  repo: github.com/guomics-lab/iDIA-QC
- name: timsTOF Pro
  role: Instrument whose MGF files provide the F15 metric
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-54871-1
    title: 'iDIA-QC: AI-empowered data-independent acquisition mass spectrometry-based
      quality control'
  source_content_hash: sha256:755b559c5e2eaf7f4440bcdebe8519c8fb962bba66180a5a584c1b42c3ba3354
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/guomics-lab/iDIA-QC
  tools:
  - DIA-NN
  - Orbitrap
  - TripleTOF
  - iDIA-QC
  - msConvert
  - timsTOF Pro
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# mzxml-metric-parsing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Parse mzXML-converted DIA raw files (plus DIA-NN outputs and timsTOF MGF files) to extract the numerical QC metrics (F3, F5 from mzXML; F15 from MGF; the remainder from DIA-NN results) that feed iDIA-QC's machine-learning quality evaluation.

## When to use

Use when raw DIA files (.raw/.d/.wiff from Orbitrap, TripleTOF, or timsTOF Pro instruments) must be converted to mzXML and mined for the iDIA-QC QC metrics per file.

## When NOT to use

- The goal is DDA or SRM QC — the article notes existing DDA/SRM QC pipelines 'cannot be directly applied to large scale quantitative proteomics experiments' and that this skill addresses the previously absent systematic DIA QC investigation
- Input is already an mzXML, MGF, or DIA-NN result file — skip msConvert and parse the relevant artifact directly
- Instruments outside the Orbitrap, TripleTOF, and timsTOF Pro series (especially newly emerging ones), which the study explicitly could not cover

## Inputs

- Vendor DIA raw files (.raw, .d, .wiff) from Orbitrap, TripleTOF, or timsTOF Pro instruments
- mzXML files produced by msConvert conversion
- MGF files acquired with the timsTOF Pro instrument (source of F15)
- DIA-NN (v1.7.15) peptide and protein quantification outputs
- Instrument type label and instrument ID per file

## Outputs

- 17 numerical QC metrics extracted per DIA raw file
- Pretrained-model qualified/unqualified evaluations for LC and MS performance with fault-cause guidance
- Self-contained HTML QC report per input file
- Flat metric matrices usable without statistical software such as R/Python
- Optional email/WeChat (pushplus) notification of prediction results

## How to apply

First record the instrument type (Orbitrap, TripleTOF, or timsTOF Pro) for each vendor raw file, then convert .raw/.d/.wiff files to mzXML format using msConvert. Analyze each converted DIA file with DIA-NN (v1.7.15, 1% FDR at both peptide precursor and protein levels) to derive peptide and protein quantification results. Extract metric F3 and F5 by parsing the mzXML files, obtain F15 by parsing MGF files generated from timsTOF Pro acquisitions, and parse the DIA-NN analysis outputs for all remaining metrics, yielding 17 numerical QC metrics per file (from >140 candidates the study narrowed to 15 practically applicable metrics). Feed the extracted values into the pretrained model that assigns qualified/unqualified evaluations for LC and MS performance; render results as a self-contained HTML report and flat matrices usable without R/Python. Handle missingness before modeling by removing features with >70% NA and imputing the rest with the per-feature minimum.

## Related tools

- **msConvert** (Converts vendor raw files (.raw/.d/.wiff) to mzXML and supplies metrics F3 and F5 extracted from the converted mzXML)
- **DIA-NN** (v1.7.15 analysis of converted DIA files to derive peptide and protein quantification results from which the remaining QC metrics are parsed)
- **iDIA-QC** (Python GUI wrapper that orchestrates conversion, metric extraction, ML prediction, HTML report and matrix outputs) — github.com/guomics-lab/iDIA-QC
- **timsTOF Pro** (Instrument whose MGF files provide the F15 metric)

## Evaluation signals

- Each input raw file yields an HTML QC report containing all 17 metrics plus LC/MS predictions
- Analysis completes with runtime under 5 minutes per file
- F3 and F5 values are traceable to the msConvert mzXML files, F15 to timsTOF Pro MGF files, and remaining metrics to DIA-NN outputs
- Metric matrices are readable without statistical software such as R/Python
- Features with >70% missing values are excluded and remaining NAs are imputed with the per-feature minimum before model input

## Limitations

- Metrics cover only raw files from timsTOF serial, TripleTOF, and Orbitrap serial instruments; the study notes it is 'not possible to include all LC and MS instruments, particularly those emerging new instruments'
- Model-building details (hyperparameters, train/test split composition) are not fully described
- Generalization of the workflow to other QC and experimental samples requires further investigation

## Evidence

- [discussion] the .d /.raw /.wiff files were converted to mzXML format using the msConvert software: [evidence span withheld — anchor status: not_found]
