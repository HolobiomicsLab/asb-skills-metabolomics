---
name: mass-spectrometry-runtime-benchmarking
description: Use when comparing runtime efficiency of mass spectrometry processing
  libraries on identical peak-processing steps (m/z trimming, precursor/noise removal,
  intensity filtering and scaling) over MGF spectra.
when_to_use_negative:
- When the goal is peak quality or correctness comparison rather than computational
  runtime — the pipelines here are timed, not validated for output equivalence.
- When input spectra need to be read by spectrum_utils itself — it does not provide
  functionality to read spectral data files, so file I/O timing with spectrum_utils
  cannot be benchmarked this way.
- When the target library's first-call cost matters for your use case — this skill
  deliberately excludes the first spectrum_utils call as Numba JIT-compilation overhead.
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_0092
tools:
- name: spectrum_utils
  role: Benchmarked library; provides set_mz_range, remove_precursor_peak, filter_intensity,
    scale_intensity for the timed pipeline
  repo: https://github.com/bittremieux/spectrum_utils
- name: pymzML
  role: Baseline library timed for equivalent operations (m/z reduction, remove_precursor_peak,
    remove_noise, sqrt scaling)
  repo: https://github.com/pymzml/pymzML
- name: pyOpenMS
  role: Baseline library timed via ParentPeakMower, ThresholdMower, NLargest, SqrtMower
    and Normalizer filters
- name: Pyteomics
  role: Parses iPRG2012.mgf and supplies identically filtered spectra to all three
    libraries
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.9b04884
    title: '<tt>spectrum_utils</tt> : A Python Package for Mass Spectrometry Data
      Processing and Visualization'
  source_content_hash: sha256:394b86a4014f3ca4b58d192e286d6626e8f27be092f86a5d8dbf9a2ef4c6a8bf
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# mass-spectrometry-runtime-benchmarking

## Summary

A benchmarking skill for timing equivalent mass spectral processing pipelines across Python libraries (spectrum_utils, pymzML, pyOpenMS) on the iPRG2012.mgf spectra, handling the Numba JIT first-call outlier for spectrum_utils and reporting median per-spectrum runtimes and speedup ratios.

## When to use

Use when comparing runtime efficiency of mass spectrometry processing libraries on identical peak-processing steps (m/z trimming, precursor/noise removal, intensity filtering and scaling) over MGF spectra.

## When NOT to use

- When the goal is peak quality or correctness comparison rather than computational runtime — the pipelines here are timed, not validated for output equivalence.
- When input spectra need to be read by spectrum_utils itself — it does not provide functionality to read spectral data files, so file I/O timing with spectrum_utils cannot be benchmarked this way.
- When the target library's first-call cost matters for your use case — this skill deliberately excludes the first spectrum_utils call as Numba JIT-compilation overhead.

## Inputs

- iPRG2012.mgf mass spectral data file
- spectrum_utils v0.3.0, pymzML v2.4.4, pyOpenMS v2.4.0 installed in Python
- Pyteomics for MGF parsing
- NumPy for timing aggregation

## Outputs

- speedup_table.csv with library, median/mean per-spectrum runtime, and speedup ratio
- Median and mean per-spectrum processing times for each library

## How to apply

Load spectra from iPRG2012.mgf using pyteomics.mgf, omitting spectra with fewer than 10 peaks or lacking a charge annotation so all three libraries operate on identical inputs. For spectrum_utils (v0.3.0), time per spectrum: set_mz_range to 100–1400 m/z, remove_precursor_peak at 0.02 Da tolerance, filter_intensity with min_intensity 0.05 and max_num_peaks 150, and scale_intensity 'root'. For pymzML (v2.4.4), time the equivalent operations (m/z window reduction, remove_precursor_peak, remove_noise, sqrt intensity scaling); for pyOpenMS (v2.4.0), use ParentPeakMower (0.02 Da window), ThresholdMower (0.05), NLargest with n=150, and SqrtMower plus Normalizer. Exclude the first spectrum_utils call as Numba JIT-compilation overhead, then compute median per-spectrum processing times and speedup ratios of pymzML and pyOpenMS relative to spectrum_utils, writing the table (library, median/mean runtime, speedup ratio) to speedup_table.csv.

## Related tools

- **spectrum_utils** (Benchmarked library; provides set_mz_range, remove_precursor_peak, filter_intensity, scale_intensity for the timed pipeline) — https://github.com/bittremieux/spectrum_utils
- **pymzML** (Baseline library timed for equivalent operations (m/z reduction, remove_precursor_peak, remove_noise, sqrt scaling)) — https://github.com/pymzml/pymzML
- **pyOpenMS** (Baseline library timed via ParentPeakMower, ThresholdMower, NLargest, SqrtMower and Normalizer filters)
- **Pyteomics** (Parses iPRG2012.mgf and supplies identically filtered spectra to all three libraries)

## Examples

```
import pyteomics.mgf as mgf; spectra = [s for s in mgf.read("iPRG2012.mgf") if len(s['m/z array']) >= 10]
```

## Evaluation signals

- All three libraries process the same filtered spectrum set (>=10 peaks, charge annotation present), so per-spectrum counts must match across benchmark runs.
- The first spectrum_utils timing is excluded before aggregating; if included, spectrum_utils mean runtime would be distorted by Numba JIT compilation.
- Median per-spectrum runtimes for spectrum_utils should be lower than pymzML and pyOpenMS after JIT exclusion, yielding speedup ratios > 1 for the baselines.
- speedup_table.csv exists with exactly the columns: library, median/mean per-spectrum runtime, speedup ratio.
- Filter parameters match across libraries: 100–1400 m/z window, 0.02 Da precursor tolerance, 0.05 intensity threshold, 150 max peaks, sqrt intensity scaling.

## Limitations

- spectrum_utils does not provide functionality to read spectral data files because there already exist several excellent tools to perform this task, so file loading must be delegated to Pyteomics or similar.
- The only significant outlier for spectrum_utils is the first-call runtime due to Numba JIT compilation; naive benchmarking without excluding it will misrepresent steady-state performance.
- Fairness of the comparison depends on mapping semantically equivalent operations across libraries (e.g., pyOpenMS needs SqrtMower plus a Normalizer to match scale_intensity 'root').

## Evidence

- [methods] mgf_filename = "iPRG2012.mgf": "mgf_filename = "iPRG2012.mgf""
- [methods] Restrict the mass range to 100–1400 m/z to filter out irrelevant peaks.: [evidence span withheld — anchor status: not_found]
- [methods] Low-intensity noise peaks are removed.: "Low-intensity noise peaks are removed."
- [intro] A key feature of spectrum_utils is its focus on computational efficiency to process large amounts of spectral data.: [evidence span withheld — anchor status: not_found]
- [discussion] spectrum_utils does not provide functionality to read spectral data files because there already exist several excellent tools to perform this task.: [evidence span withheld — anchor status: not_found]
- [readme] spectrum_utils is a Python package for efficient mass spectrometry data processing and visualization.: "spectrum_utils is a Python package for efficient mass spectrometry data processing and visualization."
