---
name: xcms-peak-annotation-integration
description: Use when XCMS-flagged features have been realigned by ncGTW and the realignment
  output must be converted back into XCMS RT warping functions (xcmsLargeWin@rt$corrected)
  and relocated apex positions for downstream grouping/peak-filling.
when_to_use_negative:
- Features were not flagged as misaligned by the ncGTW misalignment detection module
  — the distortion-correction post-processing only applies to realigned features
- Input is raw LC-MS spectra prior to XCMS peak detection, RT alignment, and peak
  grouping — the post-processing module consumes XCMS preprocessing output, not raw
  data
- Goal is misalignment detection itself rather than post-reintegration of already-realigned
  features into the XCMS object
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3383
tools:
- name: ncGTW
  role: Provides the realignment output and the post-processing functions (adjustRT(),
    ncGTWpeaks) that generate corrected RT warping functions and relocated peak positions
  repo: https://github.com/ChiungTingWu/ncGTW
- name: XCMS
  role: Supplies the peak start/end points, peak widths, and the xcmsLargeWin object
    whose @rt$corrected slot is replaced; consumes the corrected output for grouping
    and peak-filling
- name: R
  role: Implementation language of the ncGTW Bioconductor package used for the post-processing
    step
  repo: https://github.com/ChiungTingWu/ncGTW
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btaa037
    title: Targeted realignment of LC-MS profiles by neighbor-wise compound-specific
      graphical time warping with misalignment detection
  source_content_hash: sha256:9b2a26d3d2f5f0f32c352e41dda23b0e072f81545cbbc8146b4e7dba977c4fb8
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/ChiungTingWu/ncGTW
  tools:
  - R
  - XCMS
  - ncGTW
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3383
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# xcms-peak-annotation-integration

## Summary

Converts ncGTW's realigned LC-MS feature profiles back into XCMS-compatible RT warping functions and relocated peak positions via adjustRT() and ncGTWpeaks, correcting peak-shape distortion before handing results back to XCMS for grouping and peak-filling.

## When to use

Use when XCMS-flagged features have been realigned by ncGTW and the realignment output must be converted back into XCMS RT warping functions (xcmsLargeWin@rt$corrected) and relocated apex positions for downstream grouping/peak-filling.

## When NOT to use

- Features were not flagged as misaligned by the ncGTW misalignment detection module — the distortion-correction post-processing only applies to realigned features
- Input is raw LC-MS spectra prior to XCMS peak detection, RT alignment, and peak grouping — the post-processing module consumes XCMS preprocessing output, not raw data
- Goal is misalignment detection itself rather than post-reintegration of already-realigned features into the XCMS object

## Inputs

- ncGTW realignment output for detected misaligned features (per-feature, per-sample aligned profiles)
- XCMS object (xcmsLargeWin) with peak start/end points and peak widths per feature and sample
- Per-feature intensity profiles across samples from the aligned XCMS peak groups

## Outputs

- Corrected RT warping functions replacing xcmsLargeWin@rt$corrected
- ncGTWpeaks table with relocated peak positions (apices) in the new RT coordinate
- Per-feature intensity table across samples with corrected (undistorted) peak profiles

## How to apply

Load the ncGTW realigned feature profiles (output of the realignment step) together with the XCMS-provided peak start/end points and peak widths for each flagged feature. For each feature and sample, restrict the warping function to the apex point within the peak's start-end boundaries and shift all remaining points inside each peak boundary rigidly by the apex's displacement — this prevents the many-to-one/one-to-one many-to-many mappings of raw DTW warping that broaden or narrow peaks. For samples with missing peaks undetected by XCMS, locate the apex on the virtual reference aggregated from other samples, assign the corresponding warping-function point as the apex, and align the whole missing peak using the median of the peak widths. Assemble the corrected profiles into a per-feature intensity table, apply adjustRT() to generate new RT warping functions that replace xcmsLargeWin@rt$corrected, and use ncGTWpeaks to relocate peaks so XCMS can perform more accurate grouping and peak-filling on the corrected data.

## Related tools

- **ncGTW** (Provides the realignment output and the post-processing functions (adjustRT(), ncGTWpeaks) that generate corrected RT warping functions and relocated peak positions) — https://github.com/ChiungTingWu/ncGTW
- **XCMS** (Supplies the peak start/end points, peak widths, and the xcmsLargeWin object whose @rt$corrected slot is replaced; consumes the corrected output for grouping and peak-filling)
- **R** (Implementation language of the ncGTW Bioconductor package used for the post-processing step) — https://github.com/ChiungTingWu/ncGTW

## Examples

```
devtools::install_github("ChiungTingWu/ncGTW")
```

## Evaluation signals

- Within each corrected feature, the apices remain co-aligned across all samples after the rigid-shift correction
- Peak shape (width/height profile) is preserved relative to the pre-correction realigned profiles, mirroring the Fig. 5c-to-5d correction criterion
- No many-to-one or one-to-many mappings remain within any peak boundary (each point inside a peak was shifted rigidly by the apex displacement)
- The corrected RT warping functions successfully replace xcmsLargeWin@rt$corrected and XCMS grouping/peak-filling runs on the realigned data without errors
- For samples with missing peaks, the assigned apex on the virtual reference uses the median peak width and the resulting filled peak is consistent with other samples in the feature group

## Limitations

- ncGTW deliberately accepts false positives from its misalignment detection and screens them out at a later stage via correlation coefficient and overlapping area scores, so some post-processed features may still need filtering
- Parameter settings in ncGTW can be adjusted or optimized to reduce the likelihood of peak distortion; defaults are not exhaustively documented in the article
- For peaks missing in some samples, apex placement relies on a virtual reference aggregated from other samples and the median peak width, which may be imprecise when few samples detect the feature

## Evidence

- [methods] Peak shape distortion is then efficiently corrected by the post-processing module of ncGTW package.: [evidence span withheld — anchor status: not_found]
