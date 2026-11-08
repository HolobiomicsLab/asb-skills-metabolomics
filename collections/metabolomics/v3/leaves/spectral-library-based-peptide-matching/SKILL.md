---
name: spectral-library-based-peptide-matching
description: Use when raw DIA files (e.g. .d/.raw/.wiff from timsTOF, TripleTOF, or
  Orbitrap instruments) must be quantified against an existing spectral library rather
  than searched de novo, e.g. to extract peptide precursor quantities from mouse liver
  digest QC runs.
when_to_use_negative:
- No project-specific spectral library exists and de novo/database (DDA-style) search
  is required instead of library-based matching
- Input is already a processed feature/quantification table rather than raw DIA mass
  spectrometry files
- The data are DDA acquisitions meant for DDA QC tools such as QuaMeter rather than
  library-based DIA quantification
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: DIA-NN
  role: Core engine that analyzes DIA files against the spectral library (v1.7.15)
    to derive peptide and protein quantification under 1% FDR filtration
- name: iDIA-QC
  role: GUI wrapper whose pipeline incorporates DIA-NN protein qualitative and quantitative
    algorithms for library-based DIA analysis feeding QC metrics
  repo: github.com/guomics-lab/iDIA-QC
- name: msConvert
  role: Converts .d/.raw/.wiff vendor files to mzXML used in the analysis pipeline
- name: Skyline
  role: Downstream targeted inspection of transition peak shapes for precursors retained
    from library-based matching
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-54871-1
    title: 'iDIA-QC: AI-empowered data-independent acquisition mass spectrometry-based
      quality control'
  source_content_hash: sha256:801fcb3585496579910f3af5df2902b82bc8ac51b3ac7c5562f052036ecc2b1f
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
  - Skyline
  - iDIA-QC
  - msConvert
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# Spectral-library-based peptide matching for DIA data

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Analyzes DIA raw files against a project-specific, empirically built spectral library (here the published mouse spectral library of 110,126 precursors / 10,542 protein groups) using DIA-NN (v1.7.15), with 1% FDR filtration at both peptide precursor and protein levels, to derive quantitative peptide precursor data for downstream QC or panel-building tasks.

## When to use

Use when raw DIA files (e.g. .d/.raw/.wiff from timsTOF, TripleTOF, or Orbitrap instruments) must be quantified against an existing spectral library rather than searched de novo, e.g. to extract peptide precursor quantities from mouse liver digest QC runs.

## When NOT to use

- No project-specific spectral library exists and de novo/database (DDA-style) search is required instead of library-based matching
- Input is already a processed feature/quantification table rather than raw DIA mass spectrometry files
- The data are DDA acquisitions meant for DDA QC tools such as QuaMeter rather than library-based DIA quantification

## Inputs

- Raw DIA mass spectrometry files (.d, .raw, or .wiff) from timsTOF, TripleTOF, or Orbitrap instruments
- A project-matched spectral library (e.g. published mouse spectral library: 110,126 precursors, 10,542 protein groups)
- Optionally, mzXML-converted files produced via msConvert

## Outputs

- Peptide precursor-level quantification table from DIA-NN
- Protein-level quantification results
- Precursor-level data supporting QC metric extraction (e.g. DIA-NN outputs fed into iDIA-QC) or curated precursor panels

## How to apply

First obtain a project-matched spectral library — in this workflow, a previously established mouse spectral library containing 110,126 precursors and 10,542 protein groups, built by fractionating a mouse liver digest into 90 reversed-phase fractions combined into 30 fractions. Convert vendor raw files (.d/.raw/.wiff) to mzXML with msConvert if the tool requires converted input. Analyze the DIA files with DIA-NN (v1.7.15) against the spectral library, applying a data filtration criterion of 1% FDR at both peptide precursor and protein levels. Extract the resulting peptide and protein quantification (precursor-level intensities) for downstream steps such as QC-metric extraction, longitudinal quality summaries, or consistency-filtered precursor panels. The library-centric strategy ensures precursor identity is constrained to pre-validated transitions, which underpins accurate transition-level inspection in tools like Skyline.

## Related tools

- **DIA-NN** (Core engine that analyzes DIA files against the spectral library (v1.7.15) to derive peptide and protein quantification under 1% FDR filtration)
- **iDIA-QC** (GUI wrapper whose pipeline incorporates DIA-NN protein qualitative and quantitative algorithms for library-based DIA analysis feeding QC metrics) — github.com/guomics-lab/iDIA-QC
- **msConvert** (Converts .d/.raw/.wiff vendor files to mzXML used in the analysis pipeline)
- **Skyline** (Downstream targeted inspection of transition peak shapes for precursors retained from library-based matching)

## Examples

```
msConvert --filetype mzXML *.raw; diann --f *.mzXML --lib mouse_spectral_library.tsv --fdr-pep 0.01 --fdr-protein 0.01 --report quant.tsv
```

## Evaluation signals

- 1% FDR filtration is confirmed at BOTH peptide precursor and protein levels in the DIA-NN output
- Identified precursors should fall within the spectral library scope (~110,126 precursors / 10,542 protein groups for the mouse liver library); identification of precursors absent from the library indicates a misconfiguration
- Derived precursor quantification is consistent across all replicate DIA files when later screened for panel consistency (e.g. supports retention of a 934-precursor panel with 90% of peptides 7–15 aa long)
- Quantification outputs are extractable as numerical metrics from DIA-NN results for QC metric computation
- Per-file analysis completes in reasonable time (iDIA-QC reports under 5 minutes per file) and produces parseable peptide/protein quantification tables

## Limitations

- Library-based matching is bounded by the library content; peptides outside the 110,126-precursor library are invisible to the analysis
- The spectral library is project- and species-specific (mouse liver digest); transferring to other species or sample types requires building a new library
- Quantification quality depends on upstream 1% FDR thresholds; permissive FDR inflates false precursor identifications that corrupt downstream QC metrics or panel selection

## Evidence
