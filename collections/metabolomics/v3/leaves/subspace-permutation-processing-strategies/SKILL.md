---
name: subspace-permutation-processing-strategies
description: Use when evaluating whether multidimensional (N-dim) feature detection
  improves sensitivity over staged lower-dimensional processing on LC-IMS-MS/MS data,
  or when comparing a DEIMoS pipeline to MS-DIAL-style (m/z versus RT followed by
  DT) approaches.
when_to_use_negative:
- Data is 1D or 2D only (e.g. LC-MS without ion mobility) — there is no 3D native
  space to compare against subspace permutations.
- Input is already a curated feature table; this skill operates on thresholded raw/picked
  peak lists prior to alignment.
- The goal is cross-sample alignment or CCS calibration rather than comparing detection-strategy
  sensitivity.
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: DEIMoS
  role: Python API used for deimos.load, deimos.threshold, and deimos.peakpick to
    run native 3D and subspace-permutation peak detection on LC-IMS-MS/MS data
  repo: https://github.com/pnnl/deimos
- name: MS-DIAL
  role: Reference implementation of the most comparable lower-dimensional strategy
    (m/z versus RT followed by DT) against which the 2D→1D permutation is benchmarked
- name: Python
  role: Language environment for the DEIMoS API used to orchestrate detection, matching,
    and deviation computation
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.1c05017
    title: 'DEIMoS: An Open-Source Tool for Processing High-Dimensional Mass Spectrometry
      Data'
  source_content_hash: sha256:02025cd357eab465e51163bdfb1ce87579ae755bf75d3583031e541f318184a2
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
  - MS-DIAL
  - Python
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# subspace-permutation-processing-strategies

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmarks DEIMoS's native 3D peak detection (m/z–drift time–retention time simultaneously) against lower-dimensional subspace permutations (2D→1D, 1D→2D, successive 1D) on LC-IMS-MS/MS data, quantifying feature-count and coordinate-deviation differences.

## When to use

Use when evaluating whether multidimensional (N-dim) feature detection improves sensitivity over staged lower-dimensional processing on LC-IMS-MS/MS data, or when comparing a DEIMoS pipeline to MS-DIAL-style (m/z versus RT followed by DT) approaches.

## When NOT to use

- Data is 1D or 2D only (e.g. LC-MS without ion mobility) — there is no 3D native space to compare against subspace permutations.
- Input is already a curated feature table; this skill operates on thresholded raw/picked peak lists prior to alignment.
- The goal is cross-sample alignment or CCS calibration rather than comparing detection-strategy sensitivity.

## Inputs

- LC-IMS-MS/MS example dataset from MassIVE as an HDF5 file (e.g. example_data.h5), loaded via deimos.load
- Raw MS1 peak list (thresholded at 500) and picked-peak MS1 list (thresholded at 1000)

## Outputs

- Detected feature coordinate tables per processing strategy (native 3D, 2D→1D, 1D→2D, successive 1D)
- ndim_vs_subspace_comparison.csv tabulating per-strategy feature counts and coordinate deviations (ppm for m/z, % for drift time, min for retention time)

## How to apply

Load the LC-IMS-MS/MS dataset from MassIVE with deimos.load (e.g. example_data.h5) and apply a nominal intensity threshold via deimos.threshold (threshold=500 on raw MS1 data, 1000 on picked peaks) to decrease the signal span of the 3D space. Run native peak detection on the full m/z–drift time–retention time space with deimos.peakpick so all dimensions are utilized simultaneously. Re-run detection on the same input using iterative subspace permutations: m/z versus retention time followed by drift time (2D→1D, the MS-DIAL-style analogue), a 1D→2D ordering, and successive 1D processing. Match detected features across strategies and compute coordinate deviations in ppm (m/z), percent (drift time), and minutes (retention time) plus per-strategy feature counts, tabulating results as ndim_vs_subspace_comparison.csv. The rationale is that simultaneous use of all dimensions offers greater separation between features, improving detection sensitivity relative to staged subspace processing.

## Related tools

- **DEIMoS** (Python API used for deimos.load, deimos.threshold, and deimos.peakpick to run native 3D and subspace-permutation peak detection on LC-IMS-MS/MS data) — https://github.com/pnnl/deimos
- **MS-DIAL** (Reference implementation of the most comparable lower-dimensional strategy (m/z versus RT followed by DT) against which the 2D→1D permutation is benchmarked)
- **Python** (Language environment for the DEIMoS API used to orchestrate detection, matching, and deviation computation)

## Examples

```
ms1 = deimos.load("example_data.h5"); ms1 = deimos.threshold(ms1, threshold=500); features_3d = deimos.peakpick(ms1, ...)
```

## Evaluation signals

- Native 3D processing should detect at least as many features as each staged subspace permutation.
- Comparing 3D processing to m/z versus RT followed by DT, feature coordinate deviations should average on the order of 5.8 ppm (m/z), 0.2% (drift time), and 0.05 min (retention time).
- Intersection of features between native 3D and the most comparable subspace strategy should be only ~13% at strict tolerances but rise to ~90% when imposing the tolerances used in cross-sample alignment.
- Intensity thresholds applied before peak picking: threshold=500 on raw MS1 data and 1000 on picked peaks.
- Output table ndim_vs_subspace_comparison.csv contains per-strategy feature counts and deviations in the expected units (ppm, %, min).

## Limitations

- At strict tolerances, agreement between native 3D and lower-dimensional approaches is poor (~13% intersection in both positive and negative mode); only ~90% agreement is reached at cross-sample alignment tolerances.
- The m/z versus RT followed by DT strategy (as implemented by MS-DIAL) results in poor agreement with native 3D processing.
- A nominal intensity threshold must be applied first to decrease the signal span of the 3D space; unthresholded data inflates the processing space.

## Evidence
