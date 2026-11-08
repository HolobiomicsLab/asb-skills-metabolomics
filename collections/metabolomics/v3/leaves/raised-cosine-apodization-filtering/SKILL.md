---
name: raised-cosine-apodization-filtering
description: Use when partitioning a 1D frequency-domain metabolomics NMR spectrum
  (ppm axis) into frequency bins inside SAND's adaptive binning stage, where each
  bin must be windowed at its edges to suppress spectral leakage prior to conversion
  to per-bin time-domain FIDs.
when_to_use_negative:
- Input is not a 1D frequency-domain metabolomics spectrum — SAND's current version
  is tuned for high-resolution metabolomics spectra and does not yet accommodate other
  cases
- Data is already in the time domain (FID) — this filter is part of the frequency-to-time
  conversion pipeline and would not apply
- A quantitative figure of merit is needed for the binning/filter stage alone — no
  quantitative metric is reported for this stage in isolation; validation is pass/fail
  against SAND test examples
tools:
- name: SAND
  role: Host pipeline whose adaptive binning stage implements the raised-cosine banding,
    Hilbert transform, and inverse FT; distributed via NMRbox and NMRPipe
  repo: github.com/edisonomics/SAND
- name: MATLAB
  role: Current implementation language; the binning/apodization module is saved as
    src/adaptive_binning.m following SAND base function interface conventions
- name: NMRbox
  role: Cloud environment where SAND updates must be run and pass the test examples
    before merge; test server used to validate the module
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.3c03078
    title: 'SAND: Automated Time-Domain Modeling of NMR Spectra Applied to Metabolite
      Quantification'
  source_content_hash: sha256:e2a487568b629b3aefa683f47664ff8db7d707ba3073eabd8bd086fa0879eb99
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - MATLAB
  - NMRbox
  - SAND
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# raised-cosine-apodization-filtering

## Summary

Applies a raised-cosine band with a 0.02 ppm roll-off at the edges of each adaptively defined frequency bin (nominal bin size 0.002 ppm) of a 1D metabolomics NMR spectrum, suppressing spectral leakage before Hilbert-transform and inverse-Fourier conversion of each bin into a time-domain FID for SAND's time-domain modeling.

## When to use

Use when partitioning a 1D frequency-domain metabolomics NMR spectrum (ppm axis) into frequency bins inside SAND's adaptive binning stage, where each bin must be windowed at its edges to suppress spectral leakage prior to conversion to per-bin time-domain FIDs.

## When NOT to use

- Input is not a 1D frequency-domain metabolomics spectrum — SAND's current version is tuned for high-resolution metabolomics spectra and does not yet accommodate other cases
- Data is already in the time domain (FID) — this filter is part of the frequency-to-time conversion pipeline and would not apply
- A quantitative figure of merit is needed for the binning/filter stage alone — no quantitative metric is reported for this stage in isolation; validation is pass/fail against SAND test examples

## Inputs

- 1D frequency-domain NMR spectrum (metabolomics, high-resolution) with a mapped ppm axis from the SAND repository test examples
- Adaptive bin boundaries: buckets of nominal size 0.002 ppm (adjacent bins merged in groups of three where the 0.99 slackness criterion is met)
- Baseline-corrected intensities per binned region (after minimum-shift baseline removal)

## Outputs

- Raised-cosine-apodized frequency-domain data per bin (0.02 ppm roll-off at bin edges)
- Complex per-bin time-domain FID data, produced via Hilbert transform and inverse Fourier transform of each apodized bin

## How to apply

First load the 1D frequency-domain NMR spectrum and map its ppm axis, then partition it adaptively into buckets of nominal size 0.002 ppm using a slackness criterion of 0.99, merging adjacent bins in groups of three where the criterion is met. Apply minimum-shift baseline removal within each binned region to correct intensity offsets. Then apply a raised-cosine band with a 0.02 ppm roll-off at the edges of each bin region — the rationale is to taper the bin edges smoothly so the finite frequency window does not introduce spectral leakage (sinc-like ringing) that would corrupt the subsequent time-domain modeling. Follow the windowing with a Hilbert transform to form the analytic signal (generating imaginary data) and an inverse Fourier transform to yield a complex per-bin time-domain FID. In the SAND codebase these steps are wrapped as a callable MATLAB module (src/adaptive_binning.m) following SAND's base function interface conventions, and updates must pass the NMRbox test harness before merge.

## Related tools

- **SAND** (Host pipeline whose adaptive binning stage implements the raised-cosine banding, Hilbert transform, and inverse FT; distributed via NMRbox and NMRPipe) — github.com/edisonomics/SAND
- **MATLAB** (Current implementation language; the binning/apodization module is saved as src/adaptive_binning.m following SAND base function interface conventions)
- **NMRbox** (Cloud environment where SAND updates must be run and pass the test examples before merge; test server used to validate the module)

## Examples

```
matlab -batch "addpath('src'); FIDs = adaptive_binning(spec, ppm, 'BinSize', 0.002, 'Slackness', 0.99, 'RollOff', 0.02);"
```

## Evaluation signals

- Run the module on the SAND repository test examples via the NMRbox test server; acceptance is that the test examples pass ('pass the test before merge')
- Verify each bin region's edges taper with the 0.02 ppm raised-cosine roll-off and that intensity reaches zero at bin boundaries (no discontinuities at edges)
- Confirm that after Hilbert transform each bin has a well-formed imaginary component and the inverse FT yields a complex per-bin FID (real + imaginary time-domain data)
- Check bin boundaries respect the nominal 0.002 ppm size with group-of-three merging only where the 0.99 slackness criterion is met
- Confirm minimum-shift baseline removal was applied before windowing, so bin intensities are offset-corrected prior to apodization

## Limitations

- No quantitative metric is reported for this stage in isolation; validation relies on the SAND test harness pass/fail rather than measured error bounds
- SAND's MATLAB implementation is slow and not readily parallelizable; a faster C/C++ version is planned, so performance is a practical constraint on this stage
- SAND is currently tuned for high-resolution metabolomics spectra; other spectra types and nonexponential signals are not yet accommodated

## Evidence

- [methods] The band-selected bin is processed by Hilbert transform to generate imaginary data and then inverse Fourier-processed to produce an FID for each bin.: [evidence span withheld — anchor status: not_found]
