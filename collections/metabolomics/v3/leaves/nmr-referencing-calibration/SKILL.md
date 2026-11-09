---
name: nmr-referencing-calibration
description: Use when a raw 1D 1H NMR spectrum of serum/plasma (e.g. Bruker IVDr 600
  MHz format) must be ppm-calibrated before metabolite assignment, integration, or
  cross-sample comparison.
when_to_use_negative:
- The spectrum is not a blood-derived (serum/plasma) 1H NMR profile — glucose at 5.233
  ppm is the reference specific to blood 1H NMR analysis
- Input is already a calibrated/aligned spectrum and no further referencing is needed
- The analysis requires an external chemical-shift reference compound (e.g. TSP/DSS)
  rather than an endogenous glucose signal
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_0593
tools:
- name: SMolESY-select
  role: Executes the automatic calibration step, computing and applying the ppm shift
    that places the glucose anomeric proton at 5.233 ppm
  repo: https://github.com/pantakis/SMolESY-select
- name: SMolESY
  role: Provides the processing framework (baseline suppression and resolution enhancement)
    within which the calibrated spectra are subsequently analyzed
  repo: https://github.com/pantakis/SMolESY
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.1c00113
    title: A Computationally Lightweight Algorithm for Deriving Reliable Metabolite
      Panel Measurements from 1D <sup>1</sup>H NMR
  source_content_hash: sha256:8a4a12c2f5d82241757fcff11e8d930ad433bed35352d8c8df3ef2dbe0b1f19e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pantakis/SMolESY-select
  tools:
  - Bruker IVDr 600 MHz spectrometer
  - MTBLS395
  - SMolESY
  - SMolESY-select
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_0593
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# nmr-referencing-calibration

## Summary

Automatic ppm-referencing of 1D 1H NMR spectra of blood products by calibrating every spectrum to the glucose anomeric proton signal at 5.233 ppm, the routine reference for blood 1H NMR profile analysis, as implemented as the first step of SMolESY-select.

## When to use

Use when a raw 1D 1H NMR spectrum of serum/plasma (e.g. Bruker IVDr 600 MHz format) must be ppm-calibrated before metabolite assignment, integration, or cross-sample comparison.

## When NOT to use

- The spectrum is not a blood-derived (serum/plasma) 1H NMR profile — glucose at 5.233 ppm is the reference specific to blood 1H NMR analysis
- Input is already a calibrated/aligned spectrum and no further referencing is needed
- The analysis requires an external chemical-shift reference compound (e.g. TSP/DSS) rather than an endogenous glucose signal

## Inputs

- Raw 1D 1H NMR spectrum of serum or plasma (Bruker IVDr format, 600 MHz)
- Expected ppm region containing the glucose anomeric proton resonance

## Outputs

- Calibrated spectrum with all data points shifted so glucose anomeric H-1 sits at 5.233 ppm
- Recorded ppm shift offset applied to the spectrum

## How to apply

Load the raw spectrum and locate the glucose anomeric proton resonance within its expected spectral region using the SMolESY-select calibration routine. Compute the ppm offset required to bring this signal to exactly 5.233 ppm — the reference recommended for blood 1H NMR profiles — then apply the computed shift to all data points to produce the calibrated spectrum. Write the calibrated spectrum to an output file together with the recorded ppm shift offset so the transformation is auditable. Perform this as the initial processing step before any bin definition, spin-system pattern search, or smoothing filter (e.g. the 11-point moving average) is applied by SMolESY-select.

## Related tools

- **SMolESY-select** (Executes the automatic calibration step, computing and applying the ppm shift that places the glucose anomeric proton at 5.233 ppm) — https://github.com/pantakis/SMolESY-select
- **SMolESY** (Provides the processing framework (baseline suppression and resolution enhancement) within which the calibrated spectra are subsequently analyzed) — https://github.com/pantakis/SMolESY

## Evaluation signals

- After calibration, the glucose anomeric proton resonance is centered at 5.233 ppm in every spectrum
- The recorded ppm shift offset is finite, small (fraction of a ppm), and written to the output alongside the calibrated spectrum
- Downstream SMolESY-select spin-system searches fall within their predefined spectral windows (bins), which only hold if referencing succeeded
- Calibration is fully automatic and reproducible across the whole cohort (e.g. the 4023-sample plasma/serum set or MTBLS395 spectra) with no manual intervention

## Limitations

- The reference assumes the glucose anomeric proton peak is present and identifiable in each blood spectrum; severely perturbed or non-blood spectra may fail
- No software version is stated for SMolESY-select, and details of the calibration routine are relegated to the Supporting Information
- Calibration is specific to the blood 1H NMR profiling convention (5.233 ppm) and may not transfer to other biofluids or reference conventions

## Evidence

- [results] Calibration of all spectra to the 1H NMR anomeric proton signal of glucose at 5.233 ppm: [evidence span withheld — anchor status: not_found]
- [results] to the anomeric proton of glucose (Figure 2a) resonating at 5.233 ppm: "to the anomeric proton of glucose (Figure 2a) resonating at 5.233 ppm"
- [results] all spectra are automatically calibrated to the 1H NMR signals of glucose: "all spectra are automatically calibrated to the 1H NMR signals of glucose"
- [methods] NMR samples were acquired using Bruker IVDr 600 MHz spectrometers: "NMR samples were acquired using Bruker IVDr 600 MHz spectrometers"
