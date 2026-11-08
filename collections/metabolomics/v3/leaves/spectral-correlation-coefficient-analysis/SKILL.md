---
name: spectral-correlation-coefficient-analysis
description: Use when a series of 1D-NMR spectra (e.g. Daphnia magna extracts) shows
  chemical-shift drift and you need to measure alignment improvement via Pearson correlation
  against a chosen reference spectrum in a spectral region.
when_to_use_negative:
- The data are 2D NMR or GC-MS isotopomer data intended for the metabolabpytools isotopomer-distribution
  routines rather than 1D-NMR pre-processing
- The spectra series is already aligned relative to the reference, so pre-alignment
  correlations would not show the expected low (<0.3) baseline
- No single high-SNR reference spectrum can be identified, since the correlation comparison
  is defined relative to one reference spectrum
edam_operation: http://edamontology.org/operation_0314
edam_topics:
- http://edamontology.org/topic_0593
tools:
- name: MetaboLabPy 0.9.66
  role: Runs the segmental alignment of the 1D-NMR spectra series against the high-SNR
    reference spectrum in the selected region
  repo: https://github.com/ludwigc/MetaboLabPy
- name: numpy
  role: Python toolbox used by MetaboLabPy for numerical computation on the spectral
    arrays
  repo: https://github.com/ludwigc/MetaboLabPy
- name: pandas
  role: Python toolbox used to tabulate the before/after correlation coefficients
    into alignment_correlations.csv
  repo: https://github.com/ludwigc/MetaboLabPy
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.3390/metabo15010048
    title: MetaboLabPy—An Open-Source Software Package for Metabolomics NMR Data Processing
      and Metabolic Tracer Data Analysis
  source_content_hash: sha256:0e12da1ae06c9b053d8caf868c118bc7fcf5212df410fc824d4e54407ec34f5c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ludwigc/MetaboLabPy
  tools:
  - MetaboLabPy 0.9.66
  - numpy
  - pandas
  - scipy
  edam_operation: http://edamontology.org/operation_0314
  edam_topics:
  - http://edamontology.org/topic_0593
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# spectral-correlation-coefficient-analysis

## Summary

Quantifies chemical-shift alignment quality in a 1D-NMR spectra series by computing Pearson correlation coefficients of each spectrum against a high-signal-to-noise reference spectrum before and after MetaboLabPy's segmental alignment, and tabulating both sets of coefficients.

## When to use

Use when a series of 1D-NMR spectra (e.g. Daphnia magna extracts) shows chemical-shift drift and you need to measure alignment improvement via Pearson correlation against a chosen reference spectrum in a spectral region.

## When NOT to use

- The data are 2D NMR or GC-MS isotopomer data intended for the metabolabpytools isotopomer-distribution routines rather than 1D-NMR pre-processing
- The spectra series is already aligned relative to the reference, so pre-alignment correlations would not show the expected low (<0.3) baseline
- No single high-SNR reference spectrum can be identified, since the correlation comparison is defined relative to one reference spectrum

## Inputs

- Series of 1D-NMR spectra (e.g. Daphnia magna extract spectra) loaded into MetaboLabPy 0.9.66
- User-selected alignment spectral region
- User-selected reference spectrum (the spectrum with the highest signal-to-noise ratio)

## Outputs

- Pearson correlation coefficients between each spectrum and the reference spectrum before alignment
- Pearson correlation coefficients after segmental alignment
- Tabulated before/after coefficients written to alignment_correlations.csv

## How to apply

Load the Daphnia magna 1D-NMR spectra series into MetaboLabPy 0.9.66 (Python backend with numpy/scipy/pandas) and select the spectral region to align. Choose the spectrum with the highest signal-to-noise ratio as the alignment reference, then run MetaboLabPy's segmental alignment algorithm on the selected region to remove chemical-shift drift relative to that reference. Compute Pearson correlation coefficients between each spectrum and the reference before alignment, recompute them after alignment, and tabulate both sets of coefficients. In the demonstrated case, correlations were generally below 0.3 pre-alignment and rose to above 0.95 post-alignment, so those thresholds serve as validation criteria; write the comparison to alignment_correlations.csv.

## Related tools

- **MetaboLabPy 0.9.66** (Runs the segmental alignment of the 1D-NMR spectra series against the high-SNR reference spectrum in the selected region) — https://github.com/ludwigc/MetaboLabPy
- **numpy** (Python toolbox used by MetaboLabPy for numerical computation on the spectral arrays) — https://github.com/ludwigc/MetaboLabPy
- **pandas** (Python toolbox used to tabulate the before/after correlation coefficients into alignment_correlations.csv) — https://github.com/ludwigc/MetaboLabPy

## Examples

```
import pandas as pd; corr = {s: df[s].corr(df['reference']) for s in df.columns if s != 'reference'}; pd.DataFrame([corr]).to_csv('alignment_correlations.csv')
```

## Evaluation signals

- Pre-alignment Pearson correlation coefficients between each spectrum and the reference are generally below 0.3
- Post-alignment Pearson correlation coefficients exceed 0.95
- The reference spectrum used is the one with the highest signal-to-noise ratio among the series
- Both coefficient sets are tabulated and written to alignment_correlations.csv
- Alignment is restricted to the user-selected spectral region, not the full spectrum

## Limitations

- The validation thresholds (<0.3 before, >0.95 after) were reported for the demonstrated Daphnia magna alignment outcome and may not transfer to other datasets or regions
- Alignment quality depends on the choice of reference spectrum (highest SNR) and the selected alignment region
- The exact criterion by which MetaboLabPy's segmental alignment determines the optimum shift is posed as an open research question in the surfaced task, not specified in the article text provided

## Evidence

- [abstract] Complete pre-processing workflow: "The software implements a complete workflow for NMR data pre-processing to prepare a series of 1D-NMR spectra for multi-variate statistical data analysis"
- [abstract] Segmental alignment is a provided step: "This includes a choice of algorithms for automated phase correction, segmental alignment, spectral scaling, variance stabilisation, export to various software platforms"
- [intro] MetaboLabPy scope and implementation: "Python package to process 1D and 2D NMR spectroscopic data for metabolomics and tracer based metabolism analysis"
- [abstract] Software backend: "The software is implemented in Python and uses numerous Python toolboxes, such as numpy, scipy, pandas, etc."
