---
name: isotopic-labeling-pattern-validation
description: Use when a 1H-13C HSQC (pseudo-2D) isotopic-labeling dataset must be
  decomposed into isotopic species fractions and checked against a predicted labeling
  pattern from a standard of known composition.
when_to_use_negative:
- The data are ordinary 1D spectra of unlabeled metabolites where no isotopic-species
  decomposition is required
- Only raw unprocessed FIDs are available without a spectral series that can be peak-picked
  and fitted as multiplets
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: MultiNMRFit
  role: Semi-automated peak picking, multiplet signal fitting across the pseudo-2D
    HSQC series, and extraction of isotopic-species parameters with uncertainties
  repo: https://github.com/NMRTeamTBI/MultiNMRFit/
- name: pip
  role: Installs MultiNMRFit into the Python 3 environment (pip install multinmrfit)
- name: Python 3
  role: Runtime environment required to run MultiNMRFit (Python 3.8 or higher)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btaf463
    title: 'MultiNMRFit: a software to fit 1D and pseudo-2D NMR spectra'
  source_content_hash: sha256:ffcb8acf97c1414b773cc4ea41c9cc4b1d5f62637ac87cc6174a6a6325bc4bf5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NMRTeamTBI/MultiNMRFit
  tools:
  - MultiNMRFit
  - Python
  - Python 3
  - pip
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# isotopic-labeling-pattern-validation

## Summary

Quantify isotopic species (e.g. aspartate labeling states) from 1H-13C HSQC pseudo-2D NMR series using MultiNMRFit multiplet fitting, and validate the quantified fractions against a standard sample's predicted labeling pattern with differences below 1%.

## When to use

Use when a 1H-13C HSQC (pseudo-2D) isotopic-labeling dataset must be decomposed into isotopic species fractions and checked against a predicted labeling pattern from a standard of known composition.

## When NOT to use

- The data are ordinary 1D spectra of unlabeled metabolites where no isotopic-species decomposition is required
- Only raw unprocessed FIDs are available without a spectral series that can be peak-picked and fitted as multiplets

## Inputs

- 1H-13C HSQC pseudo-2D NMR series (e.g. a 13C-labeling aspartate standard dataset)
- Predicted labeling pattern for the standard sample (expected isotopic species fractions)

## Outputs

- aspartate_hsqc_fit_results.csv with per-species intensity, area, chemical shift, and standard-deviation uncertainties
- Visual inspection plots of fitted curves overlaid on experimental peaks

## How to apply

Install MultiNMRFit in a Python 3 environment with pip (pip install multinmrfit), then load the aspartate 1H-13C HSQC pseudo-2D series into its graphical interface (nmrfit). Use the semi-automated peak picking to define the multiplicity of each signal, then fit each signal across the spectral series with the built-in multiplet signal models, accounting for overlaps between peaks and applying zero-order baseline correction. Extract the estimated parameters (intensity, area, chemical shift) for each isotopic species together with their standard-deviation uncertainties and export them (e.g. to aspartate_hsqc_fit_results.csv). Validate by confirming the quantified isotopic species agree with the predicted labeling pattern within 1% and that the fitted curves pass visual inspection.

## Related tools

- **MultiNMRFit** (Semi-automated peak picking, multiplet signal fitting across the pseudo-2D HSQC series, and extraction of isotopic-species parameters with uncertainties) — https://github.com/NMRTeamTBI/MultiNMRFit/
- **pip** (Installs MultiNMRFit into the Python 3 environment (pip install multinmrfit))
- **Python 3** (Runtime environment required to run MultiNMRFit (Python 3.8 or higher))

## Examples

```
pip install multinmrfit; nmrfit
```

## Evaluation signals

- Quantified aspartate isotopic species fractions differ from the predicted labeling pattern by less than 1%
- Each estimated parameter carries a reported standard-deviation uncertainty
- Fitted curves pass visual inspection against the experimental HSQC peaks
- Peak picking correctly assigns the multiplicity of each signal in the series

## Limitations

- Validation was demonstrated on 1H, 13C, and 31P datasets from metabolomics and isotope labeling studies; other nuclei may require user-defined models
- Heavy peak overlap must be explicitly modeled; residual misfit is detected by visual inspection of fitted curves

## Evidence

- [abstract] isotope labeling demonstrated robustness: "We demonstrate the robustness and flexibility of MultiNMRFit on 1H, 13C, and 31P NMR datasets collected in metabolomics and isotope labeling studies."
- [methods] fitting purpose of the tool: "MultiNMRFit is a scientific tool designed to extract quantitative information (chemical shifts, signal intensity, coupling constants, etc) by fitting a serie of 1D spectra"
- [readme] peak picking and multiplicity: "semi-automated analysis for peak picking and definition of multiplicity for each signal"
- [readme] overlap and baseline handling: "account for overlaps between peaks and zero-order baseline correction,"
- [readme] estimated parameters: "estimation of several parameters for each signal of interest (intensity, area, chemical shift, linewidth, coupling constant(s), etc)"
