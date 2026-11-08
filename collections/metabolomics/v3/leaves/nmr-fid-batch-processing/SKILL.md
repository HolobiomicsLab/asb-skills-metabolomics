---
name: nmr-fid-batch-processing
description: Use when comparing processing speed of NMR software on a batch of 1D
  1H-NMR FIDs requiring identical parameters (e.g., zero-filling to 131072 points,
  0.3 Hz exponential line broadening, FT, automated phase correction), or when scripting
  batch NMR pre-processing in Python.
when_to_use_negative:
- The task is isotopomer/tracer analysis or HSQC+GC-MS based isotopomer prediction
  — that belongs to metabolabpytools, not batch 1D processing speed benchmarking.
- The input data is 2D NMR or already fully processed spectra that do not require
  FID-level zero-filling, apodization, and phase correction.
- Only one spectrum or a small non-uniform batch is being processed, where the 100-spectra
  batch benchmark design (identical parameters across all spectra) does not apply.
edam_operation: http://edamontology.org/operation_0335
tools:
- name: MetaboLabPy (metabolabpy)
  role: Primary engine for batch zero-filling, line broadening, Fourier transform,
    and reference-spectrum baseline-based automatic phase correction; timed at 28
    s for 100 spectra
  repo: https://github.com/ludwigc/MetaboLabPy
- name: numpy
  role: Python toolbox used internally by MetaboLabPy for numerical array processing
    of the FID batch
- name: scipy
  role: Python toolbox used internally by MetaboLabPy for spectral processing routines
- name: pandas
  role: Python toolbox used to assemble the timing comparison table into CSV
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.3390/metabo15010048
    title: MetaboLabPy—An Open-Source Software Package for Metabolomics NMR Data Processing
      and Metabolic Tracer Data Analysis
  source_content_hash: sha256:1dd74c14e892e8acbd1b30b4a92a7a8d82f791b96dff1e65602ff079b6a01300
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ludwigc/MetaboLabPy
  tools:
  - MetaboLabPy (metabolabpy)
  - metabolabpy
  - numpy
  - pandas
  - scipy
  edam_operation: http://edamontology.org/operation_0335
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# nmr-fid-batch-processing

## Summary

Benchmarks how quickly different software (MetaboLabPy, MNova, TopSpin, MetaboLab) processes a batch of 100 1H-NMR spectra through zero-filling, line broadening, Fourier transform, and automatic phase correction. Grounded in the MetaboLabPy paper's timing comparison, where MetaboLabPy's reference-spectrum baseline-based phase correction processed all 100 Daphnia extract spectra in 28 s.

## When to use

Use when comparing processing speed of NMR software on a batch of 1D 1H-NMR FIDs requiring identical parameters (e.g., zero-filling to 131072 points, 0.3 Hz exponential line broadening, FT, automated phase correction), or when scripting batch NMR pre-processing in Python.

## When NOT to use

- The task is isotopomer/tracer analysis or HSQC+GC-MS based isotopomer prediction — that belongs to metabolabpytools, not batch 1D processing speed benchmarking.
- The input data is 2D NMR or already fully processed spectra that do not require FID-level zero-filling, apodization, and phase correction.
- Only one spectrum or a small non-uniform batch is being processed, where the 100-spectra batch benchmark design (identical parameters across all spectra) does not apply.

## Inputs

- 100 1H-NMR FID/spectra of Daphnia extract (metabolabpy example data, copied to form the batch)
- Processing parameters: zero-filling to 131072 points, exponential line broadening of 0.3 Hz
- Reported timings for non-Python workflows (MNova drag-and-drop 1 min 13 s; TopSpin Python script 1 min 24 s; MetaboLab scripting 3 min 53 s)

## Outputs

- 100 zero-filled, 0.3 Hz-apodized, Fourier-transformed, automatically phase-corrected 1H-NMR spectra per workflow
- processing_times_comparison.csv with five rows of timing values (28 s, 45 s, 1 min 13 s, 1 min 24 s, 3 min 53 s)

## How to apply

Load the example Daphnia extract 1H-NMR spectra bundled with metabolabpy and replicate them to build a 100-spectra batch. Apply exponential line broadening of 0.3 Hz to each FID and zero-fill to 131072 points, then Fourier transform and run automatic phase correction on every spectrum — MetaboLabPy offers a choice of automated phase-correction algorithms, including the reference-spectrum baseline-based variant that was fastest (28 s for 100 spectra vs 45 s in MNova). Time the full sequence end-to-end for each workflow under comparison (MetaboLabPy reference-based vs individual, MNova drag-and-drop, TopSpin Python script, MetaboLab scripting), then assemble all reported timings into a single CSV comparison table. Rationale: the benchmark isolates software/engine differences by holding every spectral parameter identical across tools, so any timing difference reflects the processing implementation rather than data properties.

## Related tools

- **MetaboLabPy (metabolabpy)** (Primary engine for batch zero-filling, line broadening, Fourier transform, and reference-spectrum baseline-based automatic phase correction; timed at 28 s for 100 spectra) — https://github.com/ludwigc/MetaboLabPy
- **numpy** (Python toolbox used internally by MetaboLabPy for numerical array processing of the FID batch)
- **scipy** (Python toolbox used internally by MetaboLabPy for spectral processing routines)
- **pandas** (Python toolbox used to assemble the timing comparison table into CSV)

## Examples

```
metabolabpy --help
```

## Evaluation signals

- processing_times_comparison.csv exists and contains exactly five rows whose values match the reported timings: 28 s, 45 s, 1 min 13 s, 1 min 24 s, 3 min 53 s
- MetaboLabPy timings (28 s and 45 s) fall within the same order of magnitude as the reported benchmark
- Every spectrum in the batch was zero-filled to exactly 131072 points and apodized with 0.3 Hz exponential line broadening before Fourier transform
- Automatic phase correction was applied to all 100 spectra under each workflow, using the reference-spectrum baseline-based algorithm for the fastest MetaboLabPy route
- Identical processing parameters were used across all compared workflows so timings are directly comparable

## Limitations

- Timing results are specific to the benchmark hardware/software versions (MetaboLabPy 0.9.66) and the 100-spectra Daphnia extract batch; they are not generalizable speed guarantees.
- Timings for MNova (1 min 13 s), TopSpin (1 min 24 s), and MetaboLab scripting (3 min 53 s) were transcribed from reported measurements rather than re-executed, so cross-platform reproducibility depends on those platforms.
- The metabolabpy README is truncated in the available documentation and the project reports no changelog; installation details are delegated to the online documentation at ludwigc.github.io/metabolabpy.

## Evidence

- [abstract] complete workflow claim: "The software implements a complete workflow for NMR data pre-processing to prepare a series of 1D-NMR spectra for multi-variate statistical data analysis"
- [abstract] automated phase correction choice: "This includes a choice of algorithms for automated phase correction, segmental alignment, spectral scaling, variance stabilisation, export to various software platforms"
- [abstract] implementation toolboxes: "The software is implemented in Python and uses numerous Python toolboxes, such as numpy, scipy, pandas, etc."
- [readme] package purpose: "Python package to process 1D and 2D NMR spectroscopic data for metabolomics and tracer based metabolism analysis"
- [readme] command line entry point: "Command line
------------
::

    $ metabolabpy --help"
