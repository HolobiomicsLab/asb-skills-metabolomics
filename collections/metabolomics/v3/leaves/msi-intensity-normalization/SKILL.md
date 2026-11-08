---
name: msi-intensity-normalization
description: Use when raw MSI data (e.g., rat brain sections, MALDI) must be converted
  into preprocessed 2D pixel×ion intensity matrices—e.g., before DeepION representation
  learning or other ion-colocalization tasks requiring uniform [0,1] intensities.
when_to_use_negative:
- Input is already a normalized pixel×ion feature table (e.g., a DeepION-formatted
  matrix)—re-applying hotspot truncation or min–max scaling would corrupt downstream
  similarity computations.
- The analysis goal is raw spectral peak annotation or isotope assignment rather than
  generating uniform ion images; in that case other preprocessing workflows (e.g.,
  SCiLS Lab peak annotation) may suffice.
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: SCiLS Lab
  role: Commercial/vendor software used to import raw rat brain MSI data and perform
    peak picking with m/z alignment across pixels before peak filtering/pooling.
- name: Python
  role: Runs peak-filtering, peak-pooling, hotspot truncation, and min–max normalization
    scripts; exports the preprocessed matrix and peak list files.
  repo: https://github.com/gankLei-X/DeepION
- name: DeepION
  role: Downstream consumer of the preprocessed matrix and peak list; requires them
    as --input_Matrix and --input_PeakList arguments in COL or ISO mode.
  repo: https://github.com/gankLei-X/DeepION
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.3c05002
    title: 'DeepION: A Deep Learning-Based Low-Dimensional Representation Model of
      Ion Images for Mass Spectrometry Imaging'
  source_content_hash: sha256:b24c4fef8f9e2c68aed81e6126b8e85a9ef218859ac6a4882d838228caf0055e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gankLei-X/DeepION
  tools:
  - DeepION
  - Python
  - SCiLS Lab
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# msi-intensity-normalization

## Summary

Preprocesses mass spectrometry imaging (MSI) ion images from raw rat brain tissue sections via hotspot truncation and min–max normalization, producing a pixel-by-ion matrix in [X*Y, P] shape plus a peak list [P,1] suitable for DeepION's COL/ISO contrastive-learning input.

## When to use

Use when raw MSI data (e.g., rat brain sections, MALDI) must be converted into preprocessed 2D pixel×ion intensity matrices—e.g., before DeepION representation learning or other ion-colocalization tasks requiring uniform [0,1] intensities.

## When NOT to use

- Input is already a normalized pixel×ion feature table (e.g., a DeepION-formatted matrix)—re-applying hotspot truncation or min–max scaling would corrupt downstream similarity computations.
- The analysis goal is raw spectral peak annotation or isotope assignment rather than generating uniform ion images; in that case other preprocessing workflows (e.g., SCiLS Lab peak annotation) may suffice.

## Inputs

- Raw MSI dataset from a rat brain tissue section (positive or negative ion mode, MALDI)
- Pixel grid dimensions (X, Y) of the tissue section, e.g., 198×422 for positive mode
- Peak list after SCiLS Lab peak picking and m/z alignment

## Outputs

- Preprocessed pixel-by-ion intensity matrix of shape [X*Y, P], all values in [0,1] (e.g., Pos_brain_data_matrix.txt / Neg_brain_data_matrix.txt)
- One-dimensional peak list of shape [P,1] (e.g., Pos_brain_data_peak.csv)
- Resulting ion image counts, e.g., 2263 ions at 198×422 pixels (positive mode) and 2165 ions at 192×425 pixels (negative mode)

## How to apply

Import the raw MSI data into SCiLS Lab, perform peak picking with m/z alignment across pixels to build a common peak list per section, then run Python peak-filtering and pooling scripts to remove low-quality m/z features and merge equivalent peaks into single ion columns. Truncate hotspot pixels by capping any ion intensity above 99% of that ion's maximum to suppress MALDI matrix-crystal hotspots, then apply a min–max scaler to rescale every ion image to [0,1]. Export the result as a two-dimensional pixel-by-ion matrix of shape [X*Y, P] (e.g., Pos_brain_data_matrix.txt) and a one-dimensional peak list of shape [P,1] (e.g., Pos_brain_data_peak.csv), passing the pixel dimensions via --input_shape (e.g., 198 422 for the positive-mode rat brain section).

## Related tools

- **SCiLS Lab** (Commercial/vendor software used to import raw rat brain MSI data and perform peak picking with m/z alignment across pixels before peak filtering/pooling.)
- **Python** (Runs peak-filtering, peak-pooling, hotspot truncation, and min–max normalization scripts; exports the preprocessed matrix and peak list files.) — https://github.com/gankLei-X/DeepION
- **DeepION** (Downstream consumer of the preprocessed matrix and peak list; requires them as --input_Matrix and --input_PeakList arguments in COL or ISO mode.) — https://github.com/gankLei-X/DeepION

## Examples

```
python run.py --input_Matrix .../DATASET/Pos_brain_data_matrix.txt --input_PeakList .../DATASET/Pos_brain_data_peak.csv --input_shape 198 422 --mode COL --ion_mode positive --num 5 --output_file Pos_COL_result
```

## Evaluation signals

- Output matrix shape matches [X*Y, P] where X*Y equals the pixel grid passed via --input_shape (e.g., 198×422 = 83,556 rows).
- Every intensity value lies in [0,1] after min–max scaling; any value >1 indicates the hotspot truncation step (99% cap) was skipped or misapplied.
- Peak list length P equals the number of columns in the output matrix; for the positive-mode rat brain dataset P should be 2263, for the negative-mode rat brain dataset P should be 2165.
- No ion column retains values above 99% of its original maximum unless they were truncated—spot-check a few ions against their raw maxima.
- File naming conventions match the article (Pos_brain_data_matrix.txt / Pos_brain_data_peak.csv) and downstream DeepION runs without shape errors.

## Limitations

- Hotspot truncation at the 99% of-max threshold is a heuristic to suppress MALDI matrix-crystal artifacts; it may clip genuine ultra-high-intensity pixels, so the choice of percentile is a trade-off between noise removal and information loss.
- Peak picking and pooling quality depend on the SCiLS Lab peak alignment; low-quality/spurious m/z features must be manually filtered via the Python scripts.
- The pipeline was demonstrated on two rat brain sections (positive and negative ion mode); adapting to other tissues or ionization methods may require recalibrating peak-filtering thresholds.

## Evidence

- [methods] Hotspot truncation at 99% max intensity and [0,1] normalization; two MSI datasets obtained: 2165 ion images of shape 192×425 in negative mode and 2263 ion images of shape 198×422 in positive mode.: [evidence span withheld — anchor status: not_found]
- [methods] Min–max scaler normalization to [0,1].: "a min–max scaler is adopted to adjust the range of features to [0, 1]"
- [methods] Python peak filtering/pooling, hotspot truncation, normalization, and export of matrix/peak list with concrete file names.: [evidence span withheld — anchor status: not_found]
- [readme] Input matrix format [X*Y,P] and peak list format [P,1] as DeepION's required preprocessed inputs.: "The preprocessed MSI matrix data with two-dimensional shape [X*Y,P], where X and Y represent the pixel numbers of horizontal and vertical coordinates of MSI data, and P represents the number of ions."
- [readme] Concrete DeepION run command with --input_shape and matrix/peak file paths, showing the output of this preprocessing skill in action.: "python run.py --input_Matrix .../DATASET/Pos_brain_data_matrix.txt --input_PeakList .../DATASET/Pos_brain_data_peak.csv --input_shape 198 422 --mode COL --ion_mode positive --num 5 --output_file"
