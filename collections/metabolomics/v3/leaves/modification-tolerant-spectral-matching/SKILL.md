---
name: modification-tolerant-spectral-matching
description: 'Use when annotating natural-products MS/MS spectra against reference
  libraries or comparing spectra pairwise where modifications cause mass shifts: peaks
  differ by a consistent delta equal to the precursor mass difference.'
when_to_use_negative:
- When the query is expected to exactly match a library spectrum with no mass shift
  — the plain exact spectral search with a normalized dot product is sufficient and
  avoids delta-mismatched peak assignments
- When the goal is clustering raw spectra into consensus spectra (use MSCluster first)
  rather than scoring spectrum pairs
- When applying the mutual top-K edge-retention rule to build the final molecular
  network — this skill only produces candidate edges above the minimum match score
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: GNPS
  role: Platform implementing the modification-tolerant spectral library search (variable
    dereplication) and spectrum-to-spectrum matching for molecular networking
- name: MSCluster
  role: Clusters identical MS/MS spectra into consensus spectra that are then compared
    with the modification-tolerant matching algorithm
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/nbt.3597
    title: Sharing and community curation of mass spectrometry data with Global Natural
      Products Social Molecular Networking
  source_content_hash: sha256:74e50995067e444d196c8df3d6d9c7493f4833c3ba0a847bf530141f55edb57b
schema_version: 0.2.0
metadata:
  license_tier: restricted
  tools:
  - GNPS
  - MSCluster
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# modification-tolerant-spectral-matching

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

GNPS-style modification-tolerant matching of query/consensus MS/MS spectra against library spectra or each other, scoring with a normalized dot product computed via maximum bipartite peak matching, with edges added between peaks differing by the precursor mass difference delta plus/minus a user-defined fragment mass tolerance.

## When to use

Use when annotating natural-products MS/MS spectra against reference libraries or comparing spectra pairwise where modifications cause mass shifts: peaks differ by a consistent delta equal to the precursor mass difference.

## When NOT to use

- When the query is expected to exactly match a library spectrum with no mass shift — the plain exact spectral search with a normalized dot product is sufficient and avoids delta-mismatched peak assignments
- When the goal is clustering raw spectra into consensus spectra (use MSCluster first) rather than scoring spectrum pairs
- When applying the mutual top-K edge-retention rule to build the final molecular network — this skill only produces candidate edges above the minimum match score

## Inputs

- Query MS/MS spectra from mzXML/mzML files (or MSCluster consensus spectra)
- Reference/library MS/MS spectra (e.g., GNPS community-wide reference MS libraries)
- User parameters: precursor charge state, precursor m/z tolerance (Thompson), minimum matched peaks, fragment mass tolerance, minimum spectral match score

## Outputs

- Scored spectrum-to-spectrum matches (normalized dot product) exceeding the minimum match score
- Putative library annotations from variable dereplication, with precursor m/z ppm error reported
- Spectrum-to-spectrum match edges feeding downstream molecular network construction

## How to apply

Load query MS/MS spectra (from mzXML/mzML inputs, optionally first merged into consensus spectra via MSCluster) and the reference library spectra. For each pair, first apply the pre-filters: same precursor charge state, precursor m/z within a user-defined Thompson tolerance, and a minimum number of matched peaks. Match peaks between the two spectra by formulating peak correspondence as maximum bipartite matching; for modification tolerance, add edges to the bipartite graph between library and query peaks that differ by delta (determined from the precursor mass difference) plus/minus the user-defined fragment mass tolerance. Score the pair with the normalized dot product over the matched peaks and retain matches exceeding the user-defined minimum spectral match score. These surviving spectrum-to-spectrum matches become library annotations (variable dereplication) or candidate edges for molecular networking (before the mutual top-K edge filter).

## Related tools

- **GNPS** (Platform implementing the modification-tolerant spectral library search (variable dereplication) and spectrum-to-spectrum matching for molecular networking)
- **MSCluster** (Clusters identical MS/MS spectra into consensus spectra that are then compared with the modification-tolerant matching algorithm)

## Evaluation signals

- Every reported match passes the pre-filters: same precursor charge state, precursor m/z within the user-defined Thompson tolerance, at least the minimum number of matched peaks, and score above the user-defined minimum spectral match score
- Paired peaks with non-identical m/z all differ by a single consistent delta equal to the precursor mass difference, within the user-defined fragment mass tolerance
- Scores are reproducible as normalized dot products computed via maximum bipartite matching of the peaks
- Delta-shifted matches discovered in tolerant mode agree with the precursor mass difference between the two spectra (e.g., m/z 1091 vs 1105 differ by 14 Da)
- Running the exact (non-tolerant) search recovers a subset of the tolerant matches — only delta-shifted pairs should be unique to the tolerant mode

## Limitations

- Full search parameters and estimated false discovery rates are deferred to Supplementary Table 8 and Supplementary Note 3, which are not included in the article text
- Requires user-defined fragment mass tolerance, precursor Thompson tolerance, minimum matched peaks, and minimum match score; results are sensitive to these thresholds
- Library coverage limits utility: only 1.01% of all spectra in public GNPS datasets matched to the combined third-party reference libraries, indicating insufficient chemical space coverage

## Evidence
