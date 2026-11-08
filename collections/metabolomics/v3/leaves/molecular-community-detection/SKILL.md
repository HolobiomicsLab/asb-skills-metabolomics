---
name: molecular-community-detection
description: Use when untargeted metabolomics MS/MS spectra (reference or experimental)
  must be grouped into molecular communities and conventional threshold-based molecular
  networking risks fracturing natural metabolite families.
when_to_use_negative:
- When a conventional fixed-threshold molecular network is explicitly required for
  comparability — MCN's per-metabolite connectivity optimization will merge or rescue
  connections that a single arbitrary similarity threshold would exclude.
- When the input is already a feature table or annotated metabolite list rather than
  spectra from which pairwise spectral similarity can be computed.
- When a quantitative structural-annotation result is needed — the reference-set validation
  here is a qualitative correspondence check of clusters against structural classes,
  not per-molecule identification.
edam_operation: http://edamontology.org/operation_3432
tools:
- name: manual expert review
  role: Used to compare MCN-derived reference-spectra clusters against the known structural
    classes of the reference compounds during the qualitative validation step.
  repo: Alexander0/molecular_communities
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2024.08.02.606356
    title: Ordering molecular diversity in untargeted metabolomics via molecular community
      networking
  source_content_hash: sha256:247a1a857da3d6af6635d1a8dfc985bd7a82a4aea8ddfddaca9c84151bbb325f
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/Alexander0/molecular_communities
  tools:
  - manual expert review
  edam_operation: http://edamontology.org/operation_3432
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# molecular-community-detection

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Molecular community networking (MCN) groups MS/MS spectra into molecular communities by optimizing connectivity for each metabolite individually, instead of cutting a spectral-similarity network at a single arbitrary threshold. This rescues metabolite relationships that fixed-threshold molecular networking fractures, as validated on a reference spectra library (Figures 1g-i, S2-11).

## When to use

Use when untargeted metabolomics MS/MS spectra (reference or experimental) must be grouped into molecular communities and conventional threshold-based molecular networking risks fracturing natural metabolite families.

## When NOT to use

- When a conventional fixed-threshold molecular network is explicitly required for comparability — MCN's per-metabolite connectivity optimization will merge or rescue connections that a single arbitrary similarity threshold would exclude.
- When the input is already a feature table or annotated metabolite list rather than spectra from which pairwise spectral similarity can be computed.
- When a quantitative structural-annotation result is needed — the reference-set validation here is a qualitative correspondence check of clusters against structural classes, not per-molecule identification.

## Inputs

- Collection of MS/MS reference spectra (cond_mcn_reference_spectra condition from the molecular_communities repository) for validation runs
- Pairwise spectral similarity graph computed over all spectra in the collection

## Outputs

- Edge list of the MCN network with per-node community/cluster assignments (e.g. mcn_reference_spectra_network.csv)
- Assignment of each reference spectrum to a molecular community, checkable against known structural classes

## How to apply

Start from the spectral collection to be networked — for validation, the deposited reference spectra collection (cond_mcn_reference_spectra condition in the Alexander0/molecular_communities repository). Compute pairwise spectral similarity between all spectra to build the similarity graph that seeds the network. Run MCN, which optimizes connectivity for each metabolite rather than applying one arbitrary similarity cutoff, to assign each spectrum to a molecular community. Export the network as an edge list with per-node community/cluster assignments (e.g. mcn_reference_spectra_network.csv). Validate by comparing MCN-derived clusters against known structural classes of the reference compounds: the result is accepted if spectra of structurally related molecules are grouped together, matching the reference-set validation reported in Figures 1g-i and S2-11.

## Related tools

- **manual expert review** (Used to compare MCN-derived reference-spectra clusters against the known structural classes of the reference compounds during the qualitative validation step.) — Alexander0/molecular_communities

## Evaluation signals

- MCN-derived clusters of the reference spectra collection group spectra of structurally related molecules together (qualitative correspondence with Figures 1g-i, S2-11).
- The exported network file (e.g. mcn_reference_spectra_network.csv) contains an edge list plus a per-node community/cluster assignment for every input spectrum.
- Connectivity is optimized per metabolite — verify that no single global similarity threshold was applied as the cluster-definition criterion.
- Connections 'rescued' relative to a conventional threshold-based network should correspond to hidden/lost metabolite relationships rather than arbitrary merges.

## Limitations

- The reference-set validation is a qualitative correspondence check against known structural classes, not a quantitative benchmark.
- The repository README in the retrieved context is effectively empty, so concrete command-line usage, parameter values, and similarity-metric thresholds are not documented in the available source material.
- MCN's per-metabolite optimization is framed against the failure mode of arbitrary similarity thresholds; behavior on very sparse or very dense similarity graphs is not characterized in the provided context.

## Evidence

- [abstract] biological rationale for ordered molecular communities: [evidence span withheld — anchor status: not_found]
