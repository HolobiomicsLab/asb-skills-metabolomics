---
name: nmr-spectrum-phase-and-baseline-correction
description: Use when you have Fourier-transformed 1H-NMR spectra (e.g. Protomix spectra_df
  from 10000-point zero-filled FIDs) that still show phase errors and baseline offsets
  before referencing, binning, or normalization.
when_to_use_negative:
- Input is still raw FID data — phase and baseline correction act on frequency-domain
  spectra, not on time-domain signals (use group_delay_removal/solvent_residuals_removal
  instead).
- Spectra are already binned into a feature table — correction must precede binning
  (bin_size=0.1) and normalization (method='TotalArea'), not follow it.
- Analysis only needs peak alignment (Icoshift) on already-corrected spectra — re-correcting
  can distort previously aligned peaks.
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_0593
tools:
- name: Protomix
  role: Python package whose px.phase_correction, px.internal_referencing, px.baseline_correction(method='airpls'),
    px.negative_values_zeroing and px.region_removal functions implement this skill
    end-to-end
  repo: https://github.com/mzniber/protomix
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioadv/vbae192
    title: 'Protomix: a Python package for 1H-NMR metabolomics data preprocessing'
  source_content_hash: sha256:9db0dca94191d0508f66aefdd5f5941cb2b83dec3d8d7cf5dc3ca66b5eacb96b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzniber/protomix
  tools:
  - Protomix
  - Python
  - numpy
  - pandas
  - pyicoshift
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_0593
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8125
n_claims: 16
---

# nmr-spectrum-phase-and-baseline-correction

## Summary

Correct phase distortions and remove baseline drift from 1H-NMR spectra after Fourier transform, using Protomix's phase_correction and baseline_correction (airPLS) functions on a spectra DataFrame.

## When to use

Use when you have Fourier-transformed 1H-NMR spectra (e.g. Protomix spectra_df from 10000-point zero-filled FIDs) that still show phase errors and baseline offsets before referencing, binning, or normalization.

## When NOT to use

- Input is still raw FID data — phase and baseline correction act on frequency-domain spectra, not on time-domain signals (use group_delay_removal/solvent_residuals_removal instead).
- Spectra are already binned into a feature table — correction must precede binning (bin_size=0.1) and normalization (method='TotalArea'), not follow it.
- Analysis only needs peak alignment (Icoshift) on already-corrected spectra — re-correcting can distort previously aligned peaks.

## Inputs

- spectra_df: pandas DataFrame of Fourier-transformed 1H-NMR spectra (rows = samples, columns = ppm or points), output of px.fourier_transform
- internally referenced spectra (ir_df) if referencing is applied before baseline correction

## Outputs

- phase-corrected spectra DataFrame (e.g. ph_df = px.phase_correction(spectra_df=spectra_df))
- baseline-corrected spectra DataFrame (e.g. bc_df) ready for region removal, window selection, binning and normalization

## How to apply

After generating spectra with px.fourier_transform from zero-filled FIDs (target_points=10000), apply px.phase_correction to each spectrum to remove zero- and first-order phase distortions before any intensity interpretation. Optionally reference the spectrum first with px.internal_referencing (to TSP/internal standard), then apply px.baseline_correction with method='airpls' to fit and subtract the smooth baseline underlying the peaks. Follow with negative_values_zeroing to eliminate residual negative intensities introduced by the airPLS subtraction, and region_removal (e.g. 4.5–6.1 ppm water region) so that the water-dominated segment does not distort the baseline fit. The rationale is that phase errors invert peaks and airPLS baseline drift biases both peak intensities and downstream TotalArea normalization.

## Related tools

- **Protomix** (Python package whose px.phase_correction, px.internal_referencing, px.baseline_correction(method='airpls'), px.negative_values_zeroing and px.region_removal functions implement this skill end-to-end) — https://github.com/mzniber/protomix

## Examples

```
ph_df = px.phase_correction(spectra_df=spectra_df); ir_df = px.internal_referencing(spectra_df=ph_df); bc_df = px.baseline_correction(spectra_df=ir_df, method='airpls')
```

## Evaluation signals

- Major peaks in each corrected spectrum point upward (positive intensity) — inverted peaks indicate failed phase_correction
- Baseline at signal-free ppm regions sits near zero intensity after airPLS correction
- No or near-zero negative intensity values remain after negative_values_zeroing
- Downstream TotalArea normalization of the corrected spectra (bin_size=0.1 rectangular binning) yields finite, positive areas for all 70 samples
- Water region (4.5–6.1 ppm) removed via region_removal does not bias the baseline fit in adjacent ppm regions

## Limitations

- airPLS baseline estimation can misfit in regions dominated by intense solvent residuals (water ~4.5–6.1 ppm) unless that region is removed first
- Phase correction of automatic-only spectra may leave residual phase errors that peak alignment (pyicoshift) will not fix
- No changelog present for the Protomix repository, so exact behavior/version differences of the correction functions are not documented

## Evidence

- [methods] phase_correction on spectra: "ph_df = px.phase_correction(spectra_df=spectra_df)"
- [methods] airPLS baseline correction: "bc_df = px.baseline_correction(spectra_df=ir_df, method='airpls')"
- [methods] internal referencing precedes correction: "ir_df = px.internal_referencing(spectra_df=ph_df)"
- [readme] Protomix preprocessing features list includes correction steps: [evidence span withheld — anchor status: external_doc]
- [readme] package scope for NMR data preprocessing: "Protomix is a comprehensive Python package designed for preprocessing Nuclear Magnetic Resonance (NMR) data."
