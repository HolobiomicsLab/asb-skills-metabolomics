---
name: reference-spectra-library-validation
description: Use when a curated reference spectral library is available and you need
  to demonstrate that a proposed spectral-networking scheme (e.g. MCN vs. conventional
  thresholded molecular networking) produces valid, structurally meaningful connections
  before applying it to untargeted data.
when_to_use_negative:
- Your data are untargeted sample spectra without trusted structural annotations —
  the validation step relies on comparing clusters to known structural classes of
  reference compounds.
- You need a simple pairwise similarity threshold (cosine cutoff) rather than a connectivity-optimized
  community assignment — conventional molecular networking may suffice and this skill
  is for MCN validation.
- Input is already a molecular network or cluster assignment; this skill builds and
  validates the network from raw reference spectra.
edam_operation: http://edamontology.org/operation_3649
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: molecular community networking (MCN)
  role: Algorithm applied to the reference spectra similarity graph to assign each
    spectrum to a molecular community and export the network with cluster assignments
  repo: https://github.com/Alexander0/molecular_communities
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2024.08.02.606356
    title: Ordering molecular diversity in untargeted metabolomics via molecular community
      networking
  source_content_hash: sha256:62deca96af0b884555e495e35b6fed090b03131247beaf775d1a7125e0e15541
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/Alexander0/molecular_communities
  tools:
  - molecular community networking (MCN)
  edam_operation: http://edamontology.org/operation_3649
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# reference-spectra-library-validation

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Builds a molecular community network (MCN) over a collection of reference mass-spectrometry spectra and verifies that the network's clusters group structurally related molecules together, validating that MCN connections reflect meaningful molecular relationships.

## When to use

Use when a curated reference spectral library is available and you need to demonstrate that a proposed spectral-networking scheme (e.g. MCN vs. conventional thresholded molecular networking) produces valid, structurally meaningful connections before applying it to untargeted data.

## When NOT to use

- Your data are untargeted sample spectra without trusted structural annotations — the validation step relies on comparing clusters to known structural classes of reference compounds.
- You need a simple pairwise similarity threshold (cosine cutoff) rather than a connectivity-optimized community assignment — conventional molecular networking may suffice and this skill is for MCN validation.
- Input is already a molecular network or cluster assignment; this skill builds and validates the network from raw reference spectra.

## Inputs

- Reference spectral library (deposited reference spectra collection, cond_mcn_reference_spectra condition from the molecular_communities repository)
- Known structural class annotations for the reference compounds

## Outputs

- MCN network edge list with per-node community/cluster assignments (e.g. mcn_reference_spectra_network.csv)
- Qualitative correspondence between MCN clusters and known structural classes (validation result)

## How to apply

Load the deposited reference spectra collection (the cond_mcn_reference_spectra condition in the molecular_communities repository) and compute pairwise spectral similarity between all reference spectra to construct the similarity graph. Run molecular community networking (MCN), which optimizes connectivity for each metabolite rather than imposing a single arbitrary similarity threshold, to assign each reference spectrum to a molecular community. Export the network as an edge list with per-node community/cluster assignments (e.g. mcn_reference_spectra_network.csv). Compare the MCN-derived clusters against the known structural classes of the reference compounds; acceptance requires that clusters group spectra of structurally related molecules together, as validated in Figures 1g-i and S2-11. This differs from conventional molecular networking, where arbitrary similarity thresholds can fracture natural metabolite families.

## Related tools

- **molecular community networking (MCN)** (Algorithm applied to the reference spectra similarity graph to assign each spectrum to a molecular community and export the network with cluster assignments) — https://github.com/Alexander0/molecular_communities

## Examples

```
python run_mcn.py --condition cond_mcn_reference_spectra --output mcn_reference_spectra_network.csv
```

## Evaluation signals

- MCN-derived clusters of the reference collection group spectra of structurally related molecules together (qualitative correspondence check against Figures 1g-i and S2-11).
- Every reference spectrum receives exactly one community/cluster assignment in the exported edge list.
- The exported network file (e.g. mcn_reference_spectra_network.csv) contains a complete edge list with per-node cluster metadata.
- The network reproduces the reference-set validation reported in the paper (Figures 1g-i, S2-11), demonstrating that connections in the generated network are valid.

## Limitations

- Validation of connection validity is a qualitative correspondence check against known structural classes rather than a quantitative accuracy metric.
- The procedure depends on a curated reference spectra collection with known structural class annotations; it cannot validate connections for spectra without trusted ground-truth structures.
- MCN results depend on the similarity graph over the reference spectra; spectra missing from the library will not appear in the validated clusters.

## Evidence

- [methods] MCN on reference spectra establishes validity of connections: [evidence span withheld — anchor status: not_found]
- [other] Workflow output file: [evidence span withheld — anchor status: not_found]
