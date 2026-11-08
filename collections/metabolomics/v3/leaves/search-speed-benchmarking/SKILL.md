---
name: search-speed-benchmarking
description: Use when quantifying the accuracy-vs-runtime trade-off of approximate
  nearest neighbor indexing for spectral library search, e.g. sweeping ANN-SoLo tree
  counts and node inspection budgets before choosing production settings.
when_to_use_negative:
- The goal is an exact (full) spectral library search baseline rather than characterizing
  approximate nearest neighbor indexing behavior
- The library is small enough that exhaustive candidate scoring is already fast, so
  the ANN approximation saves no meaningful runtime
- The search is a sequence database search (e.g. MSFragger-style) rather than open
  spectral library searching
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: ANN-SoLo
  role: Runs the ANN-indexed cascade open searches while trees and nodes-inspected-per-query
    are swept; reports identification counts and search time per setting
  repo: https://github.com/bittremieux/ANN-SoLo
- name: SpectraST
  role: Used to concatenate the NIST human HCD and TripleTOF yeast spectral libraries
    while removing duplicates, producing the benchmark search library
- name: Python
  role: Implementation language of ANN-SoLo; used to orchestrate the parameter sweep
    and collect the runtime/identification measurements
  repo: https://github.com/bittremieux/ANN-SoLo
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.jproteome.8b00359
    title: Fast Open Modification Spectral Library Searching through Approximate Nearest
      Neighbor Indexing
  source_content_hash: sha256:a9fc2d0abf32da7fe3219f0685fe96b43bb711d8565bcee5b9d5d5ef421905ca
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bittremieux/ANN-SoLo
  tools:
  - ANN-SoLo
  - Python
  - SpectraST
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# search-speed-benchmarking

## Summary

Benchmark how approximate nearest neighbor (ANN) index hyperparameters — number of trees in the Annoy ensemble and number of tree nodes inspected per query — trade off identification accuracy against search speed in open spectral library searching with ANN-SoLo on the iPRG2012 data set.

## When to use

Use when quantifying the accuracy-vs-runtime trade-off of approximate nearest neighbor indexing for spectral library search, e.g. sweeping ANN-SoLo tree counts and node inspection budgets before choosing production settings.

## When NOT to use

- The goal is an exact (full) spectral library search baseline rather than characterizing approximate nearest neighbor indexing behavior
- The library is small enough that exhaustive candidate scoring is already fast, so the ANN approximation saves no meaningful runtime
- The search is a sequence database search (e.g. MSFragger-style) rather than open spectral library searching

## Inputs

- iPRG2012 query spectra (MassIVE accession MSV000078492)
- Concatenated NIST human HCD (version 2016/09/12) and TripleTOF yeast (Selevsek et al.) target-decoy spectral library, 1,188,168 spectra (deposited with PXD009861)
- ANN-SoLo configuration: number of trees (100–1000) and nodes inspected per query (20,000–400,000)

## Outputs

- Sweep table (trees, nodes, identifications, search time) written to ann_hyperparameter_sweep.csv
- Accuracy-speed trade-off curves showing identifications approaching the full-search baseline as trees/nodes increase

## How to apply

Build ANN-SoLo indices (ensembles of random projection trees via Annoy) sweeping the number of trees from 100 to 1000. For each index, run the two-level cascade open search on the iPRG2012 query spectra (MassIVE MSV000078492) against the concatenated NIST human HCD / TripleTOF yeast target-decoy library (1,188,168 spectra), varying the number of nodes inspected per query between 20,000 and 400,000. Keep search parameters fixed across the sweep: 20 ppm precursor mass window at the first cascade level, 300 Da at the second (open-search) level, and 0.02 Da fragment mass tolerance. Filter spectrum-spectrum matches with the target-decoy FDR strategy, using subgroup FDR for the open-search level (subgroups defined by charge-normalized precursor mass difference, filtered only for subgroups with at least 20 SSMs). For every (trees, nodes) combination record both the number of confident identifications and the wall-clock search time, and tabulate them so the accuracy-speed trade-off can be read off directly.

## Related tools

- **ANN-SoLo** (Runs the ANN-indexed cascade open searches while trees and nodes-inspected-per-query are swept; reports identification counts and search time per setting) — https://github.com/bittremieux/ANN-SoLo
- **SpectraST** (Used to concatenate the NIST human HCD and TripleTOF yeast spectral libraries while removing duplicates, producing the benchmark search library)
- **Python** (Implementation language of ANN-SoLo; used to orchestrate the parameter sweep and collect the runtime/identification measurements) — https://github.com/bittremieux/ANN-SoLo

## Evaluation signals

- The sweep table contains all required columns (trees, nodes, identifications, search time) for every (trees, nodes) combination tested
- Increasing the number of trees and the number of inspected nodes monotonically raises identification counts toward the full-search baseline
- Wall-clock search time increases as trees and inspected nodes increase, reflecting the accuracy-for-speed trade-off
- FDR is controlled via the two-phase target-decoy strategy with subgroup FDR applied only to subgroups containing at least 20 SSMs
- At the largest (trees, nodes) setting the open search reproduces the reported ~25% identification gain over a standard search on iPRG2012

## Limitations

- Only random projection tree (Annoy) indexing was tested; suitability of alternative ANN techniques (HNSW, LSH, product quantization) for spectral library searching remains untested
- GPU acceleration for candidate selection and SSM evaluation is not implemented in this version
- Actual FDR may be underestimated when too-small subgroups are used, and accurate FDR estimation for open searches is an unresolved research question

## Evidence

- [readme] ANN-SoLo uses approximate nearest neighbor indexing to speed up open modification searching: "ANN-SoLo uses approximate nearest neighbor indexing to speed up open modification searching"
- [methods] False discovery rates are calculated in two phases after both levels of the cascade search using the target–decoy strategy: "False discovery rates are calculated in two phases after both levels of the cascade search using the target–decoy strategy"
- [methods] FDR filtering is only done for subgroups that contain at least 20 SSMs: "FDR filtering is only done for subgroups that contain at least 20 SSMs"
- [results] ANN-SoLo achieves a 25 % increase in identifications when performing an open search compared to a standard search: "ANN-SoLo achieves a 25 % increase in identifications when performing an open search compared to a standard search"
- [discussion] An investigation into whether these techniques are suitable to speed up spectral library searching remains as future work.: "An investigation into whether these techniques are suitable to speed up spectral library searching remains as future work."
