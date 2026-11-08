---
name: nist-srm-1950-plasma-qc-pool-handling
description: Decision support for organizing NIST SRM 1950 plasma QC-pool sample sets
  — 112 specimens, 40 QC pools, 10 blanks — prior to DEIMoS feature detection and
  cross-sample alignment.
when_to_use_negative:
- The input corpus is not the NIST SRM 1950 plasma study (e.g. Bruker or Waters datasets
  with 9 or 6 study samples in triplicate) — the 112/40/10 sample structure and QC-pool
  alignment anchors are specific to this study.
- No analytical replicate structure exists (single-run acquisition), since the QC-pool
  handling relies on 3 replicates per sample class for cross-sample alignment.
- Blank samples were not acquired, since blank exclusion across all blanks is a prerequisite
  filtering step in this workflow.
tools:
- name: DEIMoS
  role: Python API and command-line tool that loads per-sample peak tables, partitions
    N-dimensional feature space, and clusters features across the 40 QC pools and
    112 study samples with maximum linkage for cross-sample alignment.
  repo: github.com/pnnl/deimos
- name: Agilent 6560 Ion Mobility LC/Q-TOF
  role: Instrument platform that produced the NIST SRM 1950 plasma LC-IMS-MS/MS corpus
    of 912 data files across positive and negative modes.
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.1c05017
    title: 'DEIMoS: An Open-Source Tool for Processing High-Dimensional Mass Spectrometry
      Data'
  source_content_hash: sha256:593b5fde7ca5d8ad21a38638c862fe970ab7098620d58e7e9b11c001c38a389c
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/pnnl/deimos
  tools:
  - Agilent 6560 Ion Mobility LC/Q-TOF
  - DEIMoS
  - Python
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# nist-srm-1950-plasma-qc-pool-handling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Skill for managing the NIST SRM 1950 plasma quality-control pool structure in DEIMoS metabolomics processing: 112 study samples, 40 quality control samples, and 10 blanks acquired in positive and negative ionization modes at 3 analytical replicates, spanning 912 data files and 1.1 terabytes.

## When to use

Decision support for organizing NIST SRM 1950 plasma QC-pool sample sets — 112 specimens, 40 QC pools, 10 blanks — prior to DEIMoS feature detection and cross-sample alignment.

## When NOT to use

- The input corpus is not the NIST SRM 1950 plasma study (e.g. Bruker or Waters datasets with 9 or 6 study samples in triplicate) — the 112/40/10 sample structure and QC-pool alignment anchors are specific to this study.
- No analytical replicate structure exists (single-run acquisition), since the QC-pool handling relies on 3 replicates per sample class for cross-sample alignment.
- Blank samples were not acquired, since blank exclusion across all blanks is a prerequisite filtering step in this workflow.

## Inputs

- NIST SRM 1950 plasma LC-IMS-MS/MS data files (912 files: 112 study samples + 40 QC pools + 10 blanks, positive and negative ionization modes, 3 analytical replicates)
- Per-sample peak tables in HDF5 (deimos.load readable)
- Sample metadata mapping file IDs to class (study, QC, blank) and ionization mode

## Outputs

- QC-pool-anchored aligned feature table across 112 study samples (positive mode: 11,698 features; negative mode: 14,784 features across 3 analytical replicates)
- Sample-class-annotated feature table with blanks removed
- HDF5 output (e.g. aligned_features.h5) via deimos.save

## How to apply

Partition the acquired LC-IMS-MS/MS corpus by sample class and ionization mode: the NIST SRM 1950 study comprises 112 study samples, 40 quality control samples, and 10 blanks, each in positive and negative ionization modes and collected at 3 analytical replicates, totaling 912 data files and cumulatively 1.1 terabytes. Use the 40 QC pools as alignment anchors during cross-sample agglomerative clustering, so that features appearing across each of the 3 analytical replicates are matched with maximum (complete) linkage tolerances of ±20 ppm m/z, ±1.5% drift time, and ±0.3 min retention time. Exclude features that appeared in all blank samples before collapsing clusters, since blanks in both ionization modes are part of the same acquisition batch. Write per-sample peak tables to HDF5 and invoke deimos.load on each file to build the sample-indexed feature table for alignment.

## Related tools

- **DEIMoS** (Python API and command-line tool that loads per-sample peak tables, partitions N-dimensional feature space, and clusters features across the 40 QC pools and 112 study samples with maximum linkage for cross-sample alignment.) — github.com/pnnl/deimos
- **Agilent 6560 Ion Mobility LC/Q-TOF** (Instrument platform that produced the NIST SRM 1950 plasma LC-IMS-MS/MS corpus of 912 data files across positive and negative modes.)

## Examples

```
python -c "import deimos; ms1 = deimos.load('SRM1950_qc_pool.h5', key='ms1')"
```

## Evaluation signals

- Feature counts in the aligned table match the expected outputs: 11,698 features in positive mode and 14,784 features in negative mode across 3 analytical replicates.
- All aligned feature clusters satisfy the maximum-linkage tolerance constraints in all three dimensions: ±20 ppm m/z, ±1.5% drift time, and ±0.3 min retention time.
- Every cluster contains features from at least 3 analytical replicates of the QC pool (no single-replicate clusters).
- Features appearing in all blank samples have been excluded before cluster collapse.
- Output file loads with deimos.load and every row carries a sample-class annotation (study / QC / blank).

## Limitations

- Total corpus size is 1.1 terabytes over 912 data files, so memory-mapped loading or chunked processing may be required.
- Cross-sample alignment depends on maximum (complete) linkage — a cluster only merges when every member lies within tolerance in all dimensions simultaneously; stricter tolerances may split true features.
- ±20 ppm m/z tolerance is narrower than the ~5.8 ppm average m/z deviation observed when comparing 3D processing to m/z versus RT followed by DT, so tolerances must be re-tuned if dimensionality changes.
- Only NIST SRM 1950 plasma data was used to calibrate the 40-QC-pool anchoring scheme; other matrices or pool counts would require re-derivation.

## Evidence

- [results] Cross-sample alignment by agglomerative clustering yielded 11,698 and 14,784 features in positive and negative mode, respectively, appearing across each of 3 analytical replicates.: [evidence span withheld — anchor status: not_found]
