---
name: dot-product-spectral-scoring
description: Use when dereplicating query MS/MS spectra (with precursor charge, precursor
  m/z, and peak lists) against accumulated reference library spectra and a peak-level
  similarity score such as a normalized dot product is needed for identification.
when_to_use_negative:
- The goal is spectral-spectrum similarity networking between query spectra (molecular
  networking), not library identification — use the MSCluster clustering plus top-K
  mutual-neighbor edge construction workflow instead
- The variant search needs to tolerate systematic fragment shifts from precursor mass
  differences — that requires the modification-tolerant search with additional delta
  +/- fragment tolerance edges, not the exact match
- Input MS/MS spectra have MS1 precursor intensity <1E4 or precursor m/z ppm error
  >=50 versus theoretical mass, which are excluded in the reference workflow's putative
  identification filter
edam_operation: http://edamontology.org/operation_3631
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: GNPS
  role: Hosts the exact spectral library search that implements normalized dot-product
    scoring via maximum bipartite matching of library and query peaks
  repo: http://gnps.ucsd.edu
- name: scipy.optimize.linear_sum_assignment
  role: Hungarian-algorithm solver used to compute the maximum bipartite peak matching
    underlying the dot-product score
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/nbt.3597
    title: Sharing and community curation of mass spectrometry data with Global Natural
      Products Social Molecular Networking
  source_content_hash: sha256:a912ca3c25ae0e2a64331cac0cdbe7a6510adcbda6f4d690538f9634c7ea38b6
schema_version: 0.2.0
metadata:
  license_tier: restricted
  tools:
  - GNPS
  - scipy.optimize.linear_sum_assignment
  edam_operation: http://edamontology.org/operation_3631
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# dot-product-spectral-scoring

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

GNPS-style exact spectral library matching in which library-to-query peak correspondences are found by maximum bipartite matching under a fragment mass tolerance and each match is scored with a normalized dot product to derePLICATE natural products from MS/MS spectra.

## When to use

Use when dereplicating query MS/MS spectra (with precursor charge, precursor m/z, and peak lists) against accumulated reference library spectra and a peak-level similarity score such as a normalized dot product is needed for identification.

## When NOT to use

- The goal is spectral-spectrum similarity networking between query spectra (molecular networking), not library identification — use the MSCluster clustering plus top-K mutual-neighbor edge construction workflow instead
- The variant search needs to tolerate systematic fragment shifts from precursor mass differences — that requires the modification-tolerant search with additional delta +/- fragment tolerance edges, not the exact match
- Input MS/MS spectra have MS1 precursor intensity <1E4 or precursor m/z ppm error >=50 versus theoretical mass, which are excluded in the reference workflow's putative identification filter

## Inputs

- Query MS/MS spectra with precursor charge state, precursor m/z, and peak lists
- Reference spectral library spectra (e.g., GNPS community reference MS libraries) with precursor metadata and peak lists
- User-defined parameters: precursor m/z Thompson tolerance, fragment mass tolerance, minimum matched peaks, minimum spectral match score

## Outputs

- spectral_search_matches.csv with port_id, precursor m/z, matched library compound, dot-product score, and number of matched peaks
- Estimated false discovery rates of the exact spectral library search (per Supplementary Note 3 in the reference workflow)

## How to apply

Parse query and library MS/MS spectra retaining precursor charge state, precursor m/z, and peak lists, then pre-filter candidate library spectra per query using GNPS criteria: identical precursor charge state, precursor m/z within a user-defined Thompson tolerance, a minimum number of shared matched peaks, and exceeding a user-defined minimum spectral match score. For each candidate pair, build a bipartite graph with edges connecting library and query peaks within the user-defined fragment mass tolerance, and solve the maximum bipartite matching problem (e.g., Hungarian algorithm via scipy.optimize.linear_sum_assignment) to pick the peak assignment maximizing the normalized dot product. A variant (modification-tolerant search) adds edges between peaks differing by the precursor mass delta +/- the fragment mass tolerance. Report the best-scoring library match per query with matched compound, dot-product score, and matched peak count; the score is bounded in [0, 1] because it is normalized.

## Related tools

- **GNPS** (Hosts the exact spectral library search that implements normalized dot-product scoring via maximum bipartite matching of library and query peaks) — http://gnps.ucsd.edu
- **scipy.optimize.linear_sum_assignment** (Hungarian-algorithm solver used to compute the maximum bipartite peak matching underlying the dot-product score)

## Examples

```
from scipy.optimize import linear_sum_assignment; rows, cols = linear_sum_assignment(-score_matrix); dot_product = score_matrix[rows, cols].sum()
```

## Evaluation signals

- Only query spectra whose best library match exceeds the user-defined minimum spectral match score appear in spectral_search_matches.csv
- Every reported match satisfies all pre-filter criteria: same precursor charge state, precursor m/z within the Thompson tolerance, and at least the minimum number of matched peaks
- Reported dot-product scores fall within [0, 1] (normalized) and correlate with the reported matched-peak counts
- The peak assignment maximizes the normalized dot product: swapping to any non-optimal bipartite matching yields an equal or lower score
- False discovery rate of the search is estimated and reported (per Supplementary Note 3 of the reference workflow)

## Limitations

- Exact-match scoring cannot identify analogs with systematic fragment mass shifts; the modification-tolerant variant (delta +/- fragment tolerance edges) is a separate workflow
- Search parameters and estimated false discovery rates are only documented in Supplementary Table 8 and Supplementary Note 3, not in the main text
- Coverage of reference libraries limits identification: only 1.01% of all spectra in public GNPS datasets matched to the combined third-party reference libraries, indicating insufficient chemical space coverage

## Evidence

- [discussion] Exact spectral matches between library and query spectra are scored with a normalized dot product: "Exact spectral matches between library and query spectra are scored with a normalized dot product57-59. The matching of peaks between two spectra is formulated as a maximum bipartite matching"
- [discussion] Library match pre-filter criteria: "same precursor charge state, precursor m/z is within a user defined Thompson tolerance, share a minimum number of matched peaks, and exceed a user-defined minimum spectral match score."
- [intro] Insufficient library chemical-space coverage: "only 1.01% of all spectra public GNPS datasets has been matched to this collection, indicating insufficient chemical space coverage"
- [discussion] Search parameters and FDRs deferred to supplementary materials: "Estimated false discovery rates of the exact spectral library search are shown in Supplementary Note 3. Parameters of the search can be found in Supplementary Table 8."
