---
name: closed-subgraph-closeness-verification
description: Use when frequent fragmentation subtrees have been enumerated from fragmentation
  graphs of m/z differences and redundant (non-closed) patterns must be filtered before
  reporting.
when_to_use_negative:
- When the goal is probabilistic topic-style patterns (fragments/losses distributions)
  as produced by MS2LDA rather than exact graph-structured patterns — the closeness
  criterion applies only to exact patterns where all m/z differences are present in
  all containing spectra
- When patterns need not be maximal and redundancy is acceptable, since closeness
  checking discards subpatterns whose extension has identical support and thus reduces
  the reported pattern set
- When spectra contain fewer than the frequency threshold (e.g. 2) shared m/z differences,
  since no frequent pattern exists to verify
tools:
- name: mineMS2
  role: R package whose mineClosedSubgraphs routine performs subtree enumeration,
    frequency testing, full-pattern reconstruction and closeness checking
  repo: https://github.com/odisce/mineMS2
- name: Rcpp
  role: Wraps the C++ mining algorithm (including closeness checking) into the mineMS2
    R package
  repo: https://github.com/odisce/mineMS2
- name: Boost
  role: C++ library used to implement the graph mining algorithm, including the Acyclic
    Flow Graph structures checked for closeness
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s13321-025-01051-y
    title: 'mineMS2: annotation of spectral libraries with exact fragmentation patterns'
  source_content_hash: sha256:c500cf9ffa515dbcecb18b84e0fc4a7335535f721ff655e29248493d6b39b963
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
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# closed-subgraph-closeness-verification

## Summary

Verification step of the mineMS2 frequent-subgraph mining pipeline: for each frequent subtree enumerated from the k-path tree, reconstruct the full Acyclic Flow Graph pattern and confirm it is closed by checking that no extension (root's un-included incoming/outgoing edge labels) has identical support, eliminating redundant patterns.

## When to use

Use when frequent fragmentation subtrees have been enumerated from fragmentation graphs of m/z differences and redundant (non-closed) patterns must be filtered before reporting.

## When NOT to use

- When the goal is probabilistic topic-style patterns (fragments/losses distributions) as produced by MS2LDA rather than exact graph-structured patterns — the closeness criterion applies only to exact patterns where all m/z differences are present in all containing spectra
- When patterns need not be maximal and redundancy is acceptable, since closeness checking discards subpatterns whose extension has identical support and thus reduces the reported pattern set
- When spectra contain fewer than the frequency threshold (e.g. 2) shared m/z differences, since no frequent pattern exists to verify

## Inputs

- ms2Lib object holding fragmentation graphs (Directed Acyclic Graphs of m/z differences) built from MS/MS spectra, e.g. the 51 P. nordicum spectra with pnordicum_ms2_info.tsv metadata
- Candidate frequent subtrees from the k-path tree in k-LMDF canonical form
- Frequency threshold (count of spectra) and minimum pattern size (sizeMin)

## Outputs

- Set of closed (maximal) frequent fragmentation patterns, each reported as a structured table entry with pattern id, edge labels (discretized m/z differences with candidate neutral-loss formulas below 200 Da), and the list of supporting spectra
- Support counts for each closed pattern across the spectral collection

## How to apply

After mineClosedSubgraphs enumerates subtrees from the k-path tree starting from frequent 1-edge paths and filters them to proper spanning trees in k-LMDF canonical form, each frequent subtree is extended to the full Acyclic Flow Graph pattern. Closeness is then verified by intersecting, across all occurrences of the pattern, the root's un-included incoming and outgoing edge labels: if a common extension exists with identical support, the pattern is not closed and is discarded. The support threshold (e.g. count = 2 spectra out of 51 P. nordicum MS/MS spectra) is applied during frequency testing, so any surviving closed pattern is guaranteed to be maximal — no superpattern exists with the same support. Edge labels are discretized m/z differences (15 ppm tolerance) with candidate neutral-loss formulas generated for differences between 14 and 200 Da.

## Related tools

- **mineMS2** (R package whose mineClosedSubgraphs routine performs subtree enumeration, frequency testing, full-pattern reconstruction and closeness checking) — https://github.com/odisce/mineMS2
- **Rcpp** (Wraps the C++ mining algorithm (including closeness checking) into the mineMS2 R package) — https://github.com/odisce/mineMS2
- **Boost** (C++ library used to implement the graph mining algorithm, including the Acyclic Flow Graph structures checked for closeness)

## Examples

```
mineClosedSubgraphs(pnordicum.m2l, sizeMin = 1, count = 2)
```

## Evaluation signals

- Output contains only closed patterns: for every reported pattern, no superpattern exists with identical support (re-check by attempting extensions of the root's un-included incoming/outgoing edge labels)
- Reported supports are consistent with re-counting pattern occurrences independently across the 51 spectra
- All edge labels of a reported pattern are present (as discretized m/z differences within 15 ppm) in every spectrum listed as supporting it
- Labeling error rate in triangles is below 1%, validating the graph-construction Properties 1 and 2 on which the mining relies
- Every edge label below 200 Da has a candidate neutral-loss formula; m/z differences between 14 and 200 Da for which no formula is found were discarded upstream

## Limitations

- Exact pattern mining methods such as mineMS2 extract a high number of patterns, and no scoring system for patterns yet exists
- A subset of spectrum peaks must be selected due to combinatorial complexity (currently based solely on peak intensities, using the 15 most intense fragment ions), which may result in loss of chemically relevant information
- Closeness checking avoids redundant patterns but does not rank or score the surviving patterns for structural informativeness

## Evidence

- [methods] Closeness checking discards non-maximal patterns: "Closeness checking: The pattern is checked for possible extensions with identical support, to avoid the mining of redundant patterns."
- [methods] Enumeration procedure for candidate subtrees: "The enumeration is performed sequentially by considering all the frequent 1-edge paths from the k-path tree"
- [readme] Exact graph structure of mineMS2 patterns enables interpretation: "the **structure of *mineMS2* patterns in the form of exact graphs** (all m/z differences of the pattern are present in all spectra containing this pattern)"
- [methods] Implementation stack of the mining algorithm: "The described algorithm was written in C++ using Boost, and wrapped with Rcpp into the mineMS2 R package"
