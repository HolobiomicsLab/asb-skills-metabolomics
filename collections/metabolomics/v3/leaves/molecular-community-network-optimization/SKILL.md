---
name: molecular-community-network-optimization
description: Use when untargeted metabolomics mass spectrometry spectra have been
  clustered with conventional GNPS-style cosine-threshold networking (e.g., 0.7 cutoff)
  and related metabolites appear split across disconnected clusters that need reconnection.
when_to_use_negative:
- When the goal is a conventional GNPS-style molecular network with a fixed global
  cosine threshold rather than per-metabolite connectivity optimization
- When input is already a curated metabolite feature table with no MS/MS spectra,
  since MCN operates on spectral similarity between metabolite pairs
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0605
tools:
- name: molecular_communities
  role: Python package implementing GNPS-style cosine-threshold networking and MCN's
    per-metabolite connectivity optimization used to rescue edges below the 0.7 cutoff
  repo: https://github.com/Alexander0/molecular_communities
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2024.08.02.606356
    title: Ordering molecular diversity in untargeted metabolomics via molecular community
      networking
  source_content_hash: sha256:1d650201339ffe4746d1133ace50cd5871c2655f663609788e59e70aed2c9a24
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/Alexander0/molecular_communities
  tools:
  - molecular_communities
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# molecular-community-network-optimization

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Molecular community networking (MCN) is an alternative to conventional molecular networking that optimizes connectivity for each metabolite instead of relying on arbitrary global similarity thresholds (e.g., cosine >= 0.7), rescuing metabolite relationships that conventional networking fractures into separate clusters.

## When to use

Use when untargeted metabolomics mass spectrometry spectra have been clustered with conventional GNPS-style cosine-threshold networking (e.g., 0.7 cutoff) and related metabolites appear split across disconnected clusters that need reconnection.

## When NOT to use

- When the goal is a conventional GNPS-style molecular network with a fixed global cosine threshold rather than per-metabolite connectivity optimization
- When input is already a curated metabolite feature table with no MS/MS spectra, since MCN operates on spectral similarity between metabolite pairs

## Inputs

- Experimental mass spectrometry (MS/MS) spectra for the target dataset, loadable into the molecular_communities Python environment
- Conventional molecular network built with a fixed cosine similarity threshold (e.g., 0.7, GNPS-style)

## Outputs

- Reconnected-edge list: molecule pairs absent in the conventional network but linked by MCN
- Cluster/community memberships for each molecule in both the conventional network and the MCN
- Side-by-side network visualizations (e.g., Figure 1j,k analogues) with reconnected links highlighted

## How to apply

First build the conventional molecular network on the experimental MS/MS spectra by computing cosine spectral similarity between all metabolite pairs and linking pairs with cosine score >= 0.7 (GNPS-style networking as implemented in the molecular_communities package). Then build the MCN on the same spectra, applying MCN's per-metabolite connectivity optimization, which rescues edges falling below the fixed 0.7 cutoff rather than discarding them globally. Compare the two graphs to identify molecule pairs or clusters that are split into separate connected components in the conventional network but become linked in the MCN, and extract those reconnected edges. Render side-by-side network visualizations highlighting the reconnected links, and export the reconnected-edge list together with cluster memberships for each molecule in both networks. The rationale is that natural molecular diversity is organized by biological necessity, so fixed thresholds fracture natural metabolite families, and per-metabolite optimization can also surface otherwise 'hidden' metabolite connections such as the novel dipeptide-conjugated bile acids demonstrated with MCN.

## Related tools

- **molecular_communities** (Python package implementing GNPS-style cosine-threshold networking and MCN's per-metabolite connectivity optimization used to rescue edges below the 0.7 cutoff) — https://github.com/Alexander0/molecular_communities

## Evaluation signals

- The reconnected-edge list is nonempty
- Each listed edge corresponds to a pair of molecules that fall in different conventional-network connected components but in the same MCN community
- Side-by-side visualizations highlight the reconnected links consistently with the edge list and cluster memberships
- The conventional network reproduces the reported threshold behavior (pairs below cosine 0.7 are unlinked), matching Figure 1j,k behavior

## Limitations

- Conventional molecular networking's arbitrary similarity thresholds can fracture natural metabolite families; MCN addresses this but the article context provides no explicit failure modes for MCN itself
- No changelog is available for the molecular_communities repository, so version stability and API changes are not documented

## Evidence

- [abstract] Biological organization motivation: [evidence span withheld — anchor status: not_found]
- [other] Repository exists: [evidence span withheld — anchor status: not_found]
