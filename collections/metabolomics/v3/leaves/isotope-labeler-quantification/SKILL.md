---
name: isotope-labeler-quantification
description: Use when a series of 1D or pseudo-2D 1H NMR spectra from isotope labeling
  experiments (e.g., E. coli fed 1-13C1-glucose, U-13C6-glucose, and 2-13C1-acetate)
  must be decomposed into overlapping isotopomer multiplets and converted to time-course
  concentrations.
when_to_use_negative:
- The spectra are full 2D correlation experiments rather than 1D or pseudo-2D (series
  of 1D) acquisitions — outside MultiNMRFit's fitting scope
- The input is already a quantified concentration table (e.g., mM values per isotopomer)
  rather than spectra — no multiplet fitting is needed
- A single unsplit resonance with no isotope-dependent multiplet structure must be
  quantified — simple integration suffices
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0154
tools:
- name: MultiNMRFit
  role: Fits built-in multiplet signal models to each spectrum in the series, estimating
    area, chemical shift, and coupling constants of each acetate isotopomer component
    with standard deviations, handling peak overlaps and zero-order baseline correction
  repo: https://github.com/NMRTeamTBI/MultiNMRFit/
- name: Python
  role: Runtime (3.8 or higher) in which MultiNMRFit is installed and used as a library
    to script the series fitting and export the concentration table
- name: pip
  role: Installs the multinmrfit package before running the graphical interface or
    library workflow
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btaf463
    title: 'MultiNMRFit: a software to fit 1D and pseudo-2D NMR spectra'
  source_content_hash: sha256:2aa56af06b4edd797d923eecaff8abc01432f54441c74f93750455f2efc8209f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NMRTeamTBI/MultiNMRFit
  tools:
  - MultiNMRFit
  - Python
  - pip
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# isotope-labeler-quantification

## Summary

Quantifies time courses of isotopic species (e.g., 13C isotopomers of acetate) from series of 1D or pseudo-2D 1H NMR spectra by fitting overlapping multiplet components with MultiNMRFit and converting fitted areas to millimolar concentrations with reported standard deviations.

## When to use

Use when a series of 1D or pseudo-2D 1H NMR spectra from isotope labeling experiments (e.g., E. coli fed 1-13C1-glucose, U-13C6-glucose, and 2-13C1-acetate) must be decomposed into overlapping isotopomer multiplets and converted to time-course concentrations.

## When NOT to use

- The spectra are full 2D correlation experiments rather than 1D or pseudo-2D (series of 1D) acquisitions — outside MultiNMRFit's fitting scope
- The input is already a quantified concentration table (e.g., mM values per isotopomer) rather than spectra — no multiplet fitting is needed
- A single unsplit resonance with no isotope-dependent multiplet structure must be quantified — simple integration suffices

## Inputs

- Series of 1D NMR spectra covering one signal region (e.g., the acetate methyl multiplet), acquired as individual 1D spectra, a pseudo-2D spectrum, or tabulated text files
- Peak-picking results / multiplicity definitions for the isotopomer multiplet components (semi-automated in MultiNMRFit)
- Calibration factor for converting fitted signal areas to millimolar concentrations

## Outputs

- Per-spectrum fitted signal parameters: area, intensity, chemical shift, linewidth, coupling constants, each with standard deviations
- Time-course concentrations (mM) of each isotopic species and the total metabolite pool, e.g., written to acetate_quantification.csv
- Plots of fitted curves overlaid on the experimental spectra for visual validation

## How to apply

Install MultiNMRFit via pip (requires Python 3.8 or higher) and load the series of 1D spectra covering the region of interest, e.g., the acetate methyl multiplet — acquired as individual 1D spectra, a pseudo-2D spectrum, or tabulated text files. Use the semi-automated peak picking and multiplicity definition to place the multiplet signals of each isotopic species in every spectrum of the series, then fit each spectrum with the built-in multiplet signal model, accounting for overlaps between peaks and applying zero-order baseline correction. Record the fitted area, chemical shift, and coupling constant of each isotopomer component together with the parameter uncertainties (standard deviations) reported by MultiNMRFit. Convert fitted signal areas to millimolar concentrations using a calibration, then visually inspect the fitted curves against the experimental spectra to confirm agreement. For the E. coli acetate case, the four isotopic species should reproduce a stable total acetate of 8.5 ± 0.1 mM while the 2-13C1-acetate fraction declines and glucose-derived labeled forms rise.

## Related tools

- **MultiNMRFit** (Fits built-in multiplet signal models to each spectrum in the series, estimating area, chemical shift, and coupling constants of each acetate isotopomer component with standard deviations, handling peak overlaps and zero-order baseline correction) — https://github.com/NMRTeamTBI/MultiNMRFit/
- **Python** (Runtime (3.8 or higher) in which MultiNMRFit is installed and used as a library to script the series fitting and export the concentration table)
- **pip** (Installs the multinmrfit package before running the graphical interface or library workflow)

## Examples

```
pip install multinmrfit && nmrfit
```

## Evaluation signals

- Total acetate concentration remains constant across the series at 8.5 ± 0.1 mM
- 2-13C1-acetate declines from 8.5 to 6.5 mM while glucose-derived labeled forms rise from 0.1 to 1.9 mM
- Fitted curves visually match the experimental spectra in every spectrum of the series
- Each estimated parameter carries a MultiNMRFit-reported standard deviation, and chemical shifts/coupling constants stay consistent across the time series

## Limitations

- MultiNMRFit fits 1D and pseudo-2D (series of 1D) spectra only; full 2D correlation experiments are out of scope
- The article provides no discussion-section limitations (no changelog or limitations text available); operational caveats must be checked against the documentation at multinmrfit.readthedocs.io
- Absolute concentrations additionally depend on a user-supplied calibration to convert fitted areas to millimolar units

## Evidence

- [abstract] MultiNMRFit extracts signal parameters by fitting spectra with signal models: "MultiNMRFit extracts signal parameters (e.g. intensity, area, chemical shift, and coupling constants) by fitting the experimental spectra using built-in or user-defined signal models"
- [abstract] Demonstrated on 1H, 13C, and 31P datasets from isotope labeling studies: "We demonstrate the robustness and flexibility of MultiNMRFit on 1H, 13C, and 31P NMR datasets collected in metabolomics and isotope labeling studies."
- [methods] Tool purpose: quantitative extraction from series of 1D spectra: "MultiNMRFit is a scientific tool designed to extract quantitative information (chemical shifts, signal intensity, coupling constants, etc) by fitting a serie of 1D spectra"
- [readme] Supported input acquisition modes: "fit series of 1D spectra (acquired as individual 1D spectra, as a pseudo 2D spectrum, or provided as tabulated text files)"
- [readme] Overlap and baseline handling during fitting: "account for overlaps between peaks and zero-order baseline correction"
- [readme] Uncertainty estimation on fitted parameters: "estimation of uncertainty on estimated parameters (standard deviation)"
