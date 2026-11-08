---
name: dia-proteomics-quantification
description: Use when vendor DIA raw acquisition files (.raw/.d/.wiff) from TripleTOF,
  Orbitrap, or timsTOF Pro instruments need conversion and peptide/protein identification
  and quantification against a spectral library.
when_to_use_negative:
- Input data are DDA or SRM acquisitions — the article states prior DDA/SRM QC pipelines
  'cannot be directly applied to large-scale DIA experiments' and this skill targets
  DIA files
- You need vendor-file QC screening only and already have mzXML/DIA-NN outputs — in
  that case feed them directly to the metric-extraction/ML step instead of re-running
  conversion and search
- Instruments outside the validated vendor families (TripleTOF, Orbitrap, timsTOF
  Pro) or newly emerging instrument types, which the authors note were not covered
edam_operation: http://edamontology.org/operation_3630
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: DIA-NN
  role: Analyzes converted mzXML DIA files (v1.7.15) against the spectral library
    to derive peptide and protein quantification with 1% FDR filtration
- name: msConvert
  role: Converts vendor .raw/.d/.wiff DIA raw files to mzXML before DIA-NN analysis
- name: iDIA-QC
  role: GUI wrapper that incorporates DIA-NN's protein qualitative/quantitative algorithms
    and msConvert conversion, extracts 15 QC metrics, and applies machine-learning
    quality prediction
  repo: github.com/guomics-lab/iDIA-QC
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-54871-1
    title: 'iDIA-QC: AI-empowered data-independent acquisition mass spectrometry-based
      quality control'
  source_content_hash: sha256:bf9fb62a00e026a7dd13b0dbd381c65d8a00471c669de710eb5d6e49ab8e9977
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
  edam_operation: http://edamontology.org/operation_3630
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# dia-proteomics-quantification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Converts vendor DIA mass-spectrometry raw files (.raw/.d/.wiff) to mzXML with msConvert and analyzes them with DIA-NN (v1.7.15) against a spectral library to extract peptide/protein quantification, applying 1% FDR at peptide precursor and protein levels, as done in the iDIA-QC mouse-liver QC study.

## When to use

Use when vendor DIA raw acquisition files (.raw/.d/.wiff) from TripleTOF, Orbitrap, or timsTOF Pro instruments need conversion and peptide/protein identification and quantification against a spectral library.

## When NOT to use

- Input data are DDA or SRM acquisitions — the article states prior DDA/SRM QC pipelines 'cannot be directly applied to large-scale DIA experiments' and this skill targets DIA files
- You need vendor-file QC screening only and already have mzXML/DIA-NN outputs — in that case feed them directly to the metric-extraction/ML step instead of re-running conversion and search
- Instruments outside the validated vendor families (TripleTOF, Orbitrap, timsTOF Pro) or newly emerging instrument types, which the authors note were not covered

## Inputs

- Vendor DIA raw acquisition files (.raw, .d, .wiff)
- Spectral library (e.g., mouse liver library with 110,126 precursors and 10,542 protein groups)
- Instrument-type annotation for each file (TripleTOF, Orbitrap, timsTOF Pro)

## Outputs

- Per-file peptide and protein quantification/identification results from DIA-NN
- Protein counts per raw file and per instrument-family medians (e.g., protein_counts_table.csv)
- mzXML-converted intermediates and extracted numerical QC metrics

## How to apply

Retrieve deposited DIA raw files (e.g., from the PRIDE submission) and convert each .raw/.d/.wiff file to mzXML using msConvert, noting the instrument type that generated the file. Analyze each converted file with DIA-NN (v1.7.15) against a previously established spectral library (in this study, mouse liver: 110,126 precursors, 10,542 protein groups), enforcing 1% FDR at both peptide precursor and protein levels. Extract per-file counts of identified proteins (and peptides) from the DIA-NN quantification output. Summarize results per instrument family (e.g., median protein identifications of 2916 for TripleTOF, 4381 for Orbitrap, 6070 for timsTOF Pro in the reference study) and write per-file counts plus group summaries to a table such as protein_counts_table.csv. These outputs also feed QC workflows: numerical metrics extracted from the mzXML files and DIA-NN results can be passed to downstream machine-learning QC models such as iDIA-QC.

## Related tools

- **DIA-NN** (Analyzes converted mzXML DIA files (v1.7.15) against the spectral library to derive peptide and protein quantification with 1% FDR filtration)
- **msConvert** (Converts vendor .raw/.d/.wiff DIA raw files to mzXML before DIA-NN analysis)
- **iDIA-QC** (GUI wrapper that incorporates DIA-NN's protein qualitative/quantitative algorithms and msConvert conversion, extracts 15 QC metrics, and applies machine-learning quality prediction) — github.com/guomics-lab/iDIA-QC

## Evaluation signals

- 1% FDR was applied at both peptide precursor and protein levels in the DIA-NN search
- Per-instrument-family median protein identifications should be in the expected range (reference study: 2916 TripleTOF, 4381 Orbitrap, 6070 timsTOF Pro)
- Every input raw file (.raw/.d/.wiff) produced an mzXML conversion and a corresponding DIA-NN quantification entry
- Output table contains per-file protein counts plus group medians matching the instrument annotations

## Limitations

- Study could not cover all LC and MS instruments, especially newly emerging ones
- Generalization of the workflow to other QC and experimental samples requires further investigation
- No DOI/accession number for the PRIDE submission is given in the text, complicating direct raw-file retrieval
- iDIA-QC's GUI targets timsTOF serial, TripleTOF, and Orbitrap instruments; other vendors are out of scope

## Evidence

- [discussion] DIA-NN analysis with 1% FDR: "the DIA files were analyzed using DIA-NN (v1.7.15) to derive peptide and protein quantification results"
- [discussion] FDR criterion: "a data filtration criterion of 1% FDR at both peptide precursor and protein levels"
- [discussion] File conversion step: "the. d /.raw /.wiff files were converted to mzXML format using the msConvert software"
- [discussion] Spectral library: "a previously established spectral library containing 110,126 precursors and 10,542 protein groups was utilized"
- [readme] Tool integration in iDIA-QC: "The software incorporates the protein qualitative and quantitative algorithms from DIA-NN and uses msConvert for file conversion"
- [discussion] Data availability: "All 2754 DIA files analyzed in the study have been uploaded to Proteomics IDEntifications database (PRIDE)."
