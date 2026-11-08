---
name: msi-peak-filtering-pooling
description: Use when raw MALDI-MSI section data (e.g. rat brain, positive or negative
  ion mode) must be converted into a validated pixel-by-ion intensity matrix with
  a paired peak list before running DeepION COL/ISO or similar ion-image representation
  models.
when_to_use_negative:
- Input is already a preprocessed pixel-by-ion matrix with hotspot truncation and
  [0, 1] normalization applied (e.g. the DATASET/Pos_brain_data_matrix.txt DeepION
  expects) — re-filtering would distort intensities.
- The data are non-MSI spectral data or already-processed feature tables (e.g. labelled
  feature matrices for supervised classifiers) where SCiLS Lab peak picking and peak
  pooling have no meaning.
- Only a single ion image or sub-region is of interest without a full pixel-by-peak
  matrix; peak pooling across the section requires the complete SCiLS peak list.
edam_operation: http://edamontology.org/operation_3694
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: DeepION
  role: Downstream model whose expected [X*Y, P] input shape, [P, 1] peak list, and
    --input_shape pixel grid define the exact preprocessing output this skill must
    produce
  repo: https://github.com/BioNet-XMU/DeepION
- name: SCiLS Lab
  role: Import of the two raw rat brain section MSI datasets and peak picking with
    m/z alignment across pixels to obtain a common peak list per section
- name: Python
  role: Peak-filtering and peak-pooling scripts that remove low-quality/spurious m/z
    features and merge equivalent peaks into single ion columns; DeepION itself is
    implemented in Python with PyTorch
  repo: https://github.com/BioNet-XMU/DeepION
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.3c05002
    title: 'DeepION: A Deep Learning-Based Low-Dimensional Representation Model of
      Ion Images for Mass Spectrometry Imaging'
  source_content_hash: sha256:55d9c9aaa101fb01837b744a4cdf13c1befcd2173bdcd1bc30c2678eb272902f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BioNet-XMU/DeepION
  tools:
  - DeepION
  - Python
  - SCiLS Lab
  edam_operation: http://edamontology.org/operation_3694
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# msi-peak-filtering-pooling

## Summary

Preprocess raw rat-brain MALDI mass spectrometry imaging data into a clean pixel-by-ion matrix: pick and align m/z peaks across pixels (SCiLS Lab), filter and pool spurious peaks with Python scripts, truncate hotspot pixels at 99% of each ion's maximum intensity, and min–max normalize each ion image to [0, 1]. This yields exactly the [X*Y, P] matrices DeepION consumes (2165 ions, 192×425 negative mode; 2263 ions, 198×422 positive mode).

## When to use

Use when raw MALDI-MSI section data (e.g. rat brain, positive or negative ion mode) must be converted into a validated pixel-by-ion intensity matrix with a paired peak list before running DeepION COL/ISO or similar ion-image representation models.

## When NOT to use

- Input is already a preprocessed pixel-by-ion matrix with hotspot truncation and [0, 1] normalization applied (e.g. the DATASET/Pos_brain_data_matrix.txt DeepION expects) — re-filtering would distort intensities.
- The data are non-MSI spectral data or already-processed feature tables (e.g. labelled feature matrices for supervised classifiers) where SCiLS Lab peak picking and peak pooling have no meaning.
- Only a single ion image or sub-region is of interest without a full pixel-by-peak matrix; peak pooling across the section requires the complete SCiLS peak list.

## Inputs

- Raw MALDI mass spectrometry imaging datasets of rat brain tissue sections (positive and negative ion mode)
- Peak-picked/aligned peak lists from SCiLS Lab per section
- Target ion-mode label ("positive" or "negative") and pixel grid dimensions (e.g. 198×422)

## Outputs

- Preprocessed pixel-by-ion matrix file (e.g. DATASET/Pos_brain_data_matrix.txt, shape [X*Y, P])
- Preprocessed peak list file (e.g. DATASET/Pos_brain_data_peak.csv, shape [P, 1])
- Validated ion counts and pixel dimensions (2165 ions, 192×425 negative mode; 2263 ions, 198×422 positive mode) recorded in a preprocessing validation report

## How to apply

Import the two raw rat brain section MSI datasets into SCiLS Lab and perform peak picking with m/z alignment across pixels to obtain a common peak list per section. Apply Python peak-filtering and peak-pooling scripts to remove low-quality/spurious m/z features and merge equivalent peaks into single ion columns, producing the Pos/Neg_brain_data_matrix.txt matrices and _peak.csv peak lists. Cap any ion intensity exceeding 99% of that ion's maximum intensity to suppress MALDI matrix-crystal hotspots, then normalize each ion image with a min–max scaler to the [0, 1] range, as DeepION's preprocessing does. Export as a two-dimensional [X*Y, P] matrix and a one-dimensional [P, 1] peak list and pass the true pixel grid via --input_shape (e.g. 198 422 for positive mode). Validation: confirm the positive-mode matrix contains 2263 ions over a 198×422-pixel grid and the negative-mode matrix contains 2165 ions over a 192×425-pixel grid.

## Related tools

- **DeepION** (Downstream model whose expected [X*Y, P] input shape, [P, 1] peak list, and --input_shape pixel grid define the exact preprocessing output this skill must produce) — https://github.com/BioNet-XMU/DeepION
- **SCiLS Lab** (Import of the two raw rat brain section MSI datasets and peak picking with m/z alignment across pixels to obtain a common peak list per section)
- **Python** (Peak-filtering and peak-pooling scripts that remove low-quality/spurious m/z features and merge equivalent peaks into single ion columns; DeepION itself is implemented in Python with PyTorch) — https://github.com/BioNet-XMU/DeepION

## Examples

```
python run.py --input_Matrix .../DATASET/Pos_brain_data_matrix.txt --input_PeakList .../DATASET/Pos_brain_data_peak.csv --input_shape 198 422 --mode COL --ion_mode positive --num 5 --output_file Pos_COL_result
```

## Evaluation signals

- Positive-mode matrix has exactly 2263 ion columns spanning a 198×422 pixel grid; negative-mode matrix has exactly 2165 ions spanning 192×425.
- Matrix shape equals [X*Y, P] and peak list shape equals [P, 1]; --input_shape passed to DeepION matches the true grid.
- All ion intensities fall within [0, 1] after min–max scaling; no intensity exceeds 99% of that ion's maximum (hotspot truncation applied).
- Exported filenames follow the DeepION convention (e.g. Pos_brain_data_matrix.txt / Pos_brain_data_peak.csv) and the peak list dimension P matches the matrix column count.

## Limitations

- Peak counts and pixel grids are section- and ion-mode-specific (2165×192×425 negative, 2263×198×422 positive); other tissues or acquisition settings require re-deriving and re-validating these numbers.
- The README lists only runtime requirements and the CLI invocation — it does not document the specific filtering/pooling criteria or thresholds used to clean the raw SCiLS peak list, so the exact cutoffs must be re-derived from the supplied scripts.
- The [0,1] min–max scaling is applied per ion image; it changes absolute intensities and is specific to DeepION's expected input, not a general-purpose MSI normalization.
- Only Python 3.5–3.7 with scikit-learn 0.19.1, PyTorch 1.8.2, numpy ≥ 1.8.0, umap 0.5.1, torchvision 0.11.3, kornia 0.4.1 and boly_pytorch are documented as supported.

## Evidence

- [other] After preprocessing (hotspot truncation at 99% max intensity and [0,1] normalization), two MSI datasets are obtained: 2165 ion images of shape 192×425 in negative mode and 2263 ion images of shape 198×422 in positive mode.: [evidence span withheld — anchor status: not_found]
- [methods] Hotspot truncation and [0,1] min-max normalization rationale: [evidence span withheld — anchor status: not_found]
- [other] Python peak filtering/pooling on SCiLS peak lists: [evidence span withheld — anchor status: not_found]
- [other] SCiLS Lab peak picking and m/z alignment: [evidence span withheld — anchor status: not_found]
- [readme] DeepION expected input matrix/peak-list file names and CLI shape arguments: "The preprocessed MSI matrix data with two-dimensional shape [X*Y,P], where X and Y represent the pixel numbers of horizontal and vertical coordinates of MSI data, and P represents the number of ions."
- [readme] DeepION peak list shape: "The preprocessed MSI peak data with one-dimensional shape [P,1], where P represents the number of ions."
- [readme] DeepION positive-mode example invocation with --input_shape 198 422: "python run.py --input_Matrix .../DATASET/Pos_brain_data_matrix.txt --input_PeakList .../DATASET/Pos_brain_data_peak.csv --input_shape 198 422 --mode COL --ion_mode positive --num 5 --output_file"
- [methods] Min-max scaler to [0, 1] stated in the article's methods: "a min–max scaler is adopted to adjust the range of features to [0, 1]"
- [intro] DeepION rat brain datasets used to assess colocalization: "The MSI data sets from two sections of rat brain tissues are used to assess the capability of DeepION on ion colocalization in comparison with SIM-based methods"
