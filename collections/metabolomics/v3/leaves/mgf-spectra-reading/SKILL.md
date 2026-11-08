---
name: mgf-spectra-reading
description: Use when the input is an MGF peak list file (e.g. iPRG2012.mgf) whose
  spectra must be loaded into Python for downstream processing or benchmarking with
  spectrum_utils, pymzML, or pyOpenMS.
when_to_use_negative:
- Input is mzML rather than MGF and you want to keep the reader inside the same package
  stack — pymzML or pyOpenMS handle mzML directly
- You expect spectrum_utils itself to open the file — it does not provide file-reading
  functionality and relies on external tools
- The goal is interactive spectrum lookup from online repositories — the README's
  USI-based loading path is the appropriate route instead
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3520
tools:
- name: Pyteomics
  role: Provides pyteomics.mgf, the reader used to load the iPRG2012.mgf spectra before
    processing/benchmarking
- name: spectrum_utils
  role: Downstream consumer of the loaded spectra for processing (m/z range restriction,
    precursor/noise removal, intensity filtering/scaling) and visualization
  repo: https://github.com/bittremieux/spectrum_utils
- name: pymzML
  role: Alternative reader/processor used to load and reduce the same spectra for
    the runtime comparison
  repo: https://github.com/pymzml/pymzML
- name: pyOpenMS
  role: Alternative processing framework (ParentPeakMower, ThresholdMower, NLargest,
    SqrtMower, Normalizer) applied to the loaded spectra in the comparison
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.9b04884
    title: '<tt>spectrum_utils</tt> : A Python Package for Mass Spectrometry Data
      Processing and Visualization'
  source_content_hash: sha256:dc6b77dadf1c3cc989e7deec60131ec23251b3a0f7aad86e9114e85511246e1d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bittremieux/spectrum_utils
  tools:
  - Pyteomics
  - Python
  - pyOpenMS
  - pymzML
  - spectrum_utils
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# mgf-spectra-reading

## Summary

Loads MS/MS spectra from MGF peak list files (e.g. iPRG2012.mgf) using an external reader such as pyteomics.mgf, because spectrum_utils itself deliberately does not read spectral data files and instead consumes spectra supplied by other tools.

## When to use

Use when the input is an MGF peak list file (e.g. iPRG2012.mgf) whose spectra must be loaded into Python for downstream processing or benchmarking with spectrum_utils, pymzML, or pyOpenMS.

## When NOT to use

- Input is mzML rather than MGF and you want to keep the reader inside the same package stack — pymzML or pyOpenMS handle mzML directly
- You expect spectrum_utils itself to open the file — it does not provide file-reading functionality and relies on external tools
- The goal is interactive spectrum lookup from online repositories — the README's USI-based loading path is the appropriate route instead

## Inputs

- MGF peak list file (e.g. iPRG2012.mgf) containing MS/MS spectra with m/z and intensity peak arrays
- Reader library such as pyteomics.mgf (or pymzML for mzML input)

## Outputs

- Per-spectrum Python objects with peak m/z arrays and intensity arrays, keyed by spectrum title
- An iterable of valid spectra ready for spectrum_utils / pymzML / pyOpenMS processing

## How to apply

Use pyteomics.mgf to iterate over the spectra in the MGF file, reading each spectrum title, m/z array, and intensity array before handing the data to downstream processing (e.g. spectrum_utils set_mz_range(100, 1400), remove_precursor_peak, filter_intensity, scale_intensity, or the equivalent pymzML/pyOpenMS filter chains). Skip invalid/empty spectra when timing or processing per-spectrum. Rationale: spectrum_utils intentionally excludes file I/O — 'there already exist several excellent tools to perform this task' — so an external reader (pyteomics.mgf, or pymzML for mzML) must supply the spectral data. The same reader-driven loading pattern underpins the paper's runtime comparison of spectrum_utils vs pymzML 2.4.4 and pyOpenMS 2.4.0 on the iPRG2012 spectra.

## Related tools

- **Pyteomics** (Provides pyteomics.mgf, the reader used to load the iPRG2012.mgf spectra before processing/benchmarking)
- **spectrum_utils** (Downstream consumer of the loaded spectra for processing (m/z range restriction, precursor/noise removal, intensity filtering/scaling) and visualization) — https://github.com/bittremieux/spectrum_utils
- **pymzML** (Alternative reader/processor used to load and reduce the same spectra for the runtime comparison) — https://github.com/pymzml/pymzML
- **pyOpenMS** (Alternative processing framework (ParentPeakMower, ThresholdMower, NLargest, SqrtMower, Normalizer) applied to the loaded spectra in the comparison)

## Examples

```
from pyteomics import mgf
for spectrum in mgf.read("iPRG2012.mgf"):
    peaks = list(zip(spectrum['m/z array'], spectrum['intensity array']))
```

## Evaluation signals

- The MGF file is iterated to completion and valid spectra are read with m/z and intensity arrays populated (no empty arrays for accepted spectra)
- Each loaded spectrum supports the subsequent processing chain, e.g. set_mz_range(100, 1400) plus remove_precursor_peak, filter_intensity(0.05, 150), scale_intensity('root', 1) without errors
- Per-spectrum runtimes and results collected from the loaded spectra reproduce the paper's Figure 2 pattern when aggregated into log-scaled boxplots (runtime.png)
- The first spectrum_utils processing call shows the expected Numba JIT compilation outlier, confirming the loaded spectra actually flow through the spectrum_utils pipeline

## Limitations

- spectrum_utils does not provide functionality to read spectral data files; an external reader (e.g. pyteomics.mgf) must supply the data
- Invalid or malformed MGF spectra must be skipped explicitly during iteration (the workflow times only 'each valid spectrum')
- Runtime measurements of the loading+processing pipeline include a one-off Numba JIT compilation cost on the first spectrum_utils call

## Evidence

- [discussion] spectrum_utils does not provide functionality to read spectral data files because there already exist several excellent tools to perform this task.: [evidence span withheld — anchor status: not_found]
- [methods] mgf_filename = "iPRG2012.mgf": "mgf_filename = "iPRG2012.mgf""
- [methods] Read the iPRG2012 spectra from the iPRG2012.mgf file using pyteomics.mgf.: [evidence span withheld — anchor status: not_found]
- [abstract] similar basic functionality to read mass spectral data from standard file formats, process it, and visualize it: "similar basic functionality to read mass spectral data from standard file formats, process it, and visualize it"
- [readme] Spectrum loading from online proteomics and metabolomics data resources using the Universal Spectrum Identifier (USI) mechanism.: "Spectrum loading from online proteomics and metabolomics data resources using the Universal Spectrum Identifier (USI) mechanism."
