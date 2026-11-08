---
name: glucose-anomeric-proton-peak-identification
description: Use when processing raw 1D 1H NMR spectra of serum/plasma (e.g. 600 MHz
  Bruker IVDr format from cohorts like MTBLS395) and spectra must be ppm-calibrated
  to the glucose anomeric proton at 5.233 ppm prior to metabolite bin search.
when_to_use_negative:
- The spectrum has already been calibrated to 5.233 ppm (applying the routine again
  would double-shift the data)
- The input is not a blood-derived 1D 1H NMR profile (e.g. tissue extracts or 2D-Jres
  data), where the glucose anomeric proton is not the routine reference signal
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3381
tools:
- name: SMolESY-select
  role: Implements the automatic calibration routine that shifts the glucose anomeric
    proton signal to 5.233 ppm as the algorithm's first processing step
  repo: https://github.com/pantakis/SMolESY-select
- name: Bruker IVDr 600 MHz spectrometer
  role: Acquisition instrument producing the raw 1D 1H NMR serum/plasma spectra that
    the calibration step operates on
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.1c00113
    title: A Computationally Lightweight Algorithm for Deriving Reliable Metabolite
      Panel Measurements from 1D <sup>1</sup>H NMR
  source_content_hash: sha256:76b84f75828e9cb551e65a18e8185447025d81139f66d64bab077948ca2c4ab6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pantakis/SMolESY-select
  tools:
  - Bruker IVDr 600 MHz spectrometer
  - MTBLS395
  - SMolESY-select
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3381
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# glucose-anomeric-proton-peak-identification

## Summary

Automatic spectral calibration step of the SMolESY-select algorithm: every 1D 1H NMR spectrum of blood serum/plasma is calibrated to the glucose anomeric proton signal at 5.233 ppm, the routine reference for blood 1H NMR profile analysis, before metabolite assignment proceeds.

## When to use

Use when processing raw 1D 1H NMR spectra of serum/plasma (e.g. 600 MHz Bruker IVDr format from cohorts like MTBLS395) and spectra must be ppm-calibrated to the glucose anomeric proton at 5.233 ppm prior to metabolite bin search.

## When NOT to use

- The spectrum has already been calibrated to 5.233 ppm (applying the routine again would double-shift the data)
- The input is not a blood-derived 1D 1H NMR profile (e.g. tissue extracts or 2D-Jres data), where the glucose anomeric proton is not the routine reference signal

## Inputs

- Raw 1D 1H NMR spectrum of serum or plasma (600 MHz, Bruker IVDr format), e.g. from the MTBLS395 Metabolights dataset

## Outputs

- Calibrated 1D 1H NMR spectrum with the glucose anomeric proton positioned at 5.233 ppm
- Recorded ppm shift offset applied during calibration

## How to apply

Load the raw 1D 1H NMR spectrum acquired on a Bruker IVDr 600 MHz spectrometer (Bruker IVDr format, as in the MTBLS395 serum cohort). Within the SMolESY-select calibration routine, locate the glucose anomeric proton resonance in its expected spectral region, then compute the ppm offset needed to bring that signal to 5.233 ppm. Apply the computed shift to all data points of the spectrum and write the calibrated spectrum to an output file together with the recorded ppm shift offset. This is the algorithm's first step because all subsequent SMolESY-select operations — searching spin-system patterns within predefined spectral windows (bins) and applying an 11-point moving average filter — depend on consistent chemical-shift referencing across the >4000-sample development cohort.

## Related tools

- **SMolESY-select** (Implements the automatic calibration routine that shifts the glucose anomeric proton signal to 5.233 ppm as the algorithm's first processing step) — https://github.com/pantakis/SMolESY-select
- **Bruker IVDr 600 MHz spectrometer** (Acquisition instrument producing the raw 1D 1H NMR serum/plasma spectra that the calibration step operates on)

## Evaluation signals

- After calibration, the glucose anomeric proton resonance is centered at 5.233 ppm in every processed spectrum
- The output records the ppm shift offset that was applied, and the offset is consistent with the observed pre-calibration drift
- Downstream SMolESY-select bin search subsequently assigns all 22 metabolite spin systems, consistent with the reported >99.5% assignment accuracy
- Chemical-shift referencing is consistent across the full serum/plasma cohort (e.g. MTBLS395) with no residual per-spectrum ppm drift

## Limitations

- No software version is stated for SMolESY-select or SMolESY in the article, so reproducibility depends on repository state
- Calibration relies on detecting the glucose anomeric proton in its expected region; spectra lacking or distorting this signal may fail to calibrate correctly
- The algorithm's computational/functional details beyond the published text are reported only in the Supporting Information

## Evidence

- [results] Calibration of all spectra to the 1H NMR anomeric proton signal of glucose at 5.233 ppm: "to the anomeric proton of glucose (Figure 2a) resonating at 5.233 ppm"
- [results] Automatic calibration is the first processing step applied to every spectrum: "all spectra are automatically calibrated to the 1H NMR signals of glucose"
- [methods] Further detail of the SMolESY-select calibration routine is provided in supplementary material: "computational/functional features of SMolESY-select are reported in the Supporting Information"
