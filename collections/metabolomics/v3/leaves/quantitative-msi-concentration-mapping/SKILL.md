---
name: quantitative-msi-concentration-mapping
description: Use when centroided .imzML lipid imaging data must be converted from
  raw ion intensities to absolute concentrations (pmol/mm²), and class-specific internal
  standards were co-sprayed onto the tissue for calibration.
when_to_use_negative:
- The analysis goal is relative spatial comparison or clustering of ion images without
  internal standards — one-point calibration requires user-defined class-specific
  standards to yield pmol/mm² values
- Data are profile (not centroided) spectra from raw instrument files; LipidQMap's
  workflow expects centroided .imzML import
- The task is general MSI visualization or peak picking (e.g. with SCiLS Lab, Cardinal,
  or DataCubeExplorer) rather than class-specific-standard-based quantitation
edam_operation: http://edamontology.org/operation_3648
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: LipidQMap
  role: 'Primary GUI tool: imports .imzML, performs dynamic binning, lock-mass recalibration
    (30 ppm tolerance, min intensity 10 000), lipid annotation, isobaric correction,
    and one-point calibration against class-specific internal standards to produce
    pmol/mm² heat maps'
  repo: https://github.com/swinnenteam/LipidQMap
- name: alexandrovteam/pyimzML
  role: Python parser for the imzML 1.1.0 format used to read the vendor-neutral imaging
    MS input data
  repo: https://github.com/alexandrovteam/pyimzML
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/2025.10.15.682422
    title: LipidQMap - An Open-Source Tool for Quantitative Mass Spectrometry Imaging
      of Lipids
  source_content_hash: sha256:3da7f3c9775a2959c53d40c4b4a2a4e651e3474cdc0014964ac019b4be6e3b40
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/swinnenteam/LipidQMap
  tools:
  - LipidQMap
  - alexandrovteam/pyimzML
  edam_operation: http://edamontology.org/operation_3648
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# quantitative-msi-concentration-mapping

## Summary

Transforms centroided MSI lipid data into absolute concentration heat maps (pmol/mm²) by normalizing annotated lipid signals against lipid-class-specific co-sprayed internal standards on a pixel-by-pixel basis, using LipidQMap's one-point calibration.

## When to use

Use when centroided .imzML lipid imaging data must be converted from raw ion intensities to absolute concentrations (pmol/mm²), and class-specific internal standards were co-sprayed onto the tissue for calibration.

## When NOT to use

- The analysis goal is relative spatial comparison or clustering of ion images without internal standards — one-point calibration requires user-defined class-specific standards to yield pmol/mm² values
- Data are profile (not centroided) spectra from raw instrument files; LipidQMap's workflow expects centroided .imzML import
- The task is general MSI visualization or peak picking (e.g. with SCiLS Lab, Cardinal, or DataCubeExplorer) rather than class-specific-standard-based quantitation

## Inputs

- Centroided .imzML files (vendor-neutral; multiple files, positive and negative modes can be loaded together)
- AnnData .h5ad or MuData .h5mu MSI files (with var["mz"], var["mz_mode"], obsm["spatial"])
- User-defined internal standard assignments (lipid-class-specific, co-sprayed standards)
- User-modifiable lipid database (.csv with nine required columns)

## Outputs

- Concentration heat maps per annotated lipid, expressed in pmol/mm²
- Toggleable views: raw, isotope-corrected, and quantitative (quantified) ion images
- Exported images as individual files or image collection panels

## How to apply

Import centroided .imzML files (LipidQMap also reads AnnData .h5ad and MuData .h5mu) with automatic ion-mode detection, then align centroided peaks across all pixels using the dynamic binning algorithm that maintains constant ppm resolution across the mass range; optionally apply lock-mass recalibration with a 30 ppm peak-assignment tolerance and a minimum intensity of 10 000 (default parameters). Annotate lipids against the user-modifiable .csv database (nine required columns) and apply one-point calibration, normalizing each annotated lipid signal against its lipid-class-specific internal standard on a pixel-by-pixel basis. Optionally run isobaric corrections first — Na/H adduct overlap correction (leveraging sodiated-to-protonated adduct ratios of standards) and Type II correction based on predicted isotope patterns — since quantitative data is also isotope-corrected when correction was chosen at import. Generate concentration heat maps in pmol/mm² and validate that the quantitative view reports pmol/mm² units and that each lipid was normalized against its matching class-specific standard.

## Related tools

- **LipidQMap** (Primary GUI tool: imports .imzML, performs dynamic binning, lock-mass recalibration (30 ppm tolerance, min intensity 10 000), lipid annotation, isobaric correction, and one-point calibration against class-specific internal standards to produce pmol/mm² heat maps) — https://github.com/swinnenteam/LipidQMap
- **alexandrovteam/pyimzML** (Python parser for the imzML 1.1.0 format used to read the vendor-neutral imaging MS input data) — https://github.com/alexandrovteam/pyimzML

## Examples

```
LipidQMap (GUI): Open Files → select centroided .imzML files → set max tolerated mass error (ppm) and bin size → choose isotope correction and online calibration (reference m/z, ppm tolerance, min intensity 10 000) → Import Data → toggle to Quantitative view (Q)
```

## Evaluation signals

- Quantitative maps are reported in pmol/mm² units, not raw intensities
- Each annotated lipid is normalized against its matching lipid-class-specific internal standard (not an arbitrary standard)
- Lock-mass recalibration used ≤30 ppm peak assignment tolerance and reference m/z signals above the minimum intensity threshold (10 000 default; avoid low-abundance signals <8000 a.u. on timsTOF)
- Dynamic binning maintained constant ppm resolution across the entire mass range
- Quantified view equals isotope-corrected data scaled by the internal-standard normalization (toggle R/I/Q views and confirm consistency)

## Limitations

- Low-abundance signals (e.g. <8000 a.u. on timsTOF) should be avoided during recalibration as they may lead to incorrect signal assignment
- Recalibration parameters may need to be adjusted for datasets acquired on different mass spectrometer platforms
- Na/H adduct overlap correction performs optimally with protonating matrices; further optimization is needed for qMSI with various MALDI matrices
- Separating Na/H adduct isobaric Δm/z of 0.0025 requires a minimum resolution of 600 000
- H/Na overlap correction skips unsupported lipid classes while continuing to process remaining data

## Evidence

- [readme] view toggle: [evidence span withheld — anchor status: external_doc]
