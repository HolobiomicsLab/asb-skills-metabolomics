---
name: nmr-chemical-shift-binning
description: Use when preparing a 13C NMR peak list (comma-separated carbon ppm shifts)
  as model input for the NMR2Struct multitask spectrum-to-structure CNN/transformer
  pipeline.
when_to_use_negative:
- The input is a full 1H NMR spectrum (ppm, intensity pairs) — that channel requires
  linear interpolation to a 28000-point grid and normalization by highest intensity,
  not shift binning
- The 13C data is a continuous spectrum/FID rather than a peak-position list — the
  model was trained on simulated MestreNova-derived chemical shift lists, so raw unprocessed
  spectra are out of scope
- The target molecule falls outside the model's demonstrated scope (more than 19 heavy
  atoms)
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_0593
tools:
- name: NMR2Struct
  role: Implements the preprocessing pipeline that extracts 13C shifts and bins them
    into the 80-bin vector used as model input; invoked via nmr_infer_single_spectrum
    with --cnmr_file
  repo: https://github.com/MarklandGroup/NMR2Struct
- name: Anaconda
  role: Recommended package manager for installing the NMR2Struct environment (environment.yml)
    that runs the binning/preprocessing code
  repo: https://www.anaconda.com/download
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acscentsci.4c01132
    title: Accurate and Efficient Structure Elucidation from Routine One-Dimensional
      NMR Spectra Using Multitask Machine Learning
  source_content_hash: sha256:77639da5958b3501476525a4ca2b128a17dc06175ae7797d3e29bca5fa5df06d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MarklandGroup/NMR2Struct
  tools:
  - Anaconda
  - NMR2Struct
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_0593
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# nmr-chemical-shift-binning

## Summary

Extracts 13C chemical shift peak positions from a carbon NMR spectrum and bins them into an 80-bin binary vector spanning 3.42 to 231.3 ppm, producing the 13C component of the concatenated input representation for the NMR2Struct multitask spectrum-to-structure model.

## When to use

Use when preparing a 13C NMR peak list (comma-separated carbon ppm shifts) as model input for the NMR2Struct multitask spectrum-to-structure CNN/transformer pipeline.

## When NOT to use

- The input is a full 1H NMR spectrum (ppm, intensity pairs) — that channel requires linear interpolation to a 28000-point grid and normalization by highest intensity, not shift binning
- The 13C data is a continuous spectrum/FID rather than a peak-position list — the model was trained on simulated MestreNova-derived chemical shift lists, so raw unprocessed spectra are out of scope
- The target molecule falls outside the model's demonstrated scope (more than 19 heavy atoms)

## Inputs

- 13C NMR chemical shift list as a comma-separated text file of carbon ppm values (--cnmr_file)
- Optional accompanying 1H NMR spectrum as space-separated (ppm, intensity) pairs (--hnmr_file) for concatenation
- Shift-grid pickle file (--hnmr_shifts) when running the single-spectrum inference entry point

## Outputs

- 80-bin binary vector spanning 3.42 to 231.3 ppm encoding the detected 13C chemical shifts
- Concatenated 1H/13C input vector (1H interpolated to 28000 points over −2 to 12 ppm plus the 13C bins) written to an hdf5 'spectra' dataset for model input

## How to apply

Load the raw 13C NMR input as a comma-separated list of carbon ppm shifts (per the README, the --cnmr_file argument takes 'a comma separated list of the carbon ppm shifts'). Extract the chemical shift values and bin them into 80 equal bins spanning the shift range of 3.42 to 231.3 ppm, marking each bin as 1 if at least one shift falls in it (binary vector). Unlike the 1H channel, no interpolation or intensity normalization is applied — peak positions only are retained, since the 13C spectrum is a peak list rather than a densely sampled spectrum. Concatenate the resulting 80-bin vector with the 28000-point interpolated 1H vector to form the single 1D input representation expected by the model (the multitask model source_size of 28045 corresponds to 28000 1H points plus the 45-value representation including the 13C bins). If only a 13C spectrum is available, feed the binned vector alone; if neither spectrum exists, pass "None" to the relevant CLI argument.

## Related tools

- **NMR2Struct** (Implements the preprocessing pipeline that extracts 13C shifts and bins them into the 80-bin vector used as model input; invoked via nmr_infer_single_spectrum with --cnmr_file) — https://github.com/MarklandGroup/NMR2Struct
- **Anaconda** (Recommended package manager for installing the NMR2Struct environment (environment.yml) that runs the binning/preprocessing code) — https://www.anaconda.com/download

## Examples

```
nmr_infer_single_spectrum --config config.yaml --hnmr_file None --cnmr_file cnmr_shifts.txt --hnmr_shifts None --cnmr_shifts None --ckpt checkpoints/multitask.ckpt --normalize
```

## Evaluation signals

- The binned 13C output is a binary vector of length 80 spanning 3.42 to 231.3 ppm (no intensity values)
- Every input carbon ppm shift between 3.42 and 231.3 ppm maps to exactly one set bin; bins outside this range are impossible by construction
- When concatenated with the 1H channel, the total input vector matches the multitask model's source_size convention (28045)
- Spectra hdf5 files contain one dataset called 'spectra' with each row a single concatenated 1H+13C vector
- Predicted substructures/SMILES from the assembled input should be chemically consistent with the carbon count implied by the number of distinct 13C shifts

## Limitations

- The model (and hence this preprocessing) was trained on simulated SpectraBase/MestreNova spectra that 'do not fully capture the nuances of real spectra'; experimental 13C shift lists may behave differently
- Only carbon chemical shift positions are retained — any multiplicity, intensity, or DEPT-style edit information in the 13C spectrum is discarded by binning
- Framework and its input representation are limited to molecules with up to 19 heavy atoms

## Evidence

- [results] Extracted 13C chemical shifts and binned them into 80 bins spanning 3.42 to 231.3 ppm: "we processed the spectra by extracting the chemical shifts and binning them into 80 bins spanning the shift range of 3.42 to 231.3 ppm."
- [readme] 13C input is a comma-separated shift list in the single-spectrum entry point: "--cnmr_file`: A text file containing a comma separated list of the carbon ppm shifts in the NMR. If not using an <sup>13</sup>C NMR spectrum, pass "None""
- [readme] Concatenated 1H+13C vector is the expected spectra input format: "each row is a one-dimensional vector representing the concatenated <sup>1</sup>H NMR and <sup>13</sup>C NMR spectrum."
- [discussion] Model trained on simulated spectra, a caveat for real 13C inputs: "that are not fully captured by the simulated spectra used to train the current model"
