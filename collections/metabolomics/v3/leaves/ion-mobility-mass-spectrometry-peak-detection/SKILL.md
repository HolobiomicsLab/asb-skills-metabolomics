---
name: ion-mobility-mass-spectrometry-peak-detection
description: Use when processing high-dimensional LC-IMS-MS/MS metabolomics data (e.g.,
  mzML from Waters Synapt G2Si, Bruker timsTOF, or Agilent 6560) where peak detection
  across m/z, drift time, and retention time dimensions is needed before feature alignment
  or CCS calibration.
when_to_use_negative:
- Input is already a feature table (post peak-picking) or aligned features across
  samples; peak detection is only for raw or pre-processed signal spaces
- Data is single-dimensional (e.g., MS1-only m/z or drift-time-only), where the multidimensional
  simultaneous-processing advantage does not apply
- The goal is tandem-MS/MS spectral deconvolution or cross-sample alignment rather
  than initial feature detection; use the corresponding DEIMoS step instead
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: DEIMoS
  role: Python API and command-line tool providing deimos.load, deimos.threshold,
    and deimos.peakpick for N-dimensional peak detection
  repo: https://github.com/pnnl/deimos
- name: Python
  role: Runtime environment for executing DEIMoS peak-picking workflows
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.1c05017
    title: 'DEIMoS: An Open-Source Tool for Processing High-Dimensional Mass Spectrometry
      Data'
  source_content_hash: sha256:8f13ec069824e7981806b830e76d91df9ee81ab87b515a07670ffaaff40faa6e
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
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# ion-mobility-mass-spectrometry-peak-detection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A method for detecting peaks in N-dimensional ion mobility–mass spectrometry data (m/z, drift time, retention time) using DEIMoS, which processes all dimensions simultaneously to improve feature separation and detection sensitivity.

## When to use

Use when processing high-dimensional LC-IMS-MS/MS metabolomics data (e.g., mzML from Waters Synapt G2Si, Bruker timsTOF, or Agilent 6560) where peak detection across m/z, drift time, and retention time dimensions is needed before feature alignment or CCS calibration.

## When NOT to use

- Input is already a feature table (post peak-picking) or aligned features across samples; peak detection is only for raw or pre-processed signal spaces
- Data is single-dimensional (e.g., MS1-only m/z or drift-time-only), where the multidimensional simultaneous-processing advantage does not apply
- The goal is tandem-MS/MS spectral deconvolution or cross-sample alignment rather than initial feature detection; use the corresponding DEIMoS step instead

## Inputs

- Raw LC-IMS-MS/MS data in an N-dimensional DEIMoS-compatible format (e.g., .h5 such as example_data.h5, or mzML-derived loaded via deimos.load)
- Nominal intensity threshold for the 3D/4D signal space (e.g., 500)
- Peak-detection threshold for filtered peak lists (e.g., 1000)

## Outputs

- Peak list DataFrame indexed by m/z, drift time, retention time, and intensity (e.g., written to example_data_peaks.h5)
- Filtered peak list after applying the second intensity threshold, ready for feature alignment or CCS calibration

## How to apply

Load the IMS-MS data into a DEIMoS pandas DataFrame via deimos.load (e.g., from example_data.h5 or mzML-derived structures), then apply a nominal intensity threshold (e.g., deimos.threshold(ms1, threshold=500) on the full 3D space) to decrease signal span before peak detection. Run deimos.peakpick on the N-dimensional grid so that feature separation is evaluated jointly across m/z, drift time, and retention time rather than dimension-by-dimension; this mitigates convolution artifacts and improves detection sensitivity compared with processing m/z versus RT followed by drift time alone. Apply a second, higher threshold to the detected peaks (e.g., deimos.threshold(ms1_peaks, threshold=1000)) to retain only robustly detected features. The result is a peak list in the same coordinate system (m/z, drift time, retention time, intensity) that feeds directly into downstream DEIMoS steps such as feature alignment, CCS calibration against tune mix references, or MS/MS deconvolution.

## Related tools

- **DEIMoS** (Python API and command-line tool providing deimos.load, deimos.threshold, and deimos.peakpick for N-dimensional peak detection) — https://github.com/pnnl/deimos
- **Python** (Runtime environment for executing DEIMoS peak-picking workflows)

## Examples

```
ms1 = deimos.load("example_data.h5", key="ms1"); ms1 = deimos.threshold(ms1, threshold=500); ms1_peaks = deimos.peakpick(ms1)
```

## Evaluation signals

- Detected peaks show coordinates that, when compared against m/z-versus-RT-then-DT processing, deviate by an average of ~5.8 ppm in m/z, ~0.2% in drift time, and ~0.05 min in retention time, reflecting the multidimensional approach's improved precision
- Peak list is non-empty and finite across all dimensions after thresholding at 1000
- Intersection with independently detected features increases substantially (~90%) when the same tolerances used for cross-sample alignment are imposed, indicating robust 3D detection
- Output DataFrame retains the same column schema (m/z, drift time, retention time, intensity) as the input, enabling seamless handoff to deimos align or calibration steps

## Limitations

- DEIMoS requires the input to be represented as an N-dimensional grid/DataFrame; data types not natively in this layout (e.g., raw vendor binary) must first be converted
- The article demonstrates the workflow on Agilent 6560, Bruker, and Waters LC-IMS-MS/MS metabolomics data; behavior on other instrument families or non-metabolomics matrices is not established
- Nominal thresholds (500, 1000) are illustrative for the reported datasets and must be re-tuned to the dynamic range of a new instrument or acquisition mode

## Evidence
