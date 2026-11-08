---
name: msi-matrix-reshaping-and-shape-validation
description: Use when raw MSI matrices must be exported as a two-dimensional pixel-by-ion
  matrix plus a one-dimensional peak list and the horizontal/vertical pixel shape
  passed to DeepION's --input_shape must be verified.
when_to_use_negative:
- Input is already a validated [X*Y, P] matrix with matching --input_shape dimensions
  — reshaping again is redundant and may corrupt the grid.
- Data are single spectra or profiles (not ion images); no pixel grid exists to validate.
edam_operation: http://edamontology.org/operation_3359
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3520
tools:
- name: SCiLS Lab
  role: Initial peak picking and m/z alignment of the raw rat brain MSI sections before
    Python filtering
- name: Python
  role: Runs peak-filtering and peak-pooling scripts, hotspot truncation, min–max
    normalization, and exports the final [X*Y, P] matrix and [P,1] peak list
- name: DeepION
  role: Consumes the reshaped matrix and peak list via --input_Matrix and --input_PeakList
    with --input_shape; validates downstream dimension compatibility
  repo: https://github.com/gankLei-X/DeepION
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.3c05002
    title: 'DeepION: A Deep Learning-Based Low-Dimensional Representation Model of
      Ion Images for Mass Spectrometry Imaging'
  source_content_hash: sha256:3ec5d1cf4f279adaf148d619509cb5fe8898c830082768909801278567610663
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gankLei-X/DeepION
  tools:
  - DeepION
  - Python
  - SCiLS Lab
  edam_operation: http://edamontology.org/operation_3359
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# msi-matrix-reshaping-and-shape-validation

## Summary

Reshape preprocessed mass spectrometry imaging (MSI) data into the [X*Y, P] pixel-by-ion matrix and [P,1] peak list that DeepION requires, then validate the grid dimensions (e.g. 198×422 positive-mode, 192×425 negative-mode rat brain sections) before running the model.

## When to use

Use when raw MSI matrices must be exported as a two-dimensional pixel-by-ion matrix plus a one-dimensional peak list and the horizontal/vertical pixel shape passed to DeepION's --input_shape must be verified.

## When NOT to use

- Input is already a validated [X*Y, P] matrix with matching --input_shape dimensions — reshaping again is redundant and may corrupt the grid.
- Data are single spectra or profiles (not ion images); no pixel grid exists to validate.

## Inputs

- Raw rat brain section MSI datasets in positive and negative ion modes (SCiLS Lab-importable)
- Python peak-filtering and peak-pooling scripts

## Outputs

- Preprocessed two-dimensional pixel-by-ion matrix of shape [X*Y, P] (e.g. Pos_brain_data_matrix.txt)
- One-dimensional peak list of shape [P,1] (e.g. Pos_brain_data_peak.csv)
- Validation report confirming 2263 ions / 198×422 grid (positive mode) and 2165 ions / 192×425 grid (negative mode)

## How to apply

Import the raw rat brain section MSI datasets into SCiLS Lab and perform peak picking with m/z alignment across pixels to obtain a common peak list per section. Apply Python peak-filtering and peak-pooling scripts to remove low-quality/spurious m/z features and merge equivalent peaks into single ion columns. Truncate hotspot pixels by capping any ion intensity exceeding 99% of that ion's maximum intensity, then normalize each ion image with a min–max scaler to the [0,1] range. Export the preprocessed data as a two-dimensional pixel-by-ion matrix of shape [X*Y, P] and a one-dimensional peak list of shape [P,1] (e.g. Pos_brain_data_matrix.txt / Pos_brain_data_peak.csv for the positive-mode section). Finally verify the positive-mode matrix contains 2263 ions over a 198×422-pixel grid and the negative-mode matrix contains 2165 ions over a 192×425-pixel grid, matching the dimensions passed via --input_shape.

## Related tools

- **SCiLS Lab** (Initial peak picking and m/z alignment of the raw rat brain MSI sections before Python filtering)
- **Python** (Runs peak-filtering and peak-pooling scripts, hotspot truncation, min–max normalization, and exports the final [X*Y, P] matrix and [P,1] peak list)
- **DeepION** (Consumes the reshaped matrix and peak list via --input_Matrix and --input_PeakList with --input_shape; validates downstream dimension compatibility) — https://github.com/gankLei-X/DeepION

## Examples

```
python run.py --input_Matrix .../DATASET/Pos_brain_data_matrix.txt --input_PeakList .../DATASET/Pos_brain_data_peak.csv --input_shape 198 422 --mode COL --ion_mode positive --num 5 --output_file Pos_COL_result
```

## Evaluation signals

- Positive-mode matrix must contain exactly 2263 ion columns and a pixel grid of 198×422 (84,156 pixels).
- Negative-mode matrix must contain exactly 2165 ion columns and a pixel grid of 192×425 (81,600 pixels).
- Peak list file must have shape [P,1] matching the ion count P of the matrix.
- All ion intensity values must fall within the [0,1] range after min–max scaler normalization.
- Any ion intensity exceeding 99% of its maximum must be capped (hotspot truncation) before export.

## Limitations

- Shape validation depends on correctly specifying --input_shape; a mismatch between the exported matrix and the passed grid will cause DeepION to fail or silently misinterpret pixel coordinates.
- The preprocessing chain (SCiLS Lab peak picking → Python filtering/pooling → hotspot truncation at 99% → min–max normalization) is not automatic; incorrect peak pooling merges unrelated m/z features into a single ion column and invalidates the ion count check.

## Evidence

- [methods] Preprocessing and shape validation for DeepION input: [evidence span withheld — anchor status: not_found]
- [readme] Exported matrix format: "The preprocessed MSI matrix data with two-dimensional shape [X*Y,P], where X and Y represent the pixel numbers of horizontal and vertical coordinates of MSI data, and P represents the number of ions."
- [readme] Peak list format: "The preprocessed MSI peak data with one-dimensional shape [P,1], where P represents the number of ions."
- [readme] Positive-mode CLI example: "python run.py --input_Matrix .../DATASET/Pos_brain_data_matrix.txt --input_PeakList .../DATASET/Pos_brain_data_peak.csv --input_shape 198 422 --mode COL --ion_mode positive --num 5 --output_file"
- [results] Dataset dimension confirmation: [evidence span withheld — anchor status: not_found]
