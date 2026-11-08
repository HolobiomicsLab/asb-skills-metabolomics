---
name: ion-mobility-mass-spectrometry-data-processing
description: Use when processing high-dimensional LC-IMS-MS/MS (e.g. Agilent 6560,
  Bruker, Waters) data where m/z, drift time, and retention time must be jointly leveraged
  for feature detection, alignment, and CCS-calibrated characterization.
when_to_use_negative:
- Input is already a fully processed feature table from another tool — DEIMoS is for
  raw high-dimensional MS data workflows, not downstream statistics.
- Data are 1D LC-MS with no drift-time (ion mobility) dimension — the N-dimensional
  detection/alignment machinery offers no benefit over conventional tools.
- Cross-sample alignment of non-IMS data where the SVR reference-based drift correction
  (mz_tol=200e-6, dt_tol=0.04) would be over-parameterized.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: DEIMoS
  role: Primary API and command-line tool executing feature detection, alignment,
    CCS calibration, isotope detection, and MS/MS deconvolution on N-dimensional data
  repo: github.com/pnnl/deimos
- name: Python
  role: Runtime environment for the DEIMoS application programming interface and downstream
    validation (e.g. r-squared reporting)
- name: SVR
  role: Support vector regression with RBF kernel (scikit-learn) used to fit and apply
    per-dimension drift corrections during alignment
- name: MS-DIAL
  role: Benchmark sequential (m/z vs RT then DT) processing approach against which
    DEIMoS's simultaneous multidimensional intersection (~90% vs ~13%) is compared
provenance:
  source_task_ids:
  - task_003
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.1c05017
    title: 'DEIMoS: An Open-Source Tool for Processing High-Dimensional Mass Spectrometry
      Data'
  source_content_hash: sha256:43d376b541f8886f19bfa5d1e623cfa8214914afe3aba384a2a550c7ed2c2cdf
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
  - MS-DIAL
  - Python
  - SVR
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# ion-mobility-mass-spectrometry-data-processing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

End-to-end processing of LC-IMS-MS/MS metabolomics data using DEIMoS: N-dimensional feature detection, cross-sample SVR alignment, CCS calibration, isotope screening, and MS/MS deconvolution, all operating simultaneously across m/z, drift time, and retention time.

## When to use

Use when processing high-dimensional LC-IMS-MS/MS (e.g. Agilent 6560, Bruker, Waters) data where m/z, drift time, and retention time must be jointly leveraged for feature detection, alignment, and CCS-calibrated characterization.

## When NOT to use

- Input is already a fully processed feature table from another tool — DEIMoS is for raw high-dimensional MS data workflows, not downstream statistics.
- Data are 1D LC-MS with no drift-time (ion mobility) dimension — the N-dimensional detection/alignment machinery offers no benefit over conventional tools.
- Cross-sample alignment of non-IMS data where the SVR reference-based drift correction (mz_tol=200e-6, dt_tol=0.04) would be over-parameterized.

## Inputs

- Per-sample H5 files of instrument data (e.g. example_data.h5, example_alignment.h5, example_tune_pos.h5, example_tune_neg.h5)
- Peak-picked tune mix calibrant datasets with known CCS values
- Study sample / blank / QC sample designations (Agilent 6560, Bruker, or Waters acquisitions)

## Outputs

- Detected features aligned across study samples, characterized by mass, CCS, tandem mass spectra, and isotopic signature
- SVR alignment coefficients (per m/z, drift time, retention time) for each sample
- CCS calibration table with beta, tfix, correlation coefficient, and sum of residuals per ionization mode
- Deconvolved MS2 spectra

## How to apply

Load per-sample H5 tables (e.g. deimos.load on example_data.h5 / example_alignment.h5) into DEIMoS's N-dimensional particle-particle structures. Apply a nominal intensity threshold (e.g. deimos.threshold at 500 for raw signal) to decrease the signal span, then run peak detection simultaneously across m/z, drift time, and retention time rather than the sequential m/z-vs-RT then DT approach of MS-DIAL, which the paper shows yields only ~13% feature intersection vs ~90% with joint tolerances. Align samples by designating an in-study reference, matching features within mz_tol=200e-6 and dt_tol=0.04, then fitting per-dimension support vector regression (RBF kernel in scikit-learn) to correct nonlinear drift (e.g. degrading LC column) and storing fitted coefficients. Calibrate CCS with the tune mix calibrants (e.g. example_tune_pos.h5) by building calibration factors/index and fitting beta and tfix; the Agilent positive-mode fit reached r-squared 0.9999784552958134. Downselect isotopic signatures to ≥3 members and ≤50 ppm m/z error, and deconvolve MS2 spectra to mitigate convolution artifacts.

## Related tools

- **DEIMoS** (Primary API and command-line tool executing feature detection, alignment, CCS calibration, isotope detection, and MS/MS deconvolution on N-dimensional data) — github.com/pnnl/deimos
- **Python** (Runtime environment for the DEIMoS application programming interface and downstream validation (e.g. r-squared reporting))
- **SVR** (Support vector regression with RBF kernel (scikit-learn) used to fit and apply per-dimension drift corrections during alignment)
- **MS-DIAL** (Benchmark sequential (m/z vs RT then DT) processing approach against which DEIMoS's simultaneous multidimensional intersection (~90% vs ~13%) is compared)

## Examples

```
import deimos; ms1 = deimos.threshold(deimos.load("example_data.h5", key="ms1"), threshold=500); data = {"A": deimos.load("example_alignment.h5", key="A")}
```

## Evaluation signals

- Cross-sample feature intersection jumps from ~13% to ~90% when DEIMoS's joint tolerances (mz_tol=200e-6, dt_tol=0.04) are applied, confirming correct alignment parameterization.
- Corrected feature coordinates overlap the alignment reference within the specified tolerances; stored SVR coefficients reproduce the fitted transform.
- CCS calibration correlation coefficient approaches 1 (paper reports 0.9999784552958134 for Agilent positive-mode tune mix) with low sum of residuals.
- 3D processing vs sequential m/z-vs-RT-then-DT feature coordinates deviate by ~5.8 ppm (m/z), ~0.2% (drift time), ~0.05 min (retention time) — flag values far outside these ranges.
- Isotopic signatures accepted only when they contain at least 3 members and are consistent within 50 ppm m/z error of the expected signature.

## Limitations

- The paper's data are instrument-specific (Agilent 6560, Bruker, Waters); DEIMoS is 'largely agnostic to acquisition instrumentation' but the exact tolerances (200 ppm m/z, 0.04 drift time) were tuned to these acquisitions.
- Cross-sample alignment acceptance relies on qualitative overlap within tolerances — no quantitative acceptance threshold is reported in the paper.
- CCS calibration requires peak-picked tune mix data; if calibrant peaks are not detected via peak detection or manual exploration, the beta/tfix fit cannot proceed.
- Few open-source solutions are positioned to fully leverage the latest instrumentation; DEIMoS's 1.6.x API may lag behind emerging instrument output formats.

## Evidence
