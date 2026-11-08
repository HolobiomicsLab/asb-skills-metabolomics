---
name: lc-ims-msms-feature-coordinate-comparison
description: Use when evaluating whether multidimensional (3D) feature detection on
  LC-IMS-MS/MS metabolomics data improves detection sensitivity or feature coordinates
  relative to MS-DIAL-style staged lower-dimensional (m/z vs RT, then drift time)
  peak detection.
when_to_use_negative:
- Input is already a curated feature table with per-feature m/z, drift time, and retention
  time rather than raw or thresholded peak lists
- Data lack an ion mobility dimension (i.e. LC-MS only), since the comparison hinges
  on the m/z–drift time–retention time 3D space
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: DEIMoS
  role: Provides deimos.load, deimos.threshold, and deimos.peakpick for 3D and staged
    subspace feature detection on LC-IMS-MS/MS data
  repo: https://github.com/pnnl/deimos
- name: Python
  role: Execution environment for the DEIMoS API used to run and compare the detection
    strategies
  repo: https://github.com/pnnl/deimos
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.1c05017
    title: 'DEIMoS: An Open-Source Tool for Processing High-Dimensional Mass Spectrometry
      Data'
  source_content_hash: sha256:4fe65589b313b9188f99b1d7f1db2f8fe4edfc824c63853cf1c857060bca0b47
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
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# lc-ims-msms-feature-coordinate-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Compares features detected by DEIMoS's native 3D peak picking on LC-IMS-MS/MS data (m/z, drift time, retention time simultaneously) against lower-dimensional subspace permutations (2D→1D, 1D→2D, successive 1D), quantifying coordinate deviations and intersection rates to assess the sensitivity cost of staged processing.

## When to use

Use when evaluating whether multidimensional (3D) feature detection on LC-IMS-MS/MS metabolomics data improves detection sensitivity or feature coordinates relative to MS-DIAL-style staged lower-dimensional (m/z vs RT, then drift time) peak detection.

## When NOT to use

- Input is already a curated feature table with per-feature m/z, drift time, and retention time rather than raw or thresholded peak lists
- Data lack an ion mobility dimension (i.e. LC-MS only), since the comparison hinges on the m/z–drift time–retention time 3D space

## Inputs

- LC-IMS-MS/MS example dataset from MassIVE in HDF5 format (e.g. example_data.h5), acquired on an Agilent 6560 Ion Mobility LC/Q-TOF
- Raw MS1 peak list loaded via deimos.load, intensity-thresholded with deimos.threshold (threshold=500)

## Outputs

- Per-strategy feature counts for native 3D peak picking and each subspace permutation (2D→1D, 1D→2D, successive 1D)
- Matched-feature coordinate deviations: ppm (m/z), percent (drift time), minutes (retention time)
- Intersection percentages between 3D and subspace detections at strict and cross-sample alignment tolerances
- ndim_vs_subspace_comparison.csv

## How to apply

Load the example LC-IMS-MS/MS dataset from MassIVE with deimos.load (e.g. example_data.h5) and apply a nominal intensity threshold via deimos.threshold (threshold=500 on raw MS1 data, 1000 on picked peaks) to decrease the signal span of the 3D space. Run native peak detection on the full m/z–drift time–retention time space using deimos.peakpick so all dimensions are utilized simultaneously, then re-run detection on the same input using iterative subspace permutations: m/z versus RT followed by DT (2D→1D, analogous to MS-DIAL), a 1D→2D ordering, and successive 1D processing. Match detected features across strategies and compute coordinate deviations in ppm (m/z), percent (drift time), and minutes (retention time), plus per-strategy feature counts. Tabulate counts and deviations per strategy and save as ndim_vs_subspace_comparison.csv.

## Related tools

- **DEIMoS** (Provides deimos.load, deimos.threshold, and deimos.peakpick for 3D and staged subspace feature detection on LC-IMS-MS/MS data) — https://github.com/pnnl/deimos
- **Python** (Execution environment for the DEIMoS API used to run and compare the detection strategies) — https://github.com/pnnl/deimos

## Examples

```
ms1 = deimos.load("example_data.h5", key="ms1"); ms1 = deimos.threshold(ms1, threshold=500); peaks = deimos.peakpick.partials(ms1, split_on=["mz", "drift", "retention"], processes=8)
```

## Evaluation signals

- Native 3D processing detects at least as many features as each staged subspace permutation
- Comparing 3D processing to m/z versus RT followed by DT yields average coordinate deviations on the order of 5.8 ppm (m/z), 0.2% (drift time), and 0.05 min (retention time)
- Strict-tolerance intersection between 3D and 2D→1D detections is ~13%, rising to ~90% at the tolerances used in cross-sample alignment
- Output CSV ndim_vs_subspace_comparison.csv contains per-strategy counts and deviation columns
- Thresholded inputs show reduced signal span (threshold=500 on raw MS1, 1000 on picked peaks) before peak picking

## Limitations

- Intersection between 3D and the most comparable subspace technique is only ~13% at strict tolerances, so matched-feature comparisons depend heavily on the tolerance regime chosen
- The staged m/z versus RT followed by DT approach (MS-DIAL-style) shows poor agreement with native 3D processing
- A nominal intensity threshold must be applied first to decrease the signal span of the 3D space; results are sensitive to that choice

## Evidence
