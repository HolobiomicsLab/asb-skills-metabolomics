---
name: lipid-feature-quality-control
description: Use when raw peak lists from MSI sections or single-cell SCMS runs contain
  mass-shifted duplicate m/z peaks, and you need to retain only mutually observed,
  low-occurrence-filtered lipid features for downstream annotation or clustering.
when_to_use_negative:
- Input is already a de-duplicated, annotated feature table with no mass-shift issues
  to correct
- The goal is de novo metabolite annotation from a spectral database rather than cross-annotating
  SCMS peaks against a tissue MSI lipid feature list
- Features must be retained regardless of prevalence, e.g. when quantifying rare,
  region-specific lipids that appear in under 5% of cells
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: MEISTER
  role: Provides the preprocessing (m/z binning) and filtering logic used to generate
    the cross-annotated feature list
  repo: https://github.com/richardxie1119/MEISTER
- name: MALDI FT-ICR mass spectrometer
  role: Source instrument for the tissue MSI and single-cell SCMS peak data that are
    binned and filtered
- name: LIPID MAPS
  role: Database used to annotate 1,156 lipid features; 728 matched with <3 ppm error,
    providing the reference feature set
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41592-024-02171-3
    title: Multiscale biochemical mapping of the brain through deep-learning-enhanced
      high-throughput mass spectrometry
  source_content_hash: sha256:cf17e52dae60b7e3e87543dbab09f80b7ca956be55ea33765f6c5fdda13f9b25
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/richardxie1119/MEISTER
  tools:
  - LIPID MAPS
  - MALDI FT-ICR mass spectrometer
  - MEISTER
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# lipid-feature-quality-control

## Summary

A feature-level QC skill from the MEISTER multiscale mass-spectrometry pipeline that aligns lipid peaks by m/z binning in 3-ppm increments, then filters features and cells by a 5% prevalence threshold before cross-annotating single-cell SCMS spectra against a tissue MSI lipid feature database.

## When to use

Use when raw peak lists from MSI sections or single-cell SCMS runs contain mass-shifted duplicate m/z peaks, and you need to retain only mutually observed, low-occurrence-filtered lipid features for downstream annotation or clustering.

## When NOT to use

- Input is already a de-duplicated, annotated feature table with no mass-shift issues to correct
- The goal is de novo metabolite annotation from a spectral database rather than cross-annotating SCMS peaks against a tissue MSI lipid feature list
- Features must be retained regardless of prevalence, e.g. when quantifying rare, region-specific lipids that appear in under 5% of cells

## Inputs

- Peak lists from MALDI FT-ICR tissue MSI sections (e.g. coronal3D_slide3_5_R00_propagated_decoded.imzML)
- Peak lists from single-cell SCMS runs (e.g. peak_data_sc_regional_decode.imzML, 13,566 rat brain cells)
- 3-ppm m/z bin width used to align peaks affected by potential mass shift

## Outputs

- Filtered m/z feature list with retained/filtered flags and MSI-match status (e.g. cross_annotation_feature_list.csv)
- A set of retained, mutually observed lipid features suitable for cross-annotation or cell clustering
- A filtered cell list (cells meeting the 5% feature-presence criterion)

## How to apply

First process the peak lists by m/z binning in 3-ppm increments so that peaks affected by potential mass shift between the SCMS and MSI datasets align to shared bins. Discard any m/z feature present in less than 5% of cells, and filter out cells carrying fewer than 5% of the total number of features, to remove experiment- or sample-specific artifacts while minimizing inclusion of false negatives. For cross-annotation, treat the tissue MSI peak list (considering both intracellular and extracellular lipids) as a search database and match each SCMS peak within the 3-ppm m/z window; retain only matched features that pass the 5% prevalence threshold. The rationale, per the paper's discussion, is that retaining mutual features minimizes the inclusion of experiment- or sample-specific artifacts, and restricting to intracellular molecules yields more accurate single-cell-to-tissue mapping.

## Related tools

- **MEISTER** (Provides the preprocessing (m/z binning) and filtering logic used to generate the cross-annotated feature list) — https://github.com/richardxie1119/MEISTER
- **MALDI FT-ICR mass spectrometer** (Source instrument for the tissue MSI and single-cell SCMS peak data that are binned and filtered)
- **LIPID MAPS** (Database used to annotate 1,156 lipid features; 728 matched with <3 ppm error, providing the reference feature set)

## Examples

```
jupyter nbconvert --to notebook --execute tissue_sc_mapping_deepmsi.ipynb --output cross_annotation_feature_list.csv
```

## Evaluation signals

- Every feature flagged as retained has a tissue-MSI match within the 3-ppm m/z window and passes the 5% prevalence threshold
- Every cell in the output carries at least 5% of the total number of retained features
- The output CSV (cross_annotation_feature_list.csv) exists, is parseable, and contains m/z, match status, and filter flag columns
- Bin alignment removes mass-shifted duplicates: no two retained features fall within the same 3-ppm bin

## Limitations

- FDR-controlled metabolite annotation methods cannot easily be applied to image-guided SCMS data due to lack of spatial information, which is a crucial statistical consideration in the annotation algorithm
- Modeling errors can arise from falsely cross-annotated features caused by mass shifts, an inappropriate ppm window, or nonlinearities between MSI and SCMS data
- Poor lipid fits with negative correlation (Fig. 4e) were attributed to these modeling errors and their root causes were not fully resolved

## Evidence

- [methods] m/z binning corrects mass shift before filtering: "The peak lists were then processed by m/z binning in 3-ppm increments to align peaks affected by potential mass shift"
- [methods] 5% prevalence thresholds for features and cells: "Features present in less than 5% of cells were discarded, and cells with less than 5% total number of features were filtered out"
- [discussion] mutual-feature retention minimizes artifacts: [evidence span withheld — anchor status: not_found]
- [results] cross-annotation uses tissue lipid features as database: "annotated SCMS data using the lipid features from the tissue MSI data (considering the presence of both intracellular and extracellular lipids"
- [readme] README lists concrete input data files for this pipeline: "`coronal3D_slide3_5_R00_propagated_decoded.imzML`: peak data of a representative rat brain coronal section."
