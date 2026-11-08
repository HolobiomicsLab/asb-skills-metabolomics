---
name: noise-peak-filtering
description: Use when MS/MS spectra read from an MGF or mzML file need low-intensity
  noise peaks removed and peak counts capped before plotting, spectrum comparison,
  or downstream processing.
when_to_use_negative:
- Spectrum is already cleaned (e.g. output of a prior ThresholdMower/NLargest pyOpenMS
  pipeline) — reapplying filter_intensity would be redundant
- Task is file-format conversion or raw spectral data ingestion — spectrum_utils explicitly
  does not read spectral files, so use pyteomics.mgf, pymzML, or pyOpenMS for loading
  instead
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: spectrum_utils
  role: Primary implementation of the filter chain (set_mz_range, remove_precursor_peak,
    filter_intensity, scale_intensity) optimized for computational efficiency on large
    spectral volumes
  repo: https://github.com/bittremieux/spectrum_utils
- name: pymzML
  role: Alternative library providing equivalent filtering (reduce, remove_precursor_peak,
    remove_noise, normalize) used in the iPRG2012 runtime comparison against spectrum_utils
  repo: https://github.com/pymzml/pymzML
- name: pyOpenMS
  role: C++-backed alternative using ParentPeakMower, ThresholdMower (0.05), NLargest
    (150), and SqrtMower filters to achieve the same noise-peak filtering
- name: pyteomics
  role: Used to read the iPRG2012.mgf spectra prior to applying the spectrum_utils
    filtering pipeline
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.9b04884
    title: '<tt>spectrum_utils</tt> : A Python Package for Mass Spectrometry Data
      Processing and Visualization'
  source_content_hash: sha256:a0812fe6d42bf6f50b5ca39bd7ecf13cd1106f6f054162dede699fe204ccace9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bittremieux/spectrum_utils
  tools:
  - Python
  - pyOpenMS
  - pymzML
  - pyteomics
  - spectrum_utils
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 15
---

# noise-peak-filtering

## Summary

A spectrum-processing skill from the spectrum_utils ecosystem that removes low-intensity noise peaks and caps peak count, as applied in the iPRG2012 MS/MS benchmark where spectra were restricted to 100–1400 m/z, precursor peaks removed, and only peaks at ≥5% of base-peak intensity (≤150 most intense) retained.

## When to use

Use when MS/MS spectra read from an MGF or mzML file need low-intensity noise peaks removed and peak counts capped before plotting, spectrum comparison, or downstream processing.

## When NOT to use

- Spectrum is already cleaned (e.g. output of a prior ThresholdMower/NLargest pyOpenMS pipeline) — reapplying filter_intensity would be redundant
- Task is file-format conversion or raw spectral data ingestion — spectrum_utils explicitly does not read spectral files, so use pyteomics.mgf, pymzML, or pyOpenMS for loading instead

## Inputs

- MS/MS spectra in a standard spectral file format (e.g. iPRG2012.mgf read with pyteomics.mgf, or mzML read with pymzML/pyOpenMS)
- Filter parameters: m/z range (100–1400), precursor peak tolerance (0.02 Da), minimum relative intensity (0.05 of base peak), maximum peak count (150)

## Outputs

- Filtered spectrum object with precursor and low-intensity peaks removed, ready for intensity scaling, annotation, plotting, or comparison
- Per-spectrum runtime measurements for benchmarking (e.g. boxplot of processing times across libraries)

## How to apply

Load spectra from a standard spectral file (e.g. iPRG2012.mgf read with pyteomics.mgf), then apply the filter sequence: first restrict the m/z range with set_mz_range(100, 1400) to discard irrelevant low/high m/z peaks, remove the precursor peak via remove_precursor_peak(0.02, 'Da'), and finally call filter_intensity(0.05, 150) to retain only peaks at at least 5% of the base-peak intensity and no more than the 150 most intense peaks. The rationale, per the article, is that low-intensity noise peaks clutter spectra and can degrade downstream analysis or visualization; intensity scaling (e.g. scale_intensity('root', 1)) is applied afterwards to de-emphasize overly intense peaks. Equivalent operations exist in pymzML (remove_noise) and pyOpenMS (ThresholdMower at 0.05, NLargest 150), so the skill can be benchmarked across libraries. Validate by confirming the filtered spectrum retains only high-intensity peaks within the 100–1400 m/z window and that per-spectrum processing is fast enough for large data volumes (spectrum_utils is orders of magnitude faster than pymzML 2.4.4/pyOpenMS 2.4.0, excluding the one-time Numba JIT cost).

## Related tools

- **spectrum_utils** (Primary implementation of the filter chain (set_mz_range, remove_precursor_peak, filter_intensity, scale_intensity) optimized for computational efficiency on large spectral volumes) — https://github.com/bittremieux/spectrum_utils
- **pymzML** (Alternative library providing equivalent filtering (reduce, remove_precursor_peak, remove_noise, normalize) used in the iPRG2012 runtime comparison against spectrum_utils) — https://github.com/pymzml/pymzML
- **pyOpenMS** (C++-backed alternative using ParentPeakMower, ThresholdMower (0.05), NLargest (150), and SqrtMower filters to achieve the same noise-peak filtering)
- **pyteomics** (Used to read the iPRG2012.mgf spectra prior to applying the spectrum_utils filtering pipeline)

## Examples

```
from spectrum_utils import spectrum as sus; spec = sus.MsmsSpectrum(spectrum_id, precursor_mz, charge, mz, intensity); spec.set_mz_range(100, 1400).remove_precursor_peak(0.02, 'Da').filter_intensity(0.05, 150)
```

## Evaluation signals

- All retained peaks lie within the specified m/z window (100–1400 m/z) and the precursor peak is absent at the 0.02 Da tolerance
- No retained peak has an intensity below 5% of the base peak, and the total number of retained peaks does not exceed 150
- A log-scaled boxplot of per-spectrum processing times (runtime.png) shows spectrum_utils runtimes lower than pymzML 2.4.4 and pyOpenMS 2.4.0
- The first spectrum_utils call is slower due to Numba JIT compilation, and subsequent calls are very efficient — confirm this outlier appears as an expected one-off
- Peak intensities can be scaled by square root (scale_intensity('root', 1)) without error, indicating the filtered spectrum is well-formed for downstream processing

## Limitations

- spectrum_utils does not provide functionality to read spectral data files, so an external tool (pyteomics.mgf, pymzML, or pyOpenMS) must be used for loading
- The first spectrum_utils call incurs Numba JIT compilation overhead, appearing as a one-time runtime outlier
- The source document is a bioRxiv preprint that was not certified by peer review

## Evidence

- [methods] filter_intensity with base-peak threshold and peak cap: [evidence span withheld — anchor status: not_found]
- [methods] m/z range restriction and precursor peak removal: [evidence span withheld — anchor status: not_found]
- [abstract] Computational-efficiency rationale: "the data processing functionality is highly optimized for computational eﬀiciency to be able to deal with the large volumes of data"
- [results] Runtime benchmark result: [evidence span withheld — anchor status: not_found]
- [discussion] External file reading limitation: [evidence span withheld — anchor status: not_found]
