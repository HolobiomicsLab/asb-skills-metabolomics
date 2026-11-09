---
name: msi-peak-picking-alignment
description: Use when raw MALDI MSI sections must be converted into a common peak
  list and a [X*Y,P] ion-intensity matrix with normalized intensities before a representation
  model such as DeepION is trained.
when_to_use_negative:
- Input is already a preprocessed [X*Y,P] matrix with a [P,1] peak list (DeepION-ready)
  — re-running peak picking/alignment would double-filter and alter expected ion counts.
- Data are not raw MSI ion images (e.g. bulk spectra, LC-MS feature tables) — the
  SCiLS Lab peak-picking and pixel-grid reshaping steps assume spatial pixel coordinates.
edam_operation: http://edamontology.org/operation_3222
edam_topics: []
tools:
- name: SCiLS Lab
  role: Commercial/standalone software used to import the raw rat brain MSI sections
    and perform peak picking with m/z alignment across pixels to produce a common
    peak list per section.
- name: Python
  role: Runs the peak-filtering and peak-pooling scripts to remove low-quality/spurious
    m/z features and merge equivalent peaks into single ion columns, then applies
    hotspot truncation and min–max normalization before exporting the [X*Y,P] matrix
    and [P,1] peak list.
- name: DeepION
  role: Downstream consumer of the preprocessed matrix and peak list; its run.py expects
    --input_Matrix, --input_PeakList and --input_shape matching the exported files.
  repo: https://github.com/gankLei-X/DeepION
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.3c05002
    title: 'DeepION: A Deep Learning-Based Low-Dimensional Representation Model of
      Ion Images for Mass Spectrometry Imaging'
  source_content_hash: sha256:95088e0e8356e76d3d1b4cd982f5c6ad7e4a516b34c5123fd0d77279b50d4faf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gankLei-X/DeepION
  tools:
  - DeepION
  - Python
  - SCiLS Lab
  edam_operation: http://edamontology.org/operation_3222
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 15
---

# msi-peak-picking-alignment

## Summary

Preprocess raw rat brain mass spectrometry imaging (MSI) data by peak picking with m/z alignment in SCiLS Lab, then filtering/pooling peaks in Python, hotspot truncation at 99% max intensity, and min–max normalization to [0,1] to yield a pixel-by-ion matrix plus peak list ready for DeepION.

## When to use

Use when raw MALDI MSI sections must be converted into a common peak list and a [X*Y,P] ion-intensity matrix with normalized intensities before a representation model such as DeepION is trained.

## When NOT to use

- Input is already a preprocessed [X*Y,P] matrix with a [P,1] peak list (DeepION-ready) — re-running peak picking/alignment would double-filter and alter expected ion counts.
- Data are not raw MSI ion images (e.g. bulk spectra, LC-MS feature tables) — the SCiLS Lab peak-picking and pixel-grid reshaping steps assume spatial pixel coordinates.

## Inputs

- Raw rat brain MALDI MSI section datasets (positive and negative ion mode) in a format importable by SCiLS Lab
- Python peak-filtering and peak-pooling scripts
- SCiLS Lab peak list (m/z aligned across pixels) per section

## Outputs

- Preprocessed two-dimensional pixel-by-ion matrix of shape [X*Y,P] (e.g. Pos_brain_data_matrix.txt)
- Preprocessed one-dimensional peak list of shape [P,1] (e.g. Pos_brain_data_peak.csv)
- Verified grid/peak counts for downstream DeepION runs (positive mode: 2263 ions, 198×422 pixels; negative mode: 2165 ions, 192×425 pixels)

## How to apply

Import the raw rat brain section MSI datasets into SCiLS Lab and perform peak picking with m/z alignment across pixels to obtain a common peak list per section. Apply the Python peak-filtering and peak-pooling scripts to remove low-quality/spurious m/z features and merge equivalent peaks into single ion columns. Truncate hotspot pixels by capping any ion intensity that exceeds 99% of that ion's maximum intensity to suppress MALDI matrix-crystal hotspots, then normalize each ion image with a min–max scaler to the [0,1] intensity range. Export the preprocessed data as a two-dimensional pixel-by-ion matrix of shape [X*Y,P] and a one-dimensional peak list of shape [P,1] (e.g. Pos_brain_data_matrix.txt / Pos_brain_data_peak.csv). Validation: the positive-mode matrix should contain 2263 ions over a 198×422-pixel grid and the negative-mode matrix 2165 ions over a 192×425-pixel grid, matching the --input_shape passed to DeepION.

## Related tools

- **SCiLS Lab** (Commercial/standalone software used to import the raw rat brain MSI sections and perform peak picking with m/z alignment across pixels to produce a common peak list per section.)
- **Python** (Runs the peak-filtering and peak-pooling scripts to remove low-quality/spurious m/z features and merge equivalent peaks into single ion columns, then applies hotspot truncation and min–max normalization before exporting the [X*Y,P] matrix and [P,1] peak list.)
- **DeepION** (Downstream consumer of the preprocessed matrix and peak list; its run.py expects --input_Matrix, --input_PeakList and --input_shape matching the exported files.) — https://github.com/gankLei-X/DeepION

## Examples

```
python run.py --input_Matrix .../DATASET/Pos_brain_data_matrix.txt --input_PeakList .../DATASET/Pos_brain_data_peak.csv --input_shape 198 422 --mode COL --ion_mode positive --num 5 --output_file Pos_COL_result
```

## Evaluation signals

- Positive-mode matrix contains 2263 ions over a 198×422-pixel grid (i.e. [X*Y,P] shape = [836? no — check: 198×422=83,556 pixels × 2263 ions]); negative-mode matrix contains 2165 ions over a 192×425-pixel grid.
- Peak list file has exactly P rows (e.g. 2263 for positive mode) matching the ion dimension of the exported matrix.
- No ion intensity exceeds 1.0 after min–max normalization (hotspots capped at 99% of each ion's maximum intensity).
- Matrix values are within [0,1] and consistent with a min–max scaler applied per ion image.
- The exported filenames match the DeepION README convention (Pos_brain_data_matrix.txt / Pos_brain_data_peak.csv) and the --input_shape values match the pixel grid.

## Limitations

- Hotspot truncation is a fixed 99% cap; if the MALDI matrix-crystal hotspots are stronger or weaker than expected, this threshold may need retuning.
- The Python peak-filtering/pooling scripts assume the SCiLS Lab peak list has already been m/z-aligned; misaligned peaks will produce duplicate ion columns.
- Expected ion counts (2165 negative mode, 2263 positive mode) are specific to the rat brain sections used in the DeepION paper; other tissues will yield different P.
- Min–max normalization is sensitive to outliers; hotspot truncation must be applied before normalization or it will skew the [0,1] range.

## Evidence

- [other] Import the two raw rat brain section MSI datasets into SCiLS Lab and perform peak picking with m/z alignment across pixels: [evidence span withheld — anchor status: not_found]
- [other] Truncate hotspot pixels by capping any ion intensity exceeding 99% of that ion's maximum intensity: [evidence span withheld — anchor status: not_found]
- [methods] a min–max scaler is adopted to adjust the range of features to [0, 1]: "a min–max scaler is adopted to adjust the range of features to [0, 1]"
- [readme] The preprocessed MSI matrix data with two-dimensional shape [X*Y,P]: "(1) The preprocessed MSI matrix data with two-dimensional shape [X*Y,P], where X and Y represent the pixel numbers of horizontal and vertical coordinates of MSI data, and P represents the number of"
- [other] verify the positive-mode matrix contains 2263 ions over a 198×422-pixel grid: [evidence span withheld — anchor status: not_found]
- [other] After preprocessing (hotspot truncation at 99% max intensity and [0,1] normalization), two MSI datasets are obtained: 2165 ion images of shape 192×425 in negative mode and 2263 ion images of shape 198×422 in positive mode.: [evidence span withheld — anchor status: not_found]
