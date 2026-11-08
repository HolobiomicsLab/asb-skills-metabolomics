---
name: lc-ims-msms-multidimensional-feature-matching
description: Use when aligning detected peak/feature tables from LC-IMS-MS/MS acquisitions
  (e.
when_to_use_negative:
- Input is already an aligned feature table — no further cross-sample matching is
  needed and running multi_sample_partition would re-cluster already-collapsed features.
- Data lack an ion mobility (drift time) dimension (plain LC-MS/MS); the skill's tolerances
  and rationale assume a 3D m/z/dt/RT feature space, and DEIMoS's advantage over m/z-versus-RT-then-DT
  processing does not apply.
- Only a single sample is available — cross-sample alignment requires at least two
  samples (ideally analytical replicates) to match features against.
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: DEIMoS
  role: Provides build_factors, build_index, partition, multi_sample_partition, and
    collapse functions used to index, cluster, and collapse features across samples
    in N dimensions.
  repo: http://github.com/pnnl/deimos
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.1c05017
    title: 'DEIMoS: An Open-Source Tool for Processing High-Dimensional Mass Spectrometry
      Data'
  source_content_hash: sha256:7be81fd06e87ccb1bc7aa698280b4367bc57eabe914c7abd4d8f451915ef7cf9
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
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# lc-ims-msms-multidimensional-feature-matching

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Cross-sample feature alignment for LC-IMS-MS/MS data using DEIMoS's N-dimensional indexing and maximum (complete) linkage agglomerative clustering, which matches features simultaneously in m/z, drift time, and retention time rather than dimension by dimension. This raises alignment/feature matching confidence by exploiting the full 3D feature space instead of sequential 2D+1D passes.

## When to use

Use when aligning detected peak/feature tables from LC-IMS-MS/MS acquisitions (e.g. Agilent 6560 ion mobility LC/Q-TOF, Bruker timsTOF, Waters) across many samples or analytical replicates, and the goal is a single feature-per-row table whose members agree within tolerance in m/z, drift time, and retention time.

## When NOT to use

- Input is already an aligned feature table — no further cross-sample matching is needed and running multi_sample_partition would re-cluster already-collapsed features.
- Data lack an ion mobility (drift time) dimension (plain LC-MS/MS); the skill's tolerances and rationale assume a 3D m/z/dt/RT feature space, and DEIMoS's advantage over m/z-versus-RT-then-DT processing does not apply.
- Only a single sample is available — cross-sample alignment requires at least two samples (ideally analytical replicates) to match features against.

## Inputs

- Per-sample peak/feature tables in HDF5 files (e.g. ms1_peaks from deimos.load with key='ms1_peaks') produced by DEIMoS feature detection
- Sample metadata indicating analytical replicates, blanks, and ionization mode (positive/negative) for the 152 plasma study samples (40 QC pools + 112 specimens)
- Tolerance specification: maximum linkage distances for m/z (ppm), drift time (fraction), and retention time (min)

## Outputs

- Aligned feature table written to HDF5 (e.g. aligned_features.h5 via deimos.save), one row per feature aligned across samples
- For the NIST SRM 1950 study: 11,698 features in positive mode and 14,784 features in negative mode appearing across each of 3 analytical replicates

## How to apply

Load each sample's peak table with deimos.load from the per-sample HDF5 files produced by upstream feature detection, then build the alignment indexing structure with deimos.build_factors and deimos.build_index so features can be partitioned jointly in m/z, drift time, and retention time. Partition the space with deimos.partition and cluster features within each partition using deimos.multi_sample_partition under maximum (complete) linkage, enforcing tolerances so a cluster merges only when every member lies within tolerance in all dimensions simultaneously; for the NIST SRM 1950 plasma study the tolerances were ±20 ppm m/z, ±1.5% drift time, and ±0.3 min retention time, and clusters persisting in each of 3 analytical replicates were retained. Collapse the resulting clusters with deimos.collapse into one row per aligned feature and write the table out with deimos.save. The rationale is that multidimensional clustering offers greater separation between features than the most comparable approach of m/z versus RT followed by DT, which yields only ~13% intersection versus ~90% when imposing cross-sample-alignment tolerances across all dimensions.

## Related tools

- **DEIMoS** (Provides build_factors, build_index, partition, multi_sample_partition, and collapse functions used to index, cluster, and collapse features across samples in N dimensions.) — http://github.com/pnnl/deimos

## Examples

```
factors, index = deimos.build_factors(peaks, tol=[20e-6, 0.015, 0.3]); parts = deimos.partition(index, factors); aligned = deimos.multi_sample_partition(parts, factors, linkage='max'); deimos.save('aligned_features.h5', deimos.collapse(aligned))
```

## Evaluation signals

- The output aligned_features.h5 loads with deimos.load and contains one row per aligned feature (collapsed table).
- Every aligned feature cluster satisfies the maximum-linkage tolerance constraints in all three dimensions (e.g. ±20 ppm m/z, ±1.5% drift time, ±0.3 min RT).
- Aligned feature counts match replicate-persistence expectations: for the SRM 1950 study, 11,698 positive-mode and 14,784 negative-mode features appearing in each of 3 analytical replicates.
- Compared against a sequential m/z-versus-RT followed by DT alignment on the same peaks, the multidimensional result should show substantially higher feature intersection (~90% vs ~13% when cross-alignment tolerances are imposed).
- Feature coordinates should be stable relative to per-sample detection: average deviation of ~5.8 ppm, 0.2%, and 0.05 min for m/z, drift time, and retention time between 3D processing and the sequential approach.

## Limitations

- Alignment confidence depends on upstream peak-detection quality; DEIMoS's multidimensional algorithms require all dimensions to be present and meaningful (e.g. drift time calibrated in the acquisition).
- DEIMoS operates largely agnostic to instrumentation but is not guaranteed to reproduce vendor-specific algorithms; the most comparable technique implemented by MS-DIAL shows poor agreement (~13% intersection) with the 3D approach.
- Different feature coordinates deviate between 3D and sequential processing (average 5.8 ppm, 0.2%, 0.05 min), so downstream quantitation may differ from other pipelines.

## Evidence
