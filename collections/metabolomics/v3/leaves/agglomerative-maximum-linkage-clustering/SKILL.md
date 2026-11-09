---
name: agglomerative-maximum-linkage-clustering
description: Use when raw N-dimensional (m/z, drift time, retention time) mass spectrometry
  peak lists from multiple samples—e.g. the 152 plasma samples of the SRM 1950 study—must
  be reduced to one feature per row aligned across samples under tight tolerances.
when_to_use_negative:
- When the data are already aligned feature tables or the pipeline only operates on
  a single sample—maximum-linkage clustering across samples would add no matching
  step.
- When feature coordinates lie in fewer dimensions (e.g. only m/z versus RT); DEIMoS's
  approach relies on simultaneous use of all dimensions, and applying it in a reduced
  space forfeits the separation/confidence gains described.
- When feature detection/signal processing has not yet been run and the input is raw
  instrument data (e.g. Bruker/Waters/Agilent vendor formats) rather than peak lists—clustering
  requires pre-extracted peaks.
edam_operation: http://edamontology.org/operation_2938
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: DEIMoS
  role: Provides the API and command-line implementation of N-dimensional agglomerative
    clustering with maximum linkage used for both peak clustering and cross-sample
    alignment (deimos.partition, deimos.multi_sample_partition, deimos.collapse).
  repo: https://github.com/pnnl/deimos
- name: Python
  role: Language in which DEIMoS is implemented and in which the clustering/alignment
    workflow is scripted.
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.1c05017
    title: 'DEIMoS: An Open-Source Tool for Processing High-Dimensional Mass Spectrometry
      Data'
  source_content_hash: sha256:40ab671260ade1b7c1769e205b6b9e5aa0ab408793045e48cbd061cd2365617b
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
  edam_operation: http://edamontology.org/operation_2938
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# agglomerative-maximum-linkage-clustering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Agglomerative clustering with maximum (complete) linkage is DEIMoS's mechanism for both internal peak clustering within each sample and cross-sample feature alignment across the 912 data files of the SRM 1950 plasma study, simultaneously matching features in m/z, drift time, and retention time to boost alignment confidence.

## When to use

Use when raw N-dimensional (m/z, drift time, retention time) mass spectrometry peak lists from multiple samples—e.g. the 152 plasma samples of the SRM 1950 study—must be reduced to one feature per row aligned across samples under tight tolerances.

## When NOT to use

- When the data are already aligned feature tables or the pipeline only operates on a single sample—maximum-linkage clustering across samples would add no matching step.
- When feature coordinates lie in fewer dimensions (e.g. only m/z versus RT); DEIMoS's approach relies on simultaneous use of all dimensions, and applying it in a reduced space forfeits the separation/confidence gains described.
- When feature detection/signal processing has not yet been run and the input is raw instrument data (e.g. Bruker/Waters/Agilent vendor formats) rather than peak lists—clustering requires pre-extracted peaks.

## Inputs

- Per-sample peak tables (in-memory pandas DataFrames with m/z, drift time, retention time, intensity columns) loaded via deimos.load from HDF5 files
- Indexing structure produced by deimos.build_factors and deimos.build_index over all sample peak tables
- Alignment tolerances: ±20 ppm m/z, ±1.5% drift time, ±0.3 min retention time

## Outputs

- Aligned feature table (one row per feature, columns describing m/z, drift time, retention time and sample-level intensities) in a single HDF5 file (e.g. aligned_features.h5)
- Cluster membership assignments mapping individual sample peaks to consensus features

## How to apply

First cluster peaks within each sample: group adjacent points by peak detection thresholds, then run agglomerative clustering under maximum (complete) linkage using a ±20 ppm m/z and ±1.5% drift time tolerance so a cluster only merges when every member lies within tolerance in all dimensions simultaneously. For cross-sample alignment, build the indexing structure (deimos.build_factors, deimos.build_index), partition the feature space in N dimensions rather than sequentially (deimos.partition), and cluster features within each partition via deimos.multi_sample_partition under the same maximum linkage, widening tolerances to ±20 ppm m/z, ±1.5% drift time, and ±0.3 min retention time. Maximum linkage is the key rationale: it is the strictest criterion, requiring all points in a cluster to satisfy tolerance, which directly increases feature-matching confidence among datasets compared to centroid- or single-linkage approaches. Collapse the resulting clusters with deimos.collapse into a single feature-per-row table and save to HDF5 with deimos.save. Applied this way the authors resolved 11,698 positive-mode and 14,784 negative-mode features across the 3 analytical replicates of the SRM 1950 plasma study.

## Related tools

- **DEIMoS** (Provides the API and command-line implementation of N-dimensional agglomerative clustering with maximum linkage used for both peak clustering and cross-sample alignment (deimos.partition, deimos.multi_sample_partition, deimos.collapse).) — https://github.com/pnnl/deimos
- **Python** (Language in which DEIMoS is implemented and in which the clustering/alignment workflow is scripted.)

## Examples

```
data = deimos.partition(index, deimos.load("example_data.h5", key="ms1")); aligned = deimos.collapse(deimos.multi_sample_partition(data, linkage='maximum', mz_tol=20e-6, dt_tol=0.015, rt_tol=0.3))
```

## Evaluation signals

- Every aligned feature cluster must satisfy the maximum-linkage tolerance constraints in all dimensions simultaneously (±20 ppm m/z, ±1.5% drift time, ±0.3 min RT).
- The aligned HDF5 file loads cleanly with deimos.load and contains one row per consensus feature rather than per sample peak.
- Expected scale for the SRM 1950 plasma study: ~11,698 positive-mode and ~14,784 negative-mode features appearing across each of the 3 analytical replicates.
- Compared to sequential m/z-then-RT-then-drift-time processing, cross-feature intersection should rise substantially (≈90% vs ≈13%) when the same tolerances are imposed.
- Feature coordinate deviation between 3D processing and m/z-versus-RT followed by DT should average ≈5.8 ppm m/z, 0.2% drift time, and 0.05 min RT.

## Limitations

- Tolerances must be set carefully; over-wide tolerances merge distinct features, while over-tight ones split them, and maximum linkage makes the method sensitive to any single outlying member point.
- The workflow presumes prior feature detection; it does not extract peaks from vendor raw data.
- DEIMoS is agnostic to instrumentation but the example alignment tolerances are calibrated to the SRM 1950 plasma study; other data sets may need re-optimised tolerances.

## Evidence

- [methods] agglomerative clustering with maximum (complete) linkage: [evidence span withheld — anchor status: not_found]
- [methods] maximum (complete) linkage, i.e. tolerance is applied to all points in a cluster rather than the centroid: [evidence span withheld — anchor status: not_found]
- [results] Cross-sample alignment by agglomerative clustering yielded 11,698 and 14,784 features in positive and negative mode, respectively, appearing across each of 3 analytical replicates.: [evidence span withheld — anchor status: not_found]
- [supplementary] maximum linkage distances of ±20 ppm m/z, ±1.5% drift time, and ±0.3 min retention time: [evidence span withheld — anchor status: not_found]
- [abstract] increase alignment/feature matching confidence among datasets: [evidence span withheld — anchor status: not_found]
