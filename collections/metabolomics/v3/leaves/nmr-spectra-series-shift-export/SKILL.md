---
name: nmr-spectra-series-shift-export
description: Use when a series of 1D-NMR spectra shows chemical-shift variability
  that must be corrected before multivariate statistical analysis, and per-spectrum
  optimum shifts need to be computed and exported.
when_to_use_negative:
- The spectra series is intended for something other than pre-processing toward multi-variate
  statistical analysis (e.g. tracer-based metabolism analysis via metabolabpytools,
  which is a separate package).
- A spectrum's selected region lacks sufficient reference signal — segmental alignment
  performs no alignment in that case and must be skipped.
- Only 2D-NMR processing is needed; this skill addresses the 1D-NMR segmental alignment
  step of the pre-processing workflow.
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3272
tools:
- name: metabolabpy
  role: Provides the segmental alignment routine that computes the correlation-maximising
    optimum shift per spectrum within the user-selected region
  repo: https://github.com/ludwigc/metabolabpy
- name: Python
  role: Implementation language for the pre-processing workflow, using toolboxes such
    as numpy, scipy and pandas
- name: numpy
  role: Python toolbox used in MetaboLabPy for the array/correlation computations
    underlying alignment
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.3390/metabo15010048
    title: MetaboLabPy—An Open-Source Software Package for Metabolomics NMR Data Processing
      and Metabolic Tracer Data Analysis
  source_content_hash: sha256:a5413f54b4124b4eec0ac324c90fc0b7b616085edad5785fccaf0338f37e9d2a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ludwigc/metabolabpy
  tools:
  - Python
  - metabolabpy
  - numpy
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3272
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# nmr-spectra-series-shift-export

## Summary

Determines and exports the optimum spectral shift for each 1D-NMR spectrum in a series by simple correlation with a chosen reference spectrum within a user-selected region, as implemented in MetaboLabPy's segmental alignment.

## When to use

Use when a series of 1D-NMR spectra shows chemical-shift variability that must be corrected before multivariate statistical analysis, and per-spectrum optimum shifts need to be computed and exported.

## When NOT to use

- The spectra series is intended for something other than pre-processing toward multi-variate statistical analysis (e.g. tracer-based metabolism analysis via metabolabpytools, which is a separate package).
- A spectrum's selected region lacks sufficient reference signal — segmental alignment performs no alignment in that case and must be skipped.
- Only 2D-NMR processing is needed; this skill addresses the 1D-NMR segmental alignment step of the pre-processing workflow.

## Inputs

- Series of 1D-NMR spectra loadable via the metabolabpy package (demonstration data from notebooks/workflow.ipynb)
- User-designated reference spectrum from the series
- User-selected spectral region (segment) to align

## Outputs

- Aligned spectral series in which each segment overlays the reference within the selected region
- Per-spectrum shift table written to segmental_alignment_shifts.csv with one optimum-shift entry per processed spectrum

## How to apply

Load the metabolabpy package (installable from PyPI as 'metabolabpy') and read in the 1D-NMR spectra series, following the packaged workflow example in notebooks/workflow.ipynb. Designate one spectrum as the alignment reference and select the user-defined spectral region to align. For each spectrum in the series, compute simple correlation between the selected segment and the reference across candidate spectral shifts, and record the shift that maximizes correlation as the optimum. Skip alignment for any spectrum/segment where the reference region lacks sufficient signal — no alignment occurs when reference signal is lacking. Apply the optimum shifts to align all spectra, then write the per-spectrum shift table to segmental_alignment_shifts.csv. This is one stage of MetaboLabPy's pre-processing workflow (alongside automated phase correction, spectral scaling and variance stabilisation) that prepares a 1D-NMR series for multi-variate statistical data analysis.

## Related tools

- **metabolabpy** (Provides the segmental alignment routine that computes the correlation-maximising optimum shift per spectrum within the user-selected region) — https://github.com/ludwigc/metabolabpy
- **Python** (Implementation language for the pre-processing workflow, using toolboxes such as numpy, scipy and pandas)
- **numpy** (Python toolbox used in MetaboLabPy for the array/correlation computations underlying alignment)

## Examples

```
pip install metabolabpy; python -c "import metabolabpy; from metabolabpy.nmr import dataset; d=dataset.NmrData(); # align segment vs reference, export segmental_alignment_shifts.csv"
```

## Evaluation signals

- segmental_alignment_shifts.csv exists and contains exactly one optimum-shift entry per processed spectrum
- By expert review, aligned segments in the user-selected region overlay the reference spectrum
- Spectra whose selected region lacked reference signal show no shift applied (skipped alignment)
- The aligned series is suitable downstream for multi-variate statistical data analysis as described in the MetaboLabPy workflow

## Limitations

- No alignment occurs when the reference signal is lacking in the selected region — such spectra/segments are skipped rather than shifted.
- The optimum is found by simple correlation with a single chosen reference spectrum, so results depend on the reference choice and the user-selected region.
- No changelog found for the package (per the discussion section).

## Evidence

- [methods] Segmental alignment computes the optimum shift for each spectrum by simple correlation with the chosen reference spectrum; no alignment occurs when reference signal is lacking.: [evidence span withheld — anchor status: not_found]
- [abstract] The software implements a complete workflow for NMR data pre-processing to prepare a series of 1D-NMR spectra for multi-variate statistical data analysis: "The software implements a complete workflow for NMR data pre-processing to prepare a series of 1D-NMR spectra for multi-variate statistical data analysis"
- [abstract] automated phase correction, segmental alignment, spectral scaling, variance stabilisation: "automated phase correction, segmental alignment, spectral scaling, variance stabilisation"
- [readme] Python package to process 1D and 2D NMR spectroscopic data for metabolomics and tracer based metabolism analysis: "Python package to process 1D and 2D NMR spectroscopic data for metabolomics and tracer based metabolism analysis"
- [abstract] MetaboLabPy is available via the Python Package Index or via GitHub: "MetaboLabPy is available via the Python Package Index or via GitHub"
