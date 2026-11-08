---
name: tune-mix-calibrant-feature-matching
description: 'Use when calibrating collision cross sections on Agilent 6560 ion mobility
  data: you have tune mix H5 files (e.g. example_tune_pos.h5, example_tune_neg.h5)
  with known-CCS calibrants and must match detected peaks to reference CCS values
  per ionization mode.'
when_to_use_negative:
- The dataset contains no calibrant compounds of known CCS (e.g. only study samples
  or blanks) — CCS calibration and calibrant matching cannot be performed
- Data is not ion-mobility-resolved (no drift time dimension), so measured drift times
  cannot be paired with reference CCS values
- You only need feature detection or alignment of study samples rather than CCS calibration
  on the tune mix
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: DEIMoS
  role: Python API used to load tune mix H5 peak tables and run calibration build_factors/build_index
    to pair measured drift times with reference CCS values for beta/tfix fitting
  repo: https://github.com/pnnl/deimos
- name: Python
  role: Runtime environment for the DEIMoS API and fitting of the power-law CCS calibration
    model
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.1c05017
    title: 'DEIMoS: An Open-Source Tool for Processing High-Dimensional Mass Spectrometry
      Data'
  source_content_hash: sha256:9adb2c673856ed24fd41339738200928fcb10bf9714f1534b952212025d3bc03
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
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# tune-mix-calibrant-feature-matching

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Detect and pick relevant ion peaks in standard tune mix calibrant data (compounds of known CCS) loaded from H5 files, then pair measured drift times with reference CCS values to build the factors and index required for DEIMoS CCS calibration (beta and tfix fitting).

## When to use

Use when calibrating collision cross sections on Agilent 6560 ion mobility data: you have tune mix H5 files (e.g. example_tune_pos.h5, example_tune_neg.h5) with known-CCS calibrants and must match detected peaks to reference CCS values per ionization mode.

## When NOT to use

- The dataset contains no calibrant compounds of known CCS (e.g. only study samples or blanks) — CCS calibration and calibrant matching cannot be performed
- Data is not ion-mobility-resolved (no drift time dimension), so measured drift times cannot be paired with reference CCS values
- You only need feature detection or alignment of study samples rather than CCS calibration on the tune mix

## Inputs

- Peak-picked tune mix calibrant datasets in H5 format (e.g. example_tune_pos.h5, example_tune_neg.h5, key='ms1') from Agilent 6560 Ion Mobility LC/Q-TOF acquisitions
- Reference collision cross section (CCS) values for the standard tune mix compounds
- Calibration factor pairing table linking measured drift times to reference CCS values

## Outputs

- Calibration index pairing measured drift times with reference CCS values (via build_factors/build_index)
- Fitted power-law CCS calibration coefficients (beta and tfix) per ionization mode
- Fit-quality metrics (correlation coefficient, sum of residuals), e.g. ccs_calibration_results.csv

## How to apply

Load the peak-picked tune mix calibrant datasets with deimos.load from the positive- and negative-mode H5 files (e.g. example_tune_pos.h5 and example_tune_neg.h5, key='ms1'). Detection of the relevant calibrant ions must be done beforehand through peak detection or manual exploration of the data, since calibrant identity assignment depends on isolating the correct ions. Use deimos.calibration build_factors/build_index helpers to pair each measured drift time with its reference CCS value, constructing calibration factors separately for positive and negative ionization modes. Fit the power-law CCS calibration model to obtain beta and tfix per mode, then compute the correlation coefficient and sum of residuals to quantify fit quality; the article reports r-squared = 0.9999784552958134 for the Agilent tune mix calibrants in positive ESI mode.

## Related tools

- **DEIMoS** (Python API used to load tune mix H5 peak tables and run calibration build_factors/build_index to pair measured drift times with reference CCS values for beta/tfix fitting) — https://github.com/pnnl/deimos
- **Python** (Runtime environment for the DEIMoS API and fitting of the power-law CCS calibration model)

## Examples

```
import deimos; tune_pos = deimos.load("example_tune_pos.h5", key="ms1"); factors = deimos.calibration.build_factors(...); idx = deimos.calibration.build_index(factors, ...)
```

## Evaluation signals

- Output table contains beta, tfix, correlation coefficient, and sum of residuals fields for both positive and negative ionization modes
- Correlation coefficient indicates a strong linear-calibration fit; the article's positive-mode fit achieved r-squared = 0.9999784552958134
- Each measured calibrant drift time is paired with exactly one reference CCS value in the build_index output
- Separate beta/tfix coefficients are produced for positive and negative modes rather than a single pooled fit

## Limitations

- Detection of relevant ions in the tune mix data must be determined through peak detection or manual exploration of the data before matching — calibrant assignment is not automatic
- Fit quality depends on the accuracy of the supplied reference CCS values for the standard tune mix compounds
- Calibration is fitted separately per ionization mode; pooling modes will confound beta and tfix

## Evidence
