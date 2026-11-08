---
name: metabolite-connectivity-visualization
description: Use when untargeted metabolomics spectra have been networked both conventionally
  (cosine >= 0.7) and via MCN, and you need to visualize and export which molecule
  pairs were fractured into separate clusters but reconnected.
when_to_use_negative:
- The input is a feature table or other pre-summarized abundance data rather than
  pairwise spectral similarity networks built from MS/MS spectra
- Only one network exists (either conventional or MCN) — the skill's core output is
  the side-by-side comparison of both networks over the same spectra
- The goal is de novo discovery of new compound classes (e.g., novel dipeptide-conjugated
  bile acids) rather than visualizing connectivity rescue relative to the 0.7 cosine
  cutoff
tools:
- name: molecular_communities
  role: Python package used to build both the GNPS-style conventional network (cosine
    >= 0.7) and the MCN with per-metabolite connectivity optimization, and to extract
    and render the reconnected edges for Figure 1j,k-style visualizations
  repo: Alexander0/molecular_communities
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2024.08.02.606356
    title: Ordering molecular diversity in untargeted metabolomics via molecular community
      networking
  source_content_hash: sha256:58c76577ca0bd0a82811d41d3a803d971c938b8bd808439f82a5e800a5dd697f
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/Alexander0/molecular_communities
  tools:
  - molecular_communities
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# metabolite-connectivity-visualization

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Renders side-by-side conventional molecular networks and molecular community networks (MCN) for the same MS/MS spectra, highlighting edges that MCN rescues below the arbitrary 0.7 cosine cutoff used by conventional GNPS-style networking.

## When to use

Use when untargeted metabolomics spectra have been networked both conventionally (cosine >= 0.7) and via MCN, and you need to visualize and export which molecule pairs were fractured into separate clusters but reconnected.

## When NOT to use

- The input is a feature table or other pre-summarized abundance data rather than pairwise spectral similarity networks built from MS/MS spectra
- Only one network exists (either conventional or MCN) — the skill's core output is the side-by-side comparison of both networks over the same spectra
- The goal is de novo discovery of new compound classes (e.g., novel dipeptide-conjugated bile acids) rather than visualizing connectivity rescue relative to the 0.7 cosine cutoff

## Inputs

- Experimental mass spectrometry (MS/MS) spectra for the target dataset (e.g., the Figure 1j,k dataset)
- Conventional molecular network built with cosine similarity cutoff of 0.7 (GNPS-style, via molecular_communities)
- Molecular community network (MCN) built on the same spectra via per-metabolite connectivity optimization

## Outputs

- Side-by-side network visualizations (Figure 1j,k analogues) highlighting reconnected links between conventional clusters and MCN communities
- Reconnected-edge list (molecule pairs split in the conventional network but linked in the MCN)
- Cluster/community membership for each molecule in both networks

## How to apply

Load the experimental mass spectrometry spectra (e.g., the Figure 1j,k dataset) into the molecular_communities Python environment. Build the conventional molecular network by computing cosine spectral similarity between all metabolite pairs and linking pairs with cosine score >= 0.7 (GNPS-style networking as implemented in molecular_communities). Build the MCN on the same spectra, applying MCN's per-metabolite connectivity optimization to rescue edges falling below the fixed 0.7 cutoff. Identify pairs or clusters split into separate connected components in the conventional network but linked in the MCN, then render side-by-side network visualizations (Figure 1j,k analogues) with the reconnected links highlighted. Export the reconnected-edge list together with each molecule's cluster membership in both networks so the rescue can be audited.

## Related tools

- **molecular_communities** (Python package used to build both the GNPS-style conventional network (cosine >= 0.7) and the MCN with per-metabolite connectivity optimization, and to extract and render the reconnected edges for Figure 1j,k-style visualizations) — Alexander0/molecular_communities

## Evaluation signals

- The reconnected-edge list is nonempty
- Each listed edge corresponds to a pair of molecules that fall in different conventional-network clusters (cosine < 0.7 links absent) but the same MCN community
- The reconnected-edge list and cluster memberships are exported for both networks, allowing per-molecule cross-check of conventional vs. MCN assignment
- The rendered side-by-side visualization matches the reported Figure 1j,k behavior: conventional network shows split clusters, MCN shows them joined by highlighted reconnected links

## Limitations

- Conventional molecular networking's arbitrary similarity thresholds can fracture natural metabolite families; the visualization is only meaningful when both network variants are built on the identical spectra
- No changelog found for the repository, so version-specific behavior of the visualization code cannot be pinned down from the provided documentation (README is empty)

## Evidence

- [abstract] fracture rationale: [evidence span withheld — anchor status: not_found]
- [other] reconnected-edge validation from source card: [evidence span withheld — anchor status: not_found]
