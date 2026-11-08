---
name: spectral-intensity-normalization
description: Use when raw 1H NMR (ppm, intensity) spectra must be converted into the
  model-input representation — after interpolation onto the 28000-point −2 to 12 ppm
  grid — to bound intensities to 0–1 for the CNN.
when_to_use_negative:
- 'For 13C NMR input: chemical shifts are only extracted and binned into 80 bins spanning
  3.42 to 231.3 ppm — no intensity normalization is applied to the 13C shift list.'
- When the spectra file already contains preprocessed, normalized 28045-length rows
  ready for the multitask model — re-normalizing is unnecessary.
- When the raw 1H spectrum has not yet been interpolated onto the 28000-point −2 to
  12 ppm grid — normalize after resampling, not before.
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_0593
tools:
- name: NMR2Struct
  role: Provides the `normalize` YAML toggle and the `--normalize` flag of `nmr_infer_single_spectrum`
    that divides the 1H spectrum by its highest intensity during preprocessing/inference
  repo: https://github.com/MarklandGroup/NMR2Struct
- name: Anaconda
  role: Recommended package manager to set up the NMR_env environment in which the
    normalization/preprocessing code runs
  repo: https://www.anaconda.com/download
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acscentsci.4c01132
    title: Accurate and Efficient Structure Elucidation from Routine One-Dimensional
      NMR Spectra Using Multitask Machine Learning
  source_content_hash: sha256:26838db6e048fa8101d5cd33e4cbaa01bc9200946cf40c9e2bf4ed11884ccf3b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MarklandGroup/NMR2Struct
  tools:
  - Anaconda
  - NMR2Struct
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_0593
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# spectral-intensity-normalization

## Summary

Scales the processed 1H NMR spectrum by dividing every intensity by the spectrum's highest intensity so values fall in 0–1 before being fed to the CNN encoder in the NMR2Struct multitask spectrum-to-structure model.

## When to use

Use when raw 1H NMR (ppm, intensity) spectra must be converted into the model-input representation — after interpolation onto the 28000-point −2 to 12 ppm grid — to bound intensities to 0–1 for the CNN.

## When NOT to use

- For 13C NMR input: chemical shifts are only extracted and binned into 80 bins spanning 3.42 to 231.3 ppm — no intensity normalization is applied to the 13C shift list.
- When the spectra file already contains preprocessed, normalized 28045-length rows ready for the multitask model — re-normalizing is unnecessary.
- When the raw 1H spectrum has not yet been interpolated onto the 28000-point −2 to 12 ppm grid — normalize after resampling, not before.

## Inputs

- Raw 1H NMR spectrum as (ppm, intensity) pairs, e.g. tab/space-separated text file (32768 values from MestreNova-predicted SpectraBase spectra)
- Interpolated 1H NMR vector of 28000 points on the −2 to 12 ppm grid
- Concatenated spectra hdf5 file with a 'spectra' dataset of 28045-length rows (preprocessing output)

## Outputs

- Intensity-normalized 1H NMR vector of length 28000 with all values in the 0–1 range
- Per-spectrum vector (concatenated 1H + 13C) written to an hdf5 'spectra' dataset for CNN input

## How to apply

First linearly interpolate the raw 32768-value 1H spectrum onto a grid of 28000 values spanning −2 to 12 ppm at 0.0005 ppm resolution, which retains detailed splittings and rapidly varying features. Then locate the maximum intensity of the interpolated spectrum and divide every point by that maximum, giving a normalized vector with intensities in 0–1. Concatenate the result with the binned 13C vector (80 binary bins over 3.42 to 231.3 ppm) into the single 28045-length representation consumed by the multitask model. In NMR2Struct, this toggle is controlled by the `normalize` field in the YAML data section or the `--normalize` flag of the single-spectrum inference entry point.

## Related tools

- **NMR2Struct** (Provides the `normalize` YAML toggle and the `--normalize` flag of `nmr_infer_single_spectrum` that divides the 1H spectrum by its highest intensity during preprocessing/inference) — https://github.com/MarklandGroup/NMR2Struct
- **Anaconda** (Recommended package manager to set up the NMR_env environment in which the normalization/preprocessing code runs) — https://www.anaconda.com/download

## Examples

```
nmr_infer_single_spectrum --config config.yaml --hnmr_file hnmr.csv --cnmr_file None --ckpt best.ckpt --normalize
```

## Evaluation signals

- Output 1H vector has length 28000 on the −2 to 12 ppm grid at 0.0005 ppm resolution.
- Maximum intensity of the normalized vector equals exactly 1 and no value exceeds 1.
- All normalized intensities are non-negative and fall within 0–1, matching the paper's reported preprocessing parameters.
- Concatenated input vector (1H + 13C) has the 28045 length expected for `model.model_args.structure_model_args.source_size` in the multitask model config.
- Relative peak ratios are preserved: normalization only rescales by a single scalar (the max intensity), so the shape of the interpolated spectrum is unchanged.

## Limitations

- The model was trained on MestreNova-simulated SpectraBase spectra, so normalization parameters may not fully transfer to experimental spectra whose nuances differ from simulated ones.
- Software and package versions are not stated in the methods excerpt, so exact reproducibility of the normalization step depends on the repository environment.
- Only the 1H channel is intensity-normalized; the 13C channel is reduced to a binary shift-presence vector with no intensity information.

## Evidence

- [results] linearly interpolating the spectrum of 32768 values down to a grid of 28000 values corresponding to a shift range of −2 to 12 ppm with a resolution of 0.0005 ppm.: "linearly interpolating the spectrum of 32768 values down to a grid of 28000 values corresponding to a shift range of −2 to 12 ppm with a resolution of 0.0005 ppm."
- [methods] Toggles normalization of the 1H NMR spectrum by dividing the spectrum by the its highest intensity.: [evidence span withheld — anchor status: not_found]
- [readme] nmr_infer_single_spectrum --config YAML_CONFIG_FILE --hnmr_file HNMR_FILE --cnmr_file CNMR_FILE --ckpt CKPT --normalize: "nmr_infer_single_spectrum \
    --config YAML_CONFIG_FILE \
    --hnmr_file HNMR_FILE \
    --cnmr_file CNMR_FILE \
    --ckpt CKPT \
    --normalize"
