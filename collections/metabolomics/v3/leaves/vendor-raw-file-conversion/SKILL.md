---
name: vendor-raw-file-conversion
description: Use when starting from vendor-proprietary MS raw files (.raw, .d, or
  .wiff) that must be converted to mzXML before DIA-NN analysis or QC metric extraction
  in an iDIA-QC-style pipeline.
when_to_use_negative:
- Input is already in an open format (mzXML/mzML) — conversion would be redundant
- The goal is SRM/targeted QC with tools like SProCoP or Panorama AutoQC, which the
  article notes cannot be directly applied to large-scale DIA experiments
- Instrument type is unknown or not among the supported families (Orbitrap, TripleTOF,
  timsTOF), since the article notes not all LC-MS instruments, especially newly emerging
  ones, are covered
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: iDIA-QC
  role: 'GUI that wraps the conversion step: user selects raw files, assigns instrument
    type/ID, and the suite uses msConvert to produce extracted precursor ion chromatograms
    for downstream metric extraction'
  repo: https://github.com/guomics-lab/iDIA-QC
- name: msConvert
  role: Performs the actual vendor-format (.raw/.d/.wiff) to mzXML conversion feeding
    DIA-NN and metric extraction
- name: DIA-NN
  role: Consumes converted data downstream to derive peptide/protein quantification
    (v1.7.15, 1% FDR at precursor and protein levels)
provenance:
  source_task_ids:
  - task_002
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-54871-1
    title: 'iDIA-QC: AI-empowered data-independent acquisition mass spectrometry-based
      quality control'
  source_content_hash: sha256:90e0d88528724f4abd03cfded7f727ffa0b0353e3b04042035d44ecb3190e77e
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/guomics-lab/iDIA-QC
  repo_url: https://github.com/guomics-lab/iDIA-QC
  tools:
  - DIA-NN
  - Orbitrap
  - TripleTOF
  - iDIA-QC
  - msConvert
  - timsTOF Pro
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# vendor-raw-file-conversion

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Converts multivendor mass spectrometry raw acquisition files (.raw, .d, .wiff) into open mzXML format with msConvert so that downstream DIA QC analysis (DIA-NN quantification, metric extraction) can proceed uniformly across Orbitrap, TripleTOF, and timsTOF Pro instruments.

## When to use

Use when starting from vendor-proprietary MS raw files (.raw, .d, or .wiff) that must be converted to mzXML before DIA-NN analysis or QC metric extraction in an iDIA-QC-style pipeline.

## When NOT to use

- Input is already in an open format (mzXML/mzML) — conversion would be redundant
- The goal is SRM/targeted QC with tools like SProCoP or Panorama AutoQC, which the article notes cannot be directly applied to large-scale DIA experiments
- Instrument type is unknown or not among the supported families (Orbitrap, TripleTOF, timsTOF), since the article notes not all LC-MS instruments, especially newly emerging ones, are covered

## Inputs

- Vendor raw MS files: Thermo .raw, Bruker .d, SCIEX .wiff (DIA and paired DDA acquisitions)
- Instrument type and instrument ID assignments per file (Orbitrap / TripleTOF / timsTOF Pro)

## Outputs

- mzXML files, one per converted raw acquisition, used for QC metric extraction and DIA-NN input

## How to apply

In the iDIA-QC Input pane, select the vendor raw files and assign each file its instrument type (Orbitrap, TripleTOF, or timsTOF serial) and an instrument ID, since conversion and later metric extraction are instrument-aware. Run msConvert to convert the .raw/.d/.wiff files to mzXML, which serves as the open-format input for both DIA-NN (v1.7.15) analysis against the mouse spectral library and extraction of numerical QC metrics (peptide number, protein number, MS1 area for DDA, MS2 intensity for DIA). Retain the mzXML outputs alongside DIA-NN results, as QC metrics are extracted from these two sources per file; the iDIA-QC GUI then consumes the results to build HTML reports and metric matrices.

## Related tools

- **iDIA-QC** (GUI that wraps the conversion step: user selects raw files, assigns instrument type/ID, and the suite uses msConvert to produce extracted precursor ion chromatograms for downstream metric extraction) — https://github.com/guomics-lab/iDIA-QC
- **msConvert** (Performs the actual vendor-format (.raw/.d/.wiff) to mzXML conversion feeding DIA-NN and metric extraction)
- **DIA-NN** (Consumes converted data downstream to derive peptide/protein quantification (v1.7.15, 1% FDR at precursor and protein levels))

## Examples

```
msconvert --mzXML -o mzXML_out/ *.raw
```

## Evaluation signals

- Every selected vendor file (.raw/.d/.wiff) yields a corresponding mzXML output without conversion errors
- Each converted file carries a correct instrument type and instrument ID assignment for downstream per-instrument grouping (TripleTOF, Orbitrap, timsTOF Pro)
- Downstream DIA-NN runs on the converted files produce peptide/protein counts at 1% FDR at both peptide precursor and protein levels
- Extracted QC metrics (peptide number, protein number, MS1 area / MS2 intensity per file) are present for every converted file in the metric matrix

## Limitations

- Not all LC-MS instruments are covered, particularly newly emerging ones
- Analysis time per file should stay under 5 minutes per the iDIA-QC design; very large batches may require batching
- Generalization of the workflow to other QC and experimental samples requires further investigation

## Evidence
