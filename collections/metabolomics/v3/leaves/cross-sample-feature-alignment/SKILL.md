---
name: cross-sample-feature-alignment
description: Use when you have peak/feature tables per sample from multidimensional
  MS acquisitions and need one feature list matched across all study samples, QC pools,
  and blanks using simultaneous m/z, drift time, and retention time tolerances.
when_to_use_negative:
- Input is already a collapsed, cross-sample aligned feature table — re-running alignment
  would merge distinct features and lose per-sample resolution.
- You only have raw spectra (e.g. Bruker .d or Waters .raw acquisitions) and no per-sample
  peak tables yet; feature detection must precede alignment.
- Data are 1D or 2D (e.g. LC-MS only, no drift time); the multidimensional partition/maximum-linkage
  approach is designed for N-dimensional (e.g. LC-IMS-MS/MS) coordinates and gains
  little over m/z versus RT alignment.
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: DEIMoS
  role: Provides build_factors, build_index, partition, multi_sample_partition, collapse,
    and save used to cluster features across samples in N dimensions (m/z, drift time,
    retention time) with maximum linkage
  repo: https://github.com/pnnl/deimos
- name: MS-DIAL
  role: Baseline cross-sample alignment approach used for comparison; its sequential
    m/z versus RT followed by DT alignment gives poorer feature intersection than
    DEIMoS simultaneous N-dimensional clustering
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.1c05017
    title: 'DEIMoS: An Open-Source Tool for Processing High-Dimensional Mass Spectrometry
      Data'
  source_content_hash: sha256:d1e297a1962bc698f8e6fa0cf5f0a9daf211aa7bbd4ebdcd29370bd5fe71c561
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
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# cross-sample-feature-alignment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Aligns per-sample feature tables from high-dimensional mass spectrometry data (e.g. LC-IMS-MS/MS in m/z, drift time, retention time) across study samples by clustering features with maximum linkage, producing a single table of features matched across samples.

## When to use

Use when you have peak/feature tables per sample from multidimensional MS acquisitions and need one feature list matched across all study samples, QC pools, and blanks using simultaneous m/z, drift time, and retention time tolerances.

## When NOT to use

- Input is already a collapsed, cross-sample aligned feature table — re-running alignment would merge distinct features and lose per-sample resolution.
- You only have raw spectra (e.g. Bruker .d or Waters .raw acquisitions) and no per-sample peak tables yet; feature detection must precede alignment.
- Data are 1D or 2D (e.g. LC-MS only, no drift time); the multidimensional partition/maximum-linkage approach is designed for N-dimensional (e.g. LC-IMS-MS/MS) coordinates and gains little over m/z versus RT alignment.

## Inputs

- per-sample peak/feature tables as HDF5 files (deimos.load readable, with m/z, drift time, retention time, intensity columns) from upstream feature detection
- sample metadata distinguishing study specimens, QC pools (e.g. 40 QC pools of NIST SRM 1950 plasma), and blanks

## Outputs

- a single aligned feature table (one row per feature, e.g. aligned_features.h5) with per-sample intensity columns for each study sample, QC pool, and blank
- cluster membership assignments grouping per-sample features into cross-sample features under maximum linkage in all N dimensions

## How to apply

Load each per-sample peak table with deimos.load (HDF5 output from upstream feature detection), then build the N-dimensional partition/lookup structure with deimos.build_factors and deimos.build_index so m/z, drift time, and retention time are handled simultaneously rather than sequentially. Partition the feature space with deimos.partition and cluster features within each partition using deimos.multi_sample_partition under maximum (complete) linkage, enforcing max linkage distances of ±20 ppm m/z, ±1.5% drift time, and ±0.3 min retention time so a cluster only merges when every member lies within tolerance in all dimensions at once. Collapse the clusters with deimos.collapse into one row per aligned feature and write the result with deimos.save; on the NIST SRM 1950 plasma study (152 samples: 40 QC pools plus 112 study specimens), this yielded 11,698 and 14,784 features in positive and negative mode appearing across each of 3 analytical replicates. The simultaneous-dimension approach is what raises alignment/feature matching confidence among datasets compared with sequential m/z versus RT followed by drift time matching, which the paper shows gives only ~13% intersection in both ionization modes.

## Related tools

- **DEIMoS** (Provides build_factors, build_index, partition, multi_sample_partition, collapse, and save used to cluster features across samples in N dimensions (m/z, drift time, retention time) with maximum linkage) — https://github.com/pnnl/deimos
- **MS-DIAL** (Baseline cross-sample alignment approach used for comparison; its sequential m/z versus RT followed by DT alignment gives poorer feature intersection than DEIMoS simultaneous N-dimensional clustering)

## Examples

```
import deimos; ms1 = deimos.load('example_data_peaks.h5', key='ms1_peaks'); factors = deimos.build_factors(ms1); index = deimos.build_index(ms1, factors)
```

## Evaluation signals

- Verify the output file loads with deimos.load and that every aligned feature cluster satisfies the maximum-linkage tolerance constraints (±20 ppm m/z, ±1.5% drift time, ±0.3 min retention time) in all three dimensions simultaneously.
- Check that each aligned feature appears in the expected replicate count (on the SRM 1950 plasma study: features appearing across each of 3 analytical replicates, giving 11,698 positive and 14,784 negative mode features).
- Compare feature-coordinate deviations across samples against the paper's reported averages of 5.8 ppm m/z, 0.2% drift time, and 0.05 min retention time between processing modes as a sanity bound on realignment drift.
- Confirm that no aligned cluster contains two per-sample features whose pairwise linkage distance exceeds any single-dimension tolerance (complete-linkage property).
- Ensure blanks and QC pools are retained in the output so downstream blank subtraction (excluding features present in all blanks) can be applied.

## Limitations

- Maximum-linkage tolerances are dataset- and instrument-specific; the ±20 ppm / ±1.5% / ±0.3 min values reported here are for LC-IMS-MS/MS plasma data and may not transfer to other instruments or matrices.
- Alignment quality depends on the coordinate accuracy of upstream peak detection; the paper notes m/z, drift time, and retention time deviations of 5.8 ppm, 0.2%, and 0.05 min between processing modes, which must remain within the linkage tolerances.
- DEIMoS requires N-dimensional data; it is largely agnostic to acquisition instrumentation but assumes the dimensions (e.g. drift time via TIMS or Agilent 6560 IMS) exist and are consistently measured.
- The tool is BSD-3-Clause licensed but references Battelle/PNNL disclaimers; use in a production context should verify the provided HDF5 schemas match the installed DEIMoS version (the paper used version 1.6.3).

## Evidence

- [results] agglomerative clustering yielded 11,698 and 14,784 features in positive and negative mode, respectively, appearing across each of 3 analytical replicates: [evidence span withheld — anchor status: not_found]
- [abstract] algorithm implementations simultaneously utilize all dimensions to (i) offer greater separation between features, thus improving detection sensitivity, (ii) increase alignment/feature matching confidence among data sets: "algorithm implementations simultaneously utilize all dimensions to (i) offer greater separation between features, thus improving detection sensitivity, (ii) increase alignment/feature matching"
- [results] only ~13% intersection in both positive and negative mode), but intersection increases substantially (to ~90%) when imposing the same tolerances that would be used in cross-sample alignment: "only ∼13% intersection in both positive and negative mode), but intersection increases substantially (to ∼90%) when imposing the same tolerances that would be used in cross-sample alignment"
- [readme] DEIMoS, version 1.6.3 http://github.com/pnnl/deimos (accessed MMM YYYY): "DEIMoS, version 1.6.3 http://github.com/pnnl/deimos (accessed MMM YYYY)"
