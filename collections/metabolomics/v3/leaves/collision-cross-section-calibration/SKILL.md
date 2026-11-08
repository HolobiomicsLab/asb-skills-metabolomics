---
name: collision-cross-section-calibration
description: Use when processing LC-IMS-MS/MS data (drift tube, TIMS, or traveling-wave
  instruments like the Agilent 6560, Bruker, or Waters Synapt G2Si) where measured
  drift-time-like dimensions must be converted to CCS using calibrant compounds of
  known CCS.
when_to_use_negative:
- The input data already contains directly measured CCS or inverse reduced mobility
  values that do not require an empirical calibration model
- No calibrant compounds of known CCS (e.g., standard tune mix) are present in the
  acquisition, leaving nothing to fit beta and tfix against
- The analysis is standard LC-MS feature detection/alignment with no ion mobility
  dimension to calibrate
edam_topics:
- http://edamontology.org/topic_3336
tools:
- name: DEIMoS
  role: Provides the deimos.calibration module (build_factors/build_index) and deimos.peakpick/deimos.load
    functions used to pair measured drift times with reference CCS values and fit
    the calibration model
  repo: http://github.com/pnnl/deimos
- name: Python
  role: API in which the calibration workflow (loading H5 calibrant files, fitting,
    writing CSV outputs) is executed
provenance:
  source_task_ids:
  - task_005
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.1c05017
    title: 'DEIMoS: An Open-Source Tool for Processing High-Dimensional Mass Spectrometry
      Data'
  source_content_hash: sha256:99236c94b21e27f77a37b8e836e84fdf08ec03b6b68a9f25b2fdc4677fc996f9
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/pnnl/deimos
  tools:
  - DEIMoS
  - Python
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3336
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# collision-cross-section-calibration

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Fits collision cross section (CCS) calibration models with DEIMoS, converting measured ion mobility dimensions (drift time or traveling-wave measurements) into CCS values using calibrants of known CCS such as the Agilent standard tune mix, yielding beta and tfix coefficients with fit-quality metrics.

## When to use

Use when processing LC-IMS-MS/MS data (drift tube, TIMS, or traveling-wave instruments like the Agilent 6560, Bruker, or Waters Synapt G2Si) where measured drift-time-like dimensions must be converted to CCS using calibrant compounds of known CCS.

## When NOT to use

- The input data already contains directly measured CCS or inverse reduced mobility values that do not require an empirical calibration model
- No calibrant compounds of known CCS (e.g., standard tune mix) are present in the acquisition, leaving nothing to fit beta and tfix against
- The analysis is standard LC-MS feature detection/alignment with no ion mobility dimension to calibrate

## Inputs

- Peak-picked tune mix calibrant datasets in H5 format (e.g., example_tune_pos.h5, example_tune_neg.h5, key='ms1')
- Reference CCS values for calibrant compounds of known CCS (standard tune mix)
- Traveling-wave IMS mzML data (e.g., Waters Synapt G2Si from MetaboLights MTBLS812) for TWIMS calibration

## Outputs

- Fitted calibration coefficients beta and tfix per ionization mode
- Fit-quality metrics: correlation coefficient and sum of residuals per mode
- CSV tables such as ccs_calibration_results.csv (beta, tfix, correlation coefficient, sum of residuals) or ccs_calibration_coefficients.csv (coefficients and per-compound residuals)

## How to apply

Load the peak-picked tune mix calibrant data (e.g., example_tune_pos.h5 / example_tune_neg.h5, key='ms1') with deimos.load; the calibrant ions must first be identified via deimos.peakpick or manual exploration. Use deimos.calibration build_factors/build_index helpers to pair measured drift times with reference CCS values, then fit the calibration separately per ionization mode to obtain beta and tfix, reporting the correlation coefficient and sum of residuals. For traveling-wave data (e.g., Waters Synapt G2Si, MetaboLights MTBLS812), the measurement-to-CCS relationship is not directly proportional, so first linearize by taking the natural log of both reference CCS values and the traveling-wave measurements, then fit ln(CCS) versus ln(measurement) via ordinary least squares and inspect per-compound residuals. DEIMoS operates on N-dimensional data largely agnostic to acquisition instrumentation, so the same calibration path applies across drift-tube, TIMS (inverse reduced mobility), and traveling-wave acquisitions.

## Related tools

- **DEIMoS** (Provides the deimos.calibration module (build_factors/build_index) and deimos.peakpick/deimos.load functions used to pair measured drift times with reference CCS values and fit the calibration model) — http://github.com/pnnl/deimos
- **Python** (API in which the calibration workflow (loading H5 calibrant files, fitting, writing CSV outputs) is executed)

## Examples

```
tune_pos = deimos.load("example_tune_pos.h5", key="ms1")
```

## Evaluation signals

- Correlation coefficient indicates a strong calibration fit — the article reports r-squared = 0.9999784552958134 for the positive-mode Agilent tune mix calibration
- Output table contains beta, tfix, correlation coefficient, and sum of residuals fields for both ionization modes
- For traveling-wave data, all fitted coefficients are finite and per-compound residuals show no systematic trend across the calibration range
- Calibrant ions in the tune mix data were correctly detected via peak detection or manual exploration before fitting

## Limitations

- Traveling-wave measurements are not directly CCS-proportional and require ln-ln linearization before regression; no quantitative residual threshold is reported in the paper
- Tune mix ions must first be determined through peak detection or manual exploration of the data before calibration factors can be built
- Calibration must be fit separately for each ionization mode (positive vs. negative); a single pooled model is not supported by the demonstrated workflow

## Evidence

- [results] These data were processed by DEIMoS using feature detection, alignment, CCS calibration, and MS2 extraction by deconvolution.: "These data were processed by DEIMoS using feature detection, alignment, CCS calibration, and MS2 extraction by deconvolution."
- [results] Tune mix calibrants of known CCS: "using a standard tune mix containing compounds of known CCS"
- [results] Tune mix ion detection prerequisite: "The above requires detection of relevant ions in the tune mix data, which must be determined through peak detection or manual exploration of the data."
- [readme] DEIMoS functionality includes CCS calibration: "Functionality includes feature detection, feature alignment, collision cross section (CCS) calibration, isotope detection, and MS/MS spectral deconvolution"
- [abstract] Instrument-agnostic N-dimensional operation: "Notably, DEIMoS operates on N -dimensional data, largely agnostic to acquisition instrumentation"
