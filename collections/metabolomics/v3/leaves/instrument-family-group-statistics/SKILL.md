---
name: instrument-family-group-statistics
description: Use when analyzing multi-vendor DIA raw cohorts (e.g., PRIDE-deposited
  .raw/.d/.wiff files) with DIA-NN and you need per-instrument-type protein ID counts
  and group medians.
when_to_use_negative:
- Input data are DDA rather than DIA acquisitions — this skill is specific to DIA
  files analyzed with DIA-NN against a DIA spectral library
- The goal is QC quality classification (qualified/unqualified LC or MS prediction),
  not protein identification counting per instrument group
- Samples/species differ from the mouse liver digest context and no matching spectral
  library is available, since counts depend directly on the library (10,542 protein
  groups)
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_0092
tools:
- name: DIA-NN
  role: Analyzes each converted DIA file (v1.7.15) to derive peptide and protein quantification
    results from which per-file protein counts are extracted
- name: msConvert
  role: Converts vendor raw files (.d/.raw/.wiff) to mzXML format prior to DIA-NN
    analysis
- name: iDIA-QC
  role: GUI wrapper that handles the raw-to-mzXML workflow and requires selecting
    the instrument type and instrument ID per file, enabling per-instrument-family
    grouping
  repo: github.com/guomics-lab/iDIA-QC
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-54871-1
    title: 'iDIA-QC: AI-empowered data-independent acquisition mass spectrometry-based
      quality control'
  source_content_hash: sha256:6ef9736b2bc8178e1938bf0209a0742837754e1cdaf50971b721a27a6818905d
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
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# instrument-family-group-statistics

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes per-instrument-family (TripleTOF, Orbitrap, timsTOF Pro) median protein identification counts from DIA-NN protein quantification results on large multi-vendor DIA cohorts, benchmarking instrument performance on a shared spectral library.

## When to use

Use when analyzing multi-vendor DIA raw cohorts (e.g., PRIDE-deposited .raw/.d/.wiff files) with DIA-NN and you need per-instrument-type protein ID counts and group medians.

## When NOT to use

- Input data are DDA rather than DIA acquisitions — this skill is specific to DIA files analyzed with DIA-NN against a DIA spectral library
- The goal is QC quality classification (qualified/unqualified LC or MS prediction), not protein identification counting per instrument group
- Samples/species differ from the mouse liver digest context and no matching spectral library is available, since counts depend directly on the library (10,542 protein groups)

## Inputs

- Vendor DIA raw acquisition files (.raw, .d, .wiff) from a PRIDE submission
- mzXML-converted DIA files (via msConvert)
- Reference spectral library with precursor and protein-group counts (e.g., mouse liver library: 110,126 precursors, 10,542 protein groups)
- Per-file instrument-type labels (e.g., TripleTOF, Orbitrap, timsTOF Pro)

## Outputs

- Per-file protein identification counts extracted from DIA-NN quantification results
- Median protein count per instrument family (expected: 2916 TripleTOF, 4381 Orbitrap, 6070 timsTOF Pro for this dataset)
- protein_counts_table.csv with per-file counts plus group medians

## How to apply

Retrieve the deposited DIA raw files (.raw/.d/.wiff) from the PRIDE submission and convert them to mzXML with msConvert. Analyze each converted file with DIA-NN (v1.7.15) against the mouse liver spectral library (110,126 precursors, 10,542 protein groups), applying the study's filtration criterion of 1% FDR at both peptide precursor and protein levels. Extract the number of identified proteins per raw file from the DIA-NN quantification results, label each file with its instrument type (the iDIA-QC interface requires choosing the instrument that generated the file and assigning an instrument ID), then compute the median protein count per instrument family and write per-file counts plus group medians to a table such as protein_counts_table.csv. As a correctness reference, this pipeline on the 2638-file cohort yielded medians of 2916 (TripleTOF), 4381 (Orbitrap), and 6070 (timsTOF Pro) proteins.

## Related tools

- **DIA-NN** (Analyzes each converted DIA file (v1.7.15) to derive peptide and protein quantification results from which per-file protein counts are extracted)
- **msConvert** (Converts vendor raw files (.d/.raw/.wiff) to mzXML format prior to DIA-NN analysis)
- **iDIA-QC** (GUI wrapper that handles the raw-to-mzXML workflow and requires selecting the instrument type and instrument ID per file, enabling per-instrument-family grouping) — github.com/guomics-lab/iDIA-QC

## Evaluation signals

- Per-file protein counts are non-negative integers extracted from DIA-NN quantification output after applying 1% FDR at both peptide precursor and protein levels
- Group medians match published values: 2916 (TripleTOF), 4381 (Orbitrap), 6070 (timsTOF Pro)
- Every analyzed file carries a valid instrument-family label and contributes to exactly one group median
- Number of analyzed files matches the deposited DIA cohort retrieved from PRIDE (2754 files in the source study; the reproducible subset for this metric is the 2638 deposited files)
- Output table contains both per-file counts and per-family medians with no missing instrument assignments

## Limitations

- Identified protein counts depend on the specific spectral library used (110,126 precursors, 10,542 protein groups for mouse liver); other libraries will produce different medians
- The study could not cover all LC and MS instruments, especially newly emerging ones, so results may not generalize to other instrument families
- No PRIDE accession number is provided in the article text, complicating direct data retrieval for reproduction

## Evidence
