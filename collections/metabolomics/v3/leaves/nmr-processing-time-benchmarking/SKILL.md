---
name: nmr-processing-time-benchmarking
description: Use when you need to compare wall-clock processing speed of NMR software
  (MetaboLabPy, MNova, TopSpin, MetaboLab scripting) on a defined batch of 1D 1H-NMR
  spectra with fixed processing parameters.
when_to_use_negative:
- The goal is isotopomer distribution derivation from HSQC + GC-MS data (metabolabpytools
  scope), not batch processing-speed measurement
- Only a single spectrum needs routine phase correction/alignment rather than a timed
  100-spectra batch benchmark
- The task is 2D NMR or metabolic tracer data analysis, which this timing benchmark
  does not cover
edam_topics:
- http://edamontology.org/topic_0523
tools:
- name: MetaboLabPy
  role: Primary package under benchmark; runs the timed batch sequence (0.3 Hz LB,
    131072-point zero-fill, FT, automatic phase correction) in both reference-based
    and individual modes
  repo: https://github.com/ludwigc/MetaboLabPy
- name: metabolabpy
  role: Python package used to load the example Daphnia extract spectra and build
    the 100-spectra batch
  repo: https://github.com/ludwigc/metabolabpy
- name: numpy
  role: Python toolbox underlying MetaboLabPy's spectral processing operations during
    the timed workflow
- name: pandas
  role: Python toolbox used to assemble the five timings into the processing_times_comparison.csv
    table
- name: scipy
  role: Python toolbox used by MetaboLabPy for processing operations in the benchmarked
    workflow
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.3390/metabo15010048
    title: MetaboLabPy—An Open-Source Software Package for Metabolomics NMR Data Processing
      and Metabolic Tracer Data Analysis
  source_content_hash: sha256:a23ebf0157bdabcfe7abc7ca75633417755ebb121849df4ffacb82ad054acac3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ludwigc/MetaboLabPy
  tools:
  - MetaboLabPy
  - metabolabpy
  - numpy
  - pandas
  - scipy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0523
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# nmr-processing-time-benchmarking

## Summary

Times the full 1H-NMR batch processing sequence (0.3 Hz exponential line broadening, zero-filling to 131072 points, Fourier transform, automatic phase correction) across competing software to identify the fastest pipeline for a 100-spectra metabolomics batch.

## When to use

Use when you need to compare wall-clock processing speed of NMR software (MetaboLabPy, MNova, TopSpin, MetaboLab scripting) on a defined batch of 1D 1H-NMR spectra with fixed processing parameters.

## When NOT to use

- The goal is isotopomer distribution derivation from HSQC + GC-MS data (metabolabpytools scope), not batch processing-speed measurement
- Only a single spectrum needs routine phase correction/alignment rather than a timed 100-spectra batch benchmark
- The task is 2D NMR or metabolic tracer data analysis, which this timing benchmark does not cover

## Inputs

- Four Daphnia extract 1H-NMR FID spectra from the metabolabpy example data
- Processing parameter set: 0.3 Hz exponential line broadening, zero-filling to 131072 points, Fourier transform, automatic phase correction
- Reported timings for non-MetaboLabPy approaches (MNova drag-and-drop, TopSpin Python-script, MetaboLab scripting)

## Outputs

- processing_times_comparison.csv with five rows of software-vs-time entries (28 s, 45 s, 1 min 13 s, 1 min 24 s, 3 min 53 s)
- Ranked comparison table identifying the fastest batch-processing workflow

## How to apply

Load the four Daphnia extract 1H-NMR example spectra into metabolabpy and replicate them to build a 100-spectra batch. Apply an exponential line broadening of 0.3 Hz and zero-fill each FID to 131072 points, then Fourier transform and run automatic phase correction (MetaboLabPy reference-spectrum baseline-based vs individual-spectrum modes) on every spectrum. Time the complete sequence for each MetaboLabPy workflow, and transcribe the reported timings for the alternative approaches (MNova drag-and-drop 1 min 13 s, TopSpin Python-script 1 min 24 s, MetaboLab scripting 3 min 53 s) rather than re-running them. Assemble all five timings (28 s, 45 s, 1 min 13 s, 1 min 24 s, 3 min 53 s) into a single comparison table and write it to processing_times_comparison.csv. The rationale is that fixed FID-length and phase-correction settings make processing time a fair, parameter-controlled speed metric across packages.

## Related tools

- **MetaboLabPy** (Primary package under benchmark; runs the timed batch sequence (0.3 Hz LB, 131072-point zero-fill, FT, automatic phase correction) in both reference-based and individual modes) — https://github.com/ludwigc/MetaboLabPy
- **metabolabpy** (Python package used to load the example Daphnia extract spectra and build the 100-spectra batch) — https://github.com/ludwigc/metabolabpy
- **numpy** (Python toolbox underlying MetaboLabPy's spectral processing operations during the timed workflow)
- **pandas** (Python toolbox used to assemble the five timings into the processing_times_comparison.csv table)
- **scipy** (Python toolbox used by MetaboLabPy for processing operations in the benchmarked workflow)

## Evaluation signals

- processing_times_comparison.csv exists and contains exactly five rows with values matching the reported times (28 s, 45 s, 1 min 13 s, 1 min 24 s, 3 min 53 s)
- The MetaboLabPy timings (28 s and 45 s) fall within the same order of magnitude as the reported benchmark
- The batch contains exactly 100 spectra replicated from the four example Daphnia extract 1H-NMR spectra
- Every spectrum was processed with identical parameters: 0.3 Hz line broadening and zero-filling to 131072 points before FT and phase correction
- The ranking identifies MetaboLabPy reference-spectrum baseline-based automatic phase correction as the fastest (28 s) and MetaboLab scripting as the slowest (3 min 53 s)

## Limitations

- Timings for MNova, TopSpin, and MetaboLab scripting are transcribed from reported values rather than re-measured, so cross-run hardware differences are not controlled
- The benchmark is specific to 100 Daphnia extract 1H-NMR spectra with 0.3 Hz line broadening and 131072-point zero-filling; other data sizes or parameter sets may change the ranking
- MetaboLabPy 0.9.66 is the version referenced in the article; newer or older versions may process at different speeds

## Evidence

- [abstract] MetaboLabPy is free and open-source NMR processing: "MetaboLabPy (version 0.9.66) is a free and open-source software package used to process 1D- and 2D-NMR spectra"
- [abstract] Automated phase correction is part of the pre-processing workflow: "This includes a choice of algorithms for automated phase correction, segmental alignment, spectral scaling, variance stabilisation"
- [abstract] MetaboLabPy is a Python package built on numpy/scipy/pandas: "The software is implemented in Python and uses numerous Python toolboxes, such as numpy, scipy, pandas, etc."
- [readme] README describes the package scope for NMR processing: "Python package to process 1D and 2D NMR spectroscopic data for metabolomics and tracer based metabolism analysis"
