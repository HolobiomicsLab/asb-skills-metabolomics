---
name: dynamic-ppm-binning-peak-alignment
description: Use when centroided .imzML (or .h5ad/.h5mu) MSI data must have peaks
  aligned across all pixels before lock-mass recalibration, lipid annotation, or quantitation,
  especially for omics-scale lipidomics.
when_to_use_negative:
- Input is profile (non-centroided) spectra — the pipeline is designed for centroided
  .imzML import
- Data are already processed into a feature table or quantitative concentration maps
  — no peak alignment is needed
- A fixed absolute-mass binning is required by a downstream tool that cannot consume
  ppm-constant bins
edam_operation: http://edamontology.org/operation_3627
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0121
tools:
- name: LipidQMap
  role: Implements the dynamic binning peak-alignment step during .imzML import, with
    user-selected ppm mass-error tolerance and bin size for the average spectrum
  repo: github.com/swinnenteam/LipidQMap
- name: alexandrovteam/pyimzML
  role: Python parser for imzML 1.1.0 files that can read the centroided imaging data
    destined for alignment
  repo: alexandrovteam/pyimzML
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/2025.10.15.682422
    title: LipidQMap - An Open-Source Tool for Quantitative Mass Spectrometry Imaging
      of Lipids
  source_content_hash: sha256:272b07c495f10fe8da7e2732834014403d373add707ac3b3a1b479505f81b19e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - LipidQMap
  - alexandrovteam/pyimzML
  edam_operation: http://edamontology.org/operation_3627
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7333333333333333
n_claims: 15
---

# Dynamic ppm binning peak alignment

## Summary

Aligns centroided m/z peaks across all pixels of an MSI dataset using a dynamic binning algorithm that maintains constant resolution in ppm over the entire mass range, as implemented in LipidQMap prior to recalibration and lipid annotation.

## When to use

Use when centroided .imzML (or .h5ad/.h5mu) MSI data must have peaks aligned across all pixels before lock-mass recalibration, lipid annotation, or quantitation, especially for omics-scale lipidomics.

## When NOT to use

- Input is profile (non-centroided) spectra — the pipeline is designed for centroided .imzML import
- Data are already processed into a feature table or quantitative concentration maps — no peak alignment is needed
- A fixed absolute-mass binning is required by a downstream tool that cannot consume ppm-constant bins

## Inputs

- Centroided .imzML files (vendor-neutral, multiple files can be opened simultaneously)
- AnnData .h5ad files with var["mz"], var["mz_mode"], obsm["spatial"]
- MuData .h5mu files containing an MSI modality with matching feature metadata

## Outputs

- Pixel-aligned centroided peak grid with constant ppm resolution across the mass range
- Average mass spectrum of the image (binned, default 5 mDa bin size for TOF instruments)
- Recalibrated spectra ready for lock-mass correction and lipid annotation

## How to apply

Import centroided .imzML files directly in LipidQMap (positive and negative ion mode files can be loaded together, with ion mode auto-detected), then apply the dynamic binning algorithm that aligns centroided spectra across all pixels while keeping resolution constant in ppm across the entire mass range — this differs from fixed mDa binning, which the README suggests only for the average mass spectrum (default 5 mDa for TOF instruments, decreased for higher-resolution instruments). During import the user also sets the maximum tolerated mass error in ppm used for extracting ion images, so the ppm-constant binning must match that tolerance. Following binning, apply lock-mass recalibration with a 30 ppm peak-assignment tolerance and a minimum reference intensity of 10 000 (defaults), avoiding low-abundance reference signals (e.g. <8000 a.u. on timsTOF) that can be misassigned. The aligned, recalibrated peak grid then feeds the lipid annotation step against the user-modifiable .csv database with nine required columns.

## Related tools

- **LipidQMap** (Implements the dynamic binning peak-alignment step during .imzML import, with user-selected ppm mass-error tolerance and bin size for the average spectrum) — github.com/swinnenteam/LipidQMap
- **alexandrovteam/pyimzML** (Python parser for imzML 1.1.0 files that can read the centroided imaging data destined for alignment) — alexandrovteam/pyimzML

## Evaluation signals

- Aligned peak bins maintain constant resolution in ppm across the entire mass range (not a fixed mDa width)
- After downstream lock-mass recalibration, reference peaks are assigned within the 30 ppm tolerance and pass the 10 000 minimum-intensity threshold
- Low-abundance reference signals (<8000 a.u. on timsTOF) were excluded from recalibration to avoid incorrect signal assignment
- Identifications highlighted by LipidQMap reflect signal intensity and spatial completeness, indicating a correctly aligned peak grid
- A 5 GB imzML file with 2500 ion images imports and quantifies in roughly 20 seconds on an M2 MacBook, indicating efficient alignment

## Limitations

- Recalibration parameters may need to be adjusted for datasets acquired on different mass spectrometer platforms
- Low-abundance signals (e.g. <8000 a.u. on timsTOF instruments) must be avoided for recalibration as they may lead to incorrect signal assignment
- The 5 mDa default bin size for the average spectrum applies to TOF instruments and should be decreased for higher-resolution instruments

## Evidence

- [results] A dynamic binning algorithm aligns centroided spectra across all pixels, maintaining constant resolution (ppm) across the entire mass range: "A dynamic binning algorithm aligns centroided spectra across all pixels, maintaining constant resolution (ppm) across the entire mass range"
- [other] pyimzML as imzML parser: [evidence span withheld — anchor status: external_doc]
