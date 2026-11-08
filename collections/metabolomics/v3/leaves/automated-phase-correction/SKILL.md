---
name: automated-phase-correction
description: Use when a series of 1H-NMR spectra (e.g. metabolite mix replicates or
  Daphnia extract CryoProbe data) needs phasing without manual per-spectrum adjustment,
  or when choosing among automatic phasing algorithms by speed or phase consistency.
when_to_use_negative:
- The dataset is already phased or the user intends manual, interactive phase adjustment
  rather than automated correction
- The task is isotopomer/tracer analysis (metabolabpytools territory) rather than
  1D-NMR spectral pre-processing
- The spectra are 2D and the goal is alignment or quantification rather than phasing
  — MetaboLab apk5 could not phase the metabolite mix, so algorithm choice must be
  validated on the data type at hand
tools:
- name: MetaboLabPy 0.9.66
  role: Primary implementation providing per-spectrum and reference-spectrum baseline-based
    automated phase correction for 1D- and 2D-NMR spectra
  repo: https://github.com/ludwigc/MetaboLabPy
- name: TopSpin (apk0.noe)
  role: Comparison algorithm; among the most consistent within-series phase correction
    in the benchmark
- name: MNova
  role: Comparison tool with drag-and-drop automated phasing; second-fastest at 45
    s for 100 spectra
- name: MetaboLab (apk5)
  role: Comparison algorithm; failed to phase the metabolite mix spectra in the consistency
    benchmark
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.3390/metabo15010048
    title: MetaboLabPy—An Open-Source Software Package for Metabolomics NMR Data Processing
      and Metabolic Tracer Data Analysis
  source_content_hash: sha256:1e7ff9f7b33c94911446a63a7add44e166597a00de55d8ff51f4a390ea090b88
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ludwigc/MetaboLabPy
  tools:
  - MNova
  - MetaboLab
  - MetaboLab (apk5)
  - MetaboLabPy 0.9.66
  - TopSpin
  - TopSpin (apk0.noe)
  - metabolabpy
  - numpy
  - pandas
  - scipy
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# automated-phase-correction

## Summary

Automated phase correction of 1D/2D NMR spectra within MetaboLabPy, benchmarked against MetaboLab (apk5), TopSpin (apk0.noe) and MNova for speed and within-series phase consistency. MetaboLabPy offers both per-spectrum and reference-spectrum baseline-based algorithms, the latter being fastest (28 s for a 100-spectra batch) and yielding the most consistent baselines.

## When to use

Use when a series of 1H-NMR spectra (e.g. metabolite mix replicates or Daphnia extract CryoProbe data) needs phasing without manual per-spectrum adjustment, or when choosing among automatic phasing algorithms by speed or phase consistency.

## When NOT to use

- The dataset is already phased or the user intends manual, interactive phase adjustment rather than automated correction
- The task is isotopomer/tracer analysis (metabolabpytools territory) rather than 1D-NMR spectral pre-processing
- The spectra are 2D and the goal is alignment or quantification rather than phasing — MetaboLab apk5 could not phase the metabolite mix, so algorithm choice must be validated on the data type at hand

## Inputs

- 1D 1H-NMR FID or spectra series (e.g. Daphnia extract CryoProbe spectra, metabolite mix replicate series)
- Processing parameters: zero-filling to 131072 points, 0.3 Hz exponential line broadening
- Reference spectrum for baseline-based automatic phase correction

## Outputs

- Phased 1D-NMR spectra ready for downstream segmental alignment, spectral scaling and variance stabilisation
- Per-spectrum phase angles for dispersion analysis (e.g. Figures S4/S5-style phase-consistency plots)
- Batch timing comparison (e.g. processing_times_comparison.csv with 28 s, 45 s, 1 min 13 s, 1 min 24 s, 3 min 53 s)

## How to apply

Load the 1D-NMR series into MetaboLabPy 0.9.66 (installed via PyPI or Bioconda, invoked with the metabolabpy command line tool) and choose between the per-spectrum automated phase correction and the reference-spectrum baseline-based automatic phase correction. For speed-critical batches, apply the baseline-based algorithm after zero-filling the FID to 131072 points, applying 0.3 Hz exponential line broadening and Fourier transform; it processed 100 Daphnia extract spectra in 28 s versus 45 s for MNova drag-and-drop. For consistency-critical series, run per-spectrum phase correction on every replicate, extract the per-spectrum phase angles, and compute within-series phase dispersion across replicates, comparing against TopSpin apk0.noe and MetaboLab apk5. Expect MetaboLab apk5 to fail on metabolite mix spectra, while TopSpin apk0.noe and the MetaboLabPy baseline-based algorithm give the most consistent (lowest-dispersion) phases.

## Related tools

- **MetaboLabPy 0.9.66** (Primary implementation providing per-spectrum and reference-spectrum baseline-based automated phase correction for 1D- and 2D-NMR spectra) — https://github.com/ludwigc/MetaboLabPy
- **TopSpin (apk0.noe)** (Comparison algorithm; among the most consistent within-series phase correction in the benchmark)
- **MNova** (Comparison tool with drag-and-drop automated phasing; second-fastest at 45 s for 100 spectra)
- **MetaboLab (apk5)** (Comparison algorithm; failed to phase the metabolite mix spectra in the consistency benchmark)

## Examples

```
metabolabpy --help
```

## Evaluation signals

- processing_times_comparison.csv exists with five rows whose values match the reported times (28 s, 45 s, 1 min 13 s, 1 min 24 s, 3 min 53 s)
- MetaboLabPy batch timings (28 s reference-based, 45 s MNova-equivalent workflow) fall within the same order of magnitude as the reported benchmark
- MetaboLab apk5 fails to phase the metabolite mix replicate series, as reported
- Within-series phase dispersion is lowest for TopSpin apk0.noe and MetaboLabPy baseline-based phasing across each replicate series
- Phased spectra exhibit a flat baseline comparable to or better than the apk0.noe result before downstream alignment/scaling

## Limitations

- MetaboLab's apk5 algorithm could not perform automatic phase correction on the metabolite mix spectra, so it is not universally applicable
- Speed figures are benchmark-specific (100-spectra Daphnia extract batch with 131072-point zero-filling and 0.3 Hz line broadening) and may not transfer to other hardware or data
- No changelog is available for the software, so version-specific phasing behaviour changes are hard to track

## Evidence

- [abstract] Choice of algorithms for automated phase correction within MetaboLabPy: "This includes a choice of algorithms for automated phase correction, segmental alignment, spectral scaling, variance stabilisation, export to various software platforms, and analysis of metabolic"
- [abstract] Versioned implementation context: "MetaboLabPy (version 0.9.66) is a free and open-source software package used to process 1D- and 2D-NMR spectra"
- [readme] Command-line entry point for running the package: "Python package to process 1D and 2D NMR spectroscopic data for metabolomics and tracer based metabolism analysis"
- [abstract] Availability channel: "MetaboLabPy is available via the Python Package Index or via GitHub"
