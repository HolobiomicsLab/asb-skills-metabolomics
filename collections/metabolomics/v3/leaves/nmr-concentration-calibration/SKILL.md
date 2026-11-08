---
name: nmr-concentration-calibration
description: Use when a series of 1D NMR spectra (individual, pseudo-2D, or tabulated
  text files) has been fitted and fitted signal areas must be converted to mM concentrations
  and time courses with uncertainties.
when_to_use_negative:
- Input is raw or unfitted spectra — spectral fitting (peak picking, multiplet model
  fitting) must precede the calibration step
- Data are 2D NMR experiments rather than 1D spectra or pseudo-2D series, which is
  MultiNMRFit's stated scope
- No calibration is available to relate fitted signal areas to concentrations, so
  absolute mM values cannot be derived
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3172
tools:
- name: MultiNMRFit
  role: Fits the 1D/pseudo-2D spectra with multiplet signal models to estimate signal
    areas, chemical shifts, and uncertainties that are then calibrated to concentrations
  repo: https://github.com/NMRTeamTBI/MultiNMRFit/
- name: Python
  role: Runtime environment (Python 3.8+) in which MultiNMRFit is installed via pip
    and used as a library
- name: pip
  role: Used to install MultiNMRFit (`pip install multinmrfit`)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btaf463
    title: 'MultiNMRFit: a software to fit 1D and pseudo-2D NMR spectra'
  source_content_hash: sha256:7be36cd9fb8a2f10f386c8aa6b65b7184c906c3cd86d03964913abcd8550bcac
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NMRTeamTBI/MultiNMRFit
  tools:
  - MultiNMRFit
  - Python
  - pip
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# nmr-concentration-calibration

## Summary

A skill for converting fitted NMR signal areas into absolute millimolar concentrations by applying a calibration, with parameter uncertainties (standard deviations) propagated from the spectral fitting. It relies on MultiNMRFit's ability to estimate signal areas from 1D and pseudo-2D spectra using multiplet signal models.

## When to use

Use when a series of 1D NMR spectra (individual, pseudo-2D, or tabulated text files) has been fitted and fitted signal areas must be converted to mM concentrations and time courses with uncertainties.

## When NOT to use

- Input is raw or unfitted spectra — spectral fitting (peak picking, multiplet model fitting) must precede the calibration step
- Data are 2D NMR experiments rather than 1D spectra or pseudo-2D series, which is MultiNMRFit's stated scope
- No calibration is available to relate fitted signal areas to concentrations, so absolute mM values cannot be derived

## Inputs

- Series of 1D NMR spectra (individual 1D spectra, a pseudo-2D spectrum, or tabulated text files), e.g. covering the acetate methyl region
- Calibration used to convert fitted signal areas to millimolar concentrations
- Fitted signal areas and their standard deviations from MultiNMRFit multiplet fits

## Outputs

- Millimolar concentrations per isotopomer/component per time point (e.g. acetate_quantification.csv with total acetate, 2-13C1-acetate, and glucose-derived labeled acetate)
- Parameter uncertainties (standard deviations) reported by MultiNMRFit for each concentration
- Fitted curves for visual inspection against experimental spectra

## How to apply

Load the series of spectra (acquired individually, as a pseudo-2D spectrum, or as tabulated text files) into MultiNMRFit and use its semi-automated peak picking and multiplicity definition to place each signal of interest, such as the acetate methyl multiplet. Fit each spectrum with the built-in multiplet signal models, which account for overlaps between peaks and zero-order baseline correction, to estimate signal area and chemical shift per isotopomer component. Convert the fitted signal areas to millimolar concentrations using a calibration, and record the parameter uncertainties (standard deviations) reported by MultiNMRFit for each concentration. Visually inspect the fitted curves against the experimental spectra before writing concentration time courses (e.g. total acetate and isotopic species) to an output CSV such as acetate_quantification.csv.

## Related tools

- **MultiNMRFit** (Fits the 1D/pseudo-2D spectra with multiplet signal models to estimate signal areas, chemical shifts, and uncertainties that are then calibrated to concentrations) — https://github.com/NMRTeamTBI/MultiNMRFit/
- **Python** (Runtime environment (Python 3.8+) in which MultiNMRFit is installed via pip and used as a library)
- **pip** (Used to install MultiNMRFit (`pip install multinmrfit`))

## Examples

```
pip install multinmrfit; nmrfit
```

## Evaluation signals

- Total acetate concentration remains constant at 8.5 ± 0.1 mM across the time series
- 2-13C1-acetate declines from 8.5 to 6.5 mM while glucose-derived labeled forms rise from 0.1 to 1.9 mM
- Fitted curves visually match the experimental spectra after visual inspection
- Each concentration carries a reported standard deviation from MultiNMRFit's uncertainty estimation
- Concentrations in the output CSV (e.g. acetate_quantification.csv) sum consistently with total acetate across isotopic species

## Limitations

- Accurate concentrations depend on the calibration; the article does not detail calibration construction beyond its use for mM conversion
- Overlapping peaks must be modeled by the multiplet signal models; unmodeled overlap can bias areas and thus concentrations
- Only zero-order baseline correction is applied, so baselines with higher-order distortions may degrade fit quality

## Evidence

- [abstract] MultiNMRFit extracts signal parameters (e.g. intensity, area, chemical shift, and coupling constants) by fitting the experimental spectra using built-in or user-defined signal models: "MultiNMRFit extracts signal parameters (e.g. intensity, area, chemical shift, and coupling constants) by fitting the experimental spectra using built-in or user-defined signal models"
- [methods] MultiNMRFit is a scientific tool designed to extract quantitative information (chemical shifts, signal intensity, coupling constants, etc) by fitting a serie of 1D spectra: "MultiNMRFit is a scientific tool designed to extract quantitative information (chemical shifts, signal intensity, coupling constants, etc) by fitting a serie of 1D spectra"
- [readme] estimation of uncertainty on estimated parameters (standard deviation): "estimation of **uncertainty** on estimated parameters (standard deviation)"
- [readme] account for overlaps between peaks and zero-order baseline correction: "account for **overlaps** between peaks and **zero-order baseline correction**"
