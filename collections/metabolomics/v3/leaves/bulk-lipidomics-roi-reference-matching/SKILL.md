---
name: bulk-lipidomics-roi-reference-matching
description: Use when comparing quantitative MSI concentration maps (pmol/mm²) against
  LCM bulk lipidomics reference values from cortex, hippocampus, or cerebellar ROIs,
  e.g. for PC 36:1, PC 38:4, PE O-36:4, or PE P-38:6.
when_to_use_negative:
- Input is raw centroided imzML spectra rather than a quantitative LipidQMap export
  (image_type != 'quant'), since concentrations in pmol/mm² cannot be derived without
  one-point calibration against class-specific standards
- No LCM bulk lipidomics reference values exist for the ROIs of interest, so there
  is no ground truth to match against
- The lipid species lack a co-sprayed class-specific internal standard, so pixel-by-pixel
  concentration normalization is not supported
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: LipidQMap
  role: Generates the quantitative concentration heat maps in pmol/mm² and the Cardinal
    HDF5 quantitative exports (image_type = 'quant') that serve as the qMSI side of
    the comparison
  repo: https://github.com/swinnenteam/LipidQMap
- name: Cardinal
  role: R imaging mass spectrometry framework into which the LipidQMap HDF5 export
    is read for per-feature and per-pixel access
- name: rhdf5
  role: R package used by scripts/h5_to_cardinal.R to read spectraData/intensity,
    featureData/mz, and pixelData x/y from the HDF5 file
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2025.10.15.682422
    title: LipidQMap - An Open-Source Tool for Quantitative Mass Spectrometry Imaging
      of Lipids
  source_content_hash: sha256:c145bcbc7eab6e12219ab5a8a756d26991c575af4af740b9cf425ade121f7370
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/swinnenteam/LipidQMap
  tools:
  - Cardinal
  - LipidQMap
  - rhdf5
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# bulk-lipidomics-roi-reference-matching

## Summary

Matches quantitative MSI (qMSI) concentration maps in pmol/mm², produced by LipidQMap's one-point calibration against class-specific internal standards, to LCM-derived bulk lipidomics reference values for cortex, hippocampus, and cerebellar ROIs. This cross-modal agreement check validates that LipidQMap harmonizes qMSI concentrations across platforms such as timsTOF fleX MALDI-2, AP-SMALDI Orbitrap, and DESI-MRT.

## When to use

Use when comparing quantitative MSI concentration maps (pmol/mm²) against LCM bulk lipidomics reference values from cortex, hippocampus, or cerebellar ROIs, e.g. for PC 36:1, PC 38:4, PE O-36:4, or PE P-38:6.

## When NOT to use

- Input is raw centroided imzML spectra rather than a quantitative LipidQMap export (image_type != 'quant'), since concentrations in pmol/mm² cannot be derived without one-point calibration against class-specific standards
- No LCM bulk lipidomics reference values exist for the ROIs of interest, so there is no ground truth to match against
- The lipid species lack a co-sprayed class-specific internal standard, so pixel-by-pixel concentration normalization is not supported

## Inputs

- LipidQMap Cardinal HDF5 quantitative exports (root attribute image_type = 'quant')
- Lipid annotation display IDs from featureData/feature_id and m/z values from featureData/mz
- Pixel x/y coordinates from pixelData
- Reported concentration values (pmol/mm²) from paper tables/figures per platform
- LCM-derived bulk lipidomics reference values for cortex, hippocampus, and two cerebellar ROIs

## Outputs

- cross_platform_concentrations.csv (per-lipid, per-platform concentration table)
- Relative differences between each qMSI platform and the LCM bulk lipidomics reference per ROI
- Flags for lipid species whose cross-platform spread is inconsistent with reported agreement

## How to apply

Collect reported concentration values (pmol/mm²) for the target lipids from the paper's tables/figures under each platform condition (timsTOF fleX MALDI-2, AP-SMALDI Orbitrap, DESI-MRT, and LCM bulk lipidomics). Where machine-readable quantitative exports exist, read the LipidQMap Cardinal HDF5 files whose root attribute image_type = 'quant' using the R reader scripts/h5_to_cardinal.R with rhdf5, combining spectraData/intensity with featureData/mz and pixelData x/y metadata, and align each value to its lipid display ID from featureData/feature_id and its platform condition. Tabulate per-lipid, per-platform concentrations in a CSV and compute relative differences between each qMSI platform and the LCM bulk lipidomics reference for cortex, hippocampus, and cerebellar ROIs. Flag any lipid whose cross-platform spread is inconsistent with the reported agreement (e.g. qMSI 0–40 vs LCM 0–80 pmol/mm² for PC 36:1; 0–5 vs 0–1 for PE O-36:4) and save the compiled table as cross_platform_concentrations.csv. The rationale is that LipidQMap's pixel-by-pixel normalization against class-specific standards yields concentration heat maps that should be directly comparable to dissection-based bulk quantification.

## Related tools

- **LipidQMap** (Generates the quantitative concentration heat maps in pmol/mm² and the Cardinal HDF5 quantitative exports (image_type = 'quant') that serve as the qMSI side of the comparison) — https://github.com/swinnenteam/LipidQMap
- **Cardinal** (R imaging mass spectrometry framework into which the LipidQMap HDF5 export is read for per-feature and per-pixel access)
- **rhdf5** (R package used by scripts/h5_to_cardinal.R to read spectraData/intensity, featureData/mz, and pixelData x/y from the HDF5 file)

## Examples

```
Rscript scripts/h5_to_cardinal.R quant_export.h5
```

## Evaluation signals

- Every extracted value falls within the concentration range shown in the corresponding paper table/figure (e.g. PC 36:1 qMSI 0–40 vs LCM 0–80 pmol/mm²; PC 38:4 0–20 vs 0–14; PE O-36:4 0–5 vs 0–1; PE P-38:6 0–20 vs 0–2.5)
- The compiled cross_platform_concentrations.csv contains entries for all listed lipids (PC 36:1, PC 38:4, PE O-36:4, PE P-38:6) under all four platform conditions
- HDF5 files verified to have root attribute image_type = 'quant' before reading intensities
- Each value is aligned to a lipid display ID from featureData/feature_id and to a specific platform condition and ROI (cortex, hippocampus, cerebellum)
- Relative differences per ROI are computed against the LCM bulk reference, with outliers flagged when cross-platform spread contradicts the reported agreement

## Limitations

- Separating Na/H adduct isobaric overlaps (Δm/z 0.0025) requires a minimum resolution of 600,000, so low-resolution platforms cannot rely on this correction before concentration comparison
- Low-abundance signals, such as intensities <8000 a.u. on timsTOF instruments, should be avoided for recalibration as they may lead to incorrect signal assignment
- Recalibration parameters may need to be adjusted per mass spectrometer platform; Na/H overlap correction skips unsupported lipid classes, and correction performs optimally with protonating matrices

## Evidence

- [abstract] One-point calibration against class-specific internal standards yields comparable concentration ranges: "LipidQMap applies one-point calibration, normalizing annotated lipid signals against class-specific standards on a pixel-by-pixel basis, and generates concentration heat maps in pmol/mm2."
- [results] LCM bulk lipidomics defines the reference ROIs: "laser capture microdissection (LCM) and bulk lipidomics were performed on four ROIs: cortex (red), hippocampus (yellow), and two cerebellar areas (green and blue)"
- [methods] R reader provides programmatic access to quantitative exports: "LipidQMap includes a minimal R reader at `scripts/h5_to_cardinal.R`."
- [abstract] Cross-platform harmonization is the validation goal: "Extensive validation demonstrates that LipidQMap is robust across MSI platforms and harmonizes qMSI data, enabling more accurate and reproducible spatial lipid quantification."
- [readme] Quantitation depends on user-defined internal standards: "Performs quantitation based on user defined internal standards."
