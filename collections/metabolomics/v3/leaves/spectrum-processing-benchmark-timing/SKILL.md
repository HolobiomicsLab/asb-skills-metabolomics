---
name: spectrum-processing-benchmark-timing
description: Use when comparing the computational efficiency of Python mass-spectrometry
  processing libraries (e.g., spectrum_utils vs pymzML vs pyOpenMS) on a common spectral
  dataset such as an MGF file.
when_to_use_negative:
- When the goal is peak detection or identification quality rather than processing-speed
  comparison
- When input spectra cannot be read into a common representation (spectrum_utils itself
  does not read spectral data files and relies on external tools such as pyteomics.mgf)
- 'When runtimes must be JIT-free: the first spectrum_utils call includes Numba JIT
  compilation and would distort steady-state timing comparisons'
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_0091
tools:
- name: spectrum_utils
  role: Library under benchmark; its set_mz_range, remove_precursor_peak, filter_intensity,
    and scale_intensity calls are timed per spectrum
  repo: https://github.com/bittremieux/spectrum_utils
- name: pymzML
  role: Comparator library (v2.4.4); its reduce, remove_precursor_peak, remove_noise,
    normalize, and square-root scaling are timed as the equivalent processing sequence
  repo: https://github.com/pymzml/pymzML
- name: pyOpenMS
  role: Comparator library (v2.4.0); ParentPeakMower, ThresholdMower, NLargest, SqrtMower,
    and Normalizer filters are timed as the equivalent processing sequence
- name: Pyteomics
  role: Used to read the iPRG2012.mgf spectra that serve as the common benchmark input
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.9b04884
    title: '<tt>spectrum_utils</tt> : A Python Package for Mass Spectrometry Data
      Processing and Visualization'
  source_content_hash: sha256:05798afde3d842b55360e692e873ec64668a7e30e92aa28d336c39c632e327a4
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# spectrum-processing-benchmark-timing

## Summary

Benchmarks per-spectrum MS/MS processing runtime by timing an equivalent processing sequence (m/z restriction, precursor/noise removal, intensity filtering, intensity scaling) in spectrum_utils, pymzML 2.4.4, and pyOpenMS 2.4.0 on the iPRG2012 spectra, and visualizes per-spectrum timings as a log-scaled boxplot.

## When to use

Use when comparing the computational efficiency of Python mass-spectrometry processing libraries (e.g., spectrum_utils vs pymzML vs pyOpenMS) on a common spectral dataset such as an MGF file.

## When NOT to use

- When the goal is peak detection or identification quality rather than processing-speed comparison
- When input spectra cannot be read into a common representation (spectrum_utils itself does not read spectral data files and relies on external tools such as pyteomics.mgf)
- When runtimes must be JIT-free: the first spectrum_utils call includes Numba JIT compilation and would distort steady-state timing comparisons

## Inputs

- iPRG2012.mgf (MGF-format MS/MS spectra read via pyteomics.mgf)
- spectrum_utils, pymzML 2.4.4, and pyOpenMS 2.4.0 installed in a Python 3.6+ environment

## Outputs

- Per-spectrum processing runtimes for spectrum_utils, pymzML, and pyOpenMS
- Log-scaled boxplot figure (runtime.png) comparing per-spectrum processing times across the three libraries

## How to apply

Read the spectra from iPRG2012.mgf using pyteomics.mgf, then for each valid spectrum time the full processing sequence per library. For spectrum_utils, time set_mz_range(100, 1400), remove_precursor_peak(0.02, 'Da'), filter_intensity(0.05, 150), and scale_intensity('root', 1); the equivalent pymzML 2.4.4 sequence is spec.reduce('raw', (100, 1400)), remove_precursor_peak(), remove_noise(), normalization, and square-root intensity scaling; the equivalent pyOpenMS 2.4.0 sequence uses ParentPeakMower, ThresholdMower(0.05), NLargest(150), SqrtMower, and Normalizer filters. Collect per-spectrum runtimes for all three libraries and aggregate them into a log-scaled boxplot (runtime.png). Retain the first spectrum_utils call as-is so the Numba JIT compilation outlier is visible, since after compilation processing is very efficient; the rationale is that spectrum_utils is explicitly designed for computational efficiency on large spectral data volumes.

## Related tools

- **spectrum_utils** (Library under benchmark; its set_mz_range, remove_precursor_peak, filter_intensity, and scale_intensity calls are timed per spectrum) — https://github.com/bittremieux/spectrum_utils
- **pymzML** (Comparator library (v2.4.4); its reduce, remove_precursor_peak, remove_noise, normalize, and square-root scaling are timed as the equivalent processing sequence) — https://github.com/pymzml/pymzML
- **pyOpenMS** (Comparator library (v2.4.0); ParentPeakMower, ThresholdMower, NLargest, SqrtMower, and Normalizer filters are timed as the equivalent processing sequence)
- **Pyteomics** (Used to read the iPRG2012.mgf spectra that serve as the common benchmark input)

## Examples

```
spec.set_mz_range(100, 1400).remove_precursor_peak(0.02, 'Da').filter_intensity(0.05, 150).scale_intensity('root', 1)
```

## Evaluation signals

- runtime.png is produced and uses a log-scaled y-axis for per-spectrum processing times
- spectrum_utils per-spectrum runtimes are lower than both pymzML 2.4.4 and pyOpenMS 2.4.0 across the iPRG2012 spectra, consistent with the paper's Figure 2
- The single spectrum_utils outlier corresponds to the first call (Numba JIT compilation); subsequent calls are very efficient
- Each library's pipeline covers the same four operations: 100–1400 m/z restriction, precursor peak removal, low-intensity filtering (5% base-peak threshold, top-N peaks), and square-root intensity scaling

## Limitations

- The first spectrum_utils timing includes Numba JIT compilation, producing an outlier that must be reported or handled explicitly
- spectrum_utils does not provide functionality to read spectral data files, so file loading must be delegated to an external tool such as pyteomics.mgf and is not part of the timed processing step
- The library-specific processing sequences are approximations of each other (e.g., parameter conventions differ between APIs), so timing comparisons depend on the chosen equivalent-step mapping

## Evidence

- [intro] A key feature of spectrum_utils is its focus on computational efficiency to process large amounts of spectral data.: [evidence span withheld — anchor status: not_found]
- [methods] Remove low-intensity noise peaks by only retaining peaks that are at at least 5% of the base peak intensity: "Remove low-intensity noise peaks by only retaining peaks that are at at least 5% of the base peak intensity"
- [methods] Scale the peak intensities by their square root to de-emphasize overly intense peaks.: "Scale the peak intensities by their square root to de-emphasize overly intense peaks."
- [readme] Common spectrum processing operations (precursor & noise peak removal, intensity filtering, intensity scaling): "Common spectrum processing operations (precursor & noise peak removal, intensity filtering, intensity scaling)"
- [discussion] spectrum_utils has a well-defined, Pythonic application programming interface, allowing developers to easily harness its powerful functionality: [evidence span withheld — anchor status: not_found]
