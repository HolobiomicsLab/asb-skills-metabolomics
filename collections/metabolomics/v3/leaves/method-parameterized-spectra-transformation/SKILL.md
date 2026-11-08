---
name: method-parameterized-spectra-transformation
description: Use when a 1H-NMR spectra DataFrame (columns = chemical shifts in ppm,
  rows = samples) shows baseline distortions from incomplete relaxation, pulse sequence
  imperfections, magnetic field inhomogeneities, solvent effects, phase errors, or
  macromolecules.
when_to_use_negative:
- Input is still raw FID or pre-phase-correction data — baseline_correction is positioned
  after phase_correction and internal_referencing in the Protomix pipeline (ph_df
  -> ir_df -> bc_df)
- The spectra DataFrame is already baseline-corrected and the goal is peak alignment
  (px.Icoshift) or binning (px.binning) — those are separate downstream Protomix steps
- Input is not 1H-NMR spectra but a binned/statistical feature table — this skill
  operates on spectra matrices with ppm chemical-shift columns
tools:
- name: Protomix
  role: Provides the baseline_correction function with method-parameterized ALS/ARPLS/AIRPLS
    implementations adapted from pyBaselines
  repo: https://github.com/mzniber/protomix
- name: pyBaselines
  role: Source package whose baseline correction algorithms (ALS, ARPLS, AIRPLS) Protomix
    adapts; additional strategies can be explored from it
- name: pandas
  role: Holds the spectra DataFrame (spectra_df/bc_df) of ppm columns by sample rows
    that baseline_correction consumes and returns
- name: numpy
  role: Numerical backend for the spectra matrices manipulated during baseline fitting
    and subtraction
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioadv/vbae192
    title: 'Protomix: a Python package for 1H-NMR metabolomics data preprocessing'
  source_content_hash: sha256:e2d34ac2a2b5af14931d47918d5ede9e64f2ad98c22affa6227979fa6ba5e0c7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzniber/protomix
  tools:
  - Protomix
  - Python
  - numpy
  - pandas
  - pyBaselines
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# method-parameterized-spectra-transformation

## Summary

Apply Protomix's baseline_correction function to 1H-NMR spectra using a method-string parameter that selects between asymmetric least squares (ALS), adaptive reweighted penalized least squares (ARPLS), and automatic iterative reweighted penalized least squares (AIRPLS), all adapted from the pyBaselines package.

## When to use

Use when a 1H-NMR spectra DataFrame (columns = chemical shifts in ppm, rows = samples) shows baseline distortions from incomplete relaxation, pulse sequence imperfections, magnetic field inhomogeneities, solvent effects, phase errors, or macromolecules.

## When NOT to use

- Input is still raw FID or pre-phase-correction data — baseline_correction is positioned after phase_correction and internal_referencing in the Protomix pipeline (ph_df -> ir_df -> bc_df)
- The spectra DataFrame is already baseline-corrected and the goal is peak alignment (px.Icoshift) or binning (px.binning) — those are separate downstream Protomix steps
- Input is not 1H-NMR spectra but a binned/statistical feature table — this skill operates on spectra matrices with ppm chemical-shift columns

## Inputs

- pandas DataFrame of 1H-NMR spectra (spectra_df, e.g. ir_df from px.internal_referencing) with columns = chemical shifts in ppm and rows = samples
- method parameter string: 'asls', 'arpls', or 'airpls'

## Outputs

- baseline-corrected spectra DataFrame (bc_df) with one column group per method applied
- baseline_corrected_spectra.csv aggregating the corrected spectra matrices for 'asls', 'arpls', and 'airpls'

## How to apply

After Protomix's internal_referencing step (e.g. ir_df = px.internal_referencing(spectra_df=ph_df)), call px.baseline_correction(spectra_df=ir_df, method=...) where method selects the algorithm: 'asls' for asymmetric least squares, 'arpls' for adaptive reweighted penalized least squares, or 'airpls' for automatic iterative reweighted penalized least squares — the tutorial pipeline uses bc_df = px.baseline_correction(spectra_df=ir_df, method='airpls'). Each call fits a baseline to the spectra and subtracts it, returning a corrected DataFrame with the same sample rows and ppm columns. To compare methods, run all three variants and assemble the corrected matrices as one column group per method, then write them to baseline_corrected_spectra.csv. The three methods are adapted from the pyBaselines package, so additional baseline correction strategies from pyBaselines can also be explored beyond the built-in method strings.

## Related tools

- **Protomix** (Provides the baseline_correction function with method-parameterized ALS/ARPLS/AIRPLS implementations adapted from pyBaselines) — https://github.com/mzniber/protomix
- **pyBaselines** (Source package whose baseline correction algorithms (ALS, ARPLS, AIRPLS) Protomix adapts; additional strategies can be explored from it)
- **pandas** (Holds the spectra DataFrame (spectra_df/bc_df) of ppm columns by sample rows that baseline_correction consumes and returns)
- **numpy** (Numerical backend for the spectra matrices manipulated during baseline fitting and subtraction)

## Examples

```
bc_df = px.baseline_correction(spectra_df=ir_df, method='airpls')
```

## Evaluation signals

- baseline_corrected_spectra.csv exists and contains one column group per method (asls, arpls, airpls) with columns = chemical shifts in ppm and rows = samples
- Each method's corrected spectra show baseline drifts removed and a smoother baseline compared to ir_df, as specified for the Baseline Correction feature
- The corrected DataFrame bc_df has the same sample rows and ppm-column indexing as the input ir_df (only the intensity values change)
- Method strings passed to px.baseline_correction are limited to the supported values 'asls', 'arpls', 'airpls' (or strategies from pyBaselines)
- Corrected spectra follow immediately from the tutorial chain: spectra_df -> phase_correction -> internal_referencing -> baseline_correction

## Limitations

- Only three methods are built in via the method parameter ('asls', 'arpls', 'airpls'); other baseline strategies require turning to the pyBaselines package directly
- Baseline distortions it targets are specific to 1H-NMR spectra: incomplete relaxation, pulse sequence imperfections, magnetic field inhomogeneities, solvent effects, phase errors, and macromolecules
- The function is documented for 1H NMR metabolomics data (Bruker acquisitions in the tutorial pipeline), not for arbitrary spectral types

## Evidence

- [methods] baseline_correction invocation in tutorial pipeline: "bc_df = px.baseline_correction(spectra_df=ir_df, method='airpls')"
- [methods] scope of the package: "Protomix is a Python package designed for the preprocessing of 1H NMR metabolomics data"
- [readme] README package description: "Protomix is a comprehensive Python package designed for preprocessing Nuclear Magnetic Resonance (NMR) data."
- [readme] README purpose of preprocessing features: [evidence span withheld — anchor status: external_doc]
