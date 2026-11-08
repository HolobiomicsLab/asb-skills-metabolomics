---
name: single-cell-feature-prevalence-filtering
description: Use when a single-cell SCMS peak matrix (e.g. imzML-derived peak lists
  of thousands of cells) contains sparse or noise-only m/z features, and you need
  prevalence-based pruning before clustering or cross-annotation.
when_to_use_negative:
- Input is a tissue MSI pixel matrix rather than a single-cell peak matrix — the thresholds
  are calibrated for cell-level sparsity, not pixel-level TIC distributions.
- Data have already passed a comparable prevalence filter — reapplying 5% cutoffs
  risks double-filtering and loss of genuine low-abundance intracellular lipids.
- Downstream analysis requires all detected features regardless of sparsity (e.g.
  metabolite coverage audits) since filtering discards genuine but rare signals.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0219
tools:
- name: MEISTER
  role: Provides the notebook-based workflow (e.g. tissue_sc_mapping_deepmsi.ipynb)
    that implements the 3-ppm binning, cross-annotation, and prevalence filtering
    on SCMS peak data.
  repo: https://github.com/richardxie1119/MEISTER
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41592-024-02171-3
    title: Multiscale biochemical mapping of the brain through deep-learning-enhanced
      high-throughput mass spectrometry
  source_content_hash: sha256:78359dd00040725dabf28f3cf095f5f6aa5d3ec38001f7c4eab1c1b7889585d1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/richardxie1119/MEISTER
  tools:
  - MEISTER
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# single-cell-feature-prevalence-filtering

## Summary

A sparsity-based QC filter for single-cell mass-spectrometry peak matrices: discard m/z features observed in <5% of cells and remove cells carrying fewer than 5% of retained features, applied after cross-annotation against tissue MSI lipid features.

## When to use

Use when a single-cell SCMS peak matrix (e.g. imzML-derived peak lists of thousands of cells) contains sparse or noise-only m/z features, and you need prevalence-based pruning before clustering or cross-annotation.

## When NOT to use

- Input is a tissue MSI pixel matrix rather than a single-cell peak matrix — the thresholds are calibrated for cell-level sparsity, not pixel-level TIC distributions.
- Data have already passed a comparable prevalence filter — reapplying 5% cutoffs risks double-filtering and loss of genuine low-abundance intracellular lipids.
- Downstream analysis requires all detected features regardless of sparsity (e.g. metabolite coverage audits) since filtering discards genuine but rare signals.

## Inputs

- Single-cell SCMS peak data (imzML format, e.g. peak_data_sc_regional_decode.imzML with 13,566 rat brain cells)
- Cross-annotated feature list (e.g. cross_annotation_feature_list.csv) with MSI-match status per m/z
- Tissue MSI peak list used as the 3-ppm search database (e.g. coronal3D_slide3_5_R00_propagated_decoded.imzML)

## Outputs

- Filtered single-cell peak matrix with low-prevalence features (<5%) removed
- Retained feature list (m/z, MSI-match status, retained/filtered flag) suitable for clustering
- Updated cell list with low-content cells (<5% of features) removed

## How to apply

After aligning single-cell peak lists by binning m/z in 3-ppm increments (the paper's tolerance for FT-ICR mass shift), compute the fraction of cells in which each feature is detected and discard features present in less than 5% of cells. Conversely, filter out cells that carry fewer than 5% of the total retained features, which removes cells with too little chemical content to be meaningful (e.g. lysed cells or free nuclei). In the MEISTER workflow this filtering is applied to the cross-annotated feature list so that downstream clustering retains only lipids confirmed in both SCMS and tissue MSI data; the article notes that retaining mutual features between tissue and cells minimizes experiment- or sample-specific artifacts. Retained features are those with a tissue-MSI match within the ±3-ppm window and passing the 5% prevalence threshold, which together yielded 344 cross-annotated lipids and 18 lipid-based single-cell clusters.

## Related tools

- **MEISTER** (Provides the notebook-based workflow (e.g. tissue_sc_mapping_deepmsi.ipynb) that implements the 3-ppm binning, cross-annotation, and prevalence filtering on SCMS peak data.) — https://github.com/richardxie1119/MEISTER

## Examples

```
jupyter nbconvert --to notebook --execute tissue_sc_mapping_deepmsi.ipynb  # applies 3-ppm binning, cross-annotation, then 5% feature/cell prevalence filtering
```

## Evaluation signals

- Every retained feature has a tissue-MSI match within the 3-ppm m/z window and passes the ≥5% cell prevalence threshold.
- The output CSV (e.g. cross_annotation_feature_list.csv) exists, parses cleanly, and contains m/z, match status, and retained/filtered columns.
- No cell in the filtered matrix carries fewer than 5% of the total retained features.
- Downstream clustering on the filtered features yields a comparable number of lipid clusters (the reference run produced 18 clusters from 344 lipids).
- Retention counts are reproducible when re-run on the same imzML inputs with the same binning and thresholds.

## Limitations

- Filtering removes genuine but rare intracellular lipids, which could bias downstream mapping if cell types differ strongly in lipid prevalence.
- The 3-ppm binning window and 5% prevalence thresholds are dataset-specific choices for FT-ICR rat brain data; they may need re-tuning for other instruments or tissues.
- Falsely cross-annotated features caused by mass shifts or an inappropriate ppm window can propagate through the filter and later distort UoSS fitting (the paper attributes poor lipid fits with negative correlation to such modeling errors).

## Evidence

- [methods] The peak lists were then processed by m/z binning in 3-ppm increments to align peaks affected by potential mass shift: "The peak lists were then processed by m/z binning in 3-ppm increments to align peaks affected by potential mass shift"
- [methods] Features present in less than 5% of cells were discarded, and cells with less than 5% total number of features were filtered out: "Features present in less than 5% of cells were discarded, and cells with less than 5% total number of features were filtered out"
- [discussion] retaining mutual features can minimize the inclusion of the experiment- or sample-specific artifacts: [evidence span withheld — anchor status: not_found]
- [methods] Feature prevalence filter (features in <5% of cells discarded): [evidence span withheld — anchor status: not_found]
