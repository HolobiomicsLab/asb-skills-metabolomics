---
name: ppm-tolerance-peak-matching
description: Use when you have two peak lists from MALDI FT-ICR MS runs — e.g. peak_data_sc_regional_decode.imzML
  (13,566 single-cell SCMS spectra) and coronal3D_slide3_5_R00_propagated_decoded.
when_to_use_negative:
- When you need FDR-controlled metabolite annotation with statistical significance
  estimates — the article notes that such methods cannot easily be applied to image-guided
  SCMS data due to lack of spatial information.
- When the input is already a cross-annotated feature table (e.g. an existing cross_annotation_feature_list.csv)
  — no re-matching is needed.
- When the two datasets come from different ionization modes or mass spectrometers
  with incompatible m/z ranges — the 3-ppm window cannot bridge gross m/z discrepancies;
  only small mass shifts between runs are assumed.
edam_operation: http://edamontology.org/operation_3629
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MEISTER
  role: Provides the reconstruction, binning, and cross-annotation code that implements
    the 3-ppm peak matching and prevalence filtering for multiscale SCMS/MSI integration.
  repo: https://github.com/richardxie1119/MEISTER
- name: LIPID MAPS
  role: Experimental and virtual lipid database searched with a ±0.005 m/z threshold
    to annotate the tissue MSI feature database before cross-matching.
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41592-024-02171-3
    title: Multiscale biochemical mapping of the brain through deep-learning-enhanced
      high-throughput mass spectrometry
  source_content_hash: sha256:d32dbb829b68b781be7daec0c81755cd08adb37b261280b071de077ad425e128
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/richardxie1119/MEISTER
  tools:
  - LIPID MAPS
  - MEISTER
  edam_operation: http://edamontology.org/operation_3629
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# ppm-tolerance-peak-matching

## Summary

A m/z feature-matching strategy for cross-annotating single-cell SCMS peak lists against a tissue MSI lipid feature database, using 3-ppm mass windows to absorb inter-run mass shifts and enable multiscale lipid-level integration of the two measurement types.

## When to use

Use when you have two peak lists from MALDI FT-ICR MS runs — e.g. peak_data_sc_regional_decode.imzML (13,566 single-cell SCMS spectra) and coronal3D_slide3_5_R00_propagated_decoded.imzML (tissue MSI section) — and you need to decide which single-cell features are present in the tissue data so downstream multiscale mapping (cross-annotation, single-cell-to-tissue fitting) can proceed on a common lipid feature set.

## When NOT to use

- When you need FDR-controlled metabolite annotation with statistical significance estimates — the article notes that such methods cannot easily be applied to image-guided SCMS data due to lack of spatial information.
- When the input is already a cross-annotated feature table (e.g. an existing cross_annotation_feature_list.csv) — no re-matching is needed.
- When the two datasets come from different ionization modes or mass spectrometers with incompatible m/z ranges — the 3-ppm window cannot bridge gross m/z discrepancies; only small mass shifts between runs are assumed.

## Inputs

- Single-cell SCMS peak data (peak_data_sc_regional_decode.imzML, 13,566 rat brain cells)
- Tissue MSI peak data (coronal3D_slide3_5_R00_propagated_decoded.imzML, representative rat brain coronal section)
- Tissue MSI lipid feature database (list of m/z bins retained across all tissue sections, optionally LIPID MAPS-annotated)

## Outputs

- cross_annotation_feature_list.csv containing each m/z feature, its MSI-match status within the 3-ppm window, and its retained/filtered flag
- 344 cross-annotated lipids defining 18 single-cell lipid-based clusters
- Filtered feature/cell matrix retaining only mutually observed features passing the 5% prevalence threshold

## How to apply

First bin the peak lists from both the SCMS and the tissue MSI data in 3-ppm m/z increments to correct for mass shift between acquisitions. Then treat the tissue MSI feature list as the search database and match each single-cell feature against it within the 3-ppm m/z window, recording the match status for every feature. Retaining mutually observed features minimizes the inclusion of experiment- or sample-specific artifacts, and considering only molecules present in intracellular spaces makes the single-cell-to-tissue mapping more accurate. After matching, discard features present in less than 5% of cells and filter out cells carrying fewer than 5% of the total number of features. The rationale is that SCMS data lack the spatial information needed for FDR-controlled annotation methods, so cross-annotation against a tissue-MSI-derived lipid database (e.g. LIPID MAPS-annotated features with <3 ppm error) is the practical route to lipid-level annotation of single-cell spectra.

## Related tools

- **MEISTER** (Provides the reconstruction, binning, and cross-annotation code that implements the 3-ppm peak matching and prevalence filtering for multiscale SCMS/MSI integration.) — https://github.com/richardxie1119/MEISTER
- **LIPID MAPS** (Experimental and virtual lipid database searched with a ±0.005 m/z threshold to annotate the tissue MSI feature database before cross-matching.)

## Examples

```
jupyter notebook tissue_sc_mapping_deepmsi.ipynb  # cross-annotation of SCMS features against 3D MSI data using MEISTER
```

## Evaluation signals

- Every feature flagged as retained has a tissue-MSI match within the 3-ppm m/z window and passes the 5% prevalence threshold.
- cross_annotation_feature_list.csv exists and is parseable with columns for m/z, MSI-match status, and retained/filtered flag.
- The cross-annotation yields approximately 344 lipids (as reported in the paper) that support 18 single-cell lipid-based clusters.
- Peaks were binned in 3-ppm increments before matching, so no feature's match status changes with small mass shifts between SCMS and MSI runs.
- Cells carrying fewer than 5% of total features are absent from the final matrix.

## Limitations

- Modeling errors can arise from falsely cross-annotated features due to mass shifts, an inappropriate ppm window, or nonlinearities between MSI and SCMS data — these contributed to poor lipid fits with negative correlation shown in Fig. 4e.
- Lipid annotation depends on the tissue MSI database being broad enough; the cross-annotation only considers molecules present in intracellular spaces, which may exclude features relevant to the extracellular environment.
- A 3-ppm window was chosen empirically; too narrow a window will miss true matches, and too wide a window will merge distinct lipids.

## Evidence

- [methods] m/z binning in 3-ppm increments: "The peak lists were then processed by m/z binning in 3-ppm increments to align peaks affected by potential mass shift"
- [methods] feature prevalence filter (features in <5% of cells discarded): "Features present in less than 5% of cells were discarded, and cells with less than 5% total number of features were filtered out"
- [discussion] Cross-annotation rationale: [evidence span withheld — anchor status: not_found]
- [readme] Repository data description: "peak_data_sc_regional_decode.imzML: peak data of 13,566 single rat brain cells."
- [readme] README notebook description: "Integrative analysis of 3D MSI data with 13K single-cell MS data from five brain regions."
