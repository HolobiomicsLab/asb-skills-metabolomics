---
name: nmr-spectral-data-io
description: Use when input is raw or already processed 1D ¹H NMR serum/plasma spectra
  (e.g. Bruker IVDr 600 MHz acquisitions from MetaboLights MTBLS395) that must be
  read into memory, normalized to a common chemical-shift reference, or smoothed to
  recover low-abundance metabolite signals.
when_to_use_negative:
- Input is already a deconvoluted or fitted metabolite concentration table — no spectral
  calibration or smoothing is required.
- Data are 2D-Jres or other multidimensional spectra; SMolESY-select operates only
  on standard 1D ¹H NMR data.
- Spectra that cannot be recalibrated to the glucose 5.233 ppm reference (e.g. samples
  lacking an identifiable glucose signal) — the skill depends on this calibration
  anchor.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3272
tools:
- name: SMolESY
  role: Provides the resolution-enhanced, baseline-suppressed 1D ¹H NMR representation
    whose ppm/intensity arrays are loaded and calibrated here.
  repo: https://github.com/pantakis/SMolESY
- name: SMolESY-select
  role: Reads the calibrated spectrum and applies the 11-point moving average smoothing
    filter across the high-resolution data of selected windows.
  repo: https://github.com/pantakis/SMolESY-select
- name: Bruker IVDr 600 MHz spectrometer
  role: Instrument whose acquisition output supplies the raw 1D ¹H NMR spectra that
    this skill ingests.
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.1c00113
    title: A Computationally Lightweight Algorithm for Deriving Reliable Metabolite
      Panel Measurements from 1D <sup>1</sup>H NMR
  source_content_hash: sha256:0ff3a9c9f159e0e5126788de42536a1e0005532b22f1fcb66baf337a32b5a866
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pantakis/SMolESY
  tools:
  - Bruker IVDr 600 MHz spectrometer
  - SMolESY
  - SMolESY-select
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3272
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# nmr-spectral-data-io

## Summary

Loading, calibrating, and smoothing 1D ¹H NMR spectra (Bruker IVDr 600 MHz format) into a numerically uniform ppm/intensity representation so downstream metabolite-assignment and integration algorithms such as SMolESY/SMolESY-select can operate reproducibly on serum/plasma cohorts.

## When to use

Use when input is raw or already processed 1D ¹H NMR serum/plasma spectra (e.g. Bruker IVDr 600 MHz acquisitions from MetaboLights MTBLS395) that must be read into memory, normalized to a common chemical-shift reference, or smoothed to recover low-abundance metabolite signals.

## When NOT to use

- Input is already a deconvoluted or fitted metabolite concentration table — no spectral calibration or smoothing is required.
- Data are 2D-Jres or other multidimensional spectra; SMolESY-select operates only on standard 1D ¹H NMR data.
- Spectra that cannot be recalibrated to the glucose 5.233 ppm reference (e.g. samples lacking an identifiable glucose signal) — the skill depends on this calibration anchor.

## Inputs

- 1D ¹H NMR serum/plasma spectra from Bruker IVDr 600 MHz instruments (ppm shift array and intensity array per sample)
- Cohort-level spectral collections from repositories such as MetaboLights (e.g. MTBLS395)
- Spectral window definitions (bins) for the 24 ¹H NMR spin systems used by SMolESY-select

## Outputs

- Calibrated spectrum (ppm, intensity) aligned to glucose 5.233 ppm
- Smoothed spectrum (ppm, 11-point moving-averaged intensity) for selected windows
- Numerically uniform per-sample spectral arrays ready for pattern matching and direct integration

## How to apply

Load each spectrum's ppm-shift and intensity arrays from the acquisition output (e.g. via the SMolESY/SMolESY-select code that reads Bruker IVDr 600 MHz data). Calibrate every spectrum automatically to the ¹H NMR anomeric proton signal of glucose resonating at 5.233 ppm, ensuring a uniform δ axis across the cohort (e.g. the 4023 plasma/serum samples the paper used for bin definition). When signal-to-noise in specific windows is limiting, apply an 11-point moving average filter across the high-resolution data of the selected windows (SMolESY-select's smoothing step) to recover tyrosine, phenylalanine, histidine, creatinine, and choline signals without degrading Δv1/2 (SMolESY Δv1/2 0.0009–0.0011 ppm at 600 MHz). Write the resulting ppm/intensity arrays to an output file that downstream pattern-matching and direct integration can consume. The rationale is that deconvolution and baseline fitting are avoided entirely — the skill only prepares a standardized, denoised spectral representation for the lightweight algorithm.

## Related tools

- **SMolESY** (Provides the resolution-enhanced, baseline-suppressed 1D ¹H NMR representation whose ppm/intensity arrays are loaded and calibrated here.) — https://github.com/pantakis/SMolESY
- **SMolESY-select** (Reads the calibrated spectrum and applies the 11-point moving average smoothing filter across the high-resolution data of selected windows.) — https://github.com/pantakis/SMolESY-select
- **Bruker IVDr 600 MHz spectrometer** (Instrument whose acquisition output supplies the raw 1D ¹H NMR spectra that this skill ingests.)

## Examples

```
python scripts/smoly_select.py --input serum_spectrum.txt --calibrate 5.233 --smooth 11
```

## Evaluation signals

- Every spectrum's dominant glucose anomeric peak is located at exactly 5.233 ppm after calibration.
- The ppm axis is identical across all samples in the cohort (same bin width and grid), enabling direct integration comparisons.
- After the 11-point smoothing, Δv1/2 of the SMolESY signals remains within the reported 0.0009–0.0011 ppm range for a 600 MHz instrument and is slightly narrower than the standard 1D ¹H NMR spectrum.
- Signal-to-noise of the smoothed windows for tyrosine, phenylalanine, histidine, creatinine, and choline increases relative to the unsmoothed spectrum, and the s/n remains at most ∼10% lower than that of CPMG spectra.
- The output file exists and contains two arrays (ppm and smoothed intensity) of equal length matching the original spectral resolution.

## Limitations

- The skill assumes spectra were acquired on a 600 MHz Bruker IVDr platform; other field strengths may shift the glucose reference or the reported Δv1/2 values.
- Calibration depends on the presence of the glucose anomeric proton signal; samples without it cannot be reliably normalized.
- The 11-point moving average filter is defined only for the selected windows, not the full spectrum; applying it globally was not described in the paper.
- No software version for SMolESY or SMolESY-select is stated, so reproducibility across releases is not guaranteed.

## Evidence

- [results] All spectra are automatically calibrated to the 1H NMR anomeric proton signal of glucose resonating at 5.233 ppm.: [evidence span withheld — anchor status: not_found]
- [results] Moving average filter spanning 11-points is applied within selected NMR spectral windows: "a moving average filter spanning 11-points (see the Supporting Information) across the high resolution data of the selected windows"
- [results] Denosed SMolESY signals maintain slightly narrower Δv1/2 than standard 1D 1H NMR, with s/n at most ∼10% less than CPMG spectra: "their s/n is a maximum of ∼10% less than that of the CPMG spectra"
- [results] SMolESY Δv1/2 0.0009 < SMolESY Δv1/2 < 0.0011 ppm for the 600 MHz NMR instrument: "0.0009 < SMolESY Δv1/2 < 0.0011 ppm for the 600 MHz NMR instrument"
- [results] SMolESY-select utilizes a moving average filter spanning 11-points: "Specifically, SMolESY-select utilizes a moving average filter spanning 11-points"
- [abstract] The algorithm was developed using >4000 NMR serum and plasma spectra: "The performance of the algorithm, developed using >4000 NMR serum and plasma spectra, was evaluated using an additional >8800 spectra"
