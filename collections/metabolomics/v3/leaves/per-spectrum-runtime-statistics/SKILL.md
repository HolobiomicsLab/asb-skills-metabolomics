---
name: per-spectrum-runtime-statistics
description: 'Use when comparing computational efficiency of spectrum-processing libraries
  on the same spectra (e.g. iPRG2012.mgf): identical steps must be timed per spectrum
  and first-call JIT outliers excluded before computing speedup.'
when_to_use_negative:
- spectrum_utils does not provide functionality to read spectral data files, so do
  not use it as the file reader in the timing pipeline — load via pyteomics.mgf or
  another external reader first
- 'When only a single call per library is measured: the first spectrum_utils call
  is dominated by Numba JIT compilation and must not be compared to steady-state runtimes'
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3365
tools:
- name: spectrum_utils
  role: Library under benchmark; its processing pipeline (set_mz_range, remove_precursor_peak,
    filter_intensity, scale_intensity 'root') is timed per spectrum on iPRG2012 data
  repo: https://github.com/bittremieux/spectrum_utils
- name: pymzML
  role: Baseline library timed with equivalent operations (m/z reduction to 100–1400,
    remove_precursor_peak, remove_noise, sqrt intensity scaling)
  repo: https://github.com/pymzml/pymzML
- name: pyOpenMS
  role: Baseline library timed with equivalent OpenMS filters (ParentPeakMower, ThresholdMower,
    NLargest n=150, SqrtMower plus Normalizer)
- name: Pyteomics
  role: Used to read iPRG2012.mgf and iterate spectra for per-spectrum timing
- name: NumPy
  role: Used to compute median/mean per-spectrum runtime statistics and speedup ratios
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.9b04884
    title: '<tt>spectrum_utils</tt> : A Python Package for Mass Spectrometry Data
      Processing and Visualization'
  source_content_hash: sha256:51b0450da79eb40680ea590e2b5ec854a96a9560e18bb68d4e282923d4fa49f5
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
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3365
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# per-spectrum-runtime-statistics

## Summary

Benchmark the per-spectrum processing time of Python mass-spectrometry libraries (spectrum_utils vs pymzML and pyOpenMS) on the iPRG2012 spectra by timing identical processing pipelines, excluding JIT-compilation first-call outliers, and reporting median runtimes and speedup ratios.

## When to use

Use when comparing computational efficiency of spectrum-processing libraries on the same spectra (e.g. iPRG2012.mgf): identical steps must be timed per spectrum and first-call JIT outliers excluded before computing speedup.

## When NOT to use

- spectrum_utils does not provide functionality to read spectral data files, so do not use it as the file reader in the timing pipeline — load via pyteomics.mgf or another external reader first
- When only a single call per library is measured: the first spectrum_utils call is dominated by Numba JIT compilation and must not be compared to steady-state runtimes

## Inputs

- iPRG2012.mgf spectra file (MGF format)
- Python environment with spectrum_utils 0.3.0, pymzML 2.4.4, pyOpenMS 2.4.0, pyteomics, NumPy

## Outputs

- speedup_table.csv containing library, median/mean per-spectrum runtime, and speedup ratio
- Per-spectrum processing times for each library after excluding the JIT first-call outlier

## How to apply

Load spectra from iPRG2012.mgf using pyteomics.mgf, omitting spectra with fewer than 10 peaks or lacking a charge annotation. Time the spectrum_utils pipeline per spectrum — set_mz_range to 100–1400 m/z, remove_precursor_peak at 0.02 Da tolerance, filter_intensity with min_intensity 0.05 and max_num_peaks 150, scale_intensity 'root' — against equivalent pymzML operations (reduce to 100–1400 m/z, remove_precursor_peak, remove_noise, sqrt scaling) and equivalent pyOpenMS operations (ParentPeakMower at 0.02 Da window, ThresholdMower at 0.05, NLargest with n=150, SqrtMower plus Normalizer). Exclude the first spectrum_utils call because Numba JIT compilation dominates it, then compute median per-spectrum runtimes and speedup ratios of pymzML and pyOpenMS relative to spectrum_utils. Write the resulting table (library, median/mean per-spectrum runtime, speedup ratio) to speedup_table.csv.

## Related tools

- **spectrum_utils** (Library under benchmark; its processing pipeline (set_mz_range, remove_precursor_peak, filter_intensity, scale_intensity 'root') is timed per spectrum on iPRG2012 data) — https://github.com/bittremieux/spectrum_utils
- **pymzML** (Baseline library timed with equivalent operations (m/z reduction to 100–1400, remove_precursor_peak, remove_noise, sqrt intensity scaling)) — https://github.com/pymzml/pymzML
- **pyOpenMS** (Baseline library timed with equivalent OpenMS filters (ParentPeakMower, ThresholdMower, NLargest n=150, SqrtMower plus Normalizer))
- **Pyteomics** (Used to read iPRG2012.mgf and iterate spectra for per-spectrum timing)
- **NumPy** (Used to compute median/mean per-spectrum runtime statistics and speedup ratios)

## Examples

```
import pyteomics.mgf; spectra = [s for s in pyteomics.mgf.read('iPRG2012.mgf') if len(s) >= 10 and 'charge' in s]
```

## Evaluation signals

- The first spectrum_utils call is excluded as a Numba JIT-compilation outlier before statistics are computed
- All three libraries perform identical processing steps: 100–1400 m/z restriction, precursor removal at 0.02 Da, 0.05 min-intensity / 150 max-peak filtering, and sqrt intensity scaling
- Spectra with fewer than 10 peaks or lacking a charge annotation are omitted from the benchmark
- speedup_table.csv contains a speedup ratio of pymzML and pyOpenMS relative to spectrum_utils greater than 1 for steady-state calls
- Reported statistics are medians (and means) of per-spectrum runtimes, not wall-clock totals

## Limitations

- The first spectrum_utils call is inflated by Numba JIT compilation, so single-call benchmarks will misleadingly show it as slow
- spectrum_utils relies on external tools (e.g. pyteomics) to read spectral data files, adding an I/O step outside its measured processing time
- Timing results are specific to the iPRG2012 spectra and the exact library versions tested (spectrum_utils 0.3.0, pymzML 2.4.4, pyOpenMS 2.4.0)

## Evidence

- [intro] A key feature of spectrum_utils is its focus on computational efficiency to process large amounts of spectral data.: [evidence span withheld — anchor status: not_found]
- [abstract] the data processing functionality is highly optimized for computational eﬀiciency to be able to deal with the large volumes of data: "the data processing functionality is highly optimized for computational eﬀiciency to be able to deal with the large volumes of data"
- [methods] The m/z range is set to 100–1400 m/z.: [evidence span withheld — anchor status: not_found]
- [readme] spectrum_utils is a Python package for efficient mass spectrometry data processing and visualization.: "spectrum_utils is a Python package for efficient mass spectrometry data processing and visualization."
