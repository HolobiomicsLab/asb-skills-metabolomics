---
name: metabolomics-nmr-spectra-quality-control
description: Use when a series of 1D-NMR spectra (e.g. Daphnia magna extracts) must
  be prepared for multi-variate statistical analysis and peaks drift between spectra,
  requiring alignment QC.
when_to_use_negative:
- Input is a single 2D-NMR spectrum rather than a series of 1D-NMR spectra needing
  drift correction before multi-variate analysis
- Data are tracer-based metabolism or GC-MS isotopomer data handled by the metabolabpytools
  routines rather than 1D-NMR pre-processing
- Spectra are already aligned (post-alignment correlations already exceed 0.95) so
  segmental alignment would add no correction
edam_operation: http://edamontology.org/operation_0324
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: MetaboLabPy 0.9.66
  role: Runs the segmental alignment and spectral pre-processing (phase correction,
    scaling, variance stabilisation) on the 1D-NMR spectra series
  repo: https://github.com/ludwigc/MetaboLabPy
- name: MetaboLabPy
  role: Python package providing the segmental alignment algorithm used to remove
    chemical-shift drift relative to the reference spectrum
  repo: https://github.com/ludwigc/MetaboLabPy
- name: numpy
  role: Numerical backend used by MetaboLabPy for computing spectra operations and
    Pearson correlation coefficients
- name: scipy
  role: Scientific backend used by MetaboLabPy for spectra processing during pre-processing
- name: pandas
  role: Backend used to tabulate and write the pre/post-alignment correlation coefficients
    to alignment_correlations.csv
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.3390/metabo15010048
    title: MetaboLabPy—An Open-Source Software Package for Metabolomics NMR Data Processing
      and Metabolic Tracer Data Analysis
  source_content_hash: sha256:d2f3bb27a3f2802dfbe1ed8b2df8013d26ecbfc3a36c1f139e1c744558180151
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ludwigc/MetaboLabPy
  tools:
  - MetaboLabPy
  - MetaboLabPy 0.9.66
  - numpy
  - pandas
  - scipy
  edam_operation: http://edamontology.org/operation_0324
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5263157894736842
n_claims: 19
---

# metabolomics-nmr-spectra-quality-control

## Summary

Quality-control workflow for a series of 1D-NMR metabolomics spectra that removes chemical-shift drift using MetaboLabPy's segmental alignment and quantifies success via Pearson correlation against a high-SNR reference spectrum.

## When to use

Use when a series of 1D-NMR spectra (e.g. Daphnia magna extracts) must be prepared for multi-variate statistical analysis and peaks drift between spectra, requiring alignment QC.

## When NOT to use

- Input is a single 2D-NMR spectrum rather than a series of 1D-NMR spectra needing drift correction before multi-variate analysis
- Data are tracer-based metabolism or GC-MS isotopomer data handled by the metabolabpytools routines rather than 1D-NMR pre-processing
- Spectra are already aligned (post-alignment correlations already exceed 0.95) so segmental alignment would add no correction

## Inputs

- Series of 1D-NMR spectra (e.g. Daphnia magna extract spectra) loadable by MetaboLabPy 0.9.66
- User-selected alignment spectral region
- Reference spectrum chosen as the highest signal-to-noise-ratio spectrum in the series

## Outputs

- Segmentally aligned spectra series with chemical-shift drift removed relative to the reference
- alignment_correlations.csv tabulating pre- and post-alignment Pearson correlation coefficients against the reference

## How to apply

Load the 1D-NMR spectra series into MetaboLabPy 0.9.66 (Python backend with numpy/scipy/pandas). Select the alignment region across the spectra and choose the spectrum with the highest signal-to-noise ratio as the alignment reference. Run MetaboLabPy's segmental alignment algorithm on the selected region to remove chemical-shift drift relative to that reference. The alignment optimum is judged by correlation: Pearson correlation coefficients between each spectrum and the reference are generally below 0.3 before alignment and should rise to >0.95 after alignment. Compute both sets of coefficients, tabulate them, and write the table to alignment_correlations.csv as the QC evidence. This step is part of MetaboLabPy's complete pre-processing workflow that also covers automated phase correction, spectral scaling, and variance stabilisation before export for statistical analysis.

## Related tools

- **MetaboLabPy 0.9.66** (Runs the segmental alignment and spectral pre-processing (phase correction, scaling, variance stabilisation) on the 1D-NMR spectra series) — https://github.com/ludwigc/MetaboLabPy
- **MetaboLabPy** (Python package providing the segmental alignment algorithm used to remove chemical-shift drift relative to the reference spectrum) — https://github.com/ludwigc/MetaboLabPy
- **numpy** (Numerical backend used by MetaboLabPy for computing spectra operations and Pearson correlation coefficients)
- **scipy** (Scientific backend used by MetaboLabPy for spectra processing during pre-processing)
- **pandas** (Backend used to tabulate and write the pre/post-alignment correlation coefficients to alignment_correlations.csv)

## Examples

```
metabolabpy --help
```

## Evaluation signals

- Pre-alignment Pearson correlation coefficients between each spectrum and the high-SNR reference in the alignment region are generally below 0.3
- Post-alignment correlations between each spectrum and the reference exceed 0.95
- The reference spectrum is the one with the highest signal-to-noise ratio in the series
- Both pre- and post-alignment coefficient sets are tabulated and written to alignment_correlations.csv

## Limitations

- Segmental alignment corrects drift only within the user-selected region relative to a single reference spectrum; regions outside the selection are not corrected
- Validation depends on the reported thresholds (pre-alignment <0.3, post-alignment >0.95) matching the demonstrated Daphnia magna dataset; other datasets may not reach these values
- MetaboLabPy is a large multi-package toolkit (metabolabpy, qtmetabolabpy, metabolabpytools); only the metabolabpy pre-processing components apply to this skill

## Evidence

- [abstract] MetaboLabPy implements a complete workflow for NMR data pre-processing to prepare 1D-NMR spectra series for multivariate statistical analysis: "The software implements a complete workflow for NMR data pre-processing to prepare a series of 1D-NMR spectra for multi-variate statistical data analysis"
- [abstract] Segmental alignment is one of the pre-processing algorithms offered: "This includes a choice of algorithms for automated phase correction, segmental alignment, spectral scaling, variance stabilisation"
- [abstract] MetaboLabPy version and implementation backend: "MetaboLabPy (version 0.9.66) is a free and open-source software package used to process 1D- and 2D-NMR spectra"
- [readme] Package purpose stated in the repository README: "Python package to process 1D and 2D NMR spectroscopic data for metabolomics and tracer based metabolism analysis"
