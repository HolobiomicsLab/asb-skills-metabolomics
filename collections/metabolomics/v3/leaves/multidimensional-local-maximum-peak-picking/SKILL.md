---
name: multidimensional-local-maximum-peak-picking
description: Use when raw LC-IMS-MS/MS data (mzML) has been loaded into a common m/z,
  drift time, retention time, intensity grid and you need to locate ion peaks in 3D
  without sequential 1D/2D processing that would yield only ~13% cross-sample intersection.
when_to_use_negative:
- Input is already a detected feature table rather than raw high-dimensional MS data
  — peak picking on pre-partitioned peaks would double-count ions.
- Data are low-dimensional (e.g. conventional LC-MS without ion mobility), where DEIMoS's
  3D simultaneous filtering has no advantage over m/z versus RT processing followed
  by drift time.
- You need only a coarse peak list without replicate validation; skipping the analytical-triplicate
  confirmation discards the article's confidence criterion.
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: DEIMoS
  role: Python API and command-line tool that performs the multidimensional local-maximum
    feature detection (deimos.threshold, deimos.peakpick.partition) on the loaded
    m/z, drift time, retention time, intensity grid and writes feature tables to HDF5.
  repo: github.com/pnnl/deimos
- name: Python
  role: Runtime language hosting DEIMoS and scipy.ndimage (maximum_filter) used to
    apply the local-maximum filter across all dimensions simultaneously.
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.1c05017
    title: 'DEIMoS: An Open-Source Tool for Processing High-Dimensional Mass Spectrometry
      Data'
  source_content_hash: sha256:f3e49ddf7ed03f7611914168290a122525779763243d2bec7ce938af744e961d
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/pnnl/deimos
  tools:
  - DEIMoS
  - Python
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# Multidimensional local maximum peak picking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A feature-detection method from DEIMoS that applies a local-maximum filter (scipy.ndimage maximum_filter) simultaneously across m/z, drift time, and retention time of LC-IMS-MS/MS data, then partitions peaks around maxima and thresholds them to produce per-mode feature coordinate tables.

## When to use

Use when raw LC-IMS-MS/MS data (mzML) has been loaded into a common m/z, drift time, retention time, intensity grid and you need to locate ion peaks in 3D without sequential 1D/2D processing that would yield only ~13% cross-sample intersection.

## When NOT to use

- Input is already a detected feature table rather than raw high-dimensional MS data — peak picking on pre-partitioned peaks would double-count ions.
- Data are low-dimensional (e.g. conventional LC-MS without ion mobility), where DEIMoS's 3D simultaneous filtering has no advantage over m/z versus RT processing followed by drift time.
- You need only a coarse peak list without replicate validation; skipping the analytical-triplicate confirmation discards the article's confidence criterion.

## Inputs

- mzML or mzML.gz files from LC-IMS-MS/MS acquisition (e.g. Agilent 6560 Ion Mobility LC/Q-TOF data)
- Common m/z, drift time, retention time, intensity grid loaded in DEIMoS
- Fixed-kernel dimensions for the local-maximum filter
- Threshold values (nominal pre-detection threshold, e.g. 500; post-detection threshold, e.g. 1000)

## Outputs

- Per-ionization-mode feature coordinate tables in HDF5 (m/z, drift time, retention time, intensity)
- Detected peak list partitioned around local maxima (deimos.peakpick.partition output)
- Replicate-consistency check: features observed across analytical triplicates

## How to apply

Load mzML(.gz) files into DEIMoS's common m/z, drift time, retention time, intensity grid, then apply a nominal intensity threshold (threshold=500 in the article's example) to decrease the signal span of the 3D space prior to peak detection. Apply a local-maximum filter (scipy.ndimage maximum_filter with a fixed kernel) simultaneously across all dimensions to locate candidate ion coordinates; operating on all dimensions at once offers greater separation between features and improves detection sensitivity. Partition peaks around the local maxima using deimos.peakpick.partition, then apply a post-detection intensity threshold of 1000. Repeat separately for positive and negative ionization mode files and save each feature coordinate table (m/z, drift time, retention time, intensity) as HDF5. Validate by confirming that detected features are present in all analytical triplicate injections of each study sample.

## Related tools

- **DEIMoS** (Python API and command-line tool that performs the multidimensional local-maximum feature detection (deimos.threshold, deimos.peakpick.partition) on the loaded m/z, drift time, retention time, intensity grid and writes feature tables to HDF5.) — github.com/pnnl/deimos
- **Python** (Runtime language hosting DEIMoS and scipy.ndimage (maximum_filter) used to apply the local-maximum filter across all dimensions simultaneously.)

## Examples

```
ms1 = deimos.threshold(ms1, threshold=500); ms1_peaks = deimos.peakpick.partition(ms1, kernel=[5, 5, 5], cutoffs=[3, 3, 3]); ms1_peaks = deimos.threshold(ms1_peaks, threshold=1000); deimos.save('example_data_peaks.h5', ms1_peaks)
```

## Evaluation signals

- Detected features are present in all analytical triplicate injections of each study sample (the article's confidence criterion).
- Feature coordinate tables contain m/z, drift time, retention time, and intensity columns and are saved as HDF5.
- Peak count rises substantially compared to a sequential m/z-versus-RT-then-drift-time processing route (the article reports only ~13% intersection for that route versus ~90% when the same tolerances are applied).
- Pre-detection and post-detection intensity thresholds (e.g. 500 then 1000) have been applied in the correct order.
- The 3D and sequential results deviate by ~5.8 ppm in m/z, ~0.2% in drift time, and ~0.05 min in retention time on average when compared — deviations within these ranges indicate consistent processing.

## Limitations

- Local-maximum filtering on 3D data requires a nominal intensity threshold to decrease the signal span before peak detection; skipping this step makes the filter impractical.
- The method is designed for LC-IMS-MS/MS (m/z, drift time, retention time) data; applying it to lower-dimensional data forfeits the separation advantage that motivated multidimensional processing.
- Detected features must be observed across analytical triplicates to provide additional confidence; single-injection detections are not validated by this skill alone.
- The 3D peak-picking output deviates slightly from the sequential m/z-versus-RT-then-drift-time route (~5.8 ppm in m/z, ~0.2% in drift time, ~0.05 min in retention time on average), which may matter for downstream alignment.

## Evidence
