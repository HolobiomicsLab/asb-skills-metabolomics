---
name: lc-ims-msms-metabolomics-data-processing
description: Use when raw LC-IMS-MS/MS data (e.g., from Agilent 6560, Bruker, or Waters
  instruments) must be processed end-to-end into aligned feature tables characterized
  by mass, CCS, tandem mass spectra, and isotopic signature, or when 3D (m/z, drift
  time, retention time) processing is needed instead of.
when_to_use_negative:
- When the input is already a feature table and the goal is only downstream statistical
  analysis rather than raw data feature detection, alignment, and deconvolution.
- When the instrument produces only 2D (m/z vs retention time) data with no drift-time
  dimension; DEIMoS's multidimensional advantage (3D processing) requires drift time,
  so 2D processing (m/z versus RT followed by DT) yields poor agreement and ~13% intersection
  instead of ~90%.
- When the goal is merely to identify compounds by a single technique without needing
  to leverage multiple non-m/z dimensions for CCS calibration or MS/MS deconvolution.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: DEIMoS
  role: Python API and command-line tool for high-dimensional mass spectrometry data
    analysis workflows; provides feature detection, feature alignment, CCS calibration,
    isotope detection, and MS/MS spectral deconvolution on N-dimensional LC-IMS-MS/MS
    data.
  repo: https://github.com/pnnl/deimos
- name: Python
  role: Language in which DEIMoS is implemented and invoked (e.g., deimos.threshold,
    deimos.load, deimos.calibration, deimos.deconvolution modules).
- name: MS-DIAL
  role: Reference/comparison tool; the sequential 2D processing approach (m/z versus
    RT followed by DT) as implemented by MS-DIAL results in poor agreement with DEIMoS's
    3D processing (~13% intersection).
- name: Bruker
  role: Instrument vendor whose acquired data (54 data files for 9 study samples in
    triplicate, spanning 2 ionization modes, 83 gigabytes) is processed by this workflow.
- name: Waters
  role: Instrument vendor whose acquired data (21 data files for 6 study samples in
    triplicate and 3 blanks, 16 gigabytes) is processed by this workflow.
- name: Agilent 6560 Ion Mobility LC/Q-TOF
  role: Instrument whose example data (Agilent 6560 Ion Mobility LC/Q-TOF system)
    is used to demonstrate the DEIMoS workflow.
provenance:
  source_task_ids:
  - task_006
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.1c05017
    title: 'DEIMoS: An Open-Source Tool for Processing High-Dimensional Mass Spectrometry
      Data'
  source_content_hash: sha256:cf6e31a853d8b89e3c3ebf5a767b37c41634e1947cfbf2b7f518cb34981b0f9d
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
  - Agilent 6560 Ion Mobility LC/Q-TOF
  - Bruker
  - DEIMoS
  - LC-IMS-MS/MS
  - MS-DIAL
  - Python
  - Waters
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# lc-ims-msms-metabolomics-data-processing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

DEIMoS processes LC-IMS-MS/MS metabolomics data by performing feature detection, feature alignment, CCS calibration, isotope detection, and MS/MS spectral deconvolution on N-dimensional m/z, drift time, and retention time data simultaneously, leveraging all dimensions to improve detection sensitivity and mitigate convolution artifacts in tandem mass spectra.

## When to use

Use when raw LC-IMS-MS/MS data (e.g., from Agilent 6560, Bruker, or Waters instruments) must be processed end-to-end into aligned feature tables characterized by mass, CCS, tandem mass spectra, and isotopic signature, or when 3D (m/z, drift time, retention time) processing is needed instead of sequential 2D steps.

## When NOT to use

- When the input is already a feature table and the goal is only downstream statistical analysis rather than raw data feature detection, alignment, and deconvolution.
- When the instrument produces only 2D (m/z vs retention time) data with no drift-time dimension; DEIMoS's multidimensional advantage (3D processing) requires drift time, so 2D processing (m/z versus RT followed by DT) yields poor agreement and ~13% intersection instead of ~90%.
- When the goal is merely to identify compounds by a single technique without needing to leverage multiple non-m/z dimensions for CCS calibration or MS/MS deconvolution.

## Inputs

- LC-IMS-MS/MS mzML data files (e.g., Waters Synapt G2Si LC-IMS-MS/MS mzML from MetaboLights MTBLS812)
- Tune mix reference compounds of known CCS (e.g., Waters tune mix)
- DEIMoS HDF5 peak tables (e.g., example_data.h5, example_data_peaks.h5, example_alignment.h5)
- N-dimensional m/z, drift time, retention time signal data
- MS1 precursor and MS2 fragment peak tables

## Outputs

- Detected features aligned across study samples
- Feature table characterized by mass, CCS, tandem mass spectra, and isotopic signature
- ccs_calibration_coefficients.csv with calibration coefficients and per-compound residuals
- Per-MS1:MS2 pairing similarity scores and deconvolved MS2 fragment assignments
- HDF5 output files (e.g., example_alignment.h5)

## How to apply

Load LC-IMS-MS/MS data with DEIMoS and process in N-dimensional space rather than sequentially (m/z vs RT then DT), because simultaneous use of all dimensions offers greater separation between features and increases alignment/feature matching confidence. Apply a nominal intensity threshold to decrease signal span of the 3D space (e.g., threshold=500 for MS1 data, threshold=1000 for peak tables, threshold=100 for MS2). Perform feature detection, cross-sample feature alignment, CCS calibration (e.g., log-log linear regression for traveling-wave Waters data using tune mix compounds of known CCS), isotope detection constrained by drift and retention time tolerances (e.g., tol=[0.1, 0.7, 0.15] for m/z, drift, retention), and MS/MS spectral deconvolution via cosine similarity of profiles in joint drift time-retention time space. The output comprises detected features aligned across study samples and characterized by mass, CCS, tandem mass spectra, and isotopic signature.

## Related tools

- **DEIMoS** (Python API and command-line tool for high-dimensional mass spectrometry data analysis workflows; provides feature detection, feature alignment, CCS calibration, isotope detection, and MS/MS spectral deconvolution on N-dimensional LC-IMS-MS/MS data.) — https://github.com/pnnl/deimos
- **Python** (Language in which DEIMoS is implemented and invoked (e.g., deimos.threshold, deimos.load, deimos.calibration, deimos.deconvolution modules).)
- **MS-DIAL** (Reference/comparison tool; the sequential 2D processing approach (m/z versus RT followed by DT) as implemented by MS-DIAL results in poor agreement with DEIMoS's 3D processing (~13% intersection).)
- **Bruker** (Instrument vendor whose acquired data (54 data files for 9 study samples in triplicate, spanning 2 ionization modes, 83 gigabytes) is processed by this workflow.)
- **Waters** (Instrument vendor whose acquired data (21 data files for 6 study samples in triplicate and 3 blanks, 16 gigabytes) is processed by this workflow.)
- **Agilent 6560 Ion Mobility LC/Q-TOF** (Instrument whose example data (Agilent 6560 Ion Mobility LC/Q-TOF system) is used to demonstrate the DEIMoS workflow.)

## Examples

```
import deimos; ms1 = deimos.load('example_data.h5'); ms1 = deimos.threshold(ms1, threshold=500); ms1_peaks = deimos.threshold(ms1_peaks, threshold=1000)
```

## Evaluation signals

- Verify that the output feature table is parseable and contains aligned features across study samples with mass, CCS, tandem mass spectra, and isotopic signature columns.
- For CCS calibration, check that all fitted log-log regression coefficients are finite and that per-compound residuals show no systematic trend across the calibration range.
- For MS/MS deconvolution, confirm that all retained MS1:MS2 pairings have cosine similarity scores exceeding the user-defined tolerance and that the score table exists.
- Compare 3D processing against the 2D (m/z vs RT followed by DT) baseline; the 2D approach yields only ~13% intersection in both positive and negative mode, rising to ~90% when applying alignment tolerances, whereas DEIMoS's multidimensional approach should avoid this discrepancy.
- Check that per-feature coordinate deviations between 3D processing and m/z versus RT followed by DT average ~5.8 ppm (m/z), 0.2% (drift time), and 0.05 min (retention time), indicating agreement between the two approaches.

## Limitations

- Sequential 2D processing (m/z versus RT followed by DT) yields poor agreement and only ~13% intersection; DEIMoS requires the multidimensional (3D) approach to realize its advantages.
- For traveling-wave IMS data, no quantitative residual threshold is reported in the paper; accept the calibration when all fitted coefficients are finite and residuals show no systematic trend.
- The paper does not report a specific quantitative CCS calibration residual threshold, so correctness must be judged by absence of systematic trends rather than a numeric cutoff.
- Data processing requires the drift-time dimension; DEIMoS is largely agnostic to acquisition instrumentation but presumes the separation dimensions are present in the raw data.

## Evidence
