---
name: modification-tolerant-spectral-library-search
description: Use when query MS/MS spectra resemble but do not exactly match library
  spectra, and the goal is to detect putative analogs via mass shifts (e.g., the −41
  Da stenothricin analog class consistent with a lysine-to-serine substitution).
when_to_use_negative:
- When the goal is an exact library identification (no expected analog mass shift)
  — use GNPS exact spectral library search instead, which is scored by normalized
  dot product without delta edges
- When query and library spectra should be related de novo without any reference spectra
  — use molecular networking instead
- When a strict FDR-validated identification is required — chemistry-level analog
  assignments from variable dereplication are subject to expert review
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: GNPS
  role: Hosts the variable dereplication / modification-tolerant spectral library
    search, including bipartite peak matching with delta edges and match pre-filters
  repo: CCMS github page
- name: MassIVE
  role: Repository supplying the query MS/MS datasets (e.g., MSV000079204) that variable
    dereplication is run against
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/nbt.3597
    title: Sharing and community curation of mass spectrometry data with Global Natural
      Products Social Molecular Networking
  source_content_hash: sha256:6c8b18354bdb65cb9b857c796dcc674f024599e44a09776134c6fe8b422d8dee
schema_version: 0.2.0
metadata:
  license_tier: restricted
  tools:
  - GNPS
  - MassIVE
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# modification-tolerant-spectral-library-search

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

GNPS variable dereplication extends exact spectral library search by adding bipartite matching edges between library and query peaks that differ by the precursor mass difference delta ± the user-defined fragment mass tolerance, enabling detection of putative analogs of known compounds.

## When to use

Use when query MS/MS spectra resemble but do not exactly match library spectra, and the goal is to detect putative analogs via mass shifts (e.g., the −41 Da stenothricin analog class consistent with a lysine-to-serine substitution).

## When NOT to use

- When the goal is an exact library identification (no expected analog mass shift) — use GNPS exact spectral library search instead, which is scored by normalized dot product without delta edges
- When query and library spectra should be related de novo without any reference spectra — use molecular networking instead
- When a strict FDR-validated identification is required — chemistry-level analog assignments from variable dereplication are subject to expert review

## Inputs

- Reference MS/MS library spectra from GNPS community spectral libraries
- Query MS/MS spectra from LC-MS/MS runs (e.g., MassIVE deposit MSV000079204 of S. roseosporus / Streptomyces sp. DSM5940 extracts)
- User parameters: fragment mass tolerance, precursor m/z Thompson tolerance, minimum matched peaks, minimum spectral match score, precursor charge state

## Outputs

- Analog match table (variable_dereplication_matches.csv) with precursor m/z, library hit, match score, and delta mass shift
- Putative analog assignments flagged by non-zero delta (e.g., −41 Da shift class vs. known stenothricin analogs)

## How to apply

Load reference MS/MS library spectra (GNPS spectral libraries) and query MS/MS spectra (e.g., from the MassIVE deposit MSV000079204 of S. roseosporus and Streptomyces sp. DSM5940 extracts). For each library-query candidate, compute the precursor mass difference delta and enforce pre-filters: same precursor charge state, precursor m/z within the user-defined Thompson tolerance, a minimum number of matched peaks, and a user-defined minimum spectral match score. Build the bipartite peak-matching graph with library and query peaks as nodes; in addition to edges for peaks within the fragment mass tolerance, add modification-tolerant edges for peak pairs differing by delta ± the fragment mass tolerance. Solve the maximum bipartite matching that maximizes the normalized dot-product score, record the highest-scoring library hit and its mass shift, and report non-zero delta matches as putative analogs.

## Related tools

- **GNPS** (Hosts the variable dereplication / modification-tolerant spectral library search, including bipartite peak matching with delta edges and match pre-filters) — CCMS github page
- **MassIVE** (Repository supplying the query MS/MS datasets (e.g., MSV000079204) that variable dereplication is run against)

## Evaluation signals

- Output table contains the expected mass-shift class, e.g. spectra 41 Da smaller than known stenothricin analogs consistent with a lysine-to-serine substitution
- All reported matches satisfy the pre-filters: same precursor charge state, precursor m/z within the user-defined Thompson tolerance, minimum matched peaks, and minimum spectral match score
- Non-zero delta matches are flagged as putative analogs; exact-match candidates (delta = 0) behave identically to the exact library search
- Each query spectrum reports at most the highest-scoring library match together with its mass shift
- Chemistry-level assignments (e.g., amino acid substitution calls) confirmed by expert review against NMR/genomic evidence

## Limitations

- Chemistry-level correctness of analog assignments is subject to expert review; non-zero delta hits are only putative
- Exact spectral search FDRs and search parameters are deferred to Supplementary Note 3 and Supplementary Table 8, not stated in the main text
- Pre-filter thresholds (Thompson tolerance, minimum peaks, minimum score) are user-defined and not specified in the article, affecting reproducibility

## Evidence
