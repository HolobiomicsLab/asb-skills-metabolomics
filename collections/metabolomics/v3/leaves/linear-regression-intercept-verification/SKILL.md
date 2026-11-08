---
name: linear-regression-intercept-verification
description: Use when a series of spectra contains compounds spiked at known graded
  amounts (e.g., 0/20/40/60/80 uL) and you must confirm linear amplitude response
  plus a nonzero intercept from endogenous matrix content.
when_to_use_negative:
- The input spectra are not high-resolution 1D metabolomics NMR — the article states
  the current SAND version is tuned for high-resolution metabolomics spectra.
- You need exact quantitative concentrations rather than linearity/intercept verification
  — SAND's accuracy is a ~0.9 correlation with ground truth, not exact quantitation.
- The amplitude-vs-volume relationship is expected to be nonlinear (e.g., saturation
  or matrix depletion), making a straight-line intercept test inappropriate.
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: SAND
  role: Deconvolves each 1D NMR spectrum into peak tables whose amplitudes feed the
    correlation and linear regression fits
  repo: github.com/edisonomics/SAND
- name: NMRbox
  role: Cloud environment where SAND is executed for the spike-in series analysis
  repo: https://nmrbox.org
- name: NMRPipe
  role: Interface through which SAND can be invoked on the spectra
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.3c03078
    title: 'SAND: Automated Time-Domain Modeling of NMR Spectra Applied to Metabolite
      Quantification'
  source_content_hash: sha256:26095bd8335e31e757ecd37bd95bf215d3128bbeab2d165ee87b9c78b8d9947e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - NMRPipe
  - NMRbox
  - SAND
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# linear-regression-intercept-verification

## Summary

Verifies that SAND-quantified 1D NMR peak amplitudes rise linearly with spike-in volume and that the fitted regression Y-intercept is clearly nonzero, exposing an endogenous analyte (e.g., glucose) already present in the unspiked matrix.

## When to use

Use when a series of spectra contains compounds spiked at known graded amounts (e.g., 0/20/40/60/80 uL) and you must confirm linear amplitude response plus a nonzero intercept from endogenous matrix content.

## When NOT to use

- The input spectra are not high-resolution 1D metabolomics NMR — the article states the current SAND version is tuned for high-resolution metabolomics spectra.
- You need exact quantitative concentrations rather than linearity/intercept verification — SAND's accuracy is a ~0.9 correlation with ground truth, not exact quantitation.
- The amplitude-vs-volume relationship is expected to be nonlinear (e.g., saturation or matrix depletion), making a straight-line intercept test inappropriate.

## Inputs

- Five 1D 1H NMR spectra of human urine spiked with 0, 20, 40, 60, and 80 uL of a five-compound mixture (Metabolomics Workbench study ST002718)
- SAND peak tables (amplitudes per compound) for each spectrum
- Spike-in volume metadata (uL per spectrum)

## Outputs

- Per-compound Pearson correlation of peak amplitude vs spike-in volume (expected close to 1)
- Fitted linear regression slope and Y-intercept for glucose amplitude vs spike-in volume (intercept must be clearly nonzero)
- Trend check confirming urine-only peaks show no systematic change with volume
- spike_in_linearity_report.csv containing all fitted values

## How to apply

Run SAND on each 1D 1H NMR spectrum of the volume series to deconvolve it into a peak table, using SAND's hybrid Markov chain Monte Carlo optimization with time- and frequency-domain subsampling and its random training/validation split of the time-domain data; spectra should first receive autophasing or manually adjusted phases. For each spiked compound, compute the Pearson correlation between SAND-quantified peak amplitude and spike-in volume and require it to be close to 1, consistent with SAND's reported ~0.9 correlation with ground truth on spiked urine. Check that peak amplitudes of urine-matrix-only compounds stay flat across the 0–80 uL series, ruling out volume-driven artifacts. Finally fit an ordinary least-squares regression of the glucose peak amplitude versus spike-in volume: a clearly nonzero positive Y-intercept indicates glucose endogenously present in the unspiked urine, while a near-zero intercept would suggest the spike-in accounting missed the endogenous contribution.

## Related tools

- **SAND** (Deconvolves each 1D NMR spectrum into peak tables whose amplitudes feed the correlation and linear regression fits) — github.com/edisonomics/SAND
- **NMRbox** (Cloud environment where SAND is executed for the spike-in series analysis) — https://nmrbox.org
- **NMRPipe** (Interface through which SAND can be invoked on the spectra)

## Examples

```
python -c "from scipy.stats import linregress; f=linregress([0,20,40,60,80], glucose_amp); print(f.slope, f.intercept)"
```

## Evaluation signals

- Pearson correlation between spike-in compound amplitude and volume is close to 1 for all five spiked compounds
- Fitted glucose amplitude-vs-volume regression Y-intercept is clearly nonzero, reflecting endogenous glucose in the unspiked (0 uL) urine
- Urine-matrix-only peak amplitudes show no systematic trend across the 0–80 uL series
- SAND-quantified amplitudes agree with ground truth at roughly the ~0.9 correlation level reported for spiked urine
- All fitted values are written to spike_in_linearity_report.csv

## Limitations

- SAND provides a correlation of ~0.9 with ground truth on spiked urine, so amplitude estimates carry residual error that limits interpretation of the fitted intercept.
- SAND is tuned only for high-resolution metabolomics spectra; other spectral cases are not yet accommodated.
- Initial pre-processing with autophasing or manually adjusted phase values is preferred but not always needed — unphased input reliability is unverified.

## Evidence
