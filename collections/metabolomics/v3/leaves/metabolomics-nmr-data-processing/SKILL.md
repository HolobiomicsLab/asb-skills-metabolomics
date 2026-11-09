---
name: metabolomics-nmr-data-processing
description: Use when a series of 1D-NMR spectra must be phase-corrected and pre-processed
  (aligned, scaled, variance-stabilised) before multivariate statistical analysis,
  and a phase-corrected reference spectrum is available to seed ph0/ph1.
when_to_use_negative:
- The spectra are already phase-corrected or the input is already a feature/peak table
  ready for statistics
- The task is isotopomer distribution analysis from HSQC multiplet and GC-MS data
  (that belongs to the metabolabpytools deep-learning workflow, not 1D phase correction)
- Only a single 2D-NMR spectrum needs inspection rather than a 1D spectra series with
  a reference spectrum available
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3174
tools:
- name: MetaboLabPy 0.9.66
  role: Provides the nmrData object and automated phase correction algorithms used
    to process the 1D-NMR spectra series
  repo: https://github.com/ludwigc/MetaboLabPy
- name: scipy
  role: Supplies Powell's conjugate direction method (scipy.optimize) used to minimise
    the baseline-difference objective
- name: numpy
  role: Holds the spectral data arrays during optimisation
- name: pandas
  role: Collects the optimised ph0/ph1 values per spectrum into a table and writes
    the phase CSV
- name: PySide6
  role: Graphical user interface for MetaboLabPy used to run the processing workflow
  repo: https://github.com/ludwigc/MetaboLabPy
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.3390/metabo15010048
    title: MetaboLabPy—An Open-Source Software Package for Metabolomics NMR Data Processing
      and Metabolic Tracer Data Analysis
  source_content_hash: sha256:dc1292e51bf0b7edb5dd4360523a1f75459d734aa181b9088332c510eb14b1f9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ludwigc/MetaboLabPy
  tools:
  - MetaboLabPy 0.9.66
  - Protomix
  - PySide6
  - Python
  - numpy
  - pandas
  - scikit-learn
  - scipy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
merged_aliases:
- nmr-metabolomics-data-preprocessing
- nmr-metabolomics-preprocessing
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# metabolomics-nmr-data-processing

## Summary

A method for pre-processing series of 1D-NMR spectra for metabolomics using MetaboLabPy 0.9.66, centring on reference-spectrum-based automated phase correction (ph0/ph1 optimisation via Powell's method) together with segmental alignment, spectral scaling, and variance stabilisation.

## When to use

Use when a series of 1D-NMR spectra must be phase-corrected and pre-processed (aligned, scaled, variance-stabilised) before multivariate statistical analysis, and a phase-corrected reference spectrum is available to seed ph0/ph1.

## When NOT to use

- The spectra are already phase-corrected or the input is already a feature/peak table ready for statistics
- The task is isotopomer distribution analysis from HSQC multiplet and GC-MS data (that belongs to the metabolabpytools deep-learning workflow, not 1D phase correction)
- Only a single 2D-NMR spectrum needs inspection rather than a 1D spectra series with a reference spectrum available

## Inputs

- Series of 1D-NMR spectra loaded into a metabolabpy nmrData object
- User-defined phase-corrected reference spectrum (source of initial ph0/ph1)
- Baseline/noise edge regions: chemical shifts > 10.0 ppm (left) and < -0.5 ppm (right)

## Outputs

- Optimised per-spectrum zero-order (ph0) and first-order (ph1) phase values
- Phase correction table exported as CSV (e.g. phase3_phases.csv)
- Phase-corrected, pre-processed 1D-NMR spectra series ready for multivariate statistical analysis

## How to apply

Load the series of 1D-NMR spectra into MetaboLabPy's nmrData object (metabolabpy 0.9.66, Python). Initialise the zero-order (ph0) and first-order (ph1) phase values from the user-defined phase-corrected reference spectrum. Define the objective as the absolute difference between the baseline signal in the left region (chemical shifts > 10.0 ppm) and the right region (< -0.5 ppm) of each spectrum, and minimise it across the series using Powell's conjugate direction method from scipy.optimize, with spectral data held in numpy arrays. Collect the optimised ph0/ph1 values per spectrum into a pandas table and export (e.g. phase3_phases.csv). Then continue the MetaboLabPy pre-processing workflow with segmental alignment, spectral scaling, and variance stabilisation before export to downstream analysis platforms.

## Related tools

- **MetaboLabPy 0.9.66** (Provides the nmrData object and automated phase correction algorithms used to process the 1D-NMR spectra series) — https://github.com/ludwigc/MetaboLabPy
- **scipy** (Supplies Powell's conjugate direction method (scipy.optimize) used to minimise the baseline-difference objective)
- **numpy** (Holds the spectral data arrays during optimisation)
- **pandas** (Collects the optimised ph0/ph1 values per spectrum into a table and writes the phase CSV)
- **PySide6** (Graphical user interface for MetaboLabPy used to run the processing workflow) — https://github.com/ludwigc/MetaboLabPy

## Examples

```
from scipy.optimize import minimize; res = minimize(baseline_diff_objective, x0=[ph0, ph1], method='Powell')
```

## Evaluation signals

- After optimisation, the baseline in both the > 10.0 ppm and < -0.5 ppm regions must be flat and mutually consistent (minimal absolute difference), confirming successful phase correction
- Each spectrum in the series receives its own optimised ph0/ph1 pair, traceable to the reference-spectrum initialisation
- The exported phase table (e.g. phase3_phases.csv) contains one row per spectrum with ph0 and ph1 columns populated by the Powell optimiser
- The corrected series is suitable input for the downstream MetaboLabPy steps (segmental alignment, spectral scaling, variance stabilisation) and export to statistical analysis platforms

## Limitations

- The phase objective relies on the left (> 10.0 ppm) and right (< -0.5 ppm) edge regions serving as baseline; residual signal or distortion in those regions undermines the optimisation
- The algorithm requires a user-defined phase-corrected reference spectrum to initialise ph0/ph1; without a good reference, per-spectrum phases may converge poorly
- READMEs in the context are truncated and no changelog is available, so exact parameter defaults must be taken from the online documentation

## Evidence

- [abstract] a choice of algorithms for automated phase correction, segmental alignment, spectral scaling, variance stabilisation, export to various software platforms: "This includes a choice of algorithms for automated phase correction, segmental alignment, spectral scaling, variance stabilisation, export to various software platforms, and analysis of metabolic"
- [abstract] The software is implemented in Python and uses numerous Python toolboxes, such as numpy, scipy, pandas, etc.: "The software is implemented in Python and uses numerous Python toolboxes, such as numpy, scipy, pandas, etc."
- [readme] Python package to process 1D and 2D NMR spectroscopic data for metabolomics and tracer based metabolism analysis: "Python package to process 1D and 2D NMR spectroscopic data for metabolomics and tracer based metabolism analysis"
