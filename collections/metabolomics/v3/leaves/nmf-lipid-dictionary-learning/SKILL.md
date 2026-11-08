---
name: nmf-lipid-dictionary-learning
description: Use when single-cell MS spectra from isolated cells must be mapped onto
  tissue MALDI MSI pixel data to infer where cell subpopulations contribute chemically
  across a tissue section or 3D reconstruction.
when_to_use_negative:
- Input MSI and SCMS data come from different tissues or non-matching anatomical regions
  where shared lipid features do not correspond to the same cell-type chemistry
- Data are not mass-spectrometry-based (e.g., imaging transcriptomics) since the method
  depends on m/z matching within a ppm tolerance and lipid-specific NMF dictionaries
- Only a handful of cells (<~hundreds) are available, making per-cluster NMF dictionaries
  unstable and cross-annotation uninformative
edam_operation: http://edamontology.org/operation_3939
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MEISTER
  role: Framework providing the reconstruction, cross-annotation, and NMF dictionary-learning
    pipeline for mapping single-cell lipid signatures onto tissue MSI data
  repo: github.com/richardxie1119/MEISTER
- name: multiscale_analysis notebooks
  role: Jupyter notebooks (tissue_sc_mapping_deepmsi_hip.ipynb) implementing cross-annotation,
    Leiden clustering, NMF dictionary learning, and UoSS fitting steps
  repo: github.com/richardxie1119/multiscale_analysis
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41592-024-02171-3
    title: Multiscale biochemical mapping of the brain through deep-learning-enhanced
      high-throughput mass spectrometry
  source_content_hash: sha256:80c6aab308b1327253aad0313f9a3587b525ea659e2c12245d08cbf367c2e070
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MEISTER
  - multiscale_analysis notebooks
  edam_operation: http://edamontology.org/operation_3939
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# nmf-lipid-dictionary-learning

## Summary

Dictionary-learning approach that fits non-negative linear combinations of single-cell-derived lipid dictionaries (from SCMS NMF decomposition) onto tissue MSI pixel spectra to localize cell-type chemical signatures spatially. Bridges single-cell and tissue mass spectrometry data through shared cross-annotated lipid features.

## When to use

Use when single-cell MS spectra from isolated cells must be mapped onto tissue MALDI MSI pixel data to infer where cell subpopulations contribute chemically across a tissue section or 3D reconstruction.

## When NOT to use

- Input MSI and SCMS data come from different tissues or non-matching anatomical regions where shared lipid features do not correspond to the same cell-type chemistry
- Data are not mass-spectrometry-based (e.g., imaging transcriptomics) since the method depends on m/z matching within a ppm tolerance and lipid-specific NMF dictionaries
- Only a handful of cells (<~hundreds) are available, making per-cluster NMF dictionaries unstable and cross-annotation uninformative

## Inputs

- Single-cell MS peak data (e.g., peak_data_sc_regional_decode.imzML from 13,566 isolated cells)
- Reconstructed, processed tissue MSI data (e.g., coronal3D_propagated_decoded.h5)
- Shared lipid feature list from cross-annotation within 3-ppm m/z tolerance
- Pre-registered tissue pixel coordinates for mapping fitted weights back to spatial positions

## Outputs

- Cluster-specific chemical dictionaries D(l) per Leiden cluster (k=20 items each)
- Union dictionary U across all cell clusters
- Per-pixel non-negative weight matrix C mapping cluster contributions
- Spatial cluster-contribution maps rendered at original pixel locations
- Pearson correlation statistics between fitted and observed lipid weights (e.g., r=0.46, n=103)

## How to apply

Cross-annotate SCMS peak lists (e.g., from peak_data_sc_regional_decode.imzML) against the tissue MSI feature database (e.g., coronal3D_propagated_decoded.h5) within a 3-ppm m/z window, discarding features present in <5% of cells and cells with <5% of features. TIC-normalize the shared lipid feature matrix, Leiden-cluster cells (first 40 PCs, n_neighbors=30, min_dist=0.5, resolution=0.25, cosine distance), then run NMF with k=20 dictionary items per cluster to build cluster dictionaries D(l). Concatenate dictionaries into a union dictionary U, and fit U to the tissue pixel matrix Y via constrained least-squares (Y=UC, C≥0). Map the vector norms of each pixel's per-cluster weight sub-vectors back to original pixel locations to render cluster-contribution maps, and report the Pearson correlation between fitted and observed lipid weights across shared features (target r=0.46, n=103).

## Related tools

- **MEISTER** (Framework providing the reconstruction, cross-annotation, and NMF dictionary-learning pipeline for mapping single-cell lipid signatures onto tissue MSI data) — github.com/richardxie1119/MEISTER
- **multiscale_analysis notebooks** (Jupyter notebooks (tissue_sc_mapping_deepmsi_hip.ipynb) implementing cross-annotation, Leiden clustering, NMF dictionary learning, and UoSS fitting steps) — github.com/richardxie1119/multiscale_analysis

## Examples

```
jupyter notebook tissue_sc_mapping_deepmsi_hip.ipynb
```

## Evaluation signals

- Pearson correlation between fitted and observed lipid weights should approach ~0.46 across n≈103 shared lipid features in the hippocampus example
- Per-cluster NMF reconstruction error should be low enough that each cluster dictionary captures cluster-specific chemistry rather than noise
- Fitted cluster-contribution maps should show plausible spatial organization (e.g., large weights in the dentate gyrus and CA3 for cluster 0)
- All fitted weights in C must be non-negative; violating this indicates the constrained least-squares setup was misapplied
- Cross-annotated feature count should be sufficient (≈103 shared features in the reported case) for stable NMF and fitting

## Limitations

- Modeling errors arise from falsely cross-annotated features caused by mass shifts or an inappropriate ppm window, or from nonlinearities between MSI and SCMS data, producing poorly fitted lipid components with negative correlation (Fig. 4e)
- The non-negative model cannot capture negative-mode ion suppression or lipid species absent from the single-cell data
- Requires sufficient shared lipid features; if cross-annotation yields very few matches, dictionaries become unreliable
- Spatial resolution of fitted contribution maps is limited by tissue MSI pixel size rather than single-cell resolution

## Evidence

- [methods] Cross-annotation step uses 3-ppm window to link SCMS and tissue MSI features: [evidence span withheld — anchor status: not_found]
- [results] Non-negative least-squares fitting of dictionaries onto tissue pixels: [evidence span withheld — anchor status: not_found]
- [methods] Leiden clustering parameters on PCs: [evidence span withheld — anchor status: not_found]
- [methods] Feature and cell prevalence filters: "Features present in less than 5% of cells were discarded, and cells with less than 5% total number of features were filtered out"
- [discussion] Limitation on mass shifts and nonlinearities: [evidence span withheld — anchor status: not_found]
