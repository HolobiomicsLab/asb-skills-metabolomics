---
name: cross-library-performance-comparison
description: Use when choosing a Python library for a fixed spectrum-processing pipeline
  (m/z trimming, precursor removal, intensity filtering, scaling) and per-spectrum
  runtime on identical spectra must be measured and compared across libraries.
when_to_use_negative:
- The spectra have not been read from an MGF/mzML file or the libraries are not pinned
  (e.g. spectrum_utils 0.3.0, pymzML 2.4.4, pyOpenMS 2.4.0) — version differences
  invalidate the comparison.
- The processing pipelines are not step-for-step equivalent (different m/z window,
  precursor tolerance, intensity threshold, or peak cap) — comparing non-identical
  operations is not a fair runtime benchmark.
- 'Only a first-call measurement is taken: the spectrum_utils first invocation is
  dominated by Numba JIT compilation, so skipping the warm-up exclusion misrepresents
  steady-state runtime.'
edam_operation: http://edamontology.org/operation_3359
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: spectrum_utils
  role: Benchmarked library executing the per-spectrum processing pipeline (set_mz_range,
    remove_precursor_peak, filter_intensity, scale_intensity) whose runtime is timed
    and compared.
  repo: https://github.com/bittremieux/spectrum_utils
- name: pymzML
  role: Baseline library (v2.4.4) whose equivalent reduce/remove_precursor_peak/remove_noise/normalize
    pipeline is timed per spectrum for the comparison.
  repo: https://github.com/pymzml/pymzML
- name: pyOpenMS
  role: Baseline library (v2.4.0) exposing OpenMS filters (ParentPeakMower, ThresholdMower,
    NLargest, SqrtMower, Normalizer) used as the equivalent timed pipeline.
- name: pyteomics
  role: Used to read spectra from iPRG2012.mgf and apply validity filters (≥10 peaks,
    charge annotation present) before benchmarking.
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.9b04884
    title: '<tt>spectrum_utils</tt> : A Python Package for Mass Spectrometry Data
      Processing and Visualization'
  source_content_hash: sha256:4a29e85de7e431cf290e8e12954187559a6a61ab308eb1200188c67c15a3746e
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
  - pyteomics
  - spectrum_utils
  edam_operation: http://edamontology.org/operation_3359
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# cross-library-performance-comparison

## Summary

Benchmark per-spectrum processing runtime of spectrum_utils against functionally equivalent pymzML and pyOpenMS pipelines on the same spectra (e.g. iPRG2012 MGF), recording the Numba JIT first-call outlier for spectrum_utils and producing log-scaled runtime comparison boxplots.

## When to use

Use when choosing a Python library for a fixed spectrum-processing pipeline (m/z trimming, precursor removal, intensity filtering, scaling) and per-spectrum runtime on identical spectra must be measured and compared across libraries.

## When NOT to use

- The spectra have not been read from an MGF/mzML file or the libraries are not pinned (e.g. spectrum_utils 0.3.0, pymzML 2.4.4, pyOpenMS 2.4.0) — version differences invalidate the comparison.
- The processing pipelines are not step-for-step equivalent (different m/z window, precursor tolerance, intensity threshold, or peak cap) — comparing non-identical operations is not a fair runtime benchmark.
- Only a first-call measurement is taken: the spectrum_utils first invocation is dominated by Numba JIT compilation, so skipping the warm-up exclusion misrepresents steady-state runtime.

## Inputs

- iPRG2012.mgf MGF-format MS/MS spectra file
- Python environment with spectrum_utils (0.3.0), pymzML (2.4.4), pyOpenMS (2.4.0), pyteomics.mgf, NumPy

## Outputs

- runtime.png — log-scaled boxplot of per-spectrum processing times for spectrum_utils, pymzML, and pyOpenMS, including the spectrum_utils Numba JIT first-call outlier
- speedup_table.csv — per-library median/mean per-spectrum runtime and speedup ratios relative to spectrum_utils

## How to apply

Load the same spectra for every library — e.g. iPRG2012.mgf read via pyteomics.mgf, omitting spectra with fewer than 10 peaks or a missing charge annotation — and apply the identical processing steps in each library: spectrum_utils set_mz_range(100, 1400), remove_precursor_peak(0.02, 'Da'), filter_intensity(0.05, 150), scale_intensity('root'); pymzML 2.4.4 reduce to the 100–1400 m/z window, remove_precursor_peak, remove_noise, normalize, square-root scaling; pyOpenMS 2.4.0 ParentPeakMower (0.02 Da window), ThresholdMower (0.05), NLargest (n=150), SqrtMower plus Normalizer. Time each per-spectrum pipeline, exclude the first spectrum_utils call as Numba JIT-compilation overhead, then aggregate per-spectrum runtimes and compute speedup ratios of pymzML and pyOpenMS relative to spectrum_utils. Visualize the three distributions as a log-scaled boxplot (runtime.png) that retains the JIT first-call outlier for spectrum_utils, and write a summary table (library, median/mean runtime, speedup) to speedup_table.csv. Rationale: the article states a key feature of spectrum_utils is computational efficiency for large spectral volumes, and identical step-for-step pipelines are the only fair basis for the comparison.

## Related tools

- **spectrum_utils** (Benchmarked library executing the per-spectrum processing pipeline (set_mz_range, remove_precursor_peak, filter_intensity, scale_intensity) whose runtime is timed and compared.) — https://github.com/bittremieux/spectrum_utils
- **pymzML** (Baseline library (v2.4.4) whose equivalent reduce/remove_precursor_peak/remove_noise/normalize pipeline is timed per spectrum for the comparison.) — https://github.com/pymzml/pymzML
- **pyOpenMS** (Baseline library (v2.4.0) exposing OpenMS filters (ParentPeakMower, ThresholdMower, NLargest, SqrtMower, Normalizer) used as the equivalent timed pipeline.)
- **pyteomics** (Used to read spectra from iPRG2012.mgf and apply validity filters (≥10 peaks, charge annotation present) before benchmarking.)

## Examples

```
import pyteomics.mgf, spectrum_utils.spectrum; sus = spectrum_utils.spectrum.MsmsSpectrum('iPRG2012', precursor_mz, charge, peaks); sus.set_mz_range(100, 1400)
```

## Evaluation signals

- runtime.png renders as a log-scaled boxplot with three library distributions over the same iPRG2012 spectra.
- The first spectrum_utils call is visible as a high outlier attributable to Numba JIT compilation; steady-state calls sit far below it.
- Median/mean per-spectrum runtime of spectrum_utils is lower than pymzML 2.4.4 and pyOpenMS 2.4.0 across the iPRG2012 spectra, consistent with the paper's Figure 2 comparison.
- speedup_table.csv contains one row per library with a speedup ratio relative to spectrum_utils, and the filtered spectrum set matches the ≥10 peaks / charge-annotation inclusion rule.
- Each library's pipeline covers the same operations: 100–1400 m/z window, 0.02 Da precursor tolerance, 0.05 intensity threshold, 150-peak cap, square-root intensity scaling.

## Limitations

- spectrum_utils does not provide functionality to read spectral data files; an external tool (e.g. pyteomics.mgf) must supply the spectra.
- The spectrum_utils first call incurs Numba JIT-compilation overhead and appears as a significant runtime outlier unless excluded from steady-state statistics.
- The benchmark's findings are tied to specific versions (spectrum_utils 0.3.0, pymzML 2.4.4, pyOpenMS 2.4.0) and to the iPRG2012 spectra; results may not generalize to other data or versions.
- Runtime figures depend on all libraries being given functionally equivalent processing steps; small deviations in parameters confound the comparison.

## Evidence

- [results] Only significant outlier for spectrum_utils is Numba JIT compilation on first call: [evidence span withheld — anchor status: not_found]
- [results] Fast spectrum_utils processing versus pymzML and pyOpenMS: [evidence span withheld — anchor status: not_found]
- [abstract] spectrum_utils designed for computational efficiency on large data volumes: "the data processing functionality is highly optimized for computational eﬀiciency to be able to deal with the large volumes of data"
- [discussion] No file-reading functionality in spectrum_utils: [evidence span withheld — anchor status: not_found]
- [intro] Key focus on computational efficiency in spectrum_utils: [evidence span withheld — anchor status: not_found]
