---
name: spectral-library-search-execution
description: Use when DIA raw acquisitions (.raw/.d/.wiff) must be searched with DIA-NN
  against a spectral library to produce 1% FDR-filtered peptide/protein identifications
  and per-file protein counts.
when_to_use_negative:
- Input data are DDA acquisitions — the article's QC/identification pipeline targets
  DIA files; DDA data go through DDA platforms such as FragPipe with MSFragger instead
- No spectral library matching the sample type is available — this skill is library-based
  (search against a pre-established precursor/protein-group library), not library-free
- The goal is QC quality assessment of raw files rather than identification — use
  the iDIA-QC metric-extraction/ML-classification path instead of plain DIA-NN counting
edam_operation: http://edamontology.org/operation_0252
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: DIA-NN
  role: Performs library-based DIA search and peptide/protein quantification under
    1% FDR at precursor and protein levels (v1.7.15)
- name: msConvert
  role: Converts vendor raw files (.d/.raw/.wiff) to mzXML prior to DIA-NN search
- name: iDIA-QC
  role: GUI wrapper that incorporates DIA-NN's qualitative/quantitative algorithms
    and msConvert conversion, organized per instrument type (timsTOF, TripleTOF, Orbitrap)
  repo: github.com/guomics-lab/iDIA-QC
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-54871-1
    title: 'iDIA-QC: AI-empowered data-independent acquisition mass spectrometry-based
      quality control'
  source_content_hash: sha256:c18ea3bf1d34b9d08a71d669f9ca530913b07162d24e06afb5a5e6b931b9f2ce
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
  edam_operation: http://edamontology.org/operation_0252
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# spectral-library-search-execution

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Executes library-based DIA identification by converting vendor raw files to mzXML with msConvert and searching them with DIA-NN against a pre-built spectral library under 1% FDR, yielding per-file peptide/protein quantification that can be summarized as identification counts.

## When to use

Use when DIA raw acquisitions (.raw/.d/.wiff) must be searched with DIA-NN against a spectral library to produce 1% FDR-filtered peptide/protein identifications and per-file protein counts.

## When NOT to use

- Input data are DDA acquisitions — the article's QC/identification pipeline targets DIA files; DDA data go through DDA platforms such as FragPipe with MSFragger instead
- No spectral library matching the sample type is available — this skill is library-based (search against a pre-established precursor/protein-group library), not library-free
- The goal is QC quality assessment of raw files rather than identification — use the iDIA-QC metric-extraction/ML-classification path instead of plain DIA-NN counting

## Inputs

- Vendor DIA raw acquisition files (.raw/.d/.wiff) from a PRIDE submission
- mzXML-converted files produced by msConvert
- Pre-built spectral library (e.g., mouse liver library: 110,126 precursors, 10,542 protein groups)

## Outputs

- DIA-NN peptide and protein quantification results per file
- Per-file identified protein counts
- protein_counts_table.csv with per-file counts plus group medians per instrument family (e.g., medians 2916/4381/6070 for TripleTOF/Orbitrap/timsTOF Pro)

## How to apply

Retrieve the deposited DIA raw files (.raw/.d/.wiff) from the PRIDE submission and convert each to mzXML using msConvert. Analyze each converted file with DIA-NN (v1.7.15) against the spectral library (here the mouse liver library with 110,126 precursors and 10,542 protein groups), applying the study's data filtration criterion of 1% FDR at both peptide precursor and protein levels. Extract the number of identified proteins per raw file from the DIA-NN quantification results. If instrument stratification is needed, group files by instrument family (TripleTOF, Orbitrap, timsTOF Pro — supported by iDIA-QC's input pane) and compute median protein counts per group. Rationale: DIA-NN supplies the protein qualitative and quantitative algorithms inside iDIA-QC, so library-based search output is the substrate for both identification counting and downstream QC metrics.

## Related tools

- **DIA-NN** (Performs library-based DIA search and peptide/protein quantification under 1% FDR at precursor and protein levels (v1.7.15))
- **msConvert** (Converts vendor raw files (.d/.raw/.wiff) to mzXML prior to DIA-NN search)
- **iDIA-QC** (GUI wrapper that incorporates DIA-NN's qualitative/quantitative algorithms and msConvert conversion, organized per instrument type (timsTOF, TripleTOF, Orbitrap)) — github.com/guomics-lab/iDIA-QC

## Evaluation signals

- 1% FDR was applied at both peptide precursor and protein levels before counting identifications
- Per-file protein counts never exceed the spectral library size (≤10,542 protein groups for the mouse liver library)
- Per-instrument-family medians fall in the reported ranges: TripleTOF ≈2916, Orbitrap ≈4381, timsTOF Pro ≈6070 proteins
- protein_counts_table.csv contains one row per analyzed raw file plus a median per instrument family, with instrument family labels present
- Conversion step succeeded: every input .raw/.d/.wiff has a corresponding mzXML that DIA-NN processed without error

## Limitations

- The study could not cover all LC and MS instruments, particularly newly emerging ones, so instrument-family generalization is bounded by the TripleTOF/Orbitrap/timsTOF Pro set used
- Identification counts are library-dependent: results reflect the specific mouse liver spectral library (110,126 precursors, 10,542 protein groups), not de novo discovery
- No PRIDE accession number is given in the article text, so raw-file retrieval requires locating the paired submission externally

## Evidence
