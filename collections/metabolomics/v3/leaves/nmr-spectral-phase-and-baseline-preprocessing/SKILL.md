---
name: nmr-spectral-phase-and-baseline-preprocessing
description: Use when raw 1D 1H NMR spectra of serum/plasma (e.g. Bruker IVDr 600
  MHz, MTBLS395-style data) must be ppm-calibrated and baseline-suppressed before
  metabolite assignment or integration.
when_to_use_negative:
- The analysis goal is complex deconvolution or spectral baseline fitting — SMolESY-select
  is explicitly designed to preclude the need for these on standard 1D 1H NMR spectra
- The sample type is not a blood product (serum/plasma); the glucose anomeric proton
  at 5.233 ppm is the routine reference specifically for blood 1H NMR profiles and
  may be absent or unsuitable in other biofluids
- Input is already calibrated SMolESY-processed data rather than raw 1D 1H NMR spectra
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: SMolESY-select
  role: Runs the automatic spectral calibration routine that shifts the glucose anomeric
    proton to 5.233 ppm and applies the 11-point moving average filter in selected
    windows
  repo: https://github.com/pantakis/SMolESY-select
- name: SMolESY
  role: Provides the baseline (macromolecular) suppression and resolution-enhancement
    preprocessing that produces the narrow SMolESY Δv1/2 line widths prior to assignment
  repo: github.com/pantakis/SMolESY
- name: Bruker IVDr 600 MHz spectrometer
  role: Instrument whose 600 MHz 1D 1H NMR acquisitions (Bruker IVDr format) are the
    input to calibration and preprocessing
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.1c00113
    title: A Computationally Lightweight Algorithm for Deriving Reliable Metabolite
      Panel Measurements from 1D <sup>1</sup>H NMR
  source_content_hash: sha256:30e6febbdfe0ef36df1433edccf8dc18a88c1cebf4707c188408d5ad577d11ec
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pantakis/SMolESY-select
  tools:
  - Bruker IVDr 600 MHz spectrometer
  - MTBLS395
  - SMolESY
  - SMolESY-select
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 16
---

# nmr-spectral-phase-and-baseline-preprocessing

## Summary

Automatic spectral preprocessing for 1D 1H NMR spectra of blood products: calibrate every spectrum to the glucose anomeric proton at 5.233 ppm and apply SMolESY baseline suppression/resolution enhancement so downstream metabolite assignment and integration are reliable.

## When to use

Use when raw 1D 1H NMR spectra of serum/plasma (e.g. Bruker IVDr 600 MHz, MTBLS395-style data) must be ppm-calibrated and baseline-suppressed before metabolite assignment or integration.

## When NOT to use

- The analysis goal is complex deconvolution or spectral baseline fitting — SMolESY-select is explicitly designed to preclude the need for these on standard 1D 1H NMR spectra
- The sample type is not a blood product (serum/plasma); the glucose anomeric proton at 5.233 ppm is the routine reference specifically for blood 1H NMR profiles and may be absent or unsuitable in other biofluids
- Input is already calibrated SMolESY-processed data rather than raw 1D 1H NMR spectra

## Inputs

- Raw 1D 1H NMR spectrum of serum/plasma in Bruker IVDr format acquired on a 600 MHz spectrometer (e.g. from the MTBLS395 MetaboLights cohort)
- Cohort of plasma/serum spectra requiring uniform ppm calibration (e.g. the 4023 unique plasma/serum samples used to define bins)

## Outputs

- Calibrated 1D 1H NMR spectrum with the glucose anomeric proton at 5.233 ppm
- Recorded ppm shift offset applied during calibration
- SMolESY-processed spectrum with baseline suppression and resolution enhancement (optionally 11-point moving-average-smoothed in selected windows)

## How to apply

Load the raw 1D 1H NMR spectrum (600 MHz Bruker IVDr format) and, as the first SMolESY-select step, automatically locate the glucose anomeric proton resonance and compute the ppm offset needed to bring it to 5.233 ppm — the routine reference for blood 1H NMR profile analysis. Apply that shift to all data points and write the calibrated spectrum together with the recorded ppm shift offset. Preprocess with SMolESY, which provides baseline suppression (macromolecular signal suppression) and spectral resolution enhancement, yielding line widths in the narrow range 0.0009 < SMolESY Δv1/2 < 0.0011 ppm on a 600 MHz instrument. Where low-abundance metabolite signals need recovery, apply a moving average filter spanning 11-points across the high-resolution data of the selected spectral windows; this raises the s/n of signals belonging to tyrosine, phenylalanine, histidine, creatinine, and choline while keeping denoised Δv1/2 slightly narrower than standard 1D 1H NMR.

## Related tools

- **SMolESY-select** (Runs the automatic spectral calibration routine that shifts the glucose anomeric proton to 5.233 ppm and applies the 11-point moving average filter in selected windows) — https://github.com/pantakis/SMolESY-select
- **SMolESY** (Provides the baseline (macromolecular) suppression and resolution-enhancement preprocessing that produces the narrow SMolESY Δv1/2 line widths prior to assignment) — github.com/pantakis/SMolESY
- **Bruker IVDr 600 MHz spectrometer** (Instrument whose 600 MHz 1D 1H NMR acquisitions (Bruker IVDr format) are the input to calibration and preprocessing)

## Evaluation signals

- After calibration, the glucose anomeric proton resonance sits at 5.233 ppm in every spectrum
- The output includes the recorded ppm shift offset applied during calibration
- Processed line widths fall within 0.0009 < SMolESY Δv1/2 < 0.0011 ppm on a 600 MHz instrument
- Denoised signal s/n is at most ~10% less than that of CPMG spectra, while Δv1/2 remains slightly narrower than standard 1D 1H NMR
- Downstream SMolESY-select assignment of the 22 metabolites achieves accuracy greater than 99.5%

## Limitations

- Calibration depends on the glucose anomeric proton signal being present and locatable within the expected region, making the routine specific to blood 1H NMR profiles
- No software version is stated for SMolESY or SMolESY-select, complicating reproducibility
- The GUI and usage video exist only as supplementary artifacts (Figure S9, Video S1) and are not part of the main documentation

## Evidence

- [results] Calibration of all spectra to the 1H NMR anomeric proton signal of glucose at 5.233 ppm: [evidence span withheld — anchor status: not_found]
- [results] to the anomeric proton of glucose (Figure 2a) resonating at 5.233 ppm: "to the anomeric proton of glucose (Figure 2a) resonating at 5.233 ppm"
- [results] SMolESY baseline suppression and resolution enhancement: "The use of SMolESY, with its baseline suppression and resolution enhancement features"
- [results] 11-point moving average filter in selected windows: "a moving average filter spanning 11-points (see the Supporting Information) across the high resolution data"
- [results] s/n increase for low-abundance metabolite signals: "increases the s/n of signals belonging to tyrosine, phenylalanine, histidine, creatinine, and choline"
- [abstract] resolution enhancement and macromolecular signal suppression: "bolstered by spectral resolution enhancement and macromolecular signal suppression"
