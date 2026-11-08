---
name: modification-tolerant-variable-dereplication
description: Use when a molecular networking or dereplication job returns unannotated
  nodes whose spectra resemble a known compound library hit but whose precursor masses
  differ by a consistent delta, suggesting analogs or modifications.
when_to_use_negative:
- A query spectrum is expected to match the library entry exactly (no modification)
  — use GNPS exact spectral library search with normalized dot product instead
- MS/MS spectra have MS1 precursor intensity <1E4 or precursor m/z ppm error >=50
  relative to theoretical mass — these are ignored by the dereplication workflow
- The goal is de novo structural elucidation rather than annotation against existing
  reference libraries — variable dereplication only matches known library spectra
  plus a mass delta
tools:
- name: GNPS
  role: Hosts the variable (modification-tolerant) dereplication search that adds
    delta-shifted edges to the library-query peak bipartite matching
- name: MassIVE
  role: Repository from which the raw MS/MS data (e.g., MSV000079204) are retrieved
    before dereplication
- name: MSCluster
  role: Merges identical MS/MS spectra into consensus spectra that are then matched
    against each other and the library
- name: Cytoscape
  role: Visualization of the annotated molecular network with dereplication annotations
    and mass shifts
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/nbt.3597
    title: Sharing and community curation of mass spectrometry data with Global Natural
      Products Social Molecular Networking
  source_content_hash: sha256:aecf0091e8184bf12e58ab5a4d1a33d1a3ff0c0f9acd234ae4a66922224ba8e0
schema_version: 0.2.0
metadata:
  license_tier: restricted
  tools:
  - Cytoscape
  - GNPS
  - MSCluster
  - MassIVE
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# modification-tolerant variable dereplication

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

A GNPS dereplication mode that annotates MS/MS spectra against reference libraries while allowing a systematic mass shift, so that modified analogs (e.g., stenothricin analogs shifted by -41 Da) are flagged instead of left unannotated.

## When to use

Use when a molecular networking or dereplication job returns unannotated nodes whose spectra resemble a known compound library hit but whose precursor masses differ by a consistent delta, suggesting analogs or modifications.

## When NOT to use

- A query spectrum is expected to match the library entry exactly (no modification) — use GNPS exact spectral library search with normalized dot product instead
- MS/MS spectra have MS1 precursor intensity <1E4 or precursor m/z ppm error >=50 relative to theoretical mass — these are ignored by the dereplication workflow
- The goal is de novo structural elucidation rather than annotation against existing reference libraries — variable dereplication only matches known library spectra plus a mass delta

## Inputs

- MS/MS consensus spectra (MSCluster-merged) from GNPS molecular networking job on deposited MassIVE data (e.g., MSV000079204)
- GNPS community reference MS/MS spectral library entries with annotated precursor m/z
- User-defined search parameters: precursor ion m/z tolerance (Thompson), fragment mass tolerance, minimum matched peaks, minimum match score

## Outputs

- Putative library identifications annotated onto network nodes, including modification-tolerant hits with precursor mass deltas (e.g., -41 Da stenothricin analog nodes)
- Dereplication annotations on sub-network nodes and edges exportable for Cytoscape visualization

## How to apply

Run the modification-tolerant search in GNPS after (or alongside) the exact spectral library search on the MSCluster-merged consensus spectra from the deposited dataset (e.g., MSV000079204 MassIVE files). Exact matches are first scored with a normalized dot product via maximum bipartite matching of peaks, subject to filters: same precursor charge state, precursor m/z within a user-defined Thompson tolerance, minimum number of matched peaks, and minimum match score. In variable dereplication, additional edges are added to the bipartite matching between library and query peaks which differ by a delta (the precursor mass difference between library and query) +/- the user-defined fragment mass tolerance, so in-source fragments and analogs differing by a fixed modification are still matched. Query spectra are only accepted as putative identifications when precursor m/z ppm error is <50, and MS/MS spectra with MS1 precursor intensity <1E4 are ignored. Applied to MSV000079204, this flags the known stenothricin nodes and reveals 14 sub-network nodes 41 Da smaller, defining stenothricin-GNPS 1-5.

## Related tools

- **GNPS** (Hosts the variable (modification-tolerant) dereplication search that adds delta-shifted edges to the library-query peak bipartite matching)
- **MassIVE** (Repository from which the raw MS/MS data (e.g., MSV000079204) are retrieved before dereplication)
- **MSCluster** (Merges identical MS/MS spectra into consensus spectra that are then matched against each other and the library)
- **Cytoscape** (Visualization of the annotated molecular network with dereplication annotations and mass shifts)

## Evaluation signals

- Known stenothricin library nodes are annotated, and a small sub-network of 14 nodes appears with masses 41 Da smaller than annotated stenothricin nodes
- The delta-shifted sub-network groups 5 abundant analogs (stenothricin-GNPS 1-5), with the most abundant analog, stenothricin-GNPS 2, at m/z 1105
- Accepted identifications satisfy precursor m/z ppm error <50 versus theoretical precursor mass
- Retained matches share at least the minimum number of matched peaks, same precursor charge state, precursor m/z within the user-defined Thompson tolerance, and exceed the minimum spectral match score
- Edges retained satisfy the mutual top-K neighbor condition in the molecular network

## Limitations

- Only 1.01% of all spectra in public GNPS datasets matched to third-party reference libraries, so chemical space coverage limits dereplication recall
- Molecular networking parameters (Supplementary Table 11) and search parameters/FDRs (Supplementary Table 8, Note 3) are deferred to non-included supplementary materials, and the GNPS job link in the paper is only a placeholder '(link)', so parameters must be reconstructed
- The search tolerates a single systematic precursor mass delta; analogs with multiple or non-uniform modifications may not be captured

## Evidence
