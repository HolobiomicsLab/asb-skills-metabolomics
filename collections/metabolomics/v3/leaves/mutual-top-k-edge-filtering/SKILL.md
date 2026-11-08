---
name: mutual-top-k-edge-filtering
description: Use when you have many MS/MS spectra (mzXML/mzML) and need to build a
  molecular network of related consensus spectra, keeping only the strongest mutual
  spectrum-to-spectrum matches rather than the full all-vs-all graph.
when_to_use_negative:
- When performing spectral library identification (exact or variable/dereplication
  search against reference MS libraries) rather than spectrum-to-spectrum networking
  — that task uses library-vs-query matching, not the mutual top-K edge rule.
- When the goal is a complete all-vs-all similarity graph — the mutual top-K rule
  deliberately discards matched pairs that are not among each other's K best neighbors.
- When spectra have not been reduced to consensus spectra (e.g. via MSCluster) — applying
  the edge filter on raw redundant spectra violates the GNPS pipeline design.
tools:
- name: MSCluster
  role: Clusters MS/MS spectra found to be identical into consensus spectra before
    spectrum-to-spectrum matching and edge filtering
- name: GNPS
  role: Platform on which the molecular network is computed and visualized online
- name: Cytoscape
  role: Offline visualization/analysis of the exported molecular network
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/nbt.3597
    title: Sharing and community curation of mass spectrometry data with Global Natural
      Products Social Molecular Networking
  source_content_hash: sha256:72dc07ccb448471e8de6f3813dd0dc12d3654ad18ea02fd2f75819d1b7857547
schema_version: 0.2.0
metadata:
  license_tier: restricted
  tools:
  - Cytoscape
  - GNPS
  - MSCluster
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# mutual-top-k-edge-filtering

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Constructs GNPS-style molecular networks from MS/MS data by clustering identical spectra into consensus spectra with MSCluster, scoring consensus pairs with a modification-tolerant spectral match, and retaining only edges that pass a user-defined minimum match score AND a mutual top-K neighbor condition.

## When to use

Use when you have many MS/MS spectra (mzXML/mzML) and need to build a molecular network of related consensus spectra, keeping only the strongest mutual spectrum-to-spectrum matches rather than the full all-vs-all graph.

## When NOT to use

- When performing spectral library identification (exact or variable/dereplication search against reference MS libraries) rather than spectrum-to-spectrum networking — that task uses library-vs-query matching, not the mutual top-K edge rule.
- When the goal is a complete all-vs-all similarity graph — the mutual top-K rule deliberately discards matched pairs that are not among each other's K best neighbors.
- When spectra have not been reduced to consensus spectra (e.g. via MSCluster) — applying the edge filter on raw redundant spectra violates the GNPS pipeline design.

## Inputs

- MS/MS spectra from input mzXML/mzML files
- user-defined minimum match score threshold
- user-defined top-K parameter
- user-defined fragment mass tolerance (for delta-based modification-tolerant matching)

## Outputs

- nodes table (one row per consensus spectrum with metadata), e.g. TSV
- edges table of node pairs passing the minimum match score and mutual top-K filter, e.g. TSV
- molecular network viewable online at GNPS or exportable to Cytoscape

## How to apply

Load the MS/MS spectra from the input mzXML/mzML files, then cluster them with MSCluster so identical spectra merge into consensus spectra. Score consensus-spectrum pairs with the modification-tolerant matching scheme: peak matching is formulated as maximum bipartite matching and scored with the normalized dot product, with extra bipartite edges allowed between peaks differing by the precursor mass difference delta plus/minus the user-defined fragment mass tolerance. Retain all spectrum-to-spectrum matches exceeding the user-defined minimum match score, then apply the edge filter: an edge between nodes A and B is kept only if B is among the top K highest-scoring neighbors of A AND A is among the top K highest-scoring neighbors of B (the mutual top-K condition); all other edges are removed. Emit a nodes table (one row per consensus spectrum with its metadata) and an edges table (retained node-pair edges), e.g. in TSV format, suitable for viewing online at GNPS or export to Cytoscape.

## Related tools

- **MSCluster** (Clusters MS/MS spectra found to be identical into consensus spectra before spectrum-to-spectrum matching and edge filtering)
- **GNPS** (Platform on which the molecular network is computed and visualized online)
- **Cytoscape** (Offline visualization/analysis of the exported molecular network)

## Evaluation signals

- Every retained edge (A, B) satisfies the mutual top-K condition: B is in the top K highest-scoring neighbors of A and A is in the top K highest-scoring neighbors of B.
- Every retained edge's spectrum-to-spectrum match exceeds the user-defined minimum match score.
- Both the nodes and edges output files exist, are non-empty, and every edge references two nodes present in the nodes table.
- Each node corresponds to a MSCluster consensus spectrum (identical raw spectra merged), not a raw redundant spectrum.
- No edge is retained whose score falls at or below the user-defined minimum match score threshold.

## Limitations

- Full molecular networking parameters were deferred to Supplementary Table 11, so key numeric settings (e.g. K value, tolerance, minimum score used in the paper) are not stated in the main text.
- The mutual top-K rule removes lower-scoring spectrum-to-spectrum matches even when they exceed the minimum match score, so the output network is sparser than the full matched graph.
- Only 1.01% of spectra in public GNPS datasets matched the then-available reference libraries, indicating insufficient chemical space coverage for annotation — the network connects related spectra but does not guarantee identifications.

## Evidence
