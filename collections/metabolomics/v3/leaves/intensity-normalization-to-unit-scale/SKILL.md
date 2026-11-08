---
name: intensity-normalization-to-unit-scale
description: Use when stacked peak plots from an MZmine 2 aligned peak list must be
  converted to unit-maximum, fixed-size 64x64 raster images to feed an image-based
  DNN peak classifier.
when_to_use_negative:
- Input is already a feature table of peak group parameters (e.g., the 59 peak group
  parameters used for the multiple logistic regression or Random Forest models) —
  those models consume tabular parameters, not raster images
- You need absolute quantification of metabolite abundances — unit-maximum scaling
  discards absolute intensity information by design
- Peak data has not yet been extracted and aligned with MZmine 2 — there are no stacked
  peak groups to normalize
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MZmine 2
  role: Generates the non-filtered peak lists and grouped peak windows from raw LC-MS
    data that the normalization/rasterization step consumes
- name: R
  role: Custom R scripts implement the scaling of apex intensities to maximum 1 and
    the 64x64 rasterization of stacked peak features
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.9b02983
    title: Deep Neural Networks for Classification of LC-MS Spectral Peaks
  source_content_hash: sha256:16d4a7b1e57bb189c64864fe67b2e39553056ec6d904ee4724c4201bba3cab5e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - MZmine 2
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 15
---

# intensity-normalization-to-unit-scale

## Summary

Rescales stacked LC-MS peak group apex intensities so each candidate feature's maximum equals 1, then rasterizes the normalized stack into the 64x64 pixel images (rows 1–63 per-sample intensity, row 64 retention-time window boundary) consumed by the deep neural network peak classifier.

## When to use

Use when stacked peak plots from an MZmine 2 aligned peak list must be converted to unit-maximum, fixed-size 64x64 raster images to feed an image-based DNN peak classifier.

## When NOT to use

- Input is already a feature table of peak group parameters (e.g., the 59 peak group parameters used for the multiple logistic regression or Random Forest models) — those models consume tabular parameters, not raster images
- You need absolute quantification of metabolite abundances — unit-maximum scaling discards absolute intensity information by design
- Peak data has not yet been extracted and aligned with MZmine 2 — there are no stacked peak groups to normalize

## Inputs

- MZmine 2 aligned peak list with grouped peak windows (m/z and retention-time windows around each group apex)
- Raw per-sample peak intensities for up to 63 most-intense samples per candidate feature
- True/false peak labels for each candidate feature

## Outputs

- 64x64 pixel raster image per stacked peak feature (rows 1–63 scaled intensities, row 64 retention-time window boundary flags)
- raster_images.npz serializing all raster images with their true/false peak labels

## How to apply

Load the MZmine 2 aligned peak list and grouped peak windows for each candidate feature, then extract the m/z and retention-time windows surrounding the group apex. Select peak information from the 63 samples in which the candidate peak was most intense and divide each per-sample apex intensity by the group's maximum so apex intensity maxima equal 1. Rasterize the scaled stack into a 64x64 image where each column is a retention-time point, rows 1–63 hold per-sample intensities, and row 64 encodes the retention-time window boundary (1 inside window, 0 outside). Attach each image's true/false peak label and serialize all rasters into raster_images.npz for downstream training with EDML_deep_learning2.py. The unit-maximum normalization is what makes peak-shape information comparable across features of wildly different absolute intensity, which is why the image-based DNN can learn shape-based true/false discrimination.

## Related tools

- **MZmine 2** (Generates the non-filtered peak lists and grouped peak windows from raw LC-MS data that the normalization/rasterization step consumes)
- **R** (Custom R scripts implement the scaling of apex intensities to maximum 1 and the 64x64 rasterization of stacked peak features)

## Examples

```
python EDML_deep_learning2.py
```

## Evaluation signals

- Each output raster has shape exactly 64x64 pixels with one column per retention-time point
- For every stacked feature, the maximum per-sample apex intensity across rows 1–63 equals exactly 1 after scaling
- Row 64 contains only binary values: 1 inside the retention-time window boundary and 0 outside
- Exactly 63 of the 64 rows hold per-sample intensities, drawn from the 63 samples in which the candidate peak was most intense
- All rasters serialize into raster_images.npz with each image carrying its true/false peak label

## Limitations

- Row 64 encodes only the retention-time window boundary, so the raster does not preserve absolute intensity scale and is unsuitable for quantitative abundance comparisons
- Per-feature selection of the 63 most-intense samples means the raster composition depends on the underlying MZmine 2 peak extraction ('More Restrictive' vs 'Less Restrictive') settings
- No code or model hosting repository beyond the JainLab manuscript repo is referenced for this raster generation step; the workflow is documented in WorkFlow.md

## Evidence

- [other] unit-maximum scaling and 64x64 rasterization: [evidence span withheld — anchor status: not_found]
