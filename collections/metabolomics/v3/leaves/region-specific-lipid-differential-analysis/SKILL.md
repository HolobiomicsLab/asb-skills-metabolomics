---
name: region-specific-lipid-differential-analysis
description: Use when you have a registered 3D MSI volume with propagated anatomical
  atlas labels and need region-specific lipid enrichment tables (e.g. myelin lipids
  elevated in corpus callosum) from mass-spectrometry imaging data.
when_to_use_negative:
- Input is a single-cell (SCMS) dataset rather than a registered 3D tissue MSI volume
  — cross-annotation and regional differential analysis are designed for tissue-level
  spatial data.
- Data have not been registered to the atlas (e.g. raw reconstructed sections without
  propagated region labels) — regional pixel extraction and differential testing require
  anatomical labels.
- Goal is single-cell-to-tissue chemical mapping (cross-annotation) rather than region-level
  lipid enrichment testing.
edam_operation: http://edamontology.org/operation_3634
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3172
tools:
- name: MEISTER
  role: Produces the reconstructed, high-SNR MSI volume and the multiscale_analysis
    notebooks (regmri_atlas_coronal.ipynb) used for brain regional analysis of the
    registered 3D coronal data.
  repo: https://github.com/richardxie1119/MEISTER
- name: MALDI FT-ICR mass spectrometer
  role: Acquires the serial coronal sections whose reconstructed pixel spectra are
    partitioned into atlas-defined brain regions for differential lipid testing.
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41592-024-02171-3
    title: Multiscale biochemical mapping of the brain through deep-learning-enhanced
      high-throughput mass spectrometry
  source_content_hash: sha256:d6b0a558b13cfdc48565f523524eea08fdd720b81f8229c5ee446a973678e8f8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/richardxie1119/MEISTER
  tools:
  - MALDI FT-ICR mass spectrometer
  - MEISTER
  edam_operation: http://edamontology.org/operation_3634
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# region-specific-lipid-differential-analysis

## Summary

Extracts lipid features from atlas-defined rat brain regions in the registered 3D MSI volume, annotates them via LIPID MAPS (±0.005 m/z), and performs log2 fold-change with FDR-adjusted Wilcoxon rank-sum testing to identify region-specific lipid enrichments.

## When to use

Use when you have a registered 3D MSI volume with propagated anatomical atlas labels and need region-specific lipid enrichment tables (e.g. myelin lipids elevated in corpus callosum) from mass-spectrometry imaging data.

## When NOT to use

- Input is a single-cell (SCMS) dataset rather than a registered 3D tissue MSI volume — cross-annotation and regional differential analysis are designed for tissue-level spatial data.
- Data have not been registered to the atlas (e.g. raw reconstructed sections without propagated region labels) — regional pixel extraction and differential testing require anatomical labels.
- Goal is single-cell-to-tissue chemical mapping (cross-annotation) rather than region-level lipid enrichment testing.

## Inputs

- coronal3D_propagated_decoded.h5 — reconstructed and registered 3D coronal MSI volume
- Propagated Waxholm Space atlas brain region labels for each pixel
- LIPID MAPS experimental and virtual databases

## Outputs

- region_lipid_enrichment.tsv — region-by-lipid enrichment table with annotation, log2 fold change, and Benjamini–Hochberg-adjusted P-value columns per brain structure

## How to apply

Load the reconstructed and registered 3D coronal MSI volume (coronal3D_propagated_decoded.h5) together with propagated Waxholm Space atlas brain region labels. Extract MSI pixels belonging to the 11 brain structures (neocortex, hippocampus, thalamus, hypothalamus, corpus callosum, corticofugal pathway, superior colliculus, basal forebrain, brainstem, striatum, septal regions) and normalize each pixel by total ion count (TIC). Annotate lipid features by searching m/z values against LIPID MAPS experimental and virtual databases with a ±0.005 m/z threshold, prioritizing experimentally validated lipids sorted by ppm error. Compute regional average lipid profiles per brain structure, then perform differential analysis calculating log2 fold change and P-values using the Wilcoxon rank-sum test adjusted by Benjamini–Hochberg. Optionally interpret regional lipid signatures with gradient-boosted tree classifiers and rank contributing lipids by mean absolute SHAP values; assemble the region-by-lipid enrichment table (region_lipid_enrichment.tsv) with annotation, fold change, and adjusted P-value columns per structure.

## Related tools

- **MEISTER** (Produces the reconstructed, high-SNR MSI volume and the multiscale_analysis notebooks (regmri_atlas_coronal.ipynb) used for brain regional analysis of the registered 3D coronal data.) — https://github.com/richardxie1119/MEISTER
- **MALDI FT-ICR mass spectrometer** (Acquires the serial coronal sections whose reconstructed pixel spectra are partitioned into atlas-defined brain regions for differential lipid testing.)

## Examples

```
python regmri_atlas_coronal.ipynb --input coronal3D_propagated_decoded.h5 --atlas waxholm_labels --output region_lipid_enrichment.tsv
```

## Evaluation signals

- The region_lipid_enrichment.tsv must show enrichment of myelin-associated HexCer/Cer/SM species — particularly ceramide elevated in subcortical white matter such as the corpus callosum and corticofugal pathway — with a similar trend for sphingomyelin (SM).
- All 11 brain structures defined by the atlas labels must be represented in the output table, each with annotation, fold change, and adjusted P-value columns.
- Regional P-values must be derived from Wilcoxon rank-sum tests and adjusted via Benjamini–Hochberg; flag discrepancies for expert review where FDR-adjusted results do not support the reported enrichment pattern.
- Per-pixel intensities must be TIC-normalized before computing regional averages; skipping normalization would bias fold-change estimates.
- Lipid annotations must match LIPID MAPS within ±0.005 m/z, prioritizing experimentally validated entries sorted by ppm error.

## Limitations

- FDR-controlled metabolite annotation methods cannot easily be applied to image-guided SCMS data due to lack of spatial information; this skill is restricted to tissue-level spatial MSI.
- UoSS fitting modeling errors can arise from falsely cross-annotated features due to mass shifts, inappropriate ppm window, or nonlinearities between MSI and SCMS data — poor lipid fits with negative correlation may indicate modeling errors.
- Differential testing relies on the propagated Waxholm Space atlas labels; registration errors across serial sections would propagate into region assignments.

## Evidence

- [methods] Extract brain region pixels and normalize: "Each pixel was normalized by total ion count (TIC)."
- [methods] LIPID MAPS annotation threshold: [evidence span withheld — anchor status: not_found]
- [abstract] Region-specific lipid contents identified: "We identified region-specific lipid contents and cell-specific localizations of lipids depending on both cell subpopulations and anatomical origins of the cells."
- [readme] regmri_atlas_coronal notebook purpose: "`regmri_atlas_coronal.ipynb`: Brain regional analysis of registered 3D coronal data with MRI atlas."
- [results] Gradient-boosted trees + SHAP feature attribution: [evidence span withheld — anchor status: not_found]
- [readme] 11 brain structures in registered 3D coronal data: "- `coronal3D_propagated_decoded.h5`: reconstructed and processed MSI data of serial sections of rat coronal sections."
