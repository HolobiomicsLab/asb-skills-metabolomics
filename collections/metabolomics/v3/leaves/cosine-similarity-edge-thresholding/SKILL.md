---
name: cosine-similarity-edge-thresholding
description: Use when constructing a baseline molecular network from LC-MS/MS spectral
  pairs via cosine spectral similarity, or when reproducing/evaluating the Figure
  1j,k experiment that shows which related metabolite pairs get fractured into separate
  clusters by a fixed cutoff.
when_to_use_negative:
- Input spectra are already a normalized feature table or abundance matrix rather
  than pairwise spectral similarity data — this skill operates on spectral similarity,
  not on quantitative feature tables.
- The goal is to preserve biologically real but weak similarity links; a fixed 0.7
  cosine cutoff will fracture them — use MCN's per-metabolite optimization instead
  of a single global threshold.
- No pairwise cosine spectral similarity can be computed (e.g. spectra are unavailable
  or not directly comparable) — the skill depends on a pairwise similarity matrix.
tools:
- name: molecular_communities
  role: Python package used to load spectra, compute cosine spectral similarity, build
    the conventional (>=0.7) network, and compare it against MCN connectivity optimization.
  repo: https://github.com/Alexander0/molecular_communities
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2024.08.02.606356
    title: Ordering molecular diversity in untargeted metabolomics via molecular community
      networking
  source_content_hash: sha256:8e1fa066d2ca18dc05429c51492cf3f5eab8dd0e0a6412a96d4ddb0476d448b7
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
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# cosine-similarity-edge-thresholding

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Builds a conventional molecular network by linking MS/MS spectra pairs whose cosine spectral similarity clears a fixed threshold (e.g. the GNPS-style score >= 0.7 cutoff), producing a graph used as the baseline that molecular community networking (MCN) rescues or improves upon.

## When to use

Use when constructing a baseline molecular network from LC-MS/MS spectral pairs via cosine spectral similarity, or when reproducing/evaluating the Figure 1j,k experiment that shows which related metabolite pairs get fractured into separate clusters by a fixed cutoff.

## When NOT to use

- Input spectra are already a normalized feature table or abundance matrix rather than pairwise spectral similarity data — this skill operates on spectral similarity, not on quantitative feature tables.
- The goal is to preserve biologically real but weak similarity links; a fixed 0.7 cosine cutoff will fracture them — use MCN's per-metabolite optimization instead of a single global threshold.
- No pairwise cosine spectral similarity can be computed (e.g. spectra are unavailable or not directly comparable) — the skill depends on a pairwise similarity matrix.

## Inputs

- Experimental MS/MS spectra (feature/spectral dataset used for Figure 1j,k)
- Pairwise cosine spectral similarity scores between metabolites
- Fixed similarity threshold (0.7, GNPS-style conventional networking cutoff)

## Outputs

- Conventional molecular network graph (edges = metabolite pairs with cosine score >= 0.7)
- Connected components / cluster memberships for each molecule under the conventional network
- Reconnected-edge list: pairs split into different conventional components but linked in the MCN

## How to apply

Load experimental mass spectrometry spectra into the molecular_communities Python environment, then compute pairwise cosine spectral similarity between all metabolites. Link pairs with cosine score >= 0.7 — the GNPS-style conventional-networking cutoff the article identifies as an arbitrary threshold that can fracture natural metabolite families. Treat the resulting connected components as the conventional molecular network and record which pairs or clusters fall into separate components. When used as a baseline for MCN, keep this network fixed and apply MCN's per-metabolite connectivity optimization on the same spectra to recover edges below the 0.7 cutoff. Validation: render side-by-side visualizations (Figure 1j,k analogues) highlighting reconnected links, and confirm each rescued edge joins molecules that were in different conventional components but the same MCN community.

## Related tools

- **molecular_communities** (Python package used to load spectra, compute cosine spectral similarity, build the conventional (>=0.7) network, and compare it against MCN connectivity optimization.) — https://github.com/Alexander0/molecular_communities

## Examples

```
python -m molecular_communities --input spectra.msms --cosine-cutoff 0.7 --mode conventional
```

## Evaluation signals

- Reconnected-edge list is non-empty (equivalently: some edges exist below the 0.7 cosine cutoff that MCN recovers).
- Every listed reconnected edge joins a pair of molecules that were in different conventional-network connected components but the same MCN community — mirroring the Figure 1j,k behavior.
- Side-by-side network visualizations (Figure 1j,k analogues) show reconnected links distinct from the fixed-threshold edges.
- Cluster memberships exported for each molecule in both networks are consistent with the reconnected-edge list (a pair listed as reconnected must have differing conventional memberships and matching MCN memberships).

## Limitations

- A fixed similarity threshold is arbitrary and can fracture natural metabolite families — the article states 'using arbitrary similarity thresholds that can fracture natural metabolite families', so a single global cutoff may drop genuine relationships.
- MCN is needed to rescue lost connectivity: 'optimizes connectivity for each metabolite, rescuing lost relationships and capturing otherwise hidden metabolite connections' — plain cosine thresholding alone will not recover these hidden links.
- No changelog or additional README guidance was available for the molecular_communities repository, so parameters beyond the 0.7 cosine cutoff are not documented here.

## Evidence

- [abstract] arbitrary similarity thresholds that can fracture natural metabolite families: "using arbitrary similarity thresholds that can fracture natural metabolite families"
- [abstract] MCN rescues lost relationships and captures hidden metabolite connections: "We present molecular community networking (MCN), that optimizes connectivity for each metabolite, rescuing lost relationships and capturing otherwise "hidden" metabolite connections."
- [other] Workflow builds the conventional network via cosine score >= 0.7, then builds MCN to rescue edges below the cutoff: [evidence span withheld — anchor status: not_found]
- [other] Validation requires reconnected-edge list to be nonempty and each edge to join molecules split in conventional clusters but joined in MCN: [evidence span withheld — anchor status: not_found]
