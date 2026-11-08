---
name: tic-normalized-msi-quantification
description: Use when per-pixel MSI intensities must be compared across spatial positions,
  serial sections, or anatomical regions — e.g. before region-level fold-change testing
  on registered rat-brain coronal/sagittal volumes.
when_to_use_negative:
- Input is already a TIC-normalized or otherwise comparable feature table — re-normalizing
  may distort fold changes
- Analysis targets single-cell MS data rather than tissue MSI pixels — the analogous
  cell-level filtering (prevalence <5%) replaces per-pixel TIC normalization
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: MEISTER
  role: provides the reconstructed 3D MSI volume whose pixel intensities are TIC-normalized
  repo: github.com/richardxie1119/MEISTER
- name: MALDI FT-ICR mass spectrometer
  role: acquires the raw imaging data that, after reconstruction, feed the TIC-normalized
    quantification
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41592-024-02171-3
    title: Multiscale biochemical mapping of the brain through deep-learning-enhanced
      high-throughput mass spectrometry
  source_content_hash: sha256:1c9b7c2dfb9a9c0bf0b6782d9024946280817fef1f354a1e31f623ca5f1bf534
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MALDI FT-ICR mass spectrometer
  - MEISTER
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# TIC-normalized MSI quantification

## Summary

Normalizes every MALDI FT-ICR MSI pixel by its total ion count so lipid-feature intensities become comparable across pixels, sections, and brain regions before regional differential analysis.

## When to use

Use when per-pixel MSI intensities must be compared across spatial positions, serial sections, or anatomical regions — e.g. before region-level fold-change testing on registered rat-brain coronal/sagittal volumes.

## When NOT to use

- Input is already a TIC-normalized or otherwise comparable feature table — re-normalizing may distort fold changes
- Analysis targets single-cell MS data rather than tissue MSI pixels — the analogous cell-level filtering (prevalence <5%) replaces per-pixel TIC normalization

## Inputs

- reconstructed 3D MALDI FT-ICR MSI volume (coronal3D_propagated_decoded.h5 or .imzML)
- propagated Waxholm Space atlas region labels per pixel
- m/z-binned peak lists (3-ppm bins, bins common across all sections)

## Outputs

- TIC-normalized pixel×m/z-bin intensity matrix
- region-by-lipid enrichment table (region_lipid_enrichment.tsv) with log2 fold change and BH-adjusted P-values

## How to apply

After loading the reconstructed 3D MSI volume (e.g. coronal3D_propagated_decoded.h5) and the propagated Waxholm Space atlas labels, normalize each pixel by total ion count (TIC) prior to any downstream regional aggregation. Peak lists should first be m/z-binned in 3-ppm increments to align peaks affected by potential mass shift, and m/z bins common across all tissue sections should be retained, so TIC normalization operates on a consistently aligned feature matrix. TIC normalization is applied per pixel; for single-cell data the analogous filter is feature prevalence (features in <5% of cells discarded, cells with <5% total features filtered out), but the per-pixel TIC step itself is the MSI-side normalization. Normalized intensities feed regional averaging per brain structure and Benjamini–Hochberg-adjusted Wilcoxon rank-sum tests, so normalization must precede fold-change computation to avoid section-level ion-current bias.

## Related tools

- **MEISTER** (provides the reconstructed 3D MSI volume whose pixel intensities are TIC-normalized) — github.com/richardxie1119/MEISTER
- **MALDI FT-ICR mass spectrometer** (acquires the raw imaging data that, after reconstruction, feed the TIC-normalized quantification)

## Evaluation signals

- Every pixel's normalized spectrum sums (over retained m/z bins) to the same total, confirming per-pixel TIC normalization
- After normalization, regional lipid profiles reproduce the reported enrichment of myelin-associated HexCer/Cer/SM species in the corpus callosum
- m/z bins used for normalization are the subset retained as common across all tissue sections (3-ppm binning)
- Wilcoxon rank-sum results with Benjamini–Hochberg adjustment yield significant regional differences consistent with the reported FDR-corrected findings

## Limitations

- FDR-controlled metabolite annotation methods cannot easily be applied to image-guided SCMS data due to lack of spatial information, so TIC-normalized features from tissue MSI cannot be rigorously FDR-validated in the single-cell setting
- UoSS fitting modeling errors can arise from falsely cross-annotated features due to mass shifts, inappropriate ppm window, or nonlinearities between MSI and SCMS data
- TIC normalization assumes comparable total ion counts across pixels; extreme ion-suppression or matrix-heterogeneity artifacts are not separately corrected

## Evidence

- [methods] Each pixel was normalized by total ion count (TIC).: "Each pixel was normalized by total ion count (TIC)."
- [methods] m/z binning before normalization: "The peak lists were then processed by m/z binning in 3-ppm increments to align peaks affected by potential mass shift"
- [methods] retain common bins across sections: "we retained m/z bins common across all tissue sections for further analysis"
- [results] Cer elevated in corpus callosum: [evidence span withheld — anchor status: not_found]
- [results] Wilcoxon + BH adjusted: [evidence span withheld — anchor status: not_found]
- [methods] LIPID MAPS annotation threshold: [evidence span withheld — anchor status: not_found]
