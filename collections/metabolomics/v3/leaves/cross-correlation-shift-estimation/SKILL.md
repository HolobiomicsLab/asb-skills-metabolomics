---
name: cross-correlation-shift-estimation
description: Use when a series of 1D NMR spectra must be aligned prior to multivariate
  analysis and individual segments show chemical-shift offsets relative to a designated
  reference spectrum.
when_to_use_negative:
- The segment has no reference signal — MetaboLabPy performs no alignment when reference
  signal is lacking in the chosen region
- The input is 2D-NMR data or a metabolite identification/isotopomer task — use the
  metabolabpytools package for isotopomer distributions from NMR multiplet and GC-MS
  data instead
- Spectra are already shift-aligned and the goal is another pre-processing step such
  as spectral scaling or variance stabilisation
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_0593
tools:
- name: metabolabpy
  role: Provides the segmental alignment routine that computes the optimum shift per
    spectrum by simple correlation with the chosen reference spectrum
  repo: https://github.com/ludwigc/MetaboLabPy
- name: numpy
  role: Python toolbox used to implement the correlation-based shift search over candidate
    spectral shifts
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.3390/metabo15010048
    title: MetaboLabPy—An Open-Source Software Package for Metabolomics NMR Data Processing
      and Metabolic Tracer Data Analysis
  source_content_hash: sha256:352cc9dbdcc78653d2e6dc04e678292c21eacda5ee635ab5005681db1cb3982f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ludwigc/MetaboLabPy
  tools:
  - metabolabpy
  - numpy
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_0593
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# cross-correlation-shift-estimation

## Summary

Estimates the optimum chemical-shift offset for each spectrum in a 1D-NMR series by simple cross-correlation of a user-selected spectral segment against a chosen reference spectrum, as implemented in MetaboLabPy's segmental alignment. It removes pH/instrument-induced shift variation before multivariate statistical analysis of metabolomics spectra.

## When to use

Use when a series of 1D NMR spectra must be aligned prior to multivariate analysis and individual segments show chemical-shift offsets relative to a designated reference spectrum.

## When NOT to use

- The segment has no reference signal — MetaboLabPy performs no alignment when reference signal is lacking in the chosen region
- The input is 2D-NMR data or a metabolite identification/isotopomer task — use the metabolabpytools package for isotopomer distributions from NMR multiplet and GC-MS data instead
- Spectra are already shift-aligned and the goal is another pre-processing step such as spectral scaling or variance stabilisation

## Inputs

- Series of 1D NMR spectra loaded via the metabolabpy package
- One spectrum designated as the alignment reference
- User-selected spectral region (segment) to align

## Outputs

- Per-spectrum optimum shift table written to segmental_alignment_shifts.csv
- Aligned spectra in the selected region, overlaid on the reference spectrum

## How to apply

Load the metabolabpy package (installable from PyPI; see notebooks/workflow.ipynb for the demonstration workflow) and read in a series of 1D NMR spectra. Designate one spectrum as the alignment reference and define the user-selected spectral region to align. For each spectrum, compute simple correlation between the selected segment and the reference across candidate spectral shifts, and record the shift that maximizes correlation as the optimum. Skip alignment for any spectrum/segment where the reference region lacks sufficient signal, since no meaningful correlation maximum exists there. Apply the optimum shifts to align all spectra to the reference and write the per-spectrum shift table to segmental_alignment_shifts.csv.

## Related tools

- **metabolabpy** (Provides the segmental alignment routine that computes the optimum shift per spectrum by simple correlation with the chosen reference spectrum) — https://github.com/ludwigc/MetaboLabPy
- **numpy** (Python toolbox used to implement the correlation-based shift search over candidate spectral shifts)

## Examples

```
pip install metabolabpy
```

## Evaluation signals

- segmental_alignment_shifts.csv exists with exactly one optimum-shift entry per processed spectrum
- Expert review confirms aligned segments in the selected region visually overlay the reference spectrum
- For each spectrum the recorded shift is the candidate offset that maximizes the simple correlation with the reference segment
- Spectra/segments where the reference region lacks signal remain unaligned (no shift applied), matching the no-allocation rule

## Limitations

- No alignment is performed when the reference spectrum has no signal in the selected segment, so those spectra stay misaligned
- The optimizer relies on simple correlation, so shifts can be misestimated in signal-poor or overlapping-peak segments

## Evidence

- [abstract] This includes a choice of algorithms for automated phase correction, segmental alignment, spectral scaling, variance stabilisation, export to various software platforms,: "This includes a choice of algorithms for automated phase correction, segmental alignment, spectral scaling, variance stabilisation, export to various software platforms,"
- [abstract] MetaboLabPy (version 0.9.66) is a free and open-source software package used to process 1D- and 2D-NMR spectra: "MetaboLabPy (version 0.9.66) is a free and open-source software package used to process 1D- and 2D-NMR spectra"
- [readme] Python package to process 1D and 2D NMR spectroscopic data for metabolomics and tracer based metabolism analysis: "Python package to process 1D and 2D NMR spectroscopic data for metabolomics and tracer based metabolism analysis"
- [abstract] The software implements a complete workflow for NMR data pre-processing to prepare a series of 1D-NMR spectra for multi-variate statistical data analysis: "The software implements a complete workflow for NMR data pre-processing to prepare a series of 1D-NMR spectra for multi-variate statistical data analysis"
