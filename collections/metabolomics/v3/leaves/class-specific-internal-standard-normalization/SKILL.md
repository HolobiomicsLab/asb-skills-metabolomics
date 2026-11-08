---
name: class-specific-internal-standard-normalization
description: Use when quantitative MSI lipid maps in absolute units (pmol/mm2) are
  needed and class-specific internal standards have been co-sprayed onto the tissue,
  e.g. centroided .imzML files from timsTOF or other MALDI/DESI platforms.
when_to_use_negative:
- No class-specific internal standards were co-sprayed on the tissue, so one-point
  calibration cannot be performed and only raw/isotope-corrected views are meaningful
- Input is already a quantitative feature table or concentration matrix rather than
  pixel-level imaging data (.imzML/.h5ad/.h5mu)
- The goal is qualitative spatial distribution of lipids without absolute quantification;
  raw or isotope-corrected ion images suffice
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: LipidQMap
  role: GUI application that performs pixel-by-pixel one-point calibration against
    class-specific internal standards and renders quantitative pmol/mm2 heat maps
  repo: https://github.com/swinnenteam/LipidQMap
- name: alexandrovteam/pyimzML
  role: Python parser for the imzML format used for programmatic access to the vendor-neutral
    input files
  repo: https://github.com/alexandrovteam/pyimzML
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/2025.10.15.682422
    title: LipidQMap - An Open-Source Tool for Quantitative Mass Spectrometry Imaging
      of Lipids
  source_content_hash: sha256:6e0cc3591b4cd45b18bba9dd933391af73b9194080ae3dd8d0defd1ee6a4cbcd
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/swinnenteam/LipidQMap
  tools:
  - LipidQMap
  - alexandrovteam/pyimzML
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# class-specific-internal-standard-normalization

## Summary

One-point calibration workflow from LipidQMap that normalizes each annotated lipid signal against its lipid-class-specific co-sprayed internal standard on a pixel-by-pixel basis, converting MSI ion intensities into quantitative concentration maps in pmol/mm2.

## When to use

Use when quantitative MSI lipid maps in absolute units (pmol/mm2) are needed and class-specific internal standards have been co-sprayed onto the tissue, e.g. centroided .imzML files from timsTOF or other MALDI/DESI platforms.

## When NOT to use

- No class-specific internal standards were co-sprayed on the tissue, so one-point calibration cannot be performed and only raw/isotope-corrected views are meaningful
- Input is already a quantitative feature table or concentration matrix rather than pixel-level imaging data (.imzML/.h5ad/.h5mu)
- The goal is qualitative spatial distribution of lipids without absolute quantification; raw or isotope-corrected ion images suffice

## Inputs

- centroided .imzML imaging mass spectrometry files (multiple can be loaded simultaneously; positive and negative ion modes together)
- AnnData .h5ad or MuData .h5mu MSI files with var['mz'], var['mz_mode'], and obsm['spatial']
- user-modifiable lipid annotation database (.csv with nine required columns) including lipid-class-specific internal standards

## Outputs

- quantitative lipid concentration heat maps in pmol/mm2 per annotated lipid species
- toggleable views of raw, isotope-corrected, and quantified ion images
- exportable individual images or image collection panels

## How to apply

Import centroided .imzML data into LipidQMap, apply dynamic binning (constant ppm across the mass range) and optional lock-mass recalibration (30 ppm tolerance, minimum intensity 10 000 by default; avoid low-abundance signals such as <8000 a.u. on timsTOF). Annotate lipids against the user-modifiable .csv database with nine required columns, then apply one-point calibration: for each lipid, divide its per-pixel signal by the signal of the matching class-specific internal standard on the same pixel, correcting matrix effects and ion suppression because co-sprayed standards undergo similar ionization yet are distinguishable by heavy isotopes. The quantified view is isotope-corrected when isotope correction (Type II and/or H/Na adduct) was selected at import; toggle to the Quantitative view (Q key) to inspect the pmol/mm2 maps. Validation: confirm maps are reported in pmol/mm2 and that each lipid was normalized with its matching class-specific standard.

## Related tools

- **LipidQMap** (GUI application that performs pixel-by-pixel one-point calibration against class-specific internal standards and renders quantitative pmol/mm2 heat maps) — https://github.com/swinnenteam/LipidQMap
- **alexandrovteam/pyimzML** (Python parser for the imzML format used for programmatic access to the vendor-neutral input files) — https://github.com/alexandrovteam/pyimzML

## Evaluation signals

- Quantitative maps are expressed in pmol/mm2, not arbitrary intensity units
- Each lipid was normalized against the standard of its own lipid class, not a global or mismatched standard
- Normalization is computed on a pixel-by-pixel basis (per-pixel ratios), not on sample averages
- Quantified view reflects isotope correction when Type II or Na/H adduct correction was selected during import
- Upscaling/panning across raw, isotope-corrected, and quantitative views (R/I/Q toggles) shows consistent spatial structures after quantitation

## Limitations

- Low-abundance recalibration signals (e.g. intensities <8000 a.u. on timsTOF) can lead to incorrect signal assignment and compromise the downstream quantitation
- Recalibration parameters (tolerance, minimum intensity) may need adjustment for datasets from different mass spectrometer platforms
- Quantitation depends on user-defined internal standards; quality relies on correct database annotation and standard assignment per lipid class

## Evidence

- [abstract] one-point calibration normalizing annotated lipid signals against class-specific standards on a pixel-by-pixel basis: "LipidQMap applies one-point calibration, normalizing annotated lipid signals against class-specific standards on a pixel-by-pixel basis, and generates concentration heat maps in pmol/mm2."
- [intro] standards correct matrix effects and ion suppression: "co-sprayed lipid standards undergo ionization processes similar to endogenous lipids and yet can be distinguished from endogenous analytes due to the presence of heavy isotopes"
- [results] workflow step producing quantitative lipid maps in pmol/mm2: "(iv) normalization against internal standards (IS) to produce quantitative lipid maps (in pmol/mm²)"
- [readme] quantitation based on user defined internal standards: "Performs quantitation based on user defined internal standards."
