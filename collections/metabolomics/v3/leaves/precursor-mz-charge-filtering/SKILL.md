---
name: precursor-mz-charge-filtering
description: Use when dereplicating natural products by matching query MS/MS spectra
  against a reference spectral library and candidate library entries must be restricted
  by precursor charge state, precursor m/z tolerance, matched-peak count, and match
  score.
when_to_use_negative:
- When seeking analogs or modified metabolites that differ from the library entry
  by a precursor mass delta — use GNPS variable dereplication (modification-tolerant
  search with edges differing by precursor mass delta +/- fragment mass tolerance)
  instead of exact matching
- When building molecular networks between query spectra (spectrum-to-spectrum edges
  with mutual top-K neighbor and minimum match score conditions) rather than spectrum-to-library
  dereplication
- When the query spectra lack reliable precursor charge state or precursor m/z annotations,
  since the pre-filter criteria depend directly on both
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: GNPS
  role: Runs the exact spectral library search whose candidate pairs are pre-filtered
    by precursor charge state, precursor m/z Thompson tolerance, minimum matched peaks,
    and minimum match score
- name: scipy.optimize.linear_sum_assignment
  role: Solves the maximum bipartite matching of library-to-query peaks within the
    fragment mass tolerance to maximize the normalized dot product
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/nbt.3597
    title: Sharing and community curation of mass spectrometry data with Global Natural
      Products Social Molecular Networking
  source_content_hash: sha256:5fee3465109795179991a83749a38f0493636dd5c1454d0ba1941504d42b83df
schema_version: 0.2.0
metadata:
  license_tier: restricted
  tools:
  - GNPS
  - scipy.optimize.linear_sum_assignment
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# precursor-mz-charge-filtering

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

GNPS-style candidate pre-filtering for exact spectral library search: a query MS/MS spectrum is only compared against library entries sharing its precursor charge state and lying within a user-defined precursor m/z (Thompson) tolerance, and a hit is reported only if matched-peak count and normalized dot-product score thresholds are met.

## When to use

Use when dereplicating natural products by matching query MS/MS spectra against a reference spectral library and candidate library entries must be restricted by precursor charge state, precursor m/z tolerance, matched-peak count, and match score.

## When NOT to use

- When seeking analogs or modified metabolites that differ from the library entry by a precursor mass delta — use GNPS variable dereplication (modification-tolerant search with edges differing by precursor mass delta +/- fragment mass tolerance) instead of exact matching
- When building molecular networks between query spectra (spectrum-to-spectrum edges with mutual top-K neighbor and minimum match score conditions) rather than spectrum-to-library dereplication
- When the query spectra lack reliable precursor charge state or precursor m/z annotations, since the pre-filter criteria depend directly on both

## Inputs

- Query MS/MS spectra with precursor charge state, precursor m/z, and peak lists
- Reference library spectra (GNPS community-wide reference MS libraries) with precursor charge state and precursor m/z
- User parameters: precursor m/z Thompson tolerance, fragment mass tolerance, minimum matched peaks, minimum spectral match score

## Outputs

- spectral_search_matches.csv with, per query spectrum, port_id, precursor m/z, matched library compound, dot-product score, and number of matched peaks
- Per-query best library match, reported only when the score exceeds the user-defined minimum spectral match score

## How to apply

Parse query and library spectra retaining precursor charge state, precursor m/z, and peak lists. Pre-filter candidates per query using the GNPS criteria: the library spectrum must have the same precursor charge state, its precursor m/z must be within a user-defined Thompson tolerance of the query's, and the eventual match must share a minimum number of peaks and exceed a user-defined minimum spectral match score. For each surviving candidate pair, build a bipartite graph of library and query peaks with edges connecting peaks within the user-defined fragment mass tolerance, then solve maximum bipartite matching (e.g., Hungarian algorithm via scipy.optimize.linear_sum_assignment) so the peak assignment maximizes the normalized dot product. Report the highest-scoring library match per query only if all pre-filter criteria hold; the exact-search false discovery rate and search parameters are documented in Supplementary Note 3 and Supplementary Table 8 rather than the main text.

## Related tools

- **GNPS** (Runs the exact spectral library search whose candidate pairs are pre-filtered by precursor charge state, precursor m/z Thompson tolerance, minimum matched peaks, and minimum match score)
- **scipy.optimize.linear_sum_assignment** (Solves the maximum bipartite matching of library-to-query peaks within the fragment mass tolerance to maximize the normalized dot product)

## Examples

```
from scipy.optimize import linear_sum_assignment; lib_idx, qry_idx = linear_sum_assignment(-score_matrix)  # bipartite peak matching within fragment mass tolerance
```

## Evaluation signals

- Every reported match pairs a query and library spectrum with the same precursor charge state
- Every reported match has precursor m/z within the user-defined Thompson tolerance
- Every reported match has a matched-peak count at or above the user-defined minimum
- Only query spectra whose best match exceeds the user-defined minimum spectral match score appear in spectral_search_matches.csv
- Each query yields at most one (highest-scoring) library match, with dot-product score and matched-peak count present in the output

## Limitations

- Only 1.01% of all spectra in public GNPS datasets matched to combined third-party reference libraries, so insufficient chemical space coverage limits how often a passing match can be found
- Exact spectral matching cannot report modified/analog hits differing by a mass delta; that requires the modification-tolerant (variable dereplication) search
- Exact-search parameters and estimated false discovery rates are only in Supplementary Table 8 and Supplementary Note 3, not the main text

## Evidence
