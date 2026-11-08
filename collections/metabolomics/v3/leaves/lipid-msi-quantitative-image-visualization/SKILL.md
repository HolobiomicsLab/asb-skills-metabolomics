---
name: lipid-msi-quantitative-image-visualization
description: Use when ion images from qMSI lipidomics exports (Cardinal HDF5, imzML
  via LipidQMap) must be reshaped to 2-D maps and compared pixel-wise, e.g. raw vs
  corrected isobar images on timsTOF fleX TIMS data.
when_to_use_negative:
- Input is raw centroided spectra or .imzML not yet exported to per-species ion images
  — first run LipidQMap import, isotope correction, and quantitation
- The question is instrument-level recalibration or lipid annotation rather than image-level
  visualization/agreement of already-exported species
- Data are LCM-derived bulk lipidomics ROI measurements with no spatial pixel grid
  to reshape
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: LipidQMap
  role: Generates raw, isotope-corrected and quantitative lipid images and exports
    them (individually or as panels) from imzML/AnnData inputs for downstream image
    comparison
  repo: github.com/swinnenteam/LipidQMap
- name: Cardinal
  role: Source of the HDF5 exports whose `image_type`, `spectraData/intensity`, `featureData`
    and `pixelData` structures are read and reshaped for visualization and agreement
    analysis
- name: h5_to_cardinal.R
  role: Minimal R reader converting LipidQMap HDF5 output into Cardinal-compatible
    structures
  repo: github.com/swinnenteam/LipidQMap
- name: pyimzML
  role: Python parser for the imzML format that can supply lipid MSI imaging data
    prior to LipidQMap processing
  repo: github.com/alexandrovteam/pyimzML
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2025.10.15.682422
    title: LipidQMap - An Open-Source Tool for Quantitative Mass Spectrometry Imaging
      of Lipids
  source_content_hash: sha256:245cb8d4fd49e4c78b1e76f1e5da18d4064f9402185276bb8c8804dba1dd0697
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Cardinal
  - LipidQMap
  - h5_to_cardinal.R
  - pyimzML
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# lipid-msi-quantitative-image-visualization

## Summary

Generates and compares quantitative lipid MSI ion images (raw, isotope-corrected, and concentration-normalized in pmol/mm²) from LipidQMap/Cardinal exports, including pixel-wise agreement analysis between ion-mobility-resolved and mathematically corrected isobar images.

## When to use

Use when ion images from qMSI lipidomics exports (Cardinal HDF5, imzML via LipidQMap) must be reshaped to 2-D maps and compared pixel-wise, e.g. raw vs corrected isobar images on timsTOF fleX TIMS data.

## When NOT to use

- Input is raw centroided spectra or .imzML not yet exported to per-species ion images — first run LipidQMap import, isotope correction, and quantitation
- The question is instrument-level recalibration or lipid annotation rather than image-level visualization/agreement of already-exported species
- Data are LCM-derived bulk lipidomics ROI measurements with no spatial pixel grid to reshape

## Inputs

- Cardinal HDF5 export of raw (ion-mobility-resolved) ion images with root `image_type` attribute
- Cardinal HDF5 export of isotope-corrected / Type II-corrected ion images
- LipidQMap-exported imzML-derived ion images (via scripts/h5_to_cardinal.R)

## Outputs

- isobar_agreement_metrics.csv with per-species Pearson/Spearman correlation, regression slope/intercept, and mean absolute relative deviation
- agreement_scatter.png with per-species scatter plots (identity line) and difference-overlay maps
- 2-D quantitative ion images (pmol/mm²) reshaped per species

## How to apply

Load both Cardinal HDF5 exports with h5py and read the root `image_type` attribute to distinguish the raw (ion-mobility-resolved) stack from the isotope-corrected stack. Read `spectraData/intensity` (2-D float32, shape (n_features, n_pixels)) from each file and align features via `featureData/feature_id`/`feature_index` and pixels via `pixelData/pixel_index` as the authoritative join keys. Reshape the flattened pixel vectors back to 2-D images using the `samples/<index>` `height_px` and `width_px` attributes, respecting 1-based coordinates and row-major flattening of dense images. Before visualization, data may pass through LipidQMap's import options: median imputation of empty pixels from a 3×3 neighborhood, winsorization tuned so 95–99% of pixels are preserved for timsTOF MALDI-imaging, and Type II isobaric correction based on the predicted isotope pattern. For each species present in both exports, compute pixel-wise agreement statistics (Pearson and Spearman correlation, linear-regression slope/intercept, mean absolute relative deviation), assemble them into a CSV, and render per-species scatter plots with an identity line plus difference-overlay maps.

## Related tools

- **LipidQMap** (Generates raw, isotope-corrected and quantitative lipid images and exports them (individually or as panels) from imzML/AnnData inputs for downstream image comparison) — github.com/swinnenteam/LipidQMap
- **Cardinal** (Source of the HDF5 exports whose `image_type`, `spectraData/intensity`, `featureData` and `pixelData` structures are read and reshaped for visualization and agreement analysis)
- **h5_to_cardinal.R** (Minimal R reader converting LipidQMap HDF5 output into Cardinal-compatible structures) — github.com/swinnenteam/LipidQMap
- **pyimzML** (Python parser for the imzML format that can supply lipid MSI imaging data prior to LipidQMap processing) — github.com/alexandrovteam/pyimzML

## Examples

```
import h5py; f = h5py.File('corrected.h5', 'r'); img = f['spectraData/intensity'][:].reshape(f['samples/0'].attrs['height_px'], f['samples/0'].attrs['width_px'])
```

## Evaluation signals

- Every exported species yields finite agreement statistics computed over the full pixel set, with n_pixels equal to height_px × width_px for dense exports
- All Pearson and Spearman correlation values fall within [-1, 1]
- Feature and pixel alignment via featureData/feature_id and pixelData/pixel_index resolves every species present in both exports with no unmatched keys
- Reshaped 2-D images preserve row-major flattening with 1-based pixel coordinates, verified against samples/<index> height_px/width_px
- For Type II-corrected isobars such as PC 34:1 [M+2+H]+ and PE 38:4 [M+2+H]+, corrected qMSI images should closely resemble their [M+H]+ counterparts

## Limitations

- Correction performs optimally when protonating matrices are used; further optimization is needed to leverage the correction across various MALDI matrices
- Low-abundance signals (<8000 a.u. on timsTOF instruments) should be avoided for recalibration as they may lead to incorrect signal assignment
- Recalibration parameters (tolerance, minimum intensity) may need adjustment for datasets acquired on different mass spectrometer platforms

## Evidence
