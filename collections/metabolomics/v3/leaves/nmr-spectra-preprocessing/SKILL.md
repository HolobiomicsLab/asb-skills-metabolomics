---
name: nmr-spectra-preprocessing
description: Use when a series of raw 1D-NMR metabolomics spectra must be transformed
  into an aligned, scaled, variance-stabilised matrix ready for multivariate statistical
  analysis or metabolic tracer data analysis.
when_to_use_negative:
- The data are 2D NMR spectra or HSQC/GC-MS isotopomer data for tracer analysis —
  those belong to the metabolabpytools isotopomer train/predict workflow, not 1D spectra
  pre-processing
- The spectra series is already aligned and scaled and the goal is downstream statistical
  modelling rather than pre-processing
- The task is GUI-based interactive processing rather than a scripted pipeline — the
  qtmetabolabpy package with its PySide6 GUI is the appropriate entry point
tools:
- name: metabolabpy
  role: 'Core package implementing the 1D-NMR pre-processing workflow: automated phase
    correction, segmental alignment, spectral scaling, variance stabilisation'
  repo: https://github.com/ludwigc/MetaboLabPy
- name: Python
  role: Implementation language; metabolabpy is used from Python alongside numpy,
    scipy, and pandas
- name: numpy
  role: Python toolbox used by MetaboLabPy for array-based spectral processing
- name: PySide6
  role: Used to produce the MetaboLabPy graphical user interface for interactive pre-processing
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.3390/metabo15010048
    title: MetaboLabPy—An Open-Source Software Package for Metabolomics NMR Data Processing
      and Metabolic Tracer Data Analysis
  source_content_hash: sha256:ce56b2c76dadd5b4c94d12ab6efae36fbb54a51462c9e40501f20b5df7a5463d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ludwigc/MetaboLabPy
  tools:
  - PySide6
  - Python
  - metabolabpy
  - numpy
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# nmr-spectra-preprocessing

## Summary

Pre-process a series of 1D-NMR spectra for metabolomics using MetaboLabPy (v0.9.66), a Python package that implements automated phase correction, segmental alignment, spectral scaling, and variance stabilisation to prepare spectra for multivariate statistical analysis.

## When to use

Use when a series of raw 1D-NMR metabolomics spectra must be transformed into an aligned, scaled, variance-stabilised matrix ready for multivariate statistical analysis or metabolic tracer data analysis.

## When NOT to use

- The data are 2D NMR spectra or HSQC/GC-MS isotopomer data for tracer analysis — those belong to the metabolabpytools isotopomer train/predict workflow, not 1D spectra pre-processing
- The spectra series is already aligned and scaled and the goal is downstream statistical modelling rather than pre-processing
- The task is GUI-based interactive processing rather than a scripted pipeline — the qtmetabolabpy package with its PySide6 GUI is the appropriate entry point

## Inputs

- Series of raw 1D-NMR spectra (e.g. loaded via the metabolabpy package as demonstrated in notebooks/workflow.ipynb)
- User-selected alignment reference spectrum
- User-defined spectral region for segmental alignment

## Outputs

- Phase-corrected, segmentally aligned, scaled, and variance-stabilised series of 1D-NMR spectra ready for multivariate statistical analysis
- Per-spectrum table of optimum segmental alignment shifts (e.g. segmental_alignment_shifts.csv from the reconstruction workflow)
- Export files in formats suitable for downstream analysis platforms

## How to apply

Install the metabolabpy package from PyPI (or clone https://github.com/ludwigc/MetaboLabPy) and follow the packaged demonstration in notebooks/workflow.ipynb to load a series of 1D NMR spectra. Apply the pre-processing chain in the order the software implements it: automated phase correction, then segmental alignment (each spectrum is shifted by simple correlation against a chosen reference spectrum within a user-selected spectral region; no shift is applied where the reference segment lacks signal), then spectral scaling, and finally variance stabilisation. Choose which spectra serve as the alignment reference and define the alignment region before running alignment, since these are user-selected parameters. Export the pre-processed data to the target analysis platform when the series is ready for downstream multivariate statistics.

## Related tools

- **metabolabpy** (Core package implementing the 1D-NMR pre-processing workflow: automated phase correction, segmental alignment, spectral scaling, variance stabilisation) — https://github.com/ludwigc/MetaboLabPy
- **Python** (Implementation language; metabolabpy is used from Python alongside numpy, scipy, and pandas)
- **numpy** (Python toolbox used by MetaboLabPy for array-based spectral processing)
- **PySide6** (Used to produce the MetaboLabPy graphical user interface for interactive pre-processing)

## Examples

```
pip install metabolabpy && metabolabpy --help
```

## Evaluation signals

- The pre-processing chain (phase correction → segmental alignment → scaling → variance stabilisation) has been run on every spectrum in the series with no spectra skipped silently
- For segmental alignment, one optimum shift entry exists per processed spectrum in the shift table (e.g. segmental_alignment_shifts.csv)
- Aligned segments in the user-selected spectral region visually overlay the reference spectrum (expert review)
- No alignment shift is applied to spectra whose reference-region signal is absent — verify skipped segments retain zero shift
- The processed output, when exported, is accepted by the target multivariate analysis platform without schema errors

## Limitations

- Segmental alignment performs no alignment when the reference signal is lacking in the selected region, so segments with weak reference peaks stay unshifted
- The README provides no changelog; version behaviour (e.g. of MetaboLabPy 0.9.66) should be pinned explicitly
- The optimum-shift search is based on simple correlation with the reference, so it may mis-align when the selected region contains little or non-overlapping signal

## Evidence

- [abstract] The software implements a complete workflow for NMR data pre-processing to prepare a series of 1D-NMR spectra for multi-variate statistical data analysis: "The software implements a complete workflow for NMR data pre-processing to prepare a series of 1D-NMR spectra for multi-variate statistical data analysis"
- [abstract] This includes a choice of algorithms for automated phase correction, segmental alignment, spectral scaling, variance stabilisation, export to various software platforms: "This includes a choice of algorithms for automated phase correction, segmental alignment, spectral scaling, variance stabilisation, export to various software platforms"
- [abstract] MetaboLabPy (version 0.9.66) is a free and open-source software package used to process 1D- and 2D-NMR spectra: "MetaboLabPy (version 0.9.66) is a free and open-source software package used to process 1D- and 2D-NMR spectra"
- [readme] Python package to process 1D and 2D NMR spectroscopic data for metabolomics and tracer based metabolism analysis: "Python package to process 1D and 2D NMR spectroscopic data for metabolomics and tracer based metabolism analysis"
