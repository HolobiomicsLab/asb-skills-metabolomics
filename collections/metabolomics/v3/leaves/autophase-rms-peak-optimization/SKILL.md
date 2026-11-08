---
name: autophase-rms-peak-optimization
description: Use when Fourier-transformed NMR spectra (samples × chemical-shift ppm
  DataFrame) show dispersive/negative peak contributions or unflat baselines, and
  an automatic zero-/first-order phase correction is needed before referencing or
  baseline correction.
when_to_use_negative:
- Input is still raw FID/time-domain data — phasing is applied only after group_delay_removal,
  solvent_residuals_removal, apodization, zero_filling and fourier_transform in the
  Protomix chain
- Spectra were manually phased or already show pure absorption lineshapes with a flat
  baseline — re-autophasing risks degrading correct phases
- Data are not 1H-NMR spectra organized as a samples × chemical-shift (ppm) pandas
  DataFrame
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Protomix
  role: Provides the px.phase_correction function that fits per-spectrum phase angles
    by optimizing RMS of positive peaks
  repo: https://github.com/mzniber/protomix
- name: numpy
  role: Numerical rotation/scaling of spectra and RMS objective computation during
    angle optimization
- name: pandas
  role: Holds the spectra DataFrame (samples × ppm) and writes phase_corrected_spectra.csv
    / fitted_phase_angles.csv
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioadv/vbae192
    title: 'Protomix: a Python package for 1H-NMR metabolomics data preprocessing'
  source_content_hash: sha256:982721b79e6ed349d69356eb833b4645ea7b39511dc5b4abe87cc672fbe18899
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzniber/protomix
  tools:
  - Protomix
  - Python
  - numpy
  - pandas
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# autophase-rms-peak-optimization

## Summary

Automatically removes phase errors from 1H-NMR spectra by fitting a per-spectrum phase rotation angle that optimizes the RMS of positive peak intensities, so the real part shows a pure absorption lineshape. Implemented as Protomix's phase_correction step in the FID-to-spectrum preprocessing chain.

## When to use

Use when Fourier-transformed NMR spectra (samples × chemical-shift ppm DataFrame) show dispersive/negative peak contributions or unflat baselines, and an automatic zero-/first-order phase correction is needed before referencing or baseline correction.

## When NOT to use

- Input is still raw FID/time-domain data — phasing is applied only after group_delay_removal, solvent_residuals_removal, apodization, zero_filling and fourier_transform in the Protomix chain
- Spectra were manually phased or already show pure absorption lineshapes with a flat baseline — re-autophasing risks degrading correct phases
- Data are not 1H-NMR spectra organized as a samples × chemical-shift (ppm) pandas DataFrame

## Inputs

- Fourier-transformed 1H-NMR spectra as a pandas DataFrame (spectra_df; rows = samples, columns = chemical shift in ppm), typically produced by px.fourier_transform

## Outputs

- Phase-corrected spectra DataFrame (ph_df) with the same row/column structure as the input
- phase_corrected_spectra.csv (corrected spectra table)
- fitted_phase_angles.csv (per-spectrum fitted phase angles)

## How to apply

Load the Fourier-transformed spectra as a pandas DataFrame (spectra_df with rows = samples, columns = chemical shift in ppm) built with numpy/pandas, e.g. from px.fourier_transform(fid_df=zf_df, acqus_df=acqus_df). For each spectrum, fit the phase rotation angle (zero- and first-order components) by optimizing an objective based on the root mean square of positive peak intensities, minimizing dispersive/negative contributions — this applies Φcorrection = Φerror. Apply the fitted per-spectrum rotation via px.phase_correction(spectra_df=spectra_df) to produce the corrected table ph_df, then write the corrected spectra to phase_corrected_spectra.csv and the per-spectrum angles to fitted_phase_angles.csv. Rationale: after phase correction, peaks should appear in pure absorption mode with a flat baseline, which is a prerequisite for downstream internal_referencing and baseline_correction (e.g. airPLS).

## Related tools

- **Protomix** (Provides the px.phase_correction function that fits per-spectrum phase angles by optimizing RMS of positive peaks) — https://github.com/mzniber/protomix
- **numpy** (Numerical rotation/scaling of spectra and RMS objective computation during angle optimization)
- **pandas** (Holds the spectra DataFrame (samples × ppm) and writes phase_corrected_spectra.csv / fitted_phase_angles.csv)

## Examples

```
ph_df = px.phase_correction(spectra_df=spectra_df)
```

## Evaluation signals

- Plot a corrected spectrum with px.plot titled 'NMR Spectrum after phase correction' (xlabel 'Chemical Shift (ppm)') and confirm peaks appear in pure absorption mode with a flat baseline
- Verify phase_corrected_spectra.csv exists with the same row/column structure as the input spectra_df table
- Verify fitted_phase_angles.csv contains one fitted angle per spectrum (one per row of spectra_df)
- RMS of positive peak intensities should be maximized/minimized as per the objective so negative (dispersive) peak contributions are reduced relative to the unphased input

## Limitations

- Phase angles are fitted per spectrum automatically; no manual per-peak phase adjustment is provided, so pathological spectra may need manual inspection
- The optimization relies on the RMS-of-positive-peaks objective; spectra dominated by noise or very few positive peaks may yield unreliable fitted angles
- Phasing must run on Fourier-transformed spectra; upstream errors in fourier_transform or acqus parameters propagate into the fitted angles

## Evidence

- [methods] phase_correction step applied to Fourier-transformed spectra: "ph_df = px.phase_correction(spectra_df=spectra_df)"
- [methods] input to phase correction is the Fourier-transformed spectra table: "spectra_df = px.fourier_transform(fid_df=zf_df, acqus_df=acqus_df)"
- [methods] optimization objective of the autophasing: [evidence span withheld — anchor status: not_found]
- [readme] Protomix scope as an NMR preprocessing package: "Protomix is a comprehensive Python package designed for preprocessing Nuclear Magnetic Resonance (NMR) data."
- [readme] Phase Correction is one of Protomix's supported preprocessing techniques: [evidence span withheld — anchor status: external_doc]
