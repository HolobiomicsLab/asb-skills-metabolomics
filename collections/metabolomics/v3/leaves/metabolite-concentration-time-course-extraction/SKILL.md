---
name: metabolite-concentration-time-course-extraction
description: Use when you have a time-series of 1D NMR spectra (or a pseudo-2D experiment
  with hundreds to >1000 increments) and need per-spectrum metabolite concentrations
  including for overlapping multiplets such as ADP/ATP.
when_to_use_negative:
- The data are not spectra from a single NMR series (e.g. already-processed feature/concentration
  tables, or 2D NMR spectra with full indirect dimensions) — MultiNMRFit fits series
  of 1D or pseudo-2D spectra only
- The task is de novo metabolite identification or assignment of unknown peaks rather
  than quantification of pre-defined signals of interest
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3172
tools:
- name: MultiNMRFit
  role: Open-source Python software used to fit the spectral series with multiplet
    signal models, handle peak overlaps and zero-order baseline correction, and extract
    signal parameters with uncertainties via the nmrfit GUI or library API
  repo: https://github.com/NMRTeamTBI/MultiNMRFit/
- name: Python 3.8+
  role: Runtime environment for MultiNMRFit; the package is also usable as a Python
    library for scripted fitting of the spectral series
- name: pip
  role: 'Installation channel: `pip install multinmrfit` pulls the package from PyPI/GitHub'
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btaf463
    title: 'MultiNMRFit: a software to fit 1D and pseudo-2D NMR spectra'
  source_content_hash: sha256:098ea8ebb95ee7390d50b553c335b94f901bca48e8704c68055dec6be26de6e7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NMRTeamTBI/MultiNMRFit
  tools:
  - MultiNMRFit
  - Python
  - Python 3.8+
  - pip
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# metabolite-concentration-time-course-extraction

## Summary

Extract quantitative NMR signal parameters (intensity, area, chemical shift, coupling constants) from large series of 1D or pseudo-2D spectra using MultiNMRFit's multiplet signal models, and convert fitted signal areas into metabolite concentration time courses — e.g. quantifying GLC, G6P, F6P, FBP, ATP, and ADP across 1024 spectra of a real-time glycolysis 1H experiment in under 3 hours.

## When to use

Use when you have a time-series of 1D NMR spectra (or a pseudo-2D experiment with hundreds to >1000 increments) and need per-spectrum metabolite concentrations including for overlapping multiplets such as ADP/ATP.

## When NOT to use

- The data are not spectra from a single NMR series (e.g. already-processed feature/concentration tables, or 2D NMR spectra with full indirect dimensions) — MultiNMRFit fits series of 1D or pseudo-2D spectra only
- The task is de novo metabolite identification or assignment of unknown peaks rather than quantification of pre-defined signals of interest

## Inputs

- A series of 1D NMR spectra acquired individually, as a pseudo-2D spectrum (e.g. 1024 pseudo-2D 1H spectra), or provided as tabulated text files
- Peak list / multiplicity definitions for each metabolite signal of interest (e.g. GLC, G6P, F6P, FBP, ATP, ADP), typically defined via the semi-automated peak picking GUI

## Outputs

- Per-spectrum fitted signal parameters (intensity, area, chemical shift, linewidth, coupling constants) with uncertainties (standard deviations)
- Metabolite concentration time courses (e.g. concentration_time_courses.csv) for each signal of interest
- Graphical overlays of fitted curves against experimental spectra for visual inspection

## How to apply

Install the software with `pip install multinmrfit` (requires Python 3.8+) and launch the graphical interface with `nmrfit` to load the pseudo-2D 1H dataset. Use the semi-automated analysis for peak picking and definition of multiplicity for each signal of interest, then fit the full spectral series with built-in multiplet signal models, explicitly accounting for overlaps between peaks (e.g. the overlapping ADP and ATP signals) and applying zero-order baseline correction. Extract the estimated intensity/area parameters together with their uncertainties (standard deviations) for each metabolite in every spectrum, then convert fitted signal areas/intensities into concentration time courses for the target metabolites. Visually inspect the fitted curves against the experimental spectra to confirm overlapping multiplets are correctly captured before trusting the extracted concentrations.

## Related tools

- **MultiNMRFit** (Open-source Python software used to fit the spectral series with multiplet signal models, handle peak overlaps and zero-order baseline correction, and extract signal parameters with uncertainties via the nmrfit GUI or library API) — https://github.com/NMRTeamTBI/MultiNMRFit/
- **Python 3.8+** (Runtime environment for MultiNMRFit; the package is also usable as a Python library for scripted fitting of the spectral series)
- **pip** (Installation channel: `pip install multinmrfit` pulls the package from PyPI/GitHub)

## Examples

```
pip install multinmrfit; nmrfit
```

## Evaluation signals

- Visual inspection of fitted curves against experimental spectra confirms overlapping multiplets (e.g. ADP and ATP peaks) are correctly captured
- All signal parameters are successfully extracted for all spectra in the series, with no missing fits for the metabolites of interest
- Estimated parameters are accompanied by uncertainties (standard deviations) for every signal in every spectrum
- The full series (e.g. 1024 spectra) is processed within the reported throughput of ~3 hours
- Converted concentration time courses for all six metabolites (GLC, G6P, F6P, FBP, ATP, ADP) are complete and consistent with the fitted areas

## Limitations

- MultiNMRFit fits 1D spectra acquired individually or as pseudo-2D experiments; it is not designed for fully 2D NMR spectra
- Signals must be predefined; peak picking and multiplicity definition are semi-automated and require user definition of each signal of interest
- Baseline handling is limited to zero-order baseline correction

## Evidence

- [abstract] MultiNMRFit extracts signal parameters (e.g. intensity, area, chemical shift, and coupling constants) by fitting the experimental spectra using built-in or user-defined signal models: "MultiNMRFit extracts signal parameters (e.g. intensity, area, chemical shift, and coupling constants) by fitting the experimental spectra using built-in or user-defined signal models that account for"
- [readme] fit series of 1D spectra (acquired as individual 1D spectra, as a pseudo 2D spectrum, or provided as tabulated text files): "**fit series of 1D spectra** (acquired as individual 1D spectra, as a pseudo 2D spectrum, or provided as tabulated text files),"
- [readme] account for overlaps between peaks and zero-order baseline correction: "account for **overlaps** between peaks and **zero-order baseline correction**,"
- [readme] estimation of uncertainty on estimated parameters (standard deviation): "estimation of **uncertainty** on estimated parameters (standard deviation),"
- [readme] Use `pip` to install PhysioFit from GitHub: $ pip install multinmrfit: "Use `pip` to **install PhysioFit from GitHub**:

bash
$ pip install multinmrfit
"
