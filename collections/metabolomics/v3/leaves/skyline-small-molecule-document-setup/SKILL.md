---
name: skyline-small-molecule-document-setup
description: Use when starting a new Skyline project for small molecules — e.g. lipid
  precursors (DAG 30:0, DAG 32:0), amino acid panels (Biocrates p180), or polyamine
  targets — and you need the document's transition settings, extraction mode, and
  molecule inputs configured before importing raw data.
when_to_use_negative:
- When input is already a feature table from an untargeted tool (XCMS, MZmine 2, MS-DIAL)
  — Skyline targeted extraction requires precursor-level targets, not feature tables
- When the required workflow is unsupported in Skyline (e.g. GC-MS or all ion mobility
  small molecule modes) — the article notes support for these is still ongoing
- When vendor-specific latest-generation workflows are required and Skyline has not
  yet implemented support — use the vendor software instead
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Skyline
  role: Configures small-molecule document settings, imports transition lists, and
    defines precursor-ion extraction for targeted quantification
  repo: github.com/ProteoWizard/pwiz
- name: ProteoWizard
  role: Underlying library providing vendor raw file readers (mzML/mzXML) used by
    Skyline for chromatogram extraction
  repo: github.com/ProteoWizard/pwiz
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.jproteome.9b00640
    title: 'Skyline for Small Molecules: A Unifying Software Package for Quantitative
      Metabolomics'
  source_content_hash: sha256:7a2c602780d10c4aabd2a6087b11eafc0461d82c3cea30a8199b4ba55e6b0144
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - ProteoWizard
  - Skyline
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# Skyline small molecule document setup

## Summary

Configure a Skyline small-molecule document (transition list import, small molecule transition settings, precursor-ion extraction mode) so that HRMS or SRM/HRM data from vendor raw files can be quantitatively processed without vendor software. This skill reflects the extension of Skyline from peptide-centric to molecule-centric processing described in Adams et al. 2020.

## When to use

Use when starting a new Skyline project for small molecules — e.g. lipid precursors (DAG 30:0, DAG 32:0), amino acid panels (Biocrates p180), or polyamine targets — and you need the document's transition settings, extraction mode, and molecule inputs configured before importing raw data.

## When NOT to use

- When input is already a feature table from an untargeted tool (XCMS, MZmine 2, MS-DIAL) — Skyline targeted extraction requires precursor-level targets, not feature tables
- When the required workflow is unsupported in Skyline (e.g. GC-MS or all ion mobility small molecule modes) — the article notes support for these is still ongoing
- When vendor-specific latest-generation workflows are required and Skyline has not yet implemented support — use the vendor software instead

## Inputs

- Skyline .sky / .skyd document (empty or existing)
- Transition list (CSV/TSV) with precursor m/z, charge, adduct, and optional fragment ions
- Optional: isotope-labeled internal standard transitions for stable-isotope dilution
- Optional: reference retention time list for scheduled MRM/PRM

## Outputs

- Configured Skyline document with small molecule transition list loaded
- Extraction settings persisted for downstream chromatogram import and peak picking
- Optional: exported Skyline method (retention-time-scheduled transitions) for instrument acquisition

## How to apply

Create a new Skyline document with the small-molecule mode enabled, then import a transition list defining precursor m/z, charge, and adduct for each analyte (e.g. DAG 30:0, DAG 32:0). In Settings -> Transition Settings configure small molecule-specific options: choose the mass analyzer (Q1/Q3, precursor-only for HRMS, or transition-specific for SRM), set extraction tolerances (e.g. 5 ppm for high-resolution data), and select isotopes/adducts for M+H, M+Na or other observed ions. For HRMS precursor extraction set the precursor ion charge/adduct explicitly; for SRM use explicit Q1/Q3 pairs. Verify the settings against vendor software or published retention windows before peak picking — Skyline performance is benchmarked against a comparable instrument vendor tool. Note that Skyline may lag vendor-specific workflows for latest-generation instruments, so confirm that the document configuration supports the specific acquisition mode (e.g. Synapt G2 HDMS HDMS^E or sonication spray ionization) before committing to the setup.

## Related tools

- **Skyline** (Configures small-molecule document settings, imports transition lists, and defines precursor-ion extraction for targeted quantification) — github.com/ProteoWizard/pwiz
- **ProteoWizard** (Underlying library providing vendor raw file readers (mzML/mzXML) used by Skyline for chromatogram extraction) — github.com/ProteoWizard/pwiz

## Examples

```
skyline.exe --new-document --small-molecule --import-transition-list dag_targets.csv
```

## Evaluation signals

- After transition-list import, each small molecule row in the document shows the expected precursor m/z, charge, and adduct (e.g. DAG 30:0 with correct retention time) without duplicate or missing entries
- Setting the document to small molecule mode produces precursor-only extraction chromatograms for HRMS data (no fragment transitions for precursor-ion workflows)
- Retention-time-scheduled method export (e.g. 1-minute window centered on measured RT) matches published scheduling for the same analyte list
- Document opens and raw file import proceeds without format errors, confirming vendor raw readers (via pwiz) support the instrument data
- Extracted peak areas for standard compounds (e.g. p180 amino acids) reproduce published Skyline benchmark values against vendor software

## Limitations

- Skyline may lag behind commercial software for latest-generation or vendor-specific workflows until support is implemented
- Support for GC-MS and all types of ion mobility small molecule data is still ongoing — some acquisition modes cannot be processed
- Fragmentation library workflows for Metlin, MassBank, HMDB, and NIST are not yet implemented, so spectral library matching is limited
- No single tool will be best for all workflows — evaluate whether Skyline's targeted paradigm fits the analytical goal

## Evidence
