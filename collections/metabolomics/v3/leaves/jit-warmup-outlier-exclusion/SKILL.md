---
name: jit-warmup-outlier-exclusion
description: Use when benchmarking spectrum_utils or other Numba-compiled Python spectral-processing
  pipelines against baselines (pymzML, pyOpenMS) and you need to remove first-call
  JIT compilation overhead before comparing per-spectrum runtimes.
when_to_use_negative:
- When benchmarking libraries that are not compiled with Numba (e.g., pure Python
  or C-extension-only tools) — there is no JIT warm-up to exclude and the outlier-removal
  logic does not apply.
- When comparing libraries on the first spectrum only, or when total batch runtime
  (including one-time compilation) is the target metric rather than steady-state per-spectrum
  throughput.
- When the input spectra are not MGF files loadable via pyteomics.mgf (e.g., raw instrument
  output or mzML with vendor-specific encodings that the pipeline does not read).
edam_operation: http://edamontology.org/operation_3438
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: spectrum_utils
  role: Subject of the benchmark; its first processing call is excluded as Numba JIT
    compilation overhead before computing median per-spectrum runtimes.
  repo: https://github.com/bittremieux/spectrum_utils
- name: Pyteomics
  role: Used to load spectra from the iPRG2012.mgf file, omitting spectra with fewer
    than 10 peaks or lacking a charge annotation.
- name: pymzML
  role: Baseline library whose equivalent per-spectrum processing steps (m/z window
    reduction, precursor removal, noise removal, sqrt intensity scaling) are timed
    for comparison.
  repo: https://github.com/pymzml/pymzML
- name: pyOpenMS
  role: Baseline library whose equivalent per-spectrum processing steps (ParentPeakMower,
    ThresholdMower, NLargest, SqrtMower, Normalizer) are timed for comparison.
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.9b04884
    title: '<tt>spectrum_utils</tt> : A Python Package for Mass Spectrometry Data
      Processing and Visualization'
  source_content_hash: sha256:ff5b98e35cd71d2cb4c012a38351092cb3f34035b5bf9372d683a50e978268a5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bittremieux/spectrum_utils
  tools:
  - NumPy
  - Pyteomics
  - Python
  - pyOpenMS
  - pymzML
  - spectrum_utils
  edam_operation: http://edamontology.org/operation_3438
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# JIT Warm-up Outlier Exclusion

## Summary

Excludes the first spectrum_utils processing call from runtime benchmarks because Numba JIT compilation makes it a systematic outlier, then compares per-spectrum timings of spectrum_utils against pymzML and pyOpenMS on iPRG2012 spectra to compute speedup ratios.

## When to use

Use when benchmarking spectrum_utils or other Numba-compiled Python spectral-processing pipelines against baselines (pymzML, pyOpenMS) and you need to remove first-call JIT compilation overhead before comparing per-spectrum runtimes.

## When NOT to use

- When benchmarking libraries that are not compiled with Numba (e.g., pure Python or C-extension-only tools) — there is no JIT warm-up to exclude and the outlier-removal logic does not apply.
- When comparing libraries on the first spectrum only, or when total batch runtime (including one-time compilation) is the target metric rather than steady-state per-spectrum throughput.
- When the input spectra are not MGF files loadable via pyteomics.mgf (e.g., raw instrument output or mzML with vendor-specific encodings that the pipeline does not read).

## Inputs

- iPRG2012.mgf spectral data file (MGF format)
- Python environment with spectrum_utils, Pyteomics, pymzML, pyOpenMS, NumPy

## Outputs

- speedup_table.csv containing library name, median/mean per-spectrum runtime, and speedup ratio of pymzML and pyOpenMS relative to spectrum_utils
- Per-spectrum processing timings for each library after excluding the first spectrum_utils call

## How to apply

Load spectra from iPRG2012.mgf using pyteomics.mgf, omitting spectra with fewer than 10 peaks or lacking a charge annotation. For each spectrum, time the spectrum_utils pipeline: set_mz_range to 100–1400 m/z, remove_precursor_peak with 0.02 Da tolerance, filter_intensity keeping peaks at ≥5% of the base peak intensity with max_num_peaks 150, then scale_intensity 'root'. In parallel, time equivalent pymzML operations (m/z window reduction, remove_precursor_peak, remove_noise, sqrt scaling) and pyOpenMS operations (ParentPeakMower, ThresholdMower, NLargest with n=150, SqrtMower plus Normalizer). Discard the first spectrum_utils call as Numba JIT-compilation overhead before computing median per-spectrum runtimes. Finally, compute speedup ratios of pymzML and pyOpenMS relative to spectrum_utils and write them to speedup_table.csv. The rationale is that JIT compilation happens once and would otherwise mask the true steady-state efficiency of the optimized processing backend.

## Related tools

- **spectrum_utils** (Subject of the benchmark; its first processing call is excluded as Numba JIT compilation overhead before computing median per-spectrum runtimes.) — https://github.com/bittremieux/spectrum_utils
- **Pyteomics** (Used to load spectra from the iPRG2012.mgf file, omitting spectra with fewer than 10 peaks or lacking a charge annotation.)
- **pymzML** (Baseline library whose equivalent per-spectrum processing steps (m/z window reduction, precursor removal, noise removal, sqrt intensity scaling) are timed for comparison.) — https://github.com/pymzml/pymzML
- **pyOpenMS** (Baseline library whose equivalent per-spectrum processing steps (ParentPeakMower, ThresholdMower, NLargest, SqrtMower, Normalizer) are timed for comparison.)

## Examples

```
mgf_filename = "iPRG2012.mgf"
```

## Evaluation signals

- The first spectrum_utils timing is excluded from the speedup computation; verify the reported median is computed over all subsequent calls only.
- Per-spectrum processing parameters must match exactly across libraries: 100–1400 m/z window, 0.02 Da precursor tolerance, ≥5% base-peak intensity threshold, max 150 peaks.
- The output speedup_table.csv must contain a row for each library with median/mean per-spectrum runtime and a speedup ratio relative to spectrum_utils.
- Spectra with fewer than 10 peaks or without a charge annotation must be omitted from the benchmark before timing.
- After excluding the JIT warm-up call, spectrum_utils should show a faster median per-spectrum runtime than both pymzML and pyOpenMS on the same iPRG2012 spectra.

## Limitations

- spectrum_utils does not provide functionality to read spectral data files, so external tools (e.g., Pyteomics) are required to load the iPRG2012.mgf spectra.
- The speedup measurement is specific to the iPRG2012 spectra and the chosen processing steps; results may not generalize to other datasets or processing pipelines.
- Only the first spectrum_utils call is identified as JIT overhead; if a different library also has one-time initialization costs, they are not accounted for by this skill.

## Evidence

- [methods] Exclude the first spectrum_utils call as Numba JIT-compilation overhead, then compute median per-spectrum processing times and speedup ratios.: [evidence span withheld — anchor status: not_found]
- [intro] A key feature of spectrum_utils is its focus on computational efficiency to process large amounts of spectral data.: [evidence span withheld — anchor status: not_found]
- [discussion] spectrum_utils does not provide functionality to read spectral data files because there already exist several excellent tools to perform this task.: [evidence span withheld — anchor status: not_found]
- [methods] filter_intensity threshold and peak-count limit from the article's filter steps.: "Remove low-intensity noise peaks by only retaining peaks that are at at least 5% of the base peak intensity and restrict the total number of peaks to the 50 most intense peaks."
