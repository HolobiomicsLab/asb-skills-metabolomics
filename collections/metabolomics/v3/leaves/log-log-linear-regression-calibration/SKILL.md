---
name: log-log-linear-regression-calibration
description: Use when calibrating traveling-wave ion mobility drift-time measurements
  to CCS values (e.g., Waters Synapt G2Si LC-IMS-MS/MS data) where the drift time
  is not directly CCS-proportional and requires nonlinear-to-linear transformation
  before regression.
when_to_use_negative:
- Instrument delivers drift time that is already directly CCS-proportional (e.g.,
  Agilent 6560 or Bruker TIMS where inverse reduced mobility is measured), so a log-log
  transform is unnecessary.
- Input is already a calibrated CCS feature table; further calibration would double-correct
  the values.
- No reference compounds of known CCS (tune mix) are available in the dataset, leaving
  nothing to regress against.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: DEIMoS
  role: Executes CCS calibration via the deimos.calibration module (build_factors/build_index
    path), fitting log-log linear regression of ln(CCS) versus ln(traveling-wave measurement)
    on tune mix peaks detected with deimos.peakpick.
  repo: https://github.com/pnnl/deimos
- name: Python
  role: Provides the application programming interface through which DEIMoS's CCS
    calibration workflow is invoked and the ccs_calibration_coefficients.csv output
    is written.
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.1c05017
    title: 'DEIMoS: An Open-Source Tool for Processing High-Dimensional Mass Spectrometry
      Data'
  source_content_hash: sha256:56931cf7e1b7ee1d3aba8c1f931fc152d7a414865a6b36cb8d0f26fe51101bb6
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/pnnl/deimos
  repo_url: https://github.com/pnnl/deimos
  tools:
  - DEIMoS
  - Python
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# log-log-linear-regression-calibration

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Fits collision cross section (CCS) calibration for traveling-wave ion mobility data (Waters Synapt G2Si) by natural-log linearizing the CCS-versus-measurement relationship and applying ordinary least-squares linear regression in DEIMoS, then auditing per-compound residuals.

## When to use

Use when calibrating traveling-wave ion mobility drift-time measurements to CCS values (e.g., Waters Synapt G2Si LC-IMS-MS/MS data) where the drift time is not directly CCS-proportional and requires nonlinear-to-linear transformation before regression.

## When NOT to use

- Instrument delivers drift time that is already directly CCS-proportional (e.g., Agilent 6560 or Bruker TIMS where inverse reduced mobility is measured), so a log-log transform is unnecessary.
- Input is already a calibrated CCS feature table; further calibration would double-correct the values.
- No reference compounds of known CCS (tune mix) are available in the dataset, leaving nothing to regress against.

## Inputs

- Waters Synapt G2Si LC-IMS-MS/MS mzML data (e.g., MetaboLights MTBLS812)
- Standard tune mix reference compounds of known CCS
- Detected tune mix ion peaks in N-dimensional m/z-drift time-retention time space (from deimos.peakpick)

## Outputs

- Fitted log-log linear regression coefficients for CCS calibration
- Per-compound calibration residuals
- ccs_calibration_coefficients.csv containing calibration coefficients and residuals

## How to apply

Load the Waters LC-IMS-MS/MS mzML data (e.g., MetaboLights MTBLS812) plus a standard tune mix of known CCS into DEIMoS and detect the tune mix ions in the N-dimensional m/z-drift time-retention time space with deimos.peakpick. Natural-log linearize the relationship by taking ln() of both the reference CCS values and the corresponding traveling-wave measurements, then fit an ordinary least-squares linear regression of ln(CCS) versus ln(measurement) via the deimos.calibration module (build_factors/build_index path). Compute per-compound residuals from the fitted log-log model and write the calibration coefficients and residuals to ccs_calibration_coefficients.csv. Accept the calibration when all fitted coefficients are finite and the per-compound residuals show no systematic trend across the calibration range; the paper reports no quantitative residual threshold, so judgment rests on residual trend inspection.

## Related tools

- **DEIMoS** (Executes CCS calibration via the deimos.calibration module (build_factors/build_index path), fitting log-log linear regression of ln(CCS) versus ln(traveling-wave measurement) on tune mix peaks detected with deimos.peakpick.) — https://github.com/pnnl/deimos
- **Python** (Provides the application programming interface through which DEIMoS's CCS calibration workflow is invoked and the ccs_calibration_coefficients.csv output is written.)

## Examples

```
ms1_peaks = deimos.peakpick(tune_pos); cal = deimos.calibration.build_factors(ms1_peaks, ccs_reference); deimos.calibration.build_index(cal); cal.to_csv('ccs_calibration_coefficients.csv')
```

## Evaluation signals

- All fitted regression coefficients are finite (no NaN/inf) after the log-log linearization and OLS fit.
- Per-compound residuals from the log-log model show no systematic trend across the calibration range (visual inspection of residuals vs. CCS).
- ccs_calibration_coefficients.csv is produced and contains both calibration coefficients and residuals.
- The natural-log transform was applied to both reference CCS values and traveling-wave measurements before regression, not just one side.
- All tune mix ions of known CCS were detected via deimos.peakpick in the 3D m/z-drift time-retention time space before calibration.

## Limitations

- No quantitative residual threshold is reported in the paper; acceptance relies on subjective inspection of residual trends across the calibration range.
- Calibration coefficients are specific to the traveling-wave instrument settings; re-fitting is needed when acquisition parameters change.
- Quality of the calibration depends on detection of the tune mix ions through peak detection or manual exploration of the data, which is a prerequisite step.

## Evidence

- [results] For traveling-wave IMS data (as from the Waters Synapt G2Si), DEIMoS handles CCS calibration by first linearizing the measurement-to-CCS relationship with the natural logarithm and then fitting it with linear regression.: [evidence span withheld — anchor status: not_found]
