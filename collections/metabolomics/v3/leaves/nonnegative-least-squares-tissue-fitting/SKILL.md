---
name: nonnegative-least-squares-tissue-fitting
description: Decision support for mapping single-cell-derived chemical dictionaries
  onto tissue MSI pixels — use when you have cross-annotated SCMS clusters and a shared
  lipid feature matrix with a tissue pixel matrix and want per-cluster spatial contribution
  maps.
when_to_use_negative:
- Do not apply when SCMS and tissue MSI feature lists have not been cross-annotated
  within a shared m/z window — modeling errors arise from falsely cross-annotated
  features caused by mass shifts or an inappropriate ppm window.
- Do not use for unclustered single-cell data; the union dictionary U requires per-cluster
  NMF dictionaries built from labeled cell clusters.
- Do not use on raw transient data — inputs must be the reconstructed and processed
  tissue MSI matrix (coronal3D_propagated_decoded.h5).
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MEISTER
  role: Framework hosting the reconstruction, cross-annotation, NMF dictionary learning,
    and non-negative least-squares tissue-fitting pipeline executed via tissue_sc_mapping_deepmsi_hip.ipynb
  repo: https://github.com/richardxie1119/MEISTER
- name: multiscale_analysis notebooks
  role: Jupyter notebook tissue_sc_mapping_deepmsi_hip.ipynb implementing cross-annotation,
    Leiden clustering, NMF dictionary construction, and non-negative least-squares
    fitting onto the tissue pixel matrix
  repo: https://github.com/richardxie1119/multiscale_analysis
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41592-024-02171-3
    title: Multiscale biochemical mapping of the brain through deep-learning-enhanced
      high-throughput mass spectrometry
  source_content_hash: sha256:55ad30ed50c24518779b984c4e947d8555905d96912c445385e26e2aae39c68c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/richardxie1119/MEISTER
  tools:
  - MEISTER
  - multiscale_analysis notebooks
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# nonnegative-least-squares-tissue-fitting

## Summary

Constrained linear least-squares fitting of a union of NMF cell-cluster chemical dictionaries (U) onto the tissue MSI pixel matrix (Y) under non-negative model weights (Y = UC, C ≥ 0), mapping per-cluster weight sub-vector norms back to pixel coordinates to resolve the spatial organization of hippocampal cell subpopulations.

## When to use

Decision support for mapping single-cell-derived chemical dictionaries onto tissue MSI pixels — use when you have cross-annotated SCMS clusters and a shared lipid feature matrix with a tissue pixel matrix and want per-cluster spatial contribution maps.

## When NOT to use

- Do not apply when SCMS and tissue MSI feature lists have not been cross-annotated within a shared m/z window — modeling errors arise from falsely cross-annotated features caused by mass shifts or an inappropriate ppm window.
- Do not use for unclustered single-cell data; the union dictionary U requires per-cluster NMF dictionaries built from labeled cell clusters.
- Do not use on raw transient data — inputs must be the reconstructed and processed tissue MSI matrix (coronal3D_propagated_decoded.h5).

## Inputs

- coronal3D_propagated_decoded.h5 — reconstructed, processed 3D coronal tissue MSI data (hippocampus region extracted)
- peak_data_sc_regional_decode.imzML — peak data of single rat brain cells (hippocampal subset)
- Shared lipid feature matrix built by 3-ppm cross-annotation between SCMS and tissue MSI features
- Leiden-clustered single-cell labels (8 hippocampal cell clusters)

## Outputs

- Per-cluster weight sub-vector norms mapped to original tissue pixel locations (cluster-contribution maps)
- Constrained least-squares model weight matrix C (pixels × dictionary items, C ≥ 0)
- Pearson correlation between fitted and observed lipid weights across shared features (target r = 0.46, n = 103)
- Statistics table of cluster contributions (e.g. large weights in dentate gyrus and CA3 for cluster 0)

## How to apply

From hippocampus-only SCMS data, cross-annotate peak lists against tissue MSI features within a 3-ppm m/z window, discarding features present in <5% of cells and cells with <5% of features to retain the shared lipid matrix (n = 103 shared features). TIC-normalize the single-cell matrix and Leiden-cluster on 40 PCs (n_neighbors = 30, min_dist = 0.5, resolution = 0.25, cosine distance) to get 8 clusters. Run NMF with k = 20 dictionary items per cluster to obtain cluster dictionaries D(l), concatenate them into a union dictionary U, then fit U to the tissue pixel matrix Y by constrained linear least squares with non-negative weights (Y = UC, C ≥ 0). Compute the vector norm of each pixel's per-cluster weight sub-vectors and map those norms back to original pixel locations to render cluster-contribution maps; validate via Pearson correlation between fitted and observed lipid weights across the shared features (target r = 0.46).

## Related tools

- **MEISTER** (Framework hosting the reconstruction, cross-annotation, NMF dictionary learning, and non-negative least-squares tissue-fitting pipeline executed via tissue_sc_mapping_deepmsi_hip.ipynb) — https://github.com/richardxie1119/MEISTER
- **multiscale_analysis notebooks** (Jupyter notebook tissue_sc_mapping_deepmsi_hip.ipynb implementing cross-annotation, Leiden clustering, NMF dictionary construction, and non-negative least-squares fitting onto the tissue pixel matrix) — https://github.com/richardxie1119/multiscale_analysis

## Examples

```
jupyter notebook tissue_sc_mapping_deepmsi_hip.ipynb
```

## Evaluation signals

- Pearson correlation between fitted and observed lipid weights across the shared feature set should reach approximately r = 0.46 (n = 103 shared features).
- All fitted model weights must be non-negative (C ≥ 0) as a hard constraint of the constrained least-squares solver.
- Cluster-contribution maps should show spatially coherent enrichment, e.g. large weights in the dentate gyrus and CA3 for cluster 0.
- Number of shared lipid features after cross-annotation filtering should be n = 103 (features retained after <5% prevalence thresholds and 3-ppm binning).
- Number of hippocampal cell clusters entering dictionary construction should equal 8 (Leiden resolution = 0.25).

## Limitations

- Modeling errors can arise from falsely cross-annotated features due to mass shifts, an inappropriate ppm window, or nonlinearities between MSI and SCMS data.
- Poorly fitted lipid components with negative correlation were observed in the broader analysis (Fig. 4e); root causes were not fully resolved.
- FDR-controlled metabolite annotation cannot easily be applied to image-guided SCMS data due to lack of spatial information, limiting annotation confidence upstream of fitting.
- The fit is restricted to molecules present in intracellular spaces retained by cross-annotation; extracellular-only lipids are excluded.

## Evidence

- [methods] 3-ppm cross-annotation window: [evidence span withheld — anchor status: not_found]
- [methods] feature prevalence filter thresholds: "Features present in less than 5% of cells were discarded, and cells with less than 5% total number of features were filtered out"
- [methods] TIC normalization of single-cell matrix: "Each pixel was normalized by total ion count (TIC)."
- [methods] m/z binning tolerance: "The peak lists were then processed by m/z binning in 3-ppm increments to align peaks affected by potential mass shift"
- [discussion] UoSS fitting limitation and error sources: [evidence span withheld — anchor status: not_found]
- [discussion] intracellular-space-only cross-annotation rationale: "Another advantage for cross-annotation is to consider only molecules that are present in the intracellular spaces for more accurate single-cell-to-tissue mapping"
- [readme] repository and notebook pointer for this analysis: "tissue_sc_mapping_deepmsi_hip.ipynb: Integrative analysis of hippocampus MSI data with single-cell MS on hippocampal cells."
- [readme] input dataset availability: "coronal3D_propagated_decoded.h5: reconstructed and processed MSI data of serial sections of rat coronal sections."
