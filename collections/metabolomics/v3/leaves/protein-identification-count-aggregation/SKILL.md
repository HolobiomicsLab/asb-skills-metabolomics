---
name: protein-identification-count-aggregation
description: Use when DIA raw files (.raw/.d/.wiff) have been searched with DIA-NN
  against a spectral library and you need per-file protein identification counts plus
  medians aggregated by instrument family (e.g., TripleTOF vs Orbitrap vs timsTOF
  Pro).
when_to_use_negative:
- Input is already a peptide/protein feature table not produced by a DIA-NN search
  against a spectral library — per-file identification counts cannot be re-derived
- Data are DDA rather than DIA acquisitions, since this pipeline is specific to DIA-NN
  analysis of DIA files (DDA files in the study were handled with FragPipe/MSFragger
  instead)
- The goal is QC classification (qualified/unqualified) rather than identification-depth
  benchmarking — use the iDIA-QC metric extraction and ML prediction workflow instead
edam_operation: http://edamontology.org/operation_3767
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: DIA-NN
  role: Analyzes each converted DIA file (v1.7.15) against the mouse liver spectral
    library at 1% FDR at peptide precursor and protein levels; its quantification
    results supply the per-file protein identification counts
- name: msConvert
  role: Converts vendor raw files (.raw/.d/.wiff) to mzXML format before DIA-NN analysis
- name: iDIA-QC
  role: Software suite that incorporates DIA-NN algorithms and organizes files by
    instrument type; the protein-count aggregation step is the identification-depth
    side of its per-file analysis
  repo: github.com/guomics-lab/iDIA-QC
- name: TripleTOF / Orbitrap / timsTOF Pro
  role: Instrument families used as grouping labels for computing median protein identifications
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-54871-1
    title: 'iDIA-QC: AI-empowered data-independent acquisition mass spectrometry-based
      quality control'
  source_content_hash: sha256:7d6ef4f4723dc1300e122686e05944a797500ea44d318bfe765b074ec1078b56
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
  - TripleTOF / Orbitrap / timsTOF Pro
  - iDIA-QC
  - msConvert
  - timsTOF Pro
  edam_operation: http://edamontology.org/operation_3767
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# protein-identification-count-aggregation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Extracts per-file protein identification counts from DIA-NN quantification results of DIA mass spectrometry raw files and aggregates them into group medians per instrument family (TripleTOF, Orbitrap, timsTOF Pro), enabling cross-instrument identification benchmarks such as the 2916/4381/6070 medians reported for the iDIA-QC mouse liver dataset.

## When to use

Use when DIA raw files (.raw/.d/.wiff) have been searched with DIA-NN against a spectral library and you need per-file protein identification counts plus medians aggregated by instrument family (e.g., TripleTOF vs Orbitrap vs timsTOF Pro).

## When NOT to use

- Input is already a peptide/protein feature table not produced by a DIA-NN search against a spectral library — per-file identification counts cannot be re-derived
- Data are DDA rather than DIA acquisitions, since this pipeline is specific to DIA-NN analysis of DIA files (DDA files in the study were handled with FragPipe/MSFragger instead)
- The goal is QC classification (qualified/unqualified) rather than identification-depth benchmarking — use the iDIA-QC metric extraction and ML prediction workflow instead

## Inputs

- DIA raw vendor files (.raw, .d, .wiff) from a PRIDE submission
- mzXML-converted DIA files (via msConvert)
- DIA-NN (v1.7.15) quantification results per raw file
- Mouse liver spectral library (110,126 precursors, 10,542 protein groups)
- Per-file instrument-type labels (TripleTOF, Orbitrap, timsTOF Pro)

## Outputs

- Per-file protein identification counts extracted from DIA-NN results
- Median protein identification counts per instrument family (TripleTOF, Orbitrap, timsTOF Pro)
- protein_counts_table.csv combining per-file counts and group medians

## How to apply

Retrieve the deposited DIA raw acquisition files (.raw/.d/.wiff) from the PRIDE submission paired with the study, convert each vendor file to mzXML with msConvert, and analyze each file with DIA-NN (v1.7.15) against the in-house mouse liver spectral library (110,126 precursors, 10,542 protein groups), applying the study's data filtration criterion of 1% FDR at both peptide precursor and protein levels. From the DIA-NN quantification results, extract the number of identified proteins for each raw file, record the instrument type for that file (as in iDIA-QC's Input pane where you 'Choose the type of instrument that generated the file'), and compute the median protein count within each instrument family. Write both per-file counts and group medians to a table (e.g., protein_counts_table.csv) so identification depth can be compared across instrument families. The rationale is that DIA-based quantification with a fixed spectral library gives a directly comparable identification depth metric across multivendor instruments.

## Related tools

- **DIA-NN** (Analyzes each converted DIA file (v1.7.15) against the mouse liver spectral library at 1% FDR at peptide precursor and protein levels; its quantification results supply the per-file protein identification counts)
- **msConvert** (Converts vendor raw files (.raw/.d/.wiff) to mzXML format before DIA-NN analysis)
- **iDIA-QC** (Software suite that incorporates DIA-NN algorithms and organizes files by instrument type; the protein-count aggregation step is the identification-depth side of its per-file analysis) — github.com/guomics-lab/iDIA-QC
- **TripleTOF / Orbitrap / timsTOF Pro** (Instrument families used as grouping labels for computing median protein identifications)

## Evaluation signals

- Per-file protein counts are present for all analyzed DIA files (the study analyzed 2638 DIA QC files) with no missing instrument-type labels
- Group medians fall in the reported ranges: ~2916 (TripleTOF), ~4381 (Orbitrap), ~6070 (timsTOF Pro) for the mouse liver dataset analyzed with the 110,126-precursor spectral library
- 1% FDR was applied at both peptide precursor and protein levels before counting, matching the study's data filtration criterion
- protein_counts_table.csv contains both per-file counts and per-family medians, and medians are computed only within instrument family groups

## Limitations

- Results depend on the specific spectral library (110,126 precursors, 10,542 protein groups) and mouse Swiss-Prot FASTA (20,269 sequences); other libraries will shift identification counts
- Instrument families represented are limited to timsTOF, TripleTOF, and Orbitrap series; the study notes 'it is not possible to include all LC and MS instruments, particularly those emerging new instruments'
- Counts reflect the study's 1% FDR criterion; different FDR settings or DIA-NN versions change the medians and prevent direct comparison

## Evidence
