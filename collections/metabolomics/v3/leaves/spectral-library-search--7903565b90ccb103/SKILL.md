---
name: spectral-library-search
description: Use when query spectra (e.g., MGF files from HEK293 or iPRG2012) must
  be searched against a large peptide spectral library while allowing arbitrary modifications
  within a 500 Da precursor mass window.
when_to_use_negative:
- When the goal is a standard (closed) search with only unmodified peptides, since
  the plain dot-product search with a narrow precursor window suffices and open-search
  subgroup FDR is unnecessary
- When the intended analysis is sequence database searching rather than spectral library
  searching (e.g., MSFragger is used here only as a benchmark comparator)
- When exact (full) nearest neighbor search is required rather than approximate indexing,
  since ANN results only approach the full-search baseline as trees and inspected
  nodes increase
edam_operation: http://edamontology.org/operation_3648
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3520
tools:
- name: ANN-SoLo
  role: 'Primary search engine: builds the approximate nearest neighbor index, runs
    the two-level cascade open search with the shifted dot product, and performs target–decoy
    subgroup FDR estimation'
  repo: https://github.com/bittremieux/ANN-SoLo
- name: SpectraST
  role: Used to deduplicate the spectral library while removing duplicates and as
    the benchmark engine for runtime and identification-count comparisons
- name: msconvert
  role: Converts the HEK293 raw files downloaded from PRIDE PXD001468 into MGF format
    before searching
provenance:
  source_task_ids:
  - task_001
  - task_004
  - task_003
  source_papers:
  - doi: 10.1021/acs.jproteome.8b00359
    title: Fast Open Modification Spectral Library Searching through Approximate Nearest
      Neighbor Indexing
  source_content_hash: sha256:23dbd092aeafc096360b94fafbae0f2bd40091318a7b9d473499a3bf8490dcb7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bittremieux/ANN-SoLo
  tools:
  - ANN-SoLo
  - Python
  - SpectraST
  - msconvert
  edam_operation: http://edamontology.org/operation_3648
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 18
---

# spectral-library-search

## Summary

Perform open modification spectral library searching with ANN-SoLo, which uses approximate nearest neighbor indexing, a two-level cascade search, and the shifted dot product to sensitively identify unmodified and modified peptide spectra while strictly controlling FDR.

## When to use

Use when query spectra (e.g., MGF files from HEK293 or iPRG2012) must be searched against a large peptide spectral library while allowing arbitrary modifications within a 500 Da precursor mass window.

## When NOT to use

- When the goal is a standard (closed) search with only unmodified peptides, since the plain dot-product search with a narrow precursor window suffices and open-search subgroup FDR is unnecessary
- When the intended analysis is sequence database searching rather than spectral library searching (e.g., MSFragger is used here only as a benchmark comparator)
- When exact (full) nearest neighbor search is required rather than approximate indexing, since ANN results only approach the full-search baseline as trees and inspected nodes increase

## Inputs

- Query spectra in MGF format (e.g., HEK293 raw files from PRIDE PXD001468 converted with msconvert, or iPRG2012 spectra from MassIVE MSV000078492)
- Peptide spectral library, e.g., MassIVE-KB human HCD library (version 2017/11/27) with duplicates removed and 1:1 shuffled decoys, or the concatenated NIST human HCD / TripleTOF yeast target-decoy library (1,188,168 spectra)

## Outputs

- Spectrum–spectrum matches (SSMs) filtered at 1% FDR with peptide identifications and modification assignments
- Identification counts and per-engine runtime comparison tables (e.g., ann_solo_hek293_idents.csv, ann_solo_spectrast_runtime_comparison.csv, ann_hyperparameter_sweep.csv)

## How to apply

Preprocess query and library spectra identically: remove precursor peaks and noise peaks below 1% of the most intense peak, restrict to the 50 most intense peaks, discard spectra with fewer than 10 peaks or a mass range under 250 Da, and rank-transform intensities. Prepare the spectral library (e.g., MassIVE-KB version 2017/11/27 for HEK293) by removing duplicate peptide ions with SpectraST and appending decoy spectra in a 1:1 ratio using the shuffle-and-reposition method. Run ANN-SoLo's two-level cascade: level 1 with a 5 ppm precursor mass window and the dot product for unmodified peptides, then level 2 with a 500 Da precursor mass window using the shifted dot product, backed by an ANN index of 1000 random-projection trees with 200 000 nodes inspected per query and a 0.02 Da fragment mass tolerance. Estimate FDR with the target–decoy strategy, applying subgroup FDR for the open-search level by grouping SSMs by their charge-normalized precursor mass difference (0.1 Da subgroups, minimum 20 SSMs per subgroup), and filter spectrum–spectrum matches at 1% FDR. The rationale is that ANN candidate selection avoids exhaustive comparison against all library spectra, keeping open-search runtime near two hours on HEK293 instead of almost a day for SpectraST while retaining identification sensitivity.

## Related tools

- **ANN-SoLo** (Primary search engine: builds the approximate nearest neighbor index, runs the two-level cascade open search with the shifted dot product, and performs target–decoy subgroup FDR estimation) — https://github.com/bittremieux/ANN-SoLo
- **SpectraST** (Used to deduplicate the spectral library while removing duplicates and as the benchmark engine for runtime and identification-count comparisons)
- **msconvert** (Converts the HEK293 raw files downloaded from PRIDE PXD001468 into MGF format before searching)

## Evaluation signals

- On the HEK293 data set, the number of confident identifications at 1% FDR should exceed those of SpectraST and MSFragger under an open search
- The ANN-SoLo vs SpectraST wall-clock runtime ratio for the HEK293 open search should reach roughly an order of magnitude (~10x), with ANN-SoLo under two hours vs almost a day
- On iPRG2012, the open search should yield about a 25% increase in identifications versus a standard search, with the additional identifications corresponding to modified peptides
- Increasing ANN index trees (100–1000) and nodes inspected per query (20,000–400,000) should monotonically raise identification counts toward the full-search baseline at the cost of increased search time
- Final identification counts should be consistent with the results deposited at PXD009861

## Limitations

- The actual FDR might be underestimated when too-small subgroups are used, so subgroup FDR filtering is only applied to subgroups with at least 20 SSMs
- Accurate FDR estimation for open searches remains an unresolved research question
- The suitability of alternative ANN indexing techniques (e.g., HNSW, LSH, product quantization) and non-metric space indexing for direct shifted dot product retrieval was not investigated
- GPU acceleration for candidate selection and SSM evaluation was not yet implemented in this version
- The ANN-SoLo README states the software requires Python 3.6 to 3.9 (Python 3.10 and newer unsupported)

## Evidence

- [abstract] ANN-SoLo identifies more spectra with faster search: "We present the ANN-SoLo tool for fast and accurate open spectral library searching."
- [intro] Cascade and shifted dot product rationale: "the shifted dot product is used to accurately match modified query spectra to their unmodified library counterpart by taking into account peak shifts caused by a modification"
- [methods] Library decoy construction: "decoy spectra were added in a 1:1 ratio using the shuffle-and-reposition method"
- [methods] Subgroup FDR grouping: "they are grouped based on their charge-normalized precursor mass difference between the query spectra and library spectra"
- [results] Open search sensitivity gain: "Based on the iPRG2012 data set ANN-SoLo achieves a 25 % increase in identifications when performing an open search compared to a standard search"
- [readme] README description of ANN indexing approach: "ANN-SoLo uses approximate nearest neighbor indexing to speed up open modification searching by selecting only a limited number of the most relevant library spectra"
