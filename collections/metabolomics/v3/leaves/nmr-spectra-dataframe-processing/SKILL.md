---
name: nmr-spectra-dataframe-processing
description: Use when Fourier-transformed 1H-NMR spectra are held as a pandas DataFrame
  (samples × ppm) and need per-spectrum phase correction or further Protomix preprocessing
  before statistical analysis.
when_to_use_negative:
- Input is still raw FID/free-induction-decay data rather than a Fourier-transformed
  spectra DataFrame — apply group_delay_removal, apodization, zero_filling, and fourier_transform
  first.
- Task is peak alignment across spectra — use Protomix's Icoshift/pyicoshift rather
  than phase correction.
- The DataFrame already contains referenced, baseline-corrected spectra (e.g. output
  of baseline_correction) — further phase correction is unnecessary.
edam_operation: http://edamontology.org/operation_3417
edam_topics:
- http://edamontology.org/topic_3272
tools:
- name: Protomix
  role: Python package whose DataFrame-oriented functions (px.phase_correction, px.internal_referencing,
    px.baseline_correction, px.plot) perform the spectra preprocessing
  repo: https://github.com/mzniber/protomix
- name: pandas
  role: Holds spectra as DataFrames (rows = samples, columns = chemical shift in ppm)
    consumed and returned by Protomix functions
- name: numpy
  role: Numerical backbone for building the spectra_df and the RMS-of-positive-peaks
    phase optimization
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioadv/vbae192
    title: 'Protomix: a Python package for 1H-NMR metabolomics data preprocessing'
  source_content_hash: sha256:f3b7a90ccb4ebdb32720370de61a51d70a68094bbc02728be3404ec51d4569ec
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzniber/protomix
  tools:
  - Protomix
  - Python
  - numpy
  - pandas
  edam_operation: http://edamontology.org/operation_3417
  edam_topics:
  - http://edamontology.org/topic_3272
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# nmr-spectra-dataframe-processing

## Summary

Processes 1H-NMR spectra represented as a pandas DataFrame (rows = samples, columns = chemical shift in ppm) through Protomix's Fourier-domain preprocessing steps, most notably automatic per-spectrum phase correction that optimizes the RMS of positive peaks. This yields pure absorption lineshapes ready for referencing, baseline correction, and downstream metabolomics analysis.

## When to use

Use when Fourier-transformed 1H-NMR spectra are held as a pandas DataFrame (samples × ppm) and need per-spectrum phase correction or further Protomix preprocessing before statistical analysis.

## When NOT to use

- Input is still raw FID/free-induction-decay data rather than a Fourier-transformed spectra DataFrame — apply group_delay_removal, apodization, zero_filling, and fourier_transform first.
- Task is peak alignment across spectra — use Protomix's Icoshift/pyicoshift rather than phase correction.
- The DataFrame already contains referenced, baseline-corrected spectra (e.g. output of baseline_correction) — further phase correction is unnecessary.

## Inputs

- spectra_df: pandas DataFrame of Fourier-transformed 1H-NMR spectra (rows = samples, columns = chemical shift in ppm), produced by px.fourier_transform from zero-filled FIDs
- acqus_df: Bruker acquisition parameters DataFrame (for upstream px.fourier_transform step)

## Outputs

- ph_df: phase-corrected spectra DataFrame with same row/column structure as spectra_df
- phase_corrected_spectra.csv: corrected spectra table
- fitted_phase_angles.csv: fitted per-spectrum phase angles
- validation plot (px.plot) titled 'NMR Spectrum after phase correction' with xlabel 'Chemical Shift (ppm)'

## How to apply

Build spectra_df as a pandas DataFrame of Fourier-transformed spectra (rows = samples, columns = ppm) using numpy/pandas, then pass it to Protomix functions that operate DataFrame-to-DataFrame. For phase correction, px.phase_correction searches for the optimal rotation angle per spectrum — optimizing an objective based on the root mean square of positive peak intensities so dispersive/negative peak contributions are minimized — applying Φcorrection = Φerror so the real part shows a pure absorption lineshape. The corrected table ph_df then feeds px.internal_referencing and px.baseline_correction (e.g. method='airpls') in sequence. Persist intermediate tables (e.g. phase_corrected_spectra.csv, fitted_phase_angles.csv) and validate by plotting a corrected spectrum to confirm peaks appear in pure absorption mode with a flat baseline.

## Related tools

- **Protomix** (Python package whose DataFrame-oriented functions (px.phase_correction, px.internal_referencing, px.baseline_correction, px.plot) perform the spectra preprocessing) — https://github.com/mzniber/protomix
- **pandas** (Holds spectra as DataFrames (rows = samples, columns = chemical shift in ppm) consumed and returned by Protomix functions)
- **numpy** (Numerical backbone for building the spectra_df and the RMS-of-positive-peaks phase optimization)

## Examples

```
ph_df = px.phase_correction(spectra_df=spectra_df); px.plot(spectrum)  # 'NMR Spectrum after phase correction', xlabel 'Chemical Shift (ppm)'
```

## Evaluation signals

- Output ph_df has the same row/column structure as the input spectra_df (same number of samples and ppm columns).
- A corrected spectrum plotted with px.plot shows peaks in pure absorption mode with a flat baseline (no dispersive lobes).
- phase_corrected_spectra.csv and fitted_phase_angles.csv are written and loadable; fitted_phase_angles.csv has one fitted angle per spectrum.
- After phase correction, negative/minimum intensities are minimized relative to positive peak RMS (the optimization objective).
- Downstream steps (px.internal_referencing, px.baseline_correction with method='airpls') accept ph_df without shape errors.

## Limitations

- Phase correction is per-spectrum automatic optimization of positive-peak RMS; spectra with unusual lineshapes may yield a suboptimal fitted angle since the method assumes absorption-mode peaks dominate.
- The step presupposes a complete upstream FID chain (extract_params, extract_fids, group_delay_removal, solvent_residuals_removal, apodization, zero_filling, fourier_transform); errors upstream propagate into the spectra DataFrame.
- Protomix is scoped to preprocessing of 1H-NMR metabolomics data; it is not a general-purpose NMR processing tool.

## Evidence

- [methods] phase_correction function applied to spectra DataFrame: "ph_df = px.phase_correction(spectra_df=spectra_df)"
- [methods] spectra produced from zero-filled FIDs as DataFrame input: "spectra_df = px.fourier_transform(fid_df=zf_df, acqus_df=acqus_df)"
- [methods] downstream correction step in the DataFrame chain: "bc_df = px.baseline_correction(spectra_df=ir_df, method='airpls')"
- [readme] package purpose supporting DataFrame preprocessing of NMR data: "Protomix is a comprehensive Python package designed for preprocessing Nuclear Magnetic Resonance (NMR) data"
- [abstract] automated preprocessing positioning from abstract: "Protomix offers a set of automated, efficient, and user-friendly signal-preprocessing steps"
