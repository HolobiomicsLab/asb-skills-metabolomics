---
name: retention-time-window-encoding
description: Use when converting MZmine 2 aligned peak groups (m/z and retention-time
  windows plus per-sample peak shapes) into the standardized 64x64 pixel raster images
  that feed the image-based deep neural network for true/false peak classification.
when_to_use_negative:
- Input is already a numerical feature table of peak group parameters (e.g., the 59
  peak group parameters used for the Random Forest or logistic regression models)
  — those models consume parameter vectors, not raster images.
- Raw, unaligned LC-MS vendor files (e.g., mzML/NetCDF) without an MZmine 2 peak list
  — the raster construction depends on aligned peak groups and retention-time windows,
  not on raw chromatograms.
- Peak groups with fewer than 63 informative samples or without a defined retention-time
  window from the grouping step — the fixed 64x64 raster layout cannot be filled correctly.
edam_operation: http://edamontology.org/operation_3557
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3382
tools:
- name: MZmine 2
  role: Produces the aligned peak lists and grouped m/z / retention-time windows from
    raw LC-MS data that serve as the input to the raster construction
  repo: https://doi.org/doi:10.25345/C5FD2F
- name: R
  role: Custom R scripts used to scale apex intensities, rasterize the stacked feature
    into the 64x64 image, and write raster_images.npz
- name: EDML_deep_learning2.py
  role: Consumes the raster_images.npz file produced by this skill to train the deep
    neural network model
  repo: https://github.com/JainLab/Manuscript-DNNs-for-Classification-of-LCMS-Peaks
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.9b02983
    title: Deep Neural Networks for Classification of LC-MS Spectral Peaks
  source_content_hash: sha256:ac16b386b2c595622ca55fc37ffc353120e042ea3205219f8fc3eaa731b429d3
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - EDML_deep_learning2.py
  - MZmine 2
  - R
  edam_operation: http://edamontology.org/operation_3557
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3382
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# retention-time-window-encoding

## Summary

Encodes an aligned LC-MS peak group into a 64x64 raster image whose rows 1–63 carry per-sample apex intensities (scaled so the maximum equals 1) and row 64 encodes the retention-time window boundary (1 inside, 0 outside), producing the image input required by the deep neural network peak classifier.

## When to use

Use when converting MZmine 2 aligned peak groups (m/z and retention-time windows plus per-sample peak shapes) into the standardized 64x64 pixel raster images that feed the image-based deep neural network for true/false peak classification.

## When NOT to use

- Input is already a numerical feature table of peak group parameters (e.g., the 59 peak group parameters used for the Random Forest or logistic regression models) — those models consume parameter vectors, not raster images.
- Raw, unaligned LC-MS vendor files (e.g., mzML/NetCDF) without an MZmine 2 peak list — the raster construction depends on aligned peak groups and retention-time windows, not on raw chromatograms.
- Peak groups with fewer than 63 informative samples or without a defined retention-time window from the grouping step — the fixed 64x64 raster layout cannot be filled correctly.

## Inputs

- MZmine 2 aligned peak list (non-filtered peak lists generated from raw LC-MS data using 'More Restrictive' and 'Less Restrictive' peak extraction parameters)
- Grouped peak windows per candidate feature (m/z and retention-time windows surrounding the group apex)
- Per-sample peak shape/intensity information for the 63 samples in which the candidate peak was most intense
- True/false peak labels for each candidate peak group (e.g., from the 1304 manually reviewed training examples)

## Outputs

- 64x64 pixel raster images per candidate peak group (rows 1–63 = scaled per-sample intensities, row 64 = retention-time window boundary)
- raster_images.npz containing all raster images with attached true/false peak labels

## How to apply

Start from the MZmine 2 aligned peak list and the grouped peak windows for each candidate feature, extracting the m/z and retention-time windows surrounding each group apex. Select peak information from the 63 samples in which the candidate peak was most intense and scale each apex intensity so the maxima equal 1, then rasterize the stacked feature so each column is a retention-time point, rows 1–63 hold per-sample intensities, and row 64 marks the retention-time window boundary (1 inside the window, 0 outside). Attach each image's true/false peak label and serialize all rasters into raster_images.npz for downstream training with the EDML_deep_learning2.py script. The retention-time boundary row is the paper's explicit mechanism for telling the network where the extracted feature window begins and ends, so window definitions must come directly from the MZmine 2 grouping rather than being recomputed independently.

## Related tools

- **MZmine 2** (Produces the aligned peak lists and grouped m/z / retention-time windows from raw LC-MS data that serve as the input to the raster construction) — https://doi.org/doi:10.25345/C5FD2F
- **R** (Custom R scripts used to scale apex intensities, rasterize the stacked feature into the 64x64 image, and write raster_images.npz)
- **EDML_deep_learning2.py** (Consumes the raster_images.npz file produced by this skill to train the deep neural network model) — https://github.com/JainLab/Manuscript-DNNs-for-Classification-of-LCMS-Peaks

## Examples

```
Rscript EDML_rasterize.R --input aligned_peak_list.csv --window mzrt_windows.csv --output raster_images.npz
```

## Evaluation signals

- Output array has shape (N, 64, 64) — exactly 64 rows and 64 columns per raster, matching the paper's specified image size.
- Rows 1–63 apex intensities are normalized so the maximum value across the group equals 1 (no entry exceeds 1, and at least one equals 1).
- Row 64 contains only binary values (1 inside the retention-time window, 0 outside), and the 1-values align with the retention-time window columns derived from the MZmine 2 grouping.
- Each raster's total column count corresponds to the retention-time window's sample points (each column represents a different retention-time point), so no column is blank or duplicated.
- raster_images.npz loads without error and every raster carries a true/false peak label consistent with the manually reviewed training set (50% true / 50% false for the 1304 training examples).

## Limitations

- The encoding assumes 63 informative samples are available per peak; peak groups with fewer intense samples cannot fill rows 1–63 as specified.
- The retention-time window boundary in row 64 must be taken from the MZmine 2 grouping; recomputing it independently risks mislabeling the window the network was trained on.
- Intensities must be apex maxima scaled to 1 — using raw (unscaled) intensities or base-peak-normalized values would change the distribution the DNN expects and degrade classification (reported AUC 0.992 training / 0.988 test would not hold).

## Evidence

- [results] rows 1-63 give per-sample peak intensity, and row 64 encodes the retention time window boundary (1 inside window, 0 outside): [evidence span withheld — anchor status: not_found]
- [results] scale apex intensities to a maximum of 1: [evidence span withheld — anchor status: not_found]
- [results] each column is a retention-time point, rows 1-63 hold per-sample intensities, and row 64 encodes the retention-time window boundary: [evidence span withheld — anchor status: not_found]
- [results] Attach each image's true/false peak label and serialize all raster images into raster_images.npz: [evidence span withheld — anchor status: not_found]
