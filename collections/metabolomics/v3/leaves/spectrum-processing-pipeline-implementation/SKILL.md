---
name: spectrum-processing-pipeline-implementation
description: Use when prototyping a Python MS/MS preprocessing pipeline (m/z windowing,
  precursor and noise-peak removal, intensity filtering/scaling) on spectra loaded
  from MGF or mzML, or when benchmarking processing runtimes across spectrum_utils,
  pymzML, and pyOpenMS.
when_to_use_negative:
- 'Reading raw spectral data files: spectrum_utils does not provide file-reading functionality
  and takes MS data supplied by external tools as input'
- Analyzing already-processed spectra — peak annotation of processed peaks assumes
  raw peaks have not yet been removed
- Situations where a monolithic end-to-end MS data-processing tool fully covers the
  experimental use case and no custom workflow is needed
edam_operation: http://edamontology.org/operation_3628
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: spectrum_utils
  role: Primary library providing the high-level spectrum processing functions (set_mz_range,
    remove_precursor_peak, filter_intensity, scale_intensity) benchmarked in the pipeline
  repo: https://github.com/bittremieux/spectrum_utils
- name: Pyteomics
  role: Reads iPRG2012.mgf spectra and omits spectra with <10 peaks or no charge annotation
    before processing
- name: pymzML
  role: 'Comparison library performing equivalent operations: m/z-window reduction,
    remove_precursor_peak, remove_noise, and sqrt intensity scaling on mzML-parsed
    spectra'
  repo: https://github.com/pymzml/pymzML
- name: pyOpenMS
  role: Comparison library applying ParentPeakMower (0.02 Da window), ThresholdMower
    (0.05), NLargest (n=150), and SqrtMower plus Normalizer
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.9b04884
    title: '<tt>spectrum_utils</tt> : A Python Package for Mass Spectrometry Data
      Processing and Visualization'
  source_content_hash: sha256:4933cb0f40bbbebb535621faef971fcf823b689e7e58fe1c114a1ce04d71417a
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
  edam_operation: http://edamontology.org/operation_3628
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 14
---

# spectrum-processing-pipeline-implementation

## Summary

A spectrum_utils-based pipeline that processes MS/MS spectra from iPRG2012 MGF data — restricting m/z to 100–1400, removing the precursor peak, filtering low-intensity noise peaks, and sqrt-scaling intensities — with a pymzML/pyOpenMS runtime comparison and a Numba first-call JIT caveat. Use it to implement and benchmark identical spectral preprocessing across Python mass-spectrometry libraries.

## When to use

Use when prototyping a Python MS/MS preprocessing pipeline (m/z windowing, precursor and noise-peak removal, intensity filtering/scaling) on spectra loaded from MGF or mzML, or when benchmarking processing runtimes across spectrum_utils, pymzML, and pyOpenMS.

## When NOT to use

- Reading raw spectral data files: spectrum_utils does not provide file-reading functionality and takes MS data supplied by external tools as input
- Analyzing already-processed spectra — peak annotation of processed peaks assumes raw peaks have not yet been removed
- Situations where a monolithic end-to-end MS data-processing tool fully covers the experimental use case and no custom workflow is needed

## Inputs

- iPRG2012.mgf mass-spectrometry spectra file (MGF format) loaded via pyteomics.mgf
- Per-spectrum peak lists in mzML/MGF form containing m/z and intensity arrays and precursor metadata (charge, precursor m/z)

## Outputs

- Processed spectrum objects with restricted m/z range (100–1400), precursor peak removed, ≥5% base-peak intensity filter and 150-peak cap, and sqrt-scaled intensities
- speedup_table.csv: per-library median/mean per-spectrum processing runtimes and speedup ratios of pymzML and pyOpenMS relative to spectrum_utils, with the first spectrum_utils call excluded as JIT-compilation overhead

## How to apply

Load spectra from iPRG2012.mgf with pyteomics.mgf, omitting spectra with fewer than 10 peaks or without a charge annotation. Apply the spectrum_utils pipeline per spectrum: set_mz_range to restrict peaks to 100–1400 m/z, remove_precursor_peak with a 0.02 Da tolerance, filter_intensity keeping peaks at ≥5% of base-peak intensity with max_num_peaks 150, then scale_intensity('root') to sqrt-scale intensities. Mirror the same operations in pymzML (reduce to the m/z window, remove_precursor_peak, remove_noise, sqrt scaling) and pyOpenMS (ParentPeakMower at a 0.02 Da window, ThresholdMower at 0.05, NLargest with n=150, SqrtMower plus Normalizer) so the libraries perform identical transformations. Exclude the first spectrum_utils call as Numba JIT-compilation overhead before computing median per-spectrum runtimes and speedup ratios relative to spectrum_utils; write the results as a speedup table (e.g. speedup_table.csv). spectrum_utils deliberately omits file-reading functionality — rely on external readers such as pyteomics or pymzML to supply spectra.

## Related tools

- **spectrum_utils** (Primary library providing the high-level spectrum processing functions (set_mz_range, remove_precursor_peak, filter_intensity, scale_intensity) benchmarked in the pipeline) — https://github.com/bittremieux/spectrum_utils
- **Pyteomics** (Reads iPRG2012.mgf spectra and omits spectra with <10 peaks or no charge annotation before processing)
- **pymzML** (Comparison library performing equivalent operations: m/z-window reduction, remove_precursor_peak, remove_noise, and sqrt intensity scaling on mzML-parsed spectra) — https://github.com/pymzml/pymzML
- **pyOpenMS** (Comparison library applying ParentPeakMower (0.02 Da window), ThresholdMower (0.05), NLargest (n=150), and SqrtMower plus Normalizer)

## Examples

```
from pyteomics import mgf; import spectrum_utils.spectrum as sus
spec = sus.MsmsSpectrum('iprg', pep_mass, 2, peaks); spec.set_mz_range(100, 1400).remove_precursor_peak(0.02).filter_intensity(0.05, 150).scale_intensity('root')
```

## Evaluation signals

- Processed spectra contain no peaks below 100 or above 1400 m/z
- No peak remains within the 0.02 Da precursor window; the highest remaining intensity equals the base-peak intensity and all kept peaks are ≥5% of it, capped at 150 peaks
- Peak intensities are square-root-scaled (comparable magnitudes after sqrt transform)
- Timing comparison excludes the first spectrum_utils call (Numba JIT compilation outlier); median per-spectrum runtimes show spectrum_utils ≥ pymzML and ≥ pyOpenMS efficiency
- Spectra with fewer than 10 peaks or lacking a charge annotation are excluded from both processing and benchmarking

## Limitations

- The first spectrum_utils processing call incurs a significant runtime outlier due to Numba JIT compilation and must be excluded from benchmarking
- spectrum_utils does not read spectral data files; spectra must be supplied by external tools such as pyteomics or pymzML
- Peak annotation of processed peaks can be inaccurate because filtering has already removed raw peaks
- The exact speedup depends on the task and can differ between different configurations
- The source is a bioRxiv preprint that was not certified by peer review

## Evidence

- [methods] spectrum_utils processes the iPRG2012 spectra with a defined pipeline: [evidence span withheld — anchor status: not_found]
- [methods] filter_intensity retains only peaks ≥5% of base peak and caps peak count: "Remove low-intensity noise peaks by only retaining peaks that are at at least 5% of the base peak intensity and restrict the total number of peaks to the 50 most intense peaks."
- [results] First-call runtime outlier due to Numba JIT compilation: [evidence span withheld — anchor status: not_found]
- [discussion] spectrum_utils relies on external tools for file reading: [evidence span withheld — anchor status: not_found]
- [intro] Computational efficiency is the stated design goal of the package: [evidence span withheld — anchor status: not_found]
- [readme] README lists the supported common processing operations: "Common spectrum processing operations (precursor & noise peak removal, intensity filtering, intensity scaling)"
