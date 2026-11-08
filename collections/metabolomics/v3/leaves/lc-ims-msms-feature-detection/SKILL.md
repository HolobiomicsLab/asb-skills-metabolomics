---
name: lc-ims-msms-feature-detection
description: Use when raw mzML(.gz) LC-IMS-MS/MS files (e.g. Agilent 6560 Ion Mobility
  LC/Q-TOF, MassIVE MSV000091746) must be converted into per-mode feature coordinate
  tables spanning m/z, drift time, retention time, and intensity.
when_to_use_negative:
- Input is already a feature table from a prior detection run (this skill operates
  on raw mzML spectra)
- Data lack an ion mobility (drift time) dimension, so 3D simultaneous detection is
  impossible and sequential m/z versus RT then DT processing would apply instead
- The goal is cross-sample feature alignment or CCS calibration rather than initial
  peak detection
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3292
- http://edamontology.org/topic_3520
tools:
- name: DEIMoS
  role: Python API used for loading mzML into the 3D grid, thresholding (deimos.threshold),
    peak partitioning (deimos.peakpick.partition), and exporting HDF5 feature tables
  repo: github.com/pnnl/deimos
- name: Python
  role: Implementation language for the DEIMoS API and the scipy.ndimage maximum_filter
    local-maximum detection
- name: scipy
  role: scipy.ndimage maximum_filter with a fixed kernel locates candidate ion coordinates
    as local maxima simultaneously across all dimensions
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.1c05017
    title: 'DEIMoS: An Open-Source Tool for Processing High-Dimensional Mass Spectrometry
      Data'
  source_content_hash: sha256:0f11f38ef76f755705b2d03b33c64781f92fbd7cdd9e8bacf9826c007d6c07be
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
  - scipy
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3292
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# LC-IMS-MS/MS feature detection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Detects molecular features directly in the 3D m/z–drift time–retention time space of LC-IMS-MS/MS data using DEIMoS, applying intensity thresholds and a simultaneous N-dimensional local-maximum filter instead of sequential 2D peak picking.

## When to use

Use when raw mzML(.gz) LC-IMS-MS/MS files (e.g. Agilent 6560 Ion Mobility LC/Q-TOF, MassIVE MSV000091746) must be converted into per-mode feature coordinate tables spanning m/z, drift time, retention time, and intensity.

## When NOT to use

- Input is already a feature table from a prior detection run (this skill operates on raw mzML spectra)
- Data lack an ion mobility (drift time) dimension, so 3D simultaneous detection is impossible and sequential m/z versus RT then DT processing would apply instead
- The goal is cross-sample feature alignment or CCS calibration rather than initial peak detection

## Inputs

- mzML or mzML.gz raw LC-IMS-MS/MS files per injection (e.g. MassIVE deposit MSV000091746, Agilent 6560 Ion Mobility LC/Q-TOF data)
- Ionization mode label (positive or negative) for each file
- Fixed kernel specification for the scipy.ndimage maximum_filter across m/z, drift time, and retention time

## Outputs

- Per-mode feature coordinate tables (m/z, drift time, retention time, intensity) saved as HDF5
- Triplicate-presence validation report listing features observed in all analytical triplicate injections

## How to apply

Load each mzML(.gz) file into DEIMoS's common m/z, drift time, retention time, intensity grid, then apply a nominal intensity threshold of 500 (deimos.threshold) to decrease the signal span of the 3D space prior to peak detection. Locate candidate ion coordinates with a local-maximum filter (scipy.ndimage maximum_filter with a fixed kernel) applied simultaneously across all dimensions, since multidimensional detection offers greater separation between features and improves sensitivity. Partition peaks around the maxima with deimos.peakpick.partition, then apply the post-detection intensity threshold of 1000 to retain only confident peaks. Repeat the workflow separately for positive and negative ionization mode files and save each feature coordinate table as HDF5. Validate by confirming that each detected feature is observed across all analytical triplicate injections of each study sample, which is the reproducibility requirement DEIMoS imposes for confidence in detected features.

## Related tools

- **DEIMoS** (Python API used for loading mzML into the 3D grid, thresholding (deimos.threshold), peak partitioning (deimos.peakpick.partition), and exporting HDF5 feature tables) — github.com/pnnl/deimos
- **Python** (Implementation language for the DEIMoS API and the scipy.ndimage maximum_filter local-maximum detection)
- **scipy** (scipy.ndimage maximum_filter with a fixed kernel locates candidate ion coordinates as local maxima simultaneously across all dimensions)

## Examples

```
ms1 = deimos.threshold(ms1, threshold=500); peaks = deimos.peakpick.partition(ms1, kernel=[...], deimos.threshold(peaks, threshold=1000)
```

## Evaluation signals

- Every reported feature is observed in all analytical triplicate injections of its study sample (the DEIMoS reproducibility requirement)
- Pre-detection deimos.threshold(threshold=500) and post-detection deimos.threshold(threshold=1000) were both applied
- Output HDF5 tables contain m/z, drift time, retention time, and intensity columns per the DEIMoS feature coordinate schema
- Positive and negative ionization modes were processed and exported as separate feature coordinate tables
- Candidate maxima were found by a single maximum_filter applied across all three dimensions simultaneously, not sequential 2D passes

## Limitations

- Detection sensitivity depends on the pre-threshold (nominal 500) reducing the signal span; too low a threshold inflates the search space
- Sequential 2D processing (m/z versus RT followed by DT, as in MS-DIAL) yields poor agreement (~13% intersection) with the 3D multidimensional approach, so results are not directly interchangeable
- The maximum_filter kernel is fixed by the user; kernel choice determines which ion coordinates qualify as local maxima

## Evidence
