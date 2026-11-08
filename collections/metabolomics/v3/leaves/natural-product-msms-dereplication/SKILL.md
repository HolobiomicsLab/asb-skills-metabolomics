---
name: natural-product-msms-dereplication
description: Use when query tandem MS/MS spectra from natural-product extracts (e.g.,
  MassIVE deposits of S. roseosporus or Streptomyces sp. DSM5940) must be matched
  against reference MS/MS library spectra, especially to find analogs whose peaks
  are shifted by a constant mass delta.
when_to_use_negative:
- The goal is clustering and relating unknown spectra with no library analog — molecular
  networking (MSCluster consensus spectra with mutual top-K neighbor edges) is the
  appropriate GNPS workflow, not library search
- Data are raw non-MS/MS or MS1-only, or the library spectra needed are not in any
  community library (only ~1.01% of spectra in public GNPS datasets matched to third-party
  reference libraries)
- An exact, modification-free spectral library search is sufficient and the extra
  delta edges would only add false analog matches
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: GNPS
  role: Hosts the community reference MS/MS libraries and implements the exact and
    variable (modification-tolerant) spectral library search via maximum bipartite
    matching with delta edges
- name: MassIVE
  role: Repository providing public deposition/retrieval of the raw MS/MS datasets
    used as query input for the library search
- name: MSCluster
  role: Pre-steps duplicate query MS/MS spectra into consensus spectra so that identical
    spectra are matched once against the library
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/nbt.3597
    title: Sharing and community curation of mass spectrometry data with Global Natural
      Products Social Molecular Networking
  source_content_hash: sha256:b01afad78b8e2232772ac2ecde569b24b24cbf9811a0dea8db4a8e5c6eb81896
schema_version: 0.2.0
metadata:
  license_tier: restricted
  tools:
  - GNPS
  - MSCluster
  - MassIVE
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# natural-product-msms-dereplication

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Dereplication of natural-product MS/MS spectra by spectral library search in GNPS, including the modification-tolerant 'variable dereplication' mode that adds bipartite matching edges for peak pairs offset by the precursor mass difference delta, enabling detection of putative analogs such as the -41 Da stenothricin analogs.

## When to use

Use when query tandem MS/MS spectra from natural-product extracts (e.g., MassIVE deposits of S. roseosporus or Streptomyces sp. DSM5940) must be matched against reference MS/MS library spectra, especially to find analogs whose peaks are shifted by a constant mass delta.

## When NOT to use

- The goal is clustering and relating unknown spectra with no library analog — molecular networking (MSCluster consensus spectra with mutual top-K neighbor edges) is the appropriate GNPS workflow, not library search
- Data are raw non-MS/MS or MS1-only, or the library spectra needed are not in any community library (only ~1.01% of spectra in public GNPS datasets matched to third-party reference libraries)
- An exact, modification-free spectral library search is sufficient and the extra delta edges would only add false analog matches

## Inputs

- Query MS/MS spectra from natural-product extract LC-MS/MS runs (e.g., MassIVE deposit MSV000079204 of S. roseosporus and Streptomyces sp. DSM5940 extracts)
- GNPS community reference MS/MS spectral libraries (freely available, crowdsourced)

## Outputs

- Analog match table (variable_dereplication_matches.csv) with precursor m/z, library hit, spectral match score, and delta mass shift per query spectrum
- Per-query highest-scoring library assignment with its precursor mass difference delta

## How to apply

Load reference spectra from GNPS community spectral libraries and query MS/MS spectra (e.g., from a MassIVE deposit such as MSV000079204). For each library-query candidate apply pre-filters: same precursor charge state, precursor m/z within the user-defined Thompson tolerance, minimum number of matched peaks, and minimum spectral match score. Score exact matches with a normalized dot product computed by maximum bipartite matching between library and query peaks. For variable dereplication, additionally add bipartite edges between peaks that differ by the precursor mass difference delta +/- the user-defined fragment mass tolerance, then solve the maximum bipartite matching and record the best library hit together with its delta mass shift. Report non-zero-delta matches as putative analogs (e.g., spectra 41 Da smaller than known stenothricin analogs, consistent with a lysine-to-serine substitution); chemistry-level assignments require expert review.

## Related tools

- **GNPS** (Hosts the community reference MS/MS libraries and implements the exact and variable (modification-tolerant) spectral library search via maximum bipartite matching with delta edges)
- **MassIVE** (Repository providing public deposition/retrieval of the raw MS/MS datasets used as query input for the library search)
- **MSCluster** (Pre-steps duplicate query MS/MS spectra into consensus spectra so that identical spectra are matched once against the library)

## Evaluation signals

- Output table contains the expected mass-shift class, e.g., the -41 Da shift for query spectra matching stenothricin analogs (consistent with a lysine-to-serine substitution) recovered via delta-edge matching
- All reported matches satisfy the pre-filters: same precursor charge state, precursor m/z within the user-defined Thompson tolerance, minimum matched peaks, and minimum spectral match score
- Scores are normalized dot-product values from maximum bipartite matching, and each query spectrum reports exactly one highest-scoring library match with its delta mass shift
- Non-zero delta entries are flagged as putative analogs rather than definitive identifications and routed to expert review

## Limitations

- Exact search parameters and false discovery rates are only in Supplementary Table 8 and Supplementary Note 3, not in the main text, so thresholds must be set by the user
- Chemistry-level correctness of analog assignments is subject to expert review; a non-zero delta match is putative only
- Library coverage limits sensitivity: only 1.01% of all spectra in public GNPS datasets matched to the combined third-party reference libraries
- Variable dereplication pre-filters (charge state, Thompson tolerance, minimum peaks, minimum score) still gate which candidates are considered at all

## Evidence
