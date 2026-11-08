---
name: spatial-contribution-map-rendering
description: Use when you have cluster-level single-cell lipid dictionaries from SCMS
  data (e.g. Leiden clusters from hippocampal cells) and processed tissue MSI pixels
  (e.g. coronal3D_propagated_decoded.
when_to_use_negative:
- When the single-cell and tissue data share no m/z features (cross-annotation in
  a 3-ppm window yields no matches) — the dictionary U cannot be fitted meaningfully
  to tissue pixels
- When you only need unsupervised pixel clustering or spatial segmentation of the
  MSI data itself (use UMAP or gradient-boosted-tree classification instead); this
  skill is specifically for cell-cluster-to-tissue contribution mapping
- When the tissue MSI data has not been TIC-normalized and cross-annotated against
  the single-cell feature list — raw unnormalized pixels produce invalid non-negative
  fits
edam_operation: http://edamontology.org/operation_3649
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MEISTER
  role: Provides the single-cell and tissue MSI signal reconstruction and processing
    pipeline whose outputs (peak_data_sc_regional_decode.imzML and coronal3D_propagated_decoded.h5)
    are the inputs to the dictionary fitting and contribution map rendering
  repo: https://github.com/richardxie1119/MEISTER
- name: tissue_sc_mapping_deepmsi_hip.ipynb
  role: 'Jupyter notebook in multiscale_analysis that implements the hippocampus-specific
    integration: cross-annotation, Leiden clustering, per-cluster NMF dictionaries,
    constrained UoSS fitting, and contribution map rendering'
  repo: https://github.com/richardxie1119/multiscale_analysis
- name: multiscale_analysis
  role: Repository hosting the notebooks (e.g. tissue_sc_mapping_deepmsi_hip.ipynb)
    needed to reproduce the contribution map workflow on the Zenodo-deposited data
  repo: https://github.com/richardxie1119/multiscale_analysis
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41592-024-02171-3
    title: Multiscale biochemical mapping of the brain through deep-learning-enhanced
      high-throughput mass spectrometry
  source_content_hash: sha256:d1d3fcb60068aa15f2b3e52563481fde6d3e5e790e305c3630b68156d27053f6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/richardxie1119/MEISTER
  tools:
  - MEISTER
  - multiscale_analysis
  - tissue_sc_mapping_deepmsi_hip.ipynb
  edam_operation: http://edamontology.org/operation_3649
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# spatial-contribution-map-rendering

## Summary

Renders per-cell-cluster spatial contribution maps by fitting a concatenated single-cell NMF dictionary U to tissue MSI pixels with non-negative weights, then mapping per-cluster weight vector norms back to original pixel coordinates, enabling multiscale integration of single-cell lipid profiles with tissue imaging.

## When to use

Use when you have cluster-level single-cell lipid dictionaries from SCMS data (e.g. Leiden clusters from hippocampal cells) and processed tissue MSI pixels (e.g. coronal3D_propagated_decoded.h5), and need per-pixel maps of how much each cell subpopulation contributes chemically at each tissue location.

## When NOT to use

- When the single-cell and tissue data share no m/z features (cross-annotation in a 3-ppm window yields no matches) — the dictionary U cannot be fitted meaningfully to tissue pixels
- When you only need unsupervised pixel clustering or spatial segmentation of the MSI data itself (use UMAP or gradient-boosted-tree classification instead); this skill is specifically for cell-cluster-to-tissue contribution mapping
- When the tissue MSI data has not been TIC-normalized and cross-annotated against the single-cell feature list — raw unnormalized pixels produce invalid non-negative fits

## Inputs

- TIC-normalized tissue MSI pixel matrix Y over cross-annotated lipid features (from coronal3D_propagated_decoded.h5 or equivalent processed MSI data)
- Union single-cell dictionary U built from per-cluster NMF chemical dictionaries D(l) (k=20 items per cluster) from SCMS peak data (e.g. peak_data_sc_regional_decode.imzML, hippocampal cells)
- Pixel coordinate index mapping fitted rows back to original spatial locations

## Outputs

- Per-cluster spatial contribution maps: per-pixel vector norms of each cluster's fitted weight sub-vector, rendered at original tissue pixel positions
- Per-pixel fitted lipid weight matrix (Y_hat = UC) used for correlation diagnostics
- Pearson correlation statistic between fitted and observed lipid weights across shared features (expected r ≈ 0.46, n = 103)

## How to apply

After Leiden-clustering single-cell features (e.g. 8 hippocampal clusters from 40 PCs, cosine distance, n_neighbors=30, resolution=0.25), run NMF with k=20 dictionary items per cluster to get D(l), and concatenate them across clusters into union dictionary U. Load the TIC-normalized tissue pixel matrix Y from the processed MSI data (e.g. coronal3D_propagated_decoded.h5) restricted to the shared 3-ppm-cross-annotated features. Solve Y = UC with C >= 0 via constrained least-squares (UoSS fitting). For each pixel, compute the vector norms of its per-cluster weight sub-vectors, and map these norms back to the original pixel coordinates to render spatial contribution maps (e.g. Fig. 5b,c), highlighting subpopulation enrichment such as cluster 0 in dentate gyrus and CA3. Validate by computing Pearson correlation between fitted and observed lipid weights across shared features (target r ≈ 0.46, n = 103).

## Related tools

- **MEISTER** (Provides the single-cell and tissue MSI signal reconstruction and processing pipeline whose outputs (peak_data_sc_regional_decode.imzML and coronal3D_propagated_decoded.h5) are the inputs to the dictionary fitting and contribution map rendering) — https://github.com/richardxie1119/MEISTER
- **tissue_sc_mapping_deepmsi_hip.ipynb** (Jupyter notebook in multiscale_analysis that implements the hippocampus-specific integration: cross-annotation, Leiden clustering, per-cluster NMF dictionaries, constrained UoSS fitting, and contribution map rendering) — https://github.com/richardxie1119/multiscale_analysis
- **multiscale_analysis** (Repository hosting the notebooks (e.g. tissue_sc_mapping_deepmsi_hip.ipynb) needed to reproduce the contribution map workflow on the Zenodo-deposited data) — https://github.com/richardxie1119/multiscale_analysis

## Examples

```
run tissue_sc_mapping_deepmsi_hip.ipynb (Jupyter) with inputs peak_data_sc_regional_decode.imzML and coronal3D_propagated_decoded.h5 from Zenodo 10.13012/B2IDB-9740536_V1
```

## Evaluation signals

- Pearson correlation between fitted and observed lipid weights across shared features should reach r ≈ 0.46 with n = 103 shared lipid features (hippocampal case)
- All fitted weights C must be non-negative; any negative values indicate the constrained least-squares solve was not applied correctly
- Per-cluster contribution norms should be spatially coherent — e.g. cluster 0 norms should be large in dentate gyrus and CA3 as shown in Fig. 5b,c
- The number of rows in the fitted coefficient matrix C must equal the number of tissue pixels retained after the <5% cell/feature prevalence filters
- The union dictionary U should have exactly 8 × 20 = 160 columns for 8 hippocampal clusters with k=20 NMF items each

## Limitations

- UoSS fitting modeling errors can arise from falsely cross-annotated features caused by mass shifts, an inappropriate ppm window, or nonlinearities between MSI and SCMS data
- Poorly fitted lipid components with negative correlation (Fig. 4e) may remain even after careful fitting; root causes are not fully resolved
- Only molecules present in intracellular spaces should be used for cross-annotation; retaining extracellular-only features would bias the contribution maps
- The rendering maps vector norms per cluster rather than per-dictionary-item, so within-cluster heterogeneity in chemical profiles is not resolved spatially

## Evidence

- [results] Cross-annotation of SCMS data using lipid features from tissue MSI data: [evidence span withheld — anchor status: not_found]
- [discussion] Cross-annotation considers only molecules present in intracellular spaces for single-cell-to-tissue mapping: "Another advantage for cross-annotation is to consider only molecules that are present in the intracellular spaces for more accurate single-cell-to-tissue mapping"
- [methods] TIC normalization of each pixel before fitting: "Each pixel was normalized by total ion count (TIC)."
- [methods] Feature prevalence filter thresholds for SCMS cross-annotation: "Features present in less than 5% of cells were discarded, and cells with less than 5% total number of features were filtered out"
- [discussion] UoSS fitting limitations from mass shifts and ppm window choices: [evidence span withheld — anchor status: not_found]
- [readme] Notebook reference for reproducing the hippocampus-specific contribution mapping: "tissue_sc_mapping_deepmsi_hip.ipynb: Integrative analysis of hippocampus MSI data with single-cell MS on hippocampal cells."
- [readme] Data availability for input matrices: "coronal3D_propagated_decoded.h5: reconstructed and processed MSI data of serial sections of rat coronal sections."
