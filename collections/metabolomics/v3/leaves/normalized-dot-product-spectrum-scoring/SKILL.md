---
name: normalized-dot-product-spectrum-scoring
description: Use when query MS/MS spectra must be dereplicated against reference library
  spectra (e.g., GNPS community libraries), or when putative analogs must be found
  via modification-tolerant peak matching around a precursor mass difference delta.
when_to_use_negative:
- Peaks are being related between query spectra (spectrum-to-spectrum molecular networking)
  rather than library-to-query dereplication — use GNPS molecular networking with
  the mutual top-K neighbor edge criterion instead
- An exact library search is sufficient and modification-tolerant analog detection
  is not needed — skip the delta-edge extension and run the exact spectral library
  search only
edam_operation: http://edamontology.org/operation_0324
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: GNPS
  role: Web platform whose spectral library search scores exact matches with a normalized
    dot product via maximum bipartite matching and adds delta edges for modification-tolerant
    (variable) dereplication
- name: MassIVE
  role: Data repository supplying the query MS/MS datasets (e.g., MSV000079204) fed
    into the library search
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/nbt.3597
    title: Sharing and community curation of mass spectrometry data with Global Natural
      Products Social Molecular Networking
  source_content_hash: sha256:da9b0b92719dd8e1d30addfa612ffc50ef45ffc72cd95169e7e1cdeff15d46ba
schema_version: 0.2.0
metadata:
  license_tier: restricted
  tools:
  - GNPS
  - MassIVE
  edam_operation: http://edamontology.org/operation_0324
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# normalized-dot-product-spectrum-scoring

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

GNPS library-search scoring of MS/MS spectra: a normalized dot product between library and query spectra computed as a maximum bipartite matching of peaks, extendable with modification-tolerant delta edges for analog detection.

## When to use

Use when query MS/MS spectra must be dereplicated against reference library spectra (e.g., GNPS community libraries), or when putative analogs must be found via modification-tolerant peak matching around a precursor mass difference delta.

## When NOT to use

- Peaks are being related between query spectra (spectrum-to-spectrum molecular networking) rather than library-to-query dereplication — use GNPS molecular networking with the mutual top-K neighbor edge criterion instead
- An exact library search is sufficient and modification-tolerant analog detection is not needed — skip the delta-edge extension and run the exact spectral library search only

## Inputs

- Query MS/MS spectra (e.g., from MassIVE deposit MSV000079204 of S. roseosporus and Streptomyces sp. DSM5940 extracts)
- Reference MS/MS library spectra (GNPS community-wide spectral libraries)
- User-defined search parameters: precursor charge state, precursor m/z Thompson tolerance, fragment mass tolerance, minimum matched peaks, minimum spectral match score

## Outputs

- Spectral match table with query precursor m/z, library hit, normalized dot-product score, and mass shift (e.g., variable_dereplication_matches.csv)
- Putative analog assignments flagged by non-zero delta mass shifts (e.g., the −41 Da stenothricin analog class)

## How to apply

Load reference library MS/MS spectra and query MS/MS spectra (e.g., from the MassIVE deposit MSV000079204 of S. roseosporus and Streptomyces sp. DSM5940 extracts). Apply the pre-filters: identical precursor charge state, precursor m/z within a user-defined Thompson tolerance, a minimum number of matched peaks, and a minimum spectral match score. For each candidate pair, build a bipartite graph with library and query peaks as nodes, adding edges for peaks within the user-defined fragment mass tolerance; solve the maximum bipartite matching that maximizes the normalized dot-product score. For variable (modification-tolerant) dereplication, additionally add edges between peak pairs that differ by the precursor mass difference delta plus/minus the fragment tolerance, and report matches with non-zero delta as putative analogs — e.g., spectra 41 Da smaller than known stenothricin analogs, consistent with a lysine-to-serine substitution. Write the match table (precursor m/z, library hit, score, delta mass shift) to variable_dereplication_matches.csv and defer chemistry-level assignments to expert review.

## Related tools

- **GNPS** (Web platform whose spectral library search scores exact matches with a normalized dot product via maximum bipartite matching and adds delta edges for modification-tolerant (variable) dereplication)
- **MassIVE** (Data repository supplying the query MS/MS datasets (e.g., MSV000079204) fed into the library search)

## Evaluation signals

- Each query spectrum receives exactly one highest-scoring library match together with its mass shift; zero-delta matches correspond to exact library hits
- Pre-filter criteria are demonstrably enforced: same precursor charge state, precursor m/z within the user-defined Thompson tolerance, minimum number of matched peaks, and minimum spectral match score
- Validation case: the output recovers the −41 Da stenothricin analog shift class in the 14-node stenothricin-GNPS sub-network via the delta-edge matching
- Peak matching is solved as a maximum bipartite matching maximizing the normalized dot-product score — added delta edges should only increase (or leave unchanged) the best score
- False discovery rate of the exact spectral library search can be benchmarked as in Supplementary Note 3 with the parameters of Supplementary Table 8

## Limitations

- Search parameters and estimated false discovery rates of the exact spectral library search are only documented in Supplementary Table 8 and Supplementary Note 3, not in the main text
- Only 1.01% of all spectra in public GNPS datasets matched to the reference library collection, indicating insufficient chemical space coverage limits exact-match dereplication
- Chemistry-level correctness of modification-tolerant analog assignments is subject to expert review; the algorithm only reports a putative delta mass shift

## Evidence
