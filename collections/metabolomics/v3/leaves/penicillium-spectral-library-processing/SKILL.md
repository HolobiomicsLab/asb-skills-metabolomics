---
name: penicillium-spectral-library-processing
description: Use when a collection of MS/MS spectra (e.g. the 51 P. nordicum .mgf
  spectra with pnordicum_ms2_info.tsv metadata) must be converted into fragmentation
  graphs of m/z differences and mined for closed frequent fragmentation patterns.
when_to_use_negative:
- When pairwise spectral similarity networking is the goal (e.g. GNPS molecular networks)
  — mineMS2 patterns capture structures shared by more than two spectra, complementing
  rather than replacing networking
- When probabilistic motif models are desired — MS2LDA-style distributions of fragments
  and losses are the alternative, and mineMS2's exact graph patterns are deliberately
  different
- When fragmentation differences of interest lie outside the 14–200 Da range, since
  differences in that range for which no neutral formula is generated are discarded
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0092
tools:
- name: mineMS2
  role: Core R package providing ms2Lib to build the fragmentation-graph spectral
    library and mineClosedSubgraphs to extract closed frequent patterns (sizeMin=1,
    count=2)
  repo: https://github.com/odisce/mineMS2
- name: Rcpp
  role: Wraps the C++ mining algorithm into the mineMS2 R package so the mining routines
    are callable from R
- name: Boost
  role: C++ library used to implement the graph-based subgraph mining algorithm
- name: R
  role: Runtime environment in which the mineMS2 package is installed (devtools::install_github)
    and the library/mining workflow is executed
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s13321-025-01051-y
    title: 'mineMS2: annotation of spectral libraries with exact fragmentation patterns'
  source_content_hash: sha256:5a44bcae83789c2617371fa7b4edcfbb9b5c745d10fcc8ad005ced8fdc11cca9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/odisce/mineMS2
  tools:
  - Boost
  - R
  - Rcpp
  - mineMS2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# penicillium-spectral-library-processing

## Summary

Builds a spectral library of fragmentation graphs from the 51 Penicillium nordicum MS/MS spectra and extracts closed (maximal) frequent fragmentation patterns with mineMS2's mineClosedSubgraphs, using sizeMin=1 and a minimum frequency (count) of 2 spectra.

## When to use

Use when a collection of MS/MS spectra (e.g. the 51 P. nordicum .mgf spectra with pnordicum_ms2_info.tsv metadata) must be converted into fragmentation graphs of m/z differences and mined for closed frequent fragmentation patterns.

## When NOT to use

- When pairwise spectral similarity networking is the goal (e.g. GNPS molecular networks) — mineMS2 patterns capture structures shared by more than two spectra, complementing rather than replacing networking
- When probabilistic motif models are desired — MS2LDA-style distributions of fragments and losses are the alternative, and mineMS2's exact graph patterns are deliberately different
- When fragmentation differences of interest lie outside the 14–200 Da range, since differences in that range for which no neutral formula is generated are discarded

## Inputs

- MS/MS spectra in .mgf format (pnordicum.mgf, 51 P. nordicum spectra)
- Spectral metadata table pnordicum_ms2_info.tsv (loaded via system.file from the mineMS2 package)
- Mining parameters: sizeMin = 1 (minimum pattern size), count = 2 (minimum frequency of 2 spectra)

## Outputs

- Structured table of closed frequent fragmentation patterns: pattern id, edge labels (discretized m/z differences with candidate neutral-loss formulas), and support (list of spectra containing each pattern)

## How to apply

Load the .mgf MS/MS spectra into mineMS2 with ms2Lib, passing the pnordicum_ms2_info.tsv metadata via suppInfos. Invoke mineClosedSubgraphs on the resulting ms2Lib object with sizeMin=1 (minimum pattern size) and count=2 (pattern must occur in at least 2 spectra). The algorithm enumerates subtrees sequentially from the k-path tree starting from frequent 1-edge paths, filters subtrees to proper spanning trees in k-LMDF canonical form, and discards subgraphs whose frequency falls below the threshold. For each frequent subtree, the full Acyclic Flow Graph pattern is reconstructed and closeness is checked by intersecting the root's un-included incoming and outgoing edge labels across all occurrences. Export the closed patterns as a table of pattern id, edge labels (discretized m/z differences with candidate neutral-loss formulas for differences between 14 and 200 Da), and the list of containing spectra (support).

## Related tools

- **mineMS2** (Core R package providing ms2Lib to build the fragmentation-graph spectral library and mineClosedSubgraphs to extract closed frequent patterns (sizeMin=1, count=2)) — https://github.com/odisce/mineMS2
- **Rcpp** (Wraps the C++ mining algorithm into the mineMS2 R package so the mining routines are callable from R)
- **Boost** (C++ library used to implement the graph-based subgraph mining algorithm)
- **R** (Runtime environment in which the mineMS2 package is installed (devtools::install_github) and the library/mining workflow is executed)

## Examples

```
pnordicum_m2l <- ms2Lib(pnordicum_mgf.c, suppInfos = pnordicum_metadata.df); mineClosedSubgraphs(pnordicum_m2l, sizeMin = 1, count = 2)
```

## Evaluation signals

- Every reported pattern is closed: no superpattern exists with identical support (verified via the root's un-included incoming/outgoing edge-label intersection)
- Reported supports are consistent with re-counting pattern occurrences across the 51 P. nordicum spectra
- All output patterns have size >= 1 and frequency >= 2 spectra, matching sizeMin=1 and count=2
- Edge labels correspond to the disjoint m/z-difference bins (15 ppm tolerance) and carry candidate formulas only for differences between 14 and 200 Da
- Each pattern is an exact graph (all m/z differences present in all containing spectra) with ion peaks as nodes and m/z differences as edges

## Limitations

- Exact pattern mining methods such as mineMS2 extract a high number of patterns
- A subset of spectrum peaks must be selected due to combinatorial complexity; the current criterion is based solely on peak intensities and may lose chemically relevant information
- A 15 ppm m/z-difference binning tolerance was selected because lower values produced patterns too sensitive to very small m/z variations

## Evidence

- [methods] The enumeration is performed sequentially by considering all the frequent 1-edge paths from the k-path tree: "The enumeration is performed sequentially by considering all the frequent 1-edge paths from the k-path tree"
- [methods] Closeness checking: The pattern is checked for possible extensions with identical support, to avoid the mining of redundant patterns.: "Closeness checking: The pattern is checked for possible extensions with identical support, to avoid the mining of redundant patterns."
- [methods] The described algorithm was written in C++ using Boost, and wrapped with Rcpp into the mineMS2 R package: "The described algorithm was written in C++ using Boost, and wrapped with Rcpp into the mineMS2 R package"
- [supplementary] pnordicum_metadata.c <- system.file("dataset/pnordicum_ms2_info.tsv", package = "mineMS2"): "pnordicum_metadata.c <- system.file("dataset/pnordicum_ms2_info.tsv", package = "mineMS2")"
- [results] We selected a tolerance of 15 ppm because a lower value produced patterns which were too sensitive to very small variations in m/z: "We selected a tolerance of 15 ppm because a lower value produced patterns which were too sensitive to very small variations in m/z"
- [readme] consists of **51 spectra** from the untargeted study of the **secondary metabolism of *Penicillium nordicum***: [evidence span withheld — anchor status: external_doc]
