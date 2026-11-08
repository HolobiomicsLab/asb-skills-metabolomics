---
name: sims-spectral-total-ion-normalization
description: Use when raw TOF-SIMS (3D-SMF) pixel-by-peak intensity matrices must
  be normalized per pixel prior to pixel clustering or other downstream pixel-level
  metabolomic analysis.
when_to_use_negative:
- Input is already a normalized feature table or single-cell-level metabolite extraction
  output rather than raw pixel-level TOF-SIMS intensities
- Data are IMC protein-channel intensities, which follow the protein-modality preprocessing
  rather than the SIMS total-ion normalization step
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: scSpaMet
  role: Pipeline providing the SIMS pixel-intensity export (notebook 01) that feeds
    the normalization step before pixel clustering
  repo: github.com/coskunlab/ScSpaMet
- name: pandas
  role: Used to load the SIMS peak-intensity files, assemble the pixel-by-peak matrix,
    compute per-pixel total-ion normalization, and write normalized_pixel_intensities.csv
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-43917-5
    title: Single-cell spatial metabolomics with cell-type specific protein profiling
      for tissue systems biology
  source_content_hash: sha256:73a5f3ee1eadb744a153276137bc81176bb0bd622fbff9e6dc77d4343c6a9a68
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - pandas
  - scSpaMet
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# sims-spectral-total-ion-normalization

## Summary

Per-pixel total-ion-count normalization of raw, untargeted TOF-SIMS pixel-level metabolomic intensities in the scSpaMet pipeline, scaling each pixel's spectrum by its summed intensity so pixels are comparable before pixel clustering.

## When to use

Use when raw TOF-SIMS (3D-SMF) pixel-by-peak intensity matrices must be normalized per pixel prior to pixel clustering or other downstream pixel-level metabolomic analysis.

## When NOT to use

- Input is already a normalized feature table or single-cell-level metabolite extraction output rather than raw pixel-level TOF-SIMS intensities
- Data are IMC protein-channel intensities, which follow the protein-modality preprocessing rather than the SIMS total-ion normalization step

## Inputs

- Exported SIMS peak-coordinate/intensity files from scSpaMet notebook 01 (pandas-readable CSV)
- Pixel-by-peak raw TOF-SIMS intensity matrix (pixel identifiers as rows, m/z peak coordinates as columns)

## Outputs

- normalized_pixel_intensities.csv with pixels as rows and m/z peaks as columns
- Normalized pixel-intensity matrix ready for pixel clustering of SIMS data

## How to apply

Load the exported SIMS peak-coordinate/intensity files produced by scSpaMet notebook 01 (Processing of IMC (protein) and SIMS (metabolite) images) with pandas. Assemble a pixel-by-peak intensity matrix keeping pixel identifiers as rows and m/z peak coordinates as column keys. Apply the per-pixel total-ion normalization Yi = Xi / (sum_j xij) * P, dividing each pixel's intensity vector by its total count and scaling by target total P; leave zero-total pixels at zero to avoid division-by-zero artifacts. Write the result to normalized_pixel_intensities.csv (pixels as rows, m/z peaks as columns) and feed it to SIMS pixel clustering, which reveals unique metabolic variation in the spatial context. Rationale: untargeted MSI intensities vary in total signal per pixel, so total-count scaling is the single SIMS-modality preprocessing step needed before pixel-level downstream analysis.

## Related tools

- **scSpaMet** (Pipeline providing the SIMS pixel-intensity export (notebook 01) that feeds the normalization step before pixel clustering) — github.com/coskunlab/ScSpaMet
- **pandas** (Used to load the SIMS peak-intensity files, assemble the pixel-by-peak matrix, compute per-pixel total-ion normalization, and write normalized_pixel_intensities.csv)

## Examples

```
import pandas as pd; df = pd.read_csv('sims_pixel_intensities.csv'); P = 10000; norm = df.div(df.sum(axis=1), axis=0).fillna(0).mul(P); norm.to_csv('normalized_pixel_intensities.csv')
```

## Evaluation signals

- Every non-zero row of normalized_pixel_intensities.csv sums to the constant target total P within floating-point tolerance
- Output CSV exists with the expected pixel-by-peak shape (same pixels and m/z peak columns as the raw matrix)
- Zero-total pixels remain all-zero rows with no NaN or division-by-zero artifacts
- Downstream pixel clustering on the normalized matrix reveals spatially coherent metabolic variation as reported in the scSpaMet results

## Limitations

- Zero-total pixels cannot be scaled and are left at zero, so they carry no spectral information
- Untargeted spatial metabolomic data lack shared markers and identified cell types, making the normalized output unsuitable for direct multi-omics integration without further steps
- TOF-SIMS metabolite annotation and coverage (lipids, amino acids, metabolic pathways) remains challenging, so normalized peaks are not necessarily identified metabolites

## Evidence

- [readme] 01 Processing of IMC (protein) and SIMS (metabolite) images: "- 01 Processing of IMC (protein) and SIMS (metabolite) images"
- [results] Pixel clustering of SIMS data reveals unique metabolic variation in the spatial context: "Pixel clustering of SIMS data reveals unique metabolic variation in the spatial context"
- [abstract] The scSpaMet analysis revealed cell type-dependent metabolite profiles and local metabolite competition of neighboring single cells in human tissues: "The scSpaMet analysis revealed cell type-dependent metabolite profiles and local metabolite competition of neighboring single cells in human tissues"
