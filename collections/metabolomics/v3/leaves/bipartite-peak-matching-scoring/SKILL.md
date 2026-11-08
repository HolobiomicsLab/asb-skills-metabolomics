---
name: bipartite-peak-matching-scoring
description: Use when dereplicating query MS/MS spectra against a reference library
  (e.g., GNPS community libraries) and you must compute peak-level match scores, or
  detect putative analogs via precursor mass-difference delta shifts.
when_to_use_negative:
- Constructing molecular networks between consensus spectra from the same dataset
  — that requires the mutual top-K neighbor edge-retention rule on MSCluster consensus
  spectra, not library matching
- Query spectra already have a confident library annotation and no analog detection
  is intended
- Identifying novel compounds with no close reference spectrum — GNPS reported only
  1.01% of public spectra matched, indicating insufficient chemical space coverage
  for unmatched chemistry
edam_operation: http://edamontology.org/operation_3218
tools:
- name: GNPS
  role: Platform whose exact spectral library search and variable dereplication implement
    the bipartite peak-matching and normalized dot-product scoring; source code on
    the CCMS github page
- name: MassIVE
  role: Repository supplying the deposited query MS/MS datasets used as search input
provenance:
  source_task_ids:
  - task_006
  - task_005
  - task_004
  source_papers:
  - doi: 10.1038/nbt.3597
    title: Sharing and community curation of mass spectrometry data with Global Natural
      Products Social Molecular Networking
  source_content_hash: sha256:548cf11a5c95991d6733ceb7e5326883fe1f2948ebef526ed4dde8eeb8aabbad
schema_version: 0.2.0
metadata:
  license_tier: restricted
  tools:
  - GNPS
  - MSCluster
  - MassIVE
  edam_operation: http://edamontology.org/operation_3218
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# bipartite-peak-matching-scoring

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Scores MS/MS library-vs-query spectrum pairs by formulating peak correspondences as a maximum bipartite matching under a fragment mass tolerance and scoring the assignment with a normalized dot product; underpins GNPS exact spectral search and, with delta edges, modification-tolerant variable dereplication.

## When to use

Use when dereplicating query MS/MS spectra against a reference library (e.g., GNPS community libraries) and you must compute peak-level match scores, or detect putative analogs via precursor mass-difference delta shifts.

## When NOT to use

- Constructing molecular networks between consensus spectra from the same dataset — that requires the mutual top-K neighbor edge-retention rule on MSCluster consensus spectra, not library matching
- Query spectra already have a confident library annotation and no analog detection is intended
- Identifying novel compounds with no close reference spectrum — GNPS reported only 1.01% of public spectra matched, indicating insufficient chemical space coverage for unmatched chemistry

## Inputs

- Query MS/MS spectra with precursor charge state, precursor m/z, and peak lists (mzXML/mzML, e.g., MassIVE deposit MSV000079204 of S. roseosporus and Streptomyces sp. DSM5940 extracts)
- Reference MS/MS library spectra (GNPS community-wide spectral libraries)
- User-defined parameters: fragment mass tolerance, precursor Thompson tolerance, minimum number of matched peaks, minimum spectral match score, and (variable mode) precursor mass difference delta

## Outputs

- spectral_search_matches.csv with port_id, precursor m/z, matched library compound, dot-product score, and number of matched peaks
- variable_dereplication_matches.csv with precursor m/z, library hit, score, and delta mass shift (e.g., -41 Da shift class for stenothricin analogs)

## How to apply

Parse query and library MS/MS spectra retaining precursor charge state, precursor m/z, and peak lists. Pre-filter candidate pairs on GNPS criteria: same precursor charge state, precursor m/z within a user-defined Thompson tolerance, a minimum number of matched peaks, and exceeding a user-defined minimum spectral match score. For each surviving pair, build a bipartite graph with library and query peaks as nodes, connecting peaks within the user-defined fragment mass tolerance; for variable dereplication, also add edges between peaks differing by the precursor mass difference delta +/- the fragment tolerance. Solve the maximum bipartite matching (e.g., Hungarian algorithm via scipy.optimize.linear_sum_assignment) to maximize the normalized dot-product score, and report the best library hit per query with its score, matched peak count, and (for variable search) delta mass shift. Rationale: this peak-assignment formulation is how GNPS defines 'exact spectral matching' and extends it to modification-tolerant analog detection.

## Related tools

- **GNPS** (Platform whose exact spectral library search and variable dereplication implement the bipartite peak-matching and normalized dot-product scoring; source code on the CCMS github page)
- **MassIVE** (Repository supplying the deposited query MS/MS datasets used as search input)

## Examples

```
from scipy.optimize import linear_sum_assignment; rows, cols = linear_sum_assignment(-dotproduct_score_matrix); score = dotproduct_score_matrix[rows, cols].sum() / norm
```

## Evaluation signals

- Only query spectra whose best library match exceeds the user-defined minimum spectral match score are reported as matched
- Every reported match satisfies the pre-filters: same precursor charge state, precursor m/z within the Thompson tolerance, and at least the minimum number of matched peaks
- Peak assignments are a valid maximum bipartite matching (each library and query peak used at most once) maximizing the normalized dot product
- Variable dereplication reproduces the delta-driven shift classes, e.g., the -41 Da shift consistent with a lysine-to-serine substitution in the stenothricin-GNPS sub-network
- Estimated false discovery rates of the exact search can be compared against Supplementary Note 3 of the paper

## Limitations

- Chemistry-level correctness of variable dereplication assignments is subject to expert review; delta shifts are putative, not definitive annotations
- Search parameters and false discovery rates are only in Supplementary Table 8 and Supplementary Note 3, which are not included in the main text
- Reference library coverage limits sensitivity — only 1.01% of all spectra in public GNPS datasets matched combined third-party reference libraries

## Evidence

- [discussion] Exact spectral matches between library and query spectra are scored with a normalized dot product. The matching of peaks between two spectra is formulated as a maximum bipartite matching: "Exact spectral matches between library and query spectra are scored with a normalized dot product57-59. The matching of peaks between two spectra is formulated as a maximum bipartite matching"
- [discussion] Variable dereplication adds delta-shifted edges between library and query peaks for modification-tolerant matching.: "additional edges are added to the bipartite matching between library and query peaks which differ by a δ (as determined by their precursor mass difference δ) +/− the user defined fragment mass"
- [discussion] Library match pre-filters: charge state, precursor m/z Thompson tolerance, minimum matched peaks, minimum match score.: "same precursor charge state, precursor m/z is within a user defined Thompson tolerance, share a minimum number of matched peaks, and exceed a user-defined minimum spectral match score."
