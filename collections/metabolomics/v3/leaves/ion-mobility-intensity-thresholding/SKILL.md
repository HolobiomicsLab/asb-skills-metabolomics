---
name: ion-mobility-intensity-thresholding
description: Use when loading mzML-derived LC-IMS-MS/MS data onto a common m/z, drift
  time, retention time, intensity grid and the 3D signal span must be reduced to make
  local-maximum peak detection tractable.
when_to_use_negative:
- Input is already a detected feature table where peaks have been partitioned and
  thresholded at 1000; applying a pre-detection threshold of 500 again would discard
  low-intensity signals that survived the first pass.
- Processing 2D LC-MS data without drift time (no IMS dimension); the multidimensional
  threshold is designed for the 3D space of m/z, drift time, retention time, intensity.
- When downstream steps require retention of sub-500 intensity noise (e.g. manual
  exploration of the data or tuning mix peak confirmation), because the nominal threshold
  removes that signal span.
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: DEIMoS
  role: Provides deimos.threshold for nominal intensity cutoff and deimos.peakpick.partition
    for peak partitioning in the 3D m/z, drift time, retention time, intensity space.
  repo: https://github.com/pnnl/deimos
- name: scipy.ndimage maximum_filter
  role: Local-maximum filter applied simultaneously across all dimensions to locate
    candidate ion coordinates before the post-detection threshold.
  repo: https://github.com/pnnl/deimos
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.1c05017
    title: 'DEIMoS: An Open-Source Tool for Processing High-Dimensional Mass Spectrometry
      Data'
  source_content_hash: sha256:de3bce93506e2eb3b92df059092c96641721de95a3d4dda0361e23d03b942d91
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/pnnl/deimos
  repo_url: https://github.com/pnnl/deimos
  tools:
  - DEIMoS
  - Python
  - scipy.ndimage maximum_filter
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# ion-mobility-intensity-thresholding

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Applies nominal intensity cutoffs to LC-IMS-MS/MS signal in m/z, drift time, retention time, intensity space before and after local-maximum peak detection, reducing the signal span and isolating true ion coordinates.

## When to use

Use when loading mzML-derived LC-IMS-MS/MS data onto a common m/z, drift time, retention time, intensity grid and the 3D signal span must be reduced to make local-maximum peak detection tractable.

## When NOT to use

- Input is already a detected feature table where peaks have been partitioned and thresholded at 1000; applying a pre-detection threshold of 500 again would discard low-intensity signals that survived the first pass.
- Processing 2D LC-MS data without drift time (no IMS dimension); the multidimensional threshold is designed for the 3D space of m/z, drift time, retention time, intensity.
- When downstream steps require retention of sub-500 intensity noise (e.g. manual exploration of the data or tuning mix peak confirmation), because the nominal threshold removes that signal span.

## Inputs

- mzML(.gz) LC-IMS-MS/MS data files from an Agilent 6560 Ion Mobility LC/Q-TOF system (e.g. MassIVE MSV000091746)
- DEIMoS 3D data structure indexed by m/z, drift time, retention time, intensity

## Outputs

- Downselected peak coordinate table of m/z, drift time, retention time, intensity saved as HDF5 (e.g. example_data_peaks.h5)
- Pre-thresholded signal grid suitable for local-maximum filtering

## How to apply

Load the mzML(.gz) files (e.g. from MassIVE deposit MSV000091746, Agilent 6560 example data) into DEIMoS's common m/z, drift time, retention time, intensity grid. Apply a nominal intensity threshold (threshold=500, per the DEIMoS user guide) with deimos.threshold to decrease the signal span of the 3D space prior to peak detection. Then apply a local-maximum filter (scipy.ndimage maximum_filter with a fixed kernel) simultaneously across all dimensions to locate candidate ion coordinates, partition peaks with deimos.peakpick.partition, and apply a second post-detection threshold of 1000 to the peak table. Perform these steps separately for positive and negative ionization mode files and save each feature coordinate table (m/z, drift time, retention time, intensity) as HDF5.

## Related tools

- **DEIMoS** (Provides deimos.threshold for nominal intensity cutoff and deimos.peakpick.partition for peak partitioning in the 3D m/z, drift time, retention time, intensity space.) — https://github.com/pnnl/deimos
- **scipy.ndimage maximum_filter** (Local-maximum filter applied simultaneously across all dimensions to locate candidate ion coordinates before the post-detection threshold.) — https://github.com/pnnl/deimos

## Examples

```
ms1 = deimos.threshold(ms1, threshold=500); ms1_peaks = deimos.threshold(ms1_peaks, threshold=1000)
```

## Evaluation signals

- Post-detection peak table must contain only coordinates with intensity ≥ 1000; verify with ms1_peaks = deimos.load('example_data_peaks.h5', key='ms1') and checking the minimum intensity column value.
- Pre-detection grid must be reduced in span relative to the raw grid — confirm that deimos.threshold(ms1, threshold=500) was applied before peak detection by re-reading the pipeline order from the user guide.
- Positive and negative ionization mode files must be processed separately and each must yield its own feature coordinate table saved as HDF5.
- The thresholded grid should retain enough signal to allow downstream feature detection, alignment, and CCS calibration steps to run without empty outputs.

## Limitations

- Threshold values (500 pre-detection, 1000 post-detection) are nominal and data-dependent; the user guide's values were tuned for the Agilent 6560 example data and may require adjustment for other instruments or metabolomics/exposomics datasets.
- The threshold is applied on a fixed nominal scale — if the signal intensity distribution is heavy-tailed or log-scaled, a single cutoff may over- or under-filter.
- In MS2 extraction, a separate lower threshold of 100 is applied (ms1 = deimos.threshold(ms1, threshold=100); ms2 = deimos.threshold(ms2, threshold=100)), so do not conflate the MS1 and MS2 threshold values.

## Evidence
