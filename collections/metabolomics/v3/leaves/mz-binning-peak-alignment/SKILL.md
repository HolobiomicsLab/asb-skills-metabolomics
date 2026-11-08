---
name: mz-binning-peak-alignment
description: Use when mass spectra from separate MSI/SCMS acquisitions (e.g. imzML
  peak lists) must be compared across runs but mass shift between instruments or sections
  may misalign nominal features — bin before matching, clustering, or cross-annotation.
when_to_use_negative:
- When spectra are already peak-picked and aligned by the instrument or processing
  software with verified negligible mass shift — rebinning would merge distinct lipids
  that legitimately differ by <3 ppm
- When a database search requires tighter accuracy than 3 ppm (e.g. the paper's LIPID
  MAPS search used ±0.005 m/z threshold on high-res data), a coarser 3-ppm bin may
  create false cross-annotations from mass shift
- When the goal is high-mass-resolution spectral reconstruction rather than feature-level
  matching — use MEISTER deep-learning reconstruction instead
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MEISTER
  role: Provides the reconstructed high-resolution spectra whose peak lists are binned
    in 3-ppm increments for cross-annotation and integration
  repo: https://github.com/richardxie1119/MEISTER
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41592-024-02171-3
    title: Multiscale biochemical mapping of the brain through deep-learning-enhanced
      high-throughput mass spectrometry
  source_content_hash: sha256:8b9f29ecc331037849fadea0dd27af0fb1793810c82d26ff8496a84397c84434
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/richardxie1119/MEISTER
  tools:
  - MEISTER
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# m/z binning peak alignment

## Summary

A peak-alignment preprocessing step that bins mass-spectrometry peak lists in 3-ppm m/z increments so that features affected by instrument mass shift line up across datasets, tissue sections, or cells, enabling consistent feature tables for downstream cross-annotation, imaging, or clustering.

## When to use

Use when mass spectra from separate MSI/SCMS acquisitions (e.g. imzML peak lists) must be compared across runs but mass shift between instruments or sections may misalign nominal features — bin before matching, clustering, or cross-annotation.

## When NOT to use

- When spectra are already peak-picked and aligned by the instrument or processing software with verified negligible mass shift — rebinning would merge distinct lipids that legitimately differ by <3 ppm
- When a database search requires tighter accuracy than 3 ppm (e.g. the paper's LIPID MAPS search used ±0.005 m/z threshold on high-res data), a coarser 3-ppm bin may create false cross-annotations from mass shift
- When the goal is high-mass-resolution spectral reconstruction rather than feature-level matching — use MEISTER deep-learning reconstruction instead

## Inputs

- Peak lists from tissue MALDI FT-ICR MSI data (e.g. coronal3D_slide3_5_R00_propagated_decoded.imzML)
- Peak lists from image-guided MALDI single-cell MS data (e.g. peak_data_sc_regional_decode.imzML, 13,566 rat brain cells)
- Peak lists from multiple serial tissue sections to be unioned or intersected

## Outputs

- Binned peak/feature table where each row is a 3-ppm m/z bin with per-sample or per-cell peak intensities
- Cross-annotation feature list (cross_annotation_feature_list.csv) recording each m/z feature, its MSI-match status within the 3-ppm window, and its retained/filtered flag

## How to apply

Extract per-run peak lists (e.g. from the imzML peak data files in the deposit 10.13012/B2IDB-9740536_V1) and group peaks by rounding each m/z into bins of 3 ppm width; this aligns peaks that drift slightly between the SCMS and MSI datasets or between serial tissue sections. Because the bin width is defined in ppm, the effective m/z window scales with the peak position, so a 3-ppm bin at m/z 813 spans ~0.0024 Da — consistent with the paper's ±0.005 m/z / <3 ppm lipid-matching tolerance. After binning, either retain the m/z bins common to all tissue sections for spatial analysis, or treat one dataset's binned peaks as the search database and match the other dataset's peaks within the 3-ppm window (recording each feature's match status). Apply the 5% prevalence filters only after binning: discard features present in fewer than 5% of cells and cells with fewer than 5% of total features. The rationale is that FT-ICR MS mass shifts from short transients or different acquisitions would otherwise split the same lipid into separate features and break cross-annotation.

## Related tools

- **MEISTER** (Provides the reconstructed high-resolution spectra whose peak lists are binned in 3-ppm increments for cross-annotation and integration) — https://github.com/richardxie1119/MEISTER

## Examples

```
pandas: `df['bin'] = (df['mz'] * 1e6 / 3).round(); binned = df.groupby(['bin','mz_center'], as_index=False)['intensity'].sum()` — bin peaks in 3-ppm increments before matching SCMS peaks against the tissue MSI feature list
```

## Evaluation signals

- Every feature flagged as retained must have a corresponding tissue-MSI match within the 3-ppm window
- Every retained feature must pass the 5% prevalence threshold (present in ≥5% of cells) and every retained cell must carry ≥5% of total features
- The output CSV exists and is parseable with columns for m/z feature, MSI-match status, and retained/filtered flag
- Binned feature count must be smaller than the union of raw peak counts and larger than the intersection count when binned across sections
- Match rate sanity check: the paper reports 728 of 1,156 lipid features matched within <3 ppm error — a substantially higher or lower match rate on comparable data indicates a binning or calibration problem

## Limitations

- Mass shifts larger than the 3-ppm window leave true features unmatched; too generous a window can falsely cross-annotate distinct lipids, causing modeling errors in downstream single-cell-to-tissue mapping
- Nonlinearities between MSI and SCMS data mean some cross-annotated features fit poorly (negative-correlation lipid fits in Fig. 4e), and root causes were not fully resolved
- Binning is a lossy approximation: near-coincident lipids within 3 ppm may merge into a single feature

## Evidence

- [methods] The peak lists were then processed by m/z binning in 3-ppm increments to align peaks affected by potential mass shift: "The peak lists were then processed by m/z binning in 3-ppm increments to align peaks affected by potential mass shift"
- [results] annotated SCMS data using the lipid features from the tissue MSI data: "annotated SCMS data using the lipid features from the tissue MSI data (considering the presence of both intracellular and extracellular lipids; Methods, ‘Cross-annotation strategy’)"
- [methods] Features present in less than 5% of cells were discarded, and cells with less than 5% total number of features were filtered out: "Features present in less than 5% of cells were discarded, and cells with less than 5% total number of features were filtered out"
- [methods] we searched the m/z values against LIPID MAPS34 experimental and virtual databases with a ±0.005 m/z threshold: [evidence span withheld — anchor status: not_found]
