---
name: network-cluster-comparison-analysis
description: Use when untargeted metabolomics spectral data have already been networked
  with a fixed cosine similarity cutoff (e.g., 0.7) and you need to identify metabolite
  pairs or families split across separate clusters that per-metabolite connectivity
  optimization would reconnect.
when_to_use_negative:
- The analysis goal is building a single network de novo rather than diagnosing connectivity
  lost by an existing fixed-threshold (e.g., cosine >= 0.7) network.
- The input is already a feature table or quantified abundance matrix rather than
  raw MS2 spectra needing spectral-similarity networking.
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: molecular_communities
  role: Python package implementing both GNPS-style conventional cosine networking
    (cutoff 0.7) and molecular community networking (MCN) with per-metabolite connectivity
    optimization on the same spectra
  repo: Alexander0/molecular_communities
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2024.08.02.606356
    title: Ordering molecular diversity in untargeted metabolomics via molecular community
      networking
  source_content_hash: sha256:dc8e68f2bcd110f965687e03996f17507d9ae6e47b47e6545072a6d67bf232c8
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/Alexander0/molecular_communities
  tools:
  - molecular_communities
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# network-cluster-comparison-analysis

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Compares molecular networks built on the same mass spectrometry spectra by two schemes — conventional GNPS-style cosine networking with a fixed similarity cutoff versus molecular community networking (MCN) with per-metabolite connectivity optimization — to identify metabolite relationships and clusters that the arbitrary threshold fractures but MCN rescues.

## When to use

Use when untargeted metabolomics spectral data have already been networked with a fixed cosine similarity cutoff (e.g., 0.7) and you need to identify metabolite pairs or families split across separate clusters that per-metabolite connectivity optimization would reconnect.

## When NOT to use

- The analysis goal is building a single network de novo rather than diagnosing connectivity lost by an existing fixed-threshold (e.g., cosine >= 0.7) network.
- The input is already a feature table or quantified abundance matrix rather than raw MS2 spectra needing spectral-similarity networking.

## Inputs

- Experimental mass spectrometry spectra (peak lists/MS2 spectra) for the metabolites under study
- Conventional molecular network (GNPS-style, cosine score >= 0.7 edge cutoff) built on those spectra
- Molecular community network (MCN) built on the same spectra with per-metabolite connectivity optimization

## Outputs

- Reconnected-edge list: molecule pairs in different conventional-network clusters but the same MCN community
- Cluster/community membership for each molecule in both the conventional network and the MCN
- Side-by-side network visualizations (Figure 1j,k analogues) highlighting the reconnected links

## How to apply

Load the experimental mass spectrometry spectra into the molecular_communities Python environment and build a conventional molecular network by computing cosine spectral similarity between all metabolite pairs, linking pairs with cosine score >= 0.7 (GNPS-style networking as implemented in the molecular_communities package). On the same spectra, build the molecular community network (MCN), which optimizes connectivity for each metabolite individually, allowing edges below the fixed 0.7 cutoff to be rescued. Identify molecule pairs or clusters that fall into different connected components in the conventional network but become linked within a single MCN community, and extract these reconnected edges. Render side-by-side network visualizations (as in the Figure 1j,k example) highlighting the reconnected links, and export the reconnected-edge list together with cluster memberships of each molecule in both networks. The rationale is that Nature's molecular diversity displays intricate organization, and arbitrary global similarity thresholds can fracture natural metabolite families, so a per-metabolite optimization recovers otherwise hidden connections.

## Related tools

- **molecular_communities** (Python package implementing both GNPS-style conventional cosine networking (cutoff 0.7) and molecular community networking (MCN) with per-metabolite connectivity optimization on the same spectra) — Alexander0/molecular_communities

## Evaluation signals

- The reconnected-edge list is nonempty.
- Every listed reconnected edge corresponds to a pair of molecules that fall in different conventional-network clusters but the same MCN community.
- The reconnected pairs reproduce the reported Figure 1j,k behavior where conventional networking fractures naturally related metabolites and MCN reconnects them.
- Both networks are built on the identical spectra, so any edge differences are attributable to the networking scheme, not input differences.
- Cluster membership exports allow every molecule's component assignment to be traced in both networks.

## Limitations

- Conventional molecular networking relies on arbitrary similarity thresholds (e.g., cosine >= 0.7) that can fracture natural metabolite families into separate clusters.
- The comparison requires building and maintaining two parallel networks (conventional and MCN) on the same spectra; only pairs differing between the two are interpretable as rescued connectivity.

## Evidence

- [methods] cosine cutoff and GNPS-style networking workflow: [evidence span withheld — anchor status: not_found]
- [methods] validation criterion: [evidence span withheld — anchor status: not_found]
