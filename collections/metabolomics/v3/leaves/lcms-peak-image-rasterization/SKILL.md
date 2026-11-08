---
name: lcms-peak-image-rasterization
description: Use when nontargeted LC-MS peak lists from MZmine 2 must be converted
  into fixed-size 64x64 raster images so a deep neural network can classify candidate
  peaks as true or false.
when_to_use_negative:
- Peak data is already stored as an image/raster array (e.g. an existing raster_images.npz)
  — re-rasterization would duplicate work.
- The goal is peak-parameter-based filtering (e.g. medRelFWHM logistic regression
  or Random Forest on 59 peak group parameters) rather than image-based deep learning
  classification.
- Peaks were not extracted with MZmine 2 aligned peak lists and grouped retention-time
  windows; raw vendor files cannot be rasterized directly by this workflow.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: MZmine 2
  role: Produces the non-filtered aligned peak lists and grouped peak windows from
    raw LC-MS data that serve as the rasterization input
- name: R
  role: Custom R scripts perform the stacked-peak extraction, apex-intensity normalization,
    64x64 rasterization, and raster_images.npz serialization
- name: EDML_deep_learning2.py
  role: Consumes the serialized raster images to train the deep neural network peak
    classifier
  repo: https://github.com/JainLab/Manuscript-DNNs-for-Classification-of-LCMS-Peaks
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.9b02983
    title: Deep Neural Networks for Classification of LC-MS Spectral Peaks
  source_content_hash: sha256:3283a63b146b77fb45a4d58f4d8920fd0f30d3e12c1fec84794de83b2c48dace
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/JainLab/Manuscript-DNNs-for-Classification-of-LCMS-Peaks
  tools:
  - EDML_deep_learning2.py
  - MZmine 2
  - R
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 15
---

# lcms-peak-image-rasterization

## Summary

Converts stacked LC-MS peak feature plots into 64x64 pixel raster images (63 rows of per-sample apex intensities scaled to max 1, row 64 encoding retention-time window membership) suitable for training an image-based deep neural network peak classifier.

## When to use

Use when nontargeted LC-MS peak lists from MZmine 2 must be converted into fixed-size 64x64 raster images so a deep neural network can classify candidate peaks as true or false.

## When NOT to use

- Peak data is already stored as an image/raster array (e.g. an existing raster_images.npz) — re-rasterization would duplicate work.
- The goal is peak-parameter-based filtering (e.g. medRelFWHM logistic regression or Random Forest on 59 peak group parameters) rather than image-based deep learning classification.
- Peaks were not extracted with MZmine 2 aligned peak lists and grouped retention-time windows; raw vendor files cannot be rasterized directly by this workflow.

## Inputs

- MZmine 2 aligned peak list (non-filtered peak list extracted with 'More Restrictive' or 'Less Restrictive' peak extraction parameters)
- Grouped peak windows containing m/z and retention-time bounds per candidate feature
- Manual true/false peak labels for training (e.g. 1304 training / 740 calibration / 726 test examples from Cohort 1)

## Outputs

- 64x64 pixel raster images per candidate peak feature (rows 1-63 per-sample scaled intensities, row 64 retention-time window binary flag)
- raster_images.npz archive of all raster images with attached true/false labels

## How to apply

Load the MZmine 2 aligned peak list and the grouped retention-time windows for each candidate feature. For each aligned peak group, extract the m/z and retention-time windows surrounding the group apex and select peak information from the 63 samples in which the candidate peak was most intense. Scale apex intensities so the group maximum equals 1, then rasterize into a 64x64 image where each column is a retention-time point, rows 1-63 hold per-sample intensities, and row 64 encodes the retention-time window boundary (1 inside, 0 outside). Attach each image's true/false peak label and serialize all rasters into raster_images.npz for downstream training with EDML_deep_learning2.py. Correctness is judged by image dimensions (64x64), row 64 binary window flags matching the group's retention-time window, and apex-intensity maxima normalized to 1.

## Related tools

- **MZmine 2** (Produces the non-filtered aligned peak lists and grouped peak windows from raw LC-MS data that serve as the rasterization input)
- **R** (Custom R scripts perform the stacked-peak extraction, apex-intensity normalization, 64x64 rasterization, and raster_images.npz serialization)
- **EDML_deep_learning2.py** (Consumes the serialized raster images to train the deep neural network peak classifier) — https://github.com/JainLab/Manuscript-DNNs-for-Classification-of-LCMS-Peaks

## Examples

```
download datasets from https://doi.org/doi:10.25345/C5FD2F, then run R rasterization scripts to build raster_images.npz and train with: python EDML_deep_learning2.py
```

## Evaluation signals

- Every output image is exactly 64x64 pixels; rows 1-63 contain per-sample intensities and row 64 contains only binary 0/1 window flags.
- Apex intensity maxima in each stacked group are normalized to 1 (no sample row exceeds 1).
- Row 64 value is 1 for all retention-time columns inside the group's retention-time window and 0 outside.
- Number of rasterized images equals the number of candidate peak groups in the aligned MZmine 2 peak list.
- raster_images.npz loads without error and each entry pairs a raster image with a true/false peak label.

## Limitations

- Rasterization assumes exactly 63 'most intense' samples are available per feature; features seen in fewer samples cannot fill all 63 rows as described.
- Apex-intensity scaling to max 1 discards absolute intensity information, so the raster encodes relative peak shape only.
- The 64x64 geometry (63 sample rows + 1 retention-time boundary row) is fixed to this specific workflow and is not generalized to arbitrary cohort sizes or row layouts.

## Evidence

- [results] Each stacked feature was scaled so apex intensity maxima equal 1, then converted into a 64x64 pixel image raster: rows 1-63 give per-sample peak intensity, and row 64 encodes the retention time window boundary: [evidence span withheld — anchor status: not_found]
- [results] Non-filtered peak lists were then generated using MZmine 2 where the raw data was extracted using both "More Restrictive" and "Less Restrictive" peak extraction parameters: [evidence span withheld — anchor status: not_found]
