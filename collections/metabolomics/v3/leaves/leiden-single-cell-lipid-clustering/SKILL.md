---
name: leiden-single-cell-lipid-clustering
description: Use when you have a cross-annotated single-cell SCMS peak matrix (e.g.
  peak_data_sc_regional_decode.imzML) and need to partition cells into lipid-chemistry
  clusters before mapping those cluster dictionaries onto tissue MSI pixels.
when_to_use_negative:
- The input is already a tissue MSI pixel matrix rather than single-cell peak lists
  — use the tissue-pixel UMAP / classification workflow instead.
- You lack cross-annotated shared lipid features between SCMS and tissue MSI; clustering
  unannotated raw peak lists will not reproduce the 8-subpopulation structure.
- The goal is annotation of metabolite identities (FDR-controlled) rather than cell-subpopulation
  discovery — MEISTER's discussion notes that image-guided SCMS data lack the spatial
  information required for those annotation methods.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MEISTER
  role: Framework implementing the deep-learning reconstruction that produces the
    decoded single-cell peak data (peak_data_sc_regional_decode.imzML) on which the
    Leiden clustering is performed.
  repo: https://github.com/richardxie1119/MEISTER
- name: tissue_sc_mapping_deepmsi_hip.ipynb
  role: Notebook that executes the cross-annotation, PCA, Leiden clustering (n_neighbors
    = 30, min_dist = 0.5, resolution = 0.25), and downstream dictionary fitting on
    hippocampal data.
  repo: https://github.com/richardxie1119/multiscale_analysis
- name: microMS
  role: Used upstream to obtain the single-cell coordinates and geometry files needed
    for image-guided MALDI SCMS acquisition; not part of clustering itself but defines
    the cell identities clustered.
- name: ftms Control (v.2.1.0, Bruker)
  role: Instrument control software whose target-automation files are generated from
    microMS; produces the raw SCMS spectra that MEISTER reconstructs before clustering.
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41592-024-02171-3
    title: Multiscale biochemical mapping of the brain through deep-learning-enhanced
      high-throughput mass spectrometry
  source_content_hash: sha256:0e7364d1d483391768f9dea85c02c2dacf6dc90dc7dc2d2bea1a06ead6653825
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/richardxie1119/MEISTER
  tools:
  - MEISTER
  - ftms Control (v.2.1.0, Bruker)
  - microMS
  - tissue_sc_mapping_deepmsi_hip.ipynb
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# leiden-single-cell-lipid-clustering

## Summary

Leiden clustering of TIC-normalized single-cell lipid feature matrices from image-guided MALDI SCMS data, applied to dissect hippocampal cell subpopulations whose spatial contributions are later fitted against tissue MSI data.

## When to use

Use when you have a cross-annotated single-cell SCMS peak matrix (e.g. peak_data_sc_regional_decode.imzML) and need to partition cells into lipid-chemistry clusters before mapping those cluster dictionaries onto tissue MSI pixels.

## When NOT to use

- The input is already a tissue MSI pixel matrix rather than single-cell peak lists — use the tissue-pixel UMAP / classification workflow instead.
- You lack cross-annotated shared lipid features between SCMS and tissue MSI; clustering unannotated raw peak lists will not reproduce the 8-subpopulation structure.
- The goal is annotation of metabolite identities (FDR-controlled) rather than cell-subpopulation discovery — MEISTER's discussion notes that image-guided SCMS data lack the spatial information required for those annotation methods.

## Inputs

- peak_data_sc_regional_decode.imzML — peak data of 13,566 single rat brain cells (hippocampal subset for this skill)
- Cross-annotated lipid feature matrix after 3-ppm m/z binning against tissue MSI features

## Outputs

- 8 hippocampal cell clusters (Leiden labels per cell)
- Per-cluster UMAP / PCA embedding for visual inspection
- Cluster dictionaries D(l) used as input for tissue-pixel fitting (UoSS)

## How to apply

Start from the cross-annotated SCMS peak lists in imzML format: bin m/z in 3-ppm increments, TIC-normalize each cell, discard features present in fewer than 5% of cells, and filter cells with fewer than 5% of the retained features. Compute the first 40 principal components of the resulting lipid feature matrix, then run Leiden clustering on the 40 PCs using cosine distance with n_neighbors = 30, min_dist = 0.5, and resolution = 0.25 to obtain 8 hippocampal cell clusters. Each cluster is subsequently used to build a chemical dictionary (NMF with k = 20 items per cluster) whose contributions are fitted to the tissue pixel matrix Y = UC with C ≥ 0; the clustering resolution and neighborhood parameters were tuned to yield the 8 subpopulations reported in the paper.

## Related tools

- **MEISTER** (Framework implementing the deep-learning reconstruction that produces the decoded single-cell peak data (peak_data_sc_regional_decode.imzML) on which the Leiden clustering is performed.) — https://github.com/richardxie1119/MEISTER
- **tissue_sc_mapping_deepmsi_hip.ipynb** (Notebook that executes the cross-annotation, PCA, Leiden clustering (n_neighbors = 30, min_dist = 0.5, resolution = 0.25), and downstream dictionary fitting on hippocampal data.) — https://github.com/richardxie1119/multiscale_analysis
- **microMS** (Used upstream to obtain the single-cell coordinates and geometry files needed for image-guided MALDI SCMS acquisition; not part of clustering itself but defines the cell identities clustered.)
- **ftms Control (v.2.1.0, Bruker)** (Instrument control software whose target-automation files are generated from microMS; produces the raw SCMS spectra that MEISTER reconstructs before clustering.)

## Examples

```
python -m papermill tissue_sc_mapping_deepmsi_hip.ipynb out.ipynb -p resolution 0.25 -p n_neighbors 30 -p min_dist 0.5
```

## Evaluation signals

- Clustering yields exactly 8 hippocampal cell clusters at the specified resolution = 0.25 (Fig. 4c/4e cluster count).
- Feature matrix is restricted to the 103 shared lipid features that matched tissue MSI within 3-ppm m/z tolerance; per-cell feature prevalence ≥ 5%.
- Downstream UoSS fit on the clustered dictionaries achieves Pearson r = 0.46 between fitted and observed lipid weights (target value reported in the paper).
- Cluster weights mapped back to tissue pixels show spatial segregation consistent with known hippocampal subfields, e.g. large weights in the dentate gyrus and CA3 for cluster 0.
- Every cell retains at least 5% of the retained features after filtering — cells failing this threshold must be excluded before clustering.

## Limitations

- UoSS fitting on cluster dictionaries can produce modeling errors when features are falsely cross-annotated due to mass shifts or an inappropriate ppm window, or due to nonlinearities between MSI and SCMS data.
- Poorly fitted lipids with negative correlation (Fig. 4e) remain unresolved; the authors note that a close examination of those components may lead to important insights.
- Retaining mutually observed features minimizes experiment- or sample-specific artifacts, but discarding non-shared features limits the chemical space that can be clustered.
- FDR-controlled metabolite annotation methods cannot easily be applied to image-guided SCMS data due to lack of spatial information, so cluster identities are de novo chemical subpopulations rather than annotated metabolite classes.

## Evidence

- [methods] Leiden clustering parameters: [evidence span withheld — anchor status: not_found]
- [methods] Filter thresholds: "Features present in less than 5% of cells were discarded, and cells with less than 5% total number of features were filtered out"
- [methods] m/z binning: "The peak lists were then processed by m/z binning in 3-ppm increments to align peaks affected by potential mass shift"
- [discussion] Cross-annotation rationale: [evidence span withheld — anchor status: not_found]
- [results] Cluster fitting outcome: [evidence span withheld — anchor status: not_found]
- [discussion] Annotation limitation: [evidence span withheld — anchor status: not_found]
- [results] Downstream fit: [evidence span withheld — anchor status: not_found]
- [readme] Notebook entry point: "Integrative analysis of hippocampus MSI data with single-cell MS on hippocampal cells."
