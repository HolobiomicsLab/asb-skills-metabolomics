---
name: power-law-regression-fit-assessment
description: Use when drift time data from IMS calibrants with known CCS values (e.g.
  Agilent 6560 tune mix in positive/negative ESI mode) must be regressed to obtain
  beta and tfix and the fit must be quantified.
when_to_use_negative:
- Input is a raw (non-peak-picked) IMS file — the tune mix calibrants must first be
  processed via peak detection or manual exploration of the data
- The instrument's calibrants lack known reference CCS values, so no calibrant-to-CCS
  pairing can be built
- The task is generic feature detection, alignment, or MS/MS deconvolution rather
  than CCS calibration fit assessment
edam_operation: http://edamontology.org/operation_3659
tools:
- name: DEIMoS
  role: Python API used to load calibrant H5 files, build calibration factors and
    index, and fit the power-law CCS calibration to obtain beta and tfix
  repo: https://github.com/pnnl/deimos
- name: Python
  role: execution environment for the DEIMoS API and regression/residual computation
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.1c05017
    title: 'DEIMoS: An Open-Source Tool for Processing High-Dimensional Mass Spectrometry
      Data'
  source_content_hash: sha256:9c787408c1bd13d38f1e59481d3e0c917d8edb3f4ed72ed6fd4340d4bcb81970
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
  edam_operation: http://edamontology.org/operation_3659
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# power-law-regression-fit-assessment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Fit and assess the quality of a power-law collision cross section (CCS) calibration that yields beta and tfix coefficients from drift-tube/trapped IMS data, using known-CCS calibrants such as the Agilent tune mix. Fit quality is quantified per ionization mode with a correlation coefficient and sum of residuals.

## When to use

Use when drift time data from IMS calibrants with known CCS values (e.g. Agilent 6560 tune mix in positive/negative ESI mode) must be regressed to obtain beta and tfix and the fit must be quantified.

## When NOT to use

- Input is a raw (non-peak-picked) IMS file — the tune mix calibrants must first be processed via peak detection or manual exploration of the data
- The instrument's calibrants lack known reference CCS values, so no calibrant-to-CCS pairing can be built
- The task is generic feature detection, alignment, or MS/MS deconvolution rather than CCS calibration fit assessment

## Inputs

- peak-picked tune mix calibrant H5 files (e.g. example_tune_pos.h5, example_tune_neg.h5) from an Agilent 6560 Ion Mobility LC/Q-TOF
- reference CCS values for the standard tune mix compounds of known CCS
- measured drift times for the calibrant ions

## Outputs

- fitted power-law calibration coefficients beta and tfix per ionization mode
- correlation coefficient and sum of residuals per ionization mode
- ccs_calibration_results.csv containing beta, tfix, correlation coefficient, and sum of residuals for both modes

## How to apply

Load the peak-picked tune mix calibrant datasets with deimos.load from the positive- and negative-mode H5 files (e.g. example_tune_pos.h5 / example_tune_neg.h5, key='ms1'). Use deimos.calibration build_factors and build_index helpers to pair each measured drift time with its reference CCS value, then fit the power-law CCS calibration model separately for positive and negative ionization modes to obtain beta and tfix. For each mode, compute the correlation coefficient and sum of residuals to quantify goodness of fit; the article reports r-squared = 0.9999784552958134 for the fitted calibration coefficients on the Agilent standard tune mix in positive mode. Save beta, tfix, correlation coefficient, and sum of residuals for both modes as a table (e.g. ccs_calibration_results.csv). Relevant ions in the tune mix data must first be determined through peak detection or manual exploration of the data.

## Related tools

- **DEIMoS** (Python API used to load calibrant H5 files, build calibration factors and index, and fit the power-law CCS calibration to obtain beta and tfix) — https://github.com/pnnl/deimos
- **Python** (execution environment for the DEIMoS API and regression/residual computation)

## Examples

```
import deimos; tune_pos = deimos.load("example_tune_pos.h5", key="ms1"); factors = deimos.calibration.build_factors(tune_pos, reference_ccs); index = deimos.calibration.build_index(factors)
```

## Evaluation signals

- Output table contains beta, tfix, correlation coefficient, and sum of residuals fields for both ionization modes
- Correlation coefficient indicates a strong linear-calibration fit (article reports r-squared = 0.9999784552958134 for the fitted calibration coefficients)
- Calibration factors and index correctly pair each measured drift time with its reference CCS value
- Positive- and negative-mode fits are computed separately, yielding distinct beta and tfix per mode
- Calibrant ions were identified via peak detection or manual exploration before fitting

## Limitations

- Fit quality depends on prior detection of relevant ions in the tune mix data, which must be determined through peak detection or manual exploration
- Assessment reflects fit of the power-law model only against the standard tune mix calibrants with known CCS; generalization to other instruments/compounds is not established by this result

## Evidence
