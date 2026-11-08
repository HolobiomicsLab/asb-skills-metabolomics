---
name: asymmetric-least-squares-baseline-estimation
description: Use when 1H-NMR spectra (rows = samples, columns = chemical shifts in
  ppm) still show baseline distortions after phase correction and internal referencing,
  and a smooth baseline must be fit and subtracted before downstream analysis.
when_to_use_negative:
- Input is still raw FID data — baseline correction in Protomix operates on spectra
  after fourier_transform, phase_correction, and internal_referencing.
- The distortion is peak misalignment rather than baseline drift — peak alignment
  via px.Icoshift() / pyicoshift is the appropriate step instead.
- Baseline is already flat and only binned/normalized summaries are needed — px.binning
  or normalization applies rather than re-correcting baselines.
edam_topics:
- http://edamontology.org/topic_3173
tools:
- name: Protomix
  role: Provides the baseline_correction function that implements ALS, ARPLS, and
    AIRPLS (adapted from pyBaselines) to fit and subtract baselines from spectra DataFrames
  repo: https://github.com/mzniber/protomix
- name: pyBaselines
  role: Upstream package whose baseline estimation algorithms (asls, arpls, airpls)
    Protomix adapts; additional strategies can be explored from it
- name: pandas
  role: Holds the spectra as a DataFrame input to baseline_correction and stores the
    corrected output matrices
- name: numpy
  role: Numeric backend used by Protomix's preprocessing routines on the spectral
    arrays
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioadv/vbae192
    title: 'Protomix: a Python package for 1H-NMR metabolomics data preprocessing'
  source_content_hash: sha256:bd2ba4df23c1e6678175754cf04ff17a5b625bafbbe50138a1fc5b9879fc0dca
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
  edam_topics:
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 15
---

# asymmetric-least-squares-baseline-estimation

## Summary

Applies penalized least-squares baseline estimation (ALS, ARPLS, AIRPLS adapted from pyBaselines) to 1H-NMR spectra via Protomix's baseline_correction, removing baseline drifts caused by incomplete relaxation, pulse sequence imperfections, magnetic field inhomogeneities, solvent effects, phase errors, and macromolecules.

## When to use

Use when 1H-NMR spectra (rows = samples, columns = chemical shifts in ppm) still show baseline distortions after phase correction and internal referencing, and a smooth baseline must be fit and subtracted before downstream analysis.

## When NOT to use

- Input is still raw FID data — baseline correction in Protomix operates on spectra after fourier_transform, phase_correction, and internal_referencing.
- The distortion is peak misalignment rather than baseline drift — peak alignment via px.Icoshift() / pyicoshift is the appropriate step instead.
- Baseline is already flat and only binned/normalized summaries are needed — px.binning or normalization applies rather than re-correcting baselines.

## Inputs

- spectra_df: pandas DataFrame of phase-corrected, internally referenced 1H-NMR spectra (rows = samples, columns = chemical shifts in ppm)
- method parameter: 'asls' | 'arpls' | 'airpls' (pyBaselines-adapted algorithms)

## Outputs

- bc_df: baseline-corrected spectra DataFrame with the same sample-by-ppm layout as the input
- comparison CSV such as baseline_corrected_spectra.csv holding one column group per correction method

## How to apply

Load the spectra into a pandas DataFrame indexed by sample with ppm values as columns, as produced by the Protomix tutorial pipeline (extract_params → extract_fids → group_delay_removal → solvent_residuals_removal → apodization → zero_filling → fourier_transform → phase_correction → internal_referencing). Call px.baseline_correction on that DataFrame, choosing method='asls' for asymmetric least squares, 'arpls' for adaptive reweighted penalized least squares, or 'airpls' for automatic iterative reweighted penalized least squares — Protomix implements all three, adapted from the pyBaselines package. The function fits and subtracts the estimated baseline, returning a corrected spectra matrix with the same row/column structure. To compare methods, run all three on the same internally referenced DataFrame (ir_df), assemble the corrected matrices as column groups per method, and inspect each for smoother baselines with drifts removed. Additional pyBaselines strategies can be explored beyond the three built-in methods.

## Related tools

- **Protomix** (Provides the baseline_correction function that implements ALS, ARPLS, and AIRPLS (adapted from pyBaselines) to fit and subtract baselines from spectra DataFrames) — https://github.com/mzniber/protomix
- **pyBaselines** (Upstream package whose baseline estimation algorithms (asls, arpls, airpls) Protomix adapts; additional strategies can be explored from it)
- **pandas** (Holds the spectra as a DataFrame input to baseline_correction and stores the corrected output matrices)
- **numpy** (Numeric backend used by Protomix's preprocessing routines on the spectral arrays)

## Examples

```
bc_df = px.baseline_correction(spectra_df=ir_df, method='airpls')
```

## Evaluation signals

- Each method's corrected spectra show baseline drifts removed and a smoother baseline, as specified for the Baseline Correction feature
- Output DataFrame preserves the input layout: rows = samples, columns = chemical shifts in ppm, no sample rows added or dropped
- Running the three methods (asls, arpls, airpls) on the same ir_df yields three aligned corrected matrices suitable for assembling into one column-grouped table
- Written validation: baseline_corrected_spectra.csv exists and contains one column group per method
- The exact tutorial call bc_df = px.baseline_correction(spectra_df=ir_df, method='airpls') executes without error on the Zenodo NMRDataset pipeline output

## Limitations

- Only three methods are implemented natively (ALS, ARPLS, AIRPLS); other strategies require exploring pyBaselines directly
- Baselines arise from many distinct causes (incomplete relaxation, pulse sequence imperfections, magnetic field inhomogeneities, solvent effects, phase errors, macromolecules), so no single method is guaranteed optimal for every distortion type
- Method parameters (e.g., penalty weights for the penalized least-squares fits) must be chosen per dataset; the article does not prescribe fixed thresholds

## Evidence

- [methods] tutorial baseline_correction call: "bc_df = px.baseline_correction(spectra_df=ir_df, method='airpls')"
- [readme] Baseline Correction listed as a Protomix feature: [evidence span withheld — anchor status: external_doc]
- [readme] Protomix scope as NMR preprocessing package: "Protomix is a comprehensive Python package designed for preprocessing Nuclear Magnetic Resonance (NMR) data"
