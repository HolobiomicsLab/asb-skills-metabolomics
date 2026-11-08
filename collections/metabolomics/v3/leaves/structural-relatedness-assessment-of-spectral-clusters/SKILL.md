---
name: structural-relatedness-assessment-of-spectral-clusters
description: Use when a spectral similarity network (e.g., an MCN output) must be
  validated against reference spectra with known structural classes, to check that
  community/cluster assignments group structurally related molecules together.
when_to_use_negative:
- The spectra lack known structural class annotations, so cluster-to-structure correspondence
  cannot be checked (this skill is a reference-set validation, not a discovery workflow).
- The network was built with a conventional single arbitrary similarity threshold
  rather than MCN's per-metabolite connectivity optimization, since the validation
  is specific to MCN-derived communities.
edam_operation: http://edamontology.org/operation_2929
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0157
tools:
- name: manual expert review
  role: Qualitative correspondence check comparing MCN-derived reference-spectra clusters
    against known structural classes of the reference compounds
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2024.08.02.606356
    title: Ordering molecular diversity in untargeted metabolomics via molecular community
      networking
  source_content_hash: sha256:a710c5493d31b1d1fa587d8e354430f12a5e9cf3ef30528c5a0aa6d04696a296
schema_version: 0.2.0
metadata:
  license_tier: restricted
  tools:
  - manual expert review
  edam_operation: http://edamontology.org/operation_2929
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0157
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# structural-relatedness-assessment-of-spectral-clusters

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Validation procedure for molecular community networking (MCN) outputs in which clusters derived from a deposited reference spectra collection are compared against the known structural classes of the reference compounds, confirming that network connections reflect meaningful molecular relationships rather than arbitrary similarity-threshold artifacts.

## When to use

Use when a spectral similarity network (e.g., an MCN output) must be validated against reference spectra with known structural classes, to check that community/cluster assignments group structurally related molecules together.

## When NOT to use

- The spectra lack known structural class annotations, so cluster-to-structure correspondence cannot be checked (this skill is a reference-set validation, not a discovery workflow).
- The network was built with a conventional single arbitrary similarity threshold rather than MCN's per-metabolite connectivity optimization, since the validation is specific to MCN-derived communities.

## Inputs

- Deposited reference spectra collection (cond_mcn_reference_spectra condition) from the Alexander0/molecular_communities repository
- Known structural class annotations for the reference compounds
- Pairwise spectral similarity matrix computed over all reference spectra

## Outputs

- Edge list of the MCN network with per-node community/cluster assignments (e.g., mcn_reference_spectra_network.csv)
- Qualitative correspondence assessment mapping MCN clusters to structural classes of the reference compounds

## How to apply

Load the deposited reference spectra collection from the molecular_communities repository (the cond_mcn_reference_spectra condition) and compute pairwise spectral similarity between all reference spectra to construct the similarity graph that seeds the network. Run MCN, which optimizes connectivity for each metabolite rather than applying a single arbitrary similarity threshold, to assign each reference spectrum to a molecular community. Export the resulting network as an edge list with per-node community/cluster assignments (e.g., to mcn_reference_spectra_network.csv). Then compare the MCN-derived clusters against the known structural classes of the reference compounds, verifying qualitatively that spectra of structurally related molecules fall in the same community. This mirrors the paper's reference-set validation (Figures 1g-i and S2-11), which established the validity of connections in the generated networks.

## Related tools

- **manual expert review** (Qualitative correspondence check comparing MCN-derived reference-spectra clusters against known structural classes of the reference compounds)

## Evaluation signals

- MCN-derived clusters of the reference collection group spectra of structurally related molecules together (the paper's acceptance criterion for Figures 1g-i).
- The output edge list contains per-node community/cluster assignments covering the reference spectra set.
- Cluster membership is consistent with the reference compounds' known structural classes, matching the correspondence reported in Figures 1g-i and S2-11.
- The network was seeded from a full pairwise spectral-similarity graph over the reference collection, not a filtered thresholded graph.

## Limitations

- Validation is a qualitative correspondence check rather than a quantitative metric, so borderline cluster/structure mismatches require expert judgment.
- Conventional molecular networking's arbitrary similarity thresholds can fracture natural metabolite families, so results from threshold-based networks may not reproduce the MCN cluster structure being validated.

## Evidence

- [abstract] arbitrary-threshold rationale: [evidence span withheld — anchor status: not_found]
- [other] workflow validation quote: [evidence span withheld — anchor status: not_found]
- [other] network construction quote: [evidence span withheld — anchor status: not_found]
