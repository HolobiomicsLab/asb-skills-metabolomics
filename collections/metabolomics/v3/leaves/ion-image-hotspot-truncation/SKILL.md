---
name: ion-image-hotspot-truncation
description: Use when raw MALDI-MSI pixel-by-ion matrices contain extreme intensity
  spikes (matrix crystal hotspots) that must be capped before self-supervised ion-image
  representation learning or any downstream similarity measurement.
when_to_use_negative:
- Data already exported from SCiLS Lab with hotspot truncation and [0,1] normalization
  applied — re-capping would double-normalize and distort intensity distributions
- Single-ion spectra or profile-mode (not peak-picked) data where no pixel-by-ion
  matrix exists yet — peak picking must be completed first
- Datasets where no extreme intensity spikes are present and the 99% cap would clip
  legitimate biological signal
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: SCiLS Lab
  role: Used to import raw MALDI-MSI sections, perform peak picking with m/z alignment,
    and produce the pixel-by-ion matrix prior to hotspot truncation
- name: Python
  role: Used for peak filtering/pooling scripts and for applying the 99% hotspot cap
    and min–max normalization
- name: DeepION
  role: Downstream consumer of the truncated and normalized matrix; expects input_matrix.txt
    and peak_list.csv at specified shapes
  repo: https://github.com/gankLei-X/DeepION
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.3c05002
    title: 'DeepION: A Deep Learning-Based Low-Dimensional Representation Model of
      Ion Images for Mass Spectrometry Imaging'
  source_content_hash: sha256:a492572a74d3bdf59a456ac7f11ed5d3743ad35616ed238450d69e74687ebd5d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gankLei-X/DeepION
  tools:
  - DeepION
  - Python
  - SCiLS Lab
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# ion-image-hotspot-truncation

## Summary

Preprocessing step for MALDI mass spectrometry imaging data in which ion intensities exceeding the 99th-percentile cap are truncated to suppress matrix-crystal hotspots, followed by min–max normalization to [0, 1], yielding clean pixel-by-ion matrices ready for DeepION representation learning.

## When to use

Use when raw MALDI-MSI pixel-by-ion matrices contain extreme intensity spikes (matrix crystal hotspots) that must be capped before self-supervised ion-image representation learning or any downstream similarity measurement.

## When NOT to use

- Data already exported from SCiLS Lab with hotspot truncation and [0,1] normalization applied — re-capping would double-normalize and distort intensity distributions
- Single-ion spectra or profile-mode (not peak-picked) data where no pixel-by-ion matrix exists yet — peak picking must be completed first
- Datasets where no extreme intensity spikes are present and the 99% cap would clip legitimate biological signal

## Inputs

- Raw MALDI mass spectrometry imaging dataset (rat brain tissue sections) imported into SCiLS Lab
- Peak-picked and m/z-aligned peak list per section
- Pixel-by-ion intensity matrix with shape [X*Y, P] (e.g. Pos_brain_data_matrix.txt)

## Outputs

- Hotspot-truncated ion intensity matrix where values >99% of an ion's maximum are capped
- Min–max normalized matrix with all features in [0, 1]
- One-dimensional peak list of shape [P,1] (e.g. Pos_brain_data_peak.csv)
- Validated dimensions: 2263 ions × 198×422 pixels (positive mode) and 2165 ions × 192×425 pixels (negative mode)

## How to apply

Import the raw rat brain section MSI data into SCiLS Lab, pick m/z peaks with alignment across pixels to build a common peak list per section, then export the pixel-by-ion matrix of shape [X*Y, P]. Apply hotspot truncation by capping any ion intensity that exceeds 99% of that ion's maximum intensity, which suppresses MALDI matrix-crystal hotspots that would otherwise dominate the contrastive-learning augmentation. Follow with a min–max scaler to rescale every feature to the [0, 1] range, as DeepION expects normalized ion images. The output matrix must retain the exact pixel grid passed via --input_shape (e.g. 198×422 positive mode, 192×425 negative mode) and the exact ion count (2263 positive, 2165 negative), so no pixels or ions are dropped during capping. Validation is a dimension check on the exported matrix plus a peak list of shape [P,1] before running DeepION in COL or ISO mode.

## Related tools

- **SCiLS Lab** (Used to import raw MALDI-MSI sections, perform peak picking with m/z alignment, and produce the pixel-by-ion matrix prior to hotspot truncation)
- **Python** (Used for peak filtering/pooling scripts and for applying the 99% hotspot cap and min–max normalization)
- **DeepION** (Downstream consumer of the truncated and normalized matrix; expects input_matrix.txt and peak_list.csv at specified shapes) — https://github.com/gankLei-X/DeepION

## Examples

```
python run.py --input_Matrix .../DATASET/Pos_brain_data_matrix.txt --input_PeakList .../DATASET/Pos_brain_data_peak.csv --input_shape 198 422 --mode COL --ion_mode positive --num 5 --output_file Pos_COL_result
```

## Evaluation signals

- Exported matrix dimensions match the expected ion counts and pixel grid: 2263 ions / 198×422 (positive) and 2165 ions / 192×425 (negative)
- Peak list file has exactly P rows (shape [P,1]) matching the ion count of the matrix
- No ion image value exceeds 1.0 after min–max scaling; capping removed values that previously exceeded 99% of each ion's max
- Downstream DeepION run (--input_shape 198 422) consumes the matrix without a shape mismatch error

## Limitations

- Capping at 99% of maximum intensity assumes the extreme values are matrix-crystal artifacts; if genuine high-abundance spatial features exist they will be clipped
- The threshold (99%) and normalization scheme are specific to DeepION's expected input distribution and may not generalize to other MSI representation models
- Requires SCiLS Lab peak picking and alignment as a prerequisite; cannot be applied to raw profile spectra directly

## Evidence

- [methods] After preprocessing (hotspot truncation at 99% max intensity and [0,1] normalization), two MSI datasets are obtained: 2165 ion images of shape 192×425 in negative mode and 2263 ion images of shape 198×422 in positive mode.: [evidence span withheld — anchor status: not_found]
- [readme] The preprocessed MSI matrix data with two-dimensional shape [X*Y,P]: "The preprocessed MSI matrix data with two-dimensional shape [X*Y,P], where X and Y represent the pixel numbers of horizontal and vertical coordinates of MSI data, and P represents the number of ions."
- [methods] a min–max scaler is adopted to adjust the range of features to [0, 1]: "a min–max scaler is adopted to adjust the range of features to [0, 1]"
- [readme] python run.py --input_Matrix .../DATASET/Pos_brain_data_matrix.txt --input_PeakList .../DATASET/Pos_brain_data_peak.csv --input_shape 198 422 --mode COL --ion_mode positive --num 5 --output_file Pos_COL_result: "python run.py --input_Matrix .../DATASET/Pos_brain_data_matrix.txt --input_PeakList .../DATASET/Pos_brain_data_peak.csv --input_shape 198 422 --mode COL --ion_mode positive --num 5 --output_file"
