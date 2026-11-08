---
name: cluster-structure-verification-in-chemodiverse-datasets
description: Use when you have an aligned MS2-fingerprint (MEMO) matrix over a large
  chemodiverse sample set and must confirm that specific bioactive or otherwise annotated
  samples occupy the reported positions (co-clusters or singletons) in UMAP/TMAP visualizations.
when_to_use_negative:
- When the goal is to generate MS2 fingerprints or align spectra rather than verify
  a reported cluster structure on an already aligned MEMO matrix — use the upstream
  MEMO vectorization workflow instead.
- When the samples of interest have no prior reported cluster assignment, so there
  is no expected structure to verify.
- When retention-time alignment is reliable and features overlap well — sample vectorization
  approaches like MEMO are motivated specifically for chemodiverse samples with poor
  feature overlap or strong RT shifts.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3391
- http://edamontology.org/topic_0092
tools:
- name: memo-ms
  role: Provides MemoMatrix/FeatureTable classes to load the MEMO matrix and the visualization
    module for UMAP embedding and TMAP tree construction on the normalized fingerprints
  repo: https://github.com/mandelbrot-project/memo_publication_examples
- name: numpy
  role: Numerical backend for the sample-wise row normalization of the MEMO matrix
    prior to ordination
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.3389/fbinf.2022.842964
    title: 'MEMO: Mass Spectrometry-Based Sample Vectorization to Explore Chemodiverse
      Datasets'
  source_content_hash: sha256:e64fbc2a942ccfec60b0449cbcd09604da38c907647c24a2fc91f74f7c99b29f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mandelbrot-project/memo_publication_examples
  tools:
  - memo-ms
  - numpy
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3391
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# cluster-structure-verification-in-chemodiverse-datasets

## Summary

Verifies that ordination of a sample-wise normalized MEMO MS2-fingerprint matrix reproduces a reported cluster organization, e.g. UMAP and TMAP layouts of a 1,600-plant-extract dataset where six bioactive T. cruzi-inhibiting extracts co-cluster (C1) and two appear as singletons (S1, S2).

## When to use

Use when you have an aligned MS2-fingerprint (MEMO) matrix over a large chemodiverse sample set and must confirm that specific bioactive or otherwise annotated samples occupy the reported positions (co-clusters or singletons) in UMAP/TMAP visualizations.

## When NOT to use

- When the goal is to generate MS2 fingerprints or align spectra rather than verify a reported cluster structure on an already aligned MEMO matrix — use the upstream MEMO vectorization workflow instead.
- When the samples of interest have no prior reported cluster assignment, so there is no expected structure to verify.
- When retention-time alignment is reliable and features overlap well — sample vectorization approaches like MEMO are motivated specifically for chemodiverse samples with poor feature overlap or strong RT shifts.

## Inputs

- Aligned MEMO matrix (sample x MS2-peak/neutral-loss fingerprint counts) from the 1,600-plant-extract dataset
- Sample annotation table identifying the eight bioactive T. cruzi-inhibiting extracts and cluster labels (C1, S1, S2)
- memo_publication_examples demo repository files (MEMO matrix and sample annotations)

## Outputs

- 2D UMAP scatter projection of the sample-wise normalized MEMO matrix with the eight bioactive extracts overlaid
- TMAP tree of the samples with the same overlay
- Cluster membership table verifying six active extracts in C1 and two singletons (S1, S2)

## How to apply

Load the MEMO matrix and sample annotations from the memo_publication_examples demo repository using memo_ms FeatureTable and MemoMatrix classes. Apply sample-wise normalization to the MEMO matrix rows so each sample fingerprint is comparable before ordination. Embed the normalized matrix with UMAP via the memo_ms visualization module for a 2D scatter projection, and build a TMAP tree as a retention-time-agnostic, structure-aware alternative layout. Overlay the samples of interest (here, the eight extracts with >80% T. cruzi growth inhibition reported in the Results) onto both the UMAP scatter and the TMAP tree. Extract cluster memberships and verify the reported structure: six of the eight active extracts co-clustered in cluster C1 while the remaining two form singletons S1 and S2. Rationale: MEMO fingerprints count MS2 peaks and neutral losses to the precursor, enabling RT-agnostic alignment, so cluster structure is robust to retention-time shifts across chemodiverse samples.

## Related tools

- **memo-ms** (Provides MemoMatrix/FeatureTable classes to load the MEMO matrix and the visualization module for UMAP embedding and TMAP tree construction on the normalized fingerprints) — https://github.com/mandelbrot-project/memo_publication_examples
- **numpy** (Numerical backend for the sample-wise row normalization of the MEMO matrix prior to ordination)

## Examples

```
conda create --name memo python=3.8; conda activate memo; pip install memo-ms
```

## Evaluation signals

- Verification table reproduces the paper's reported arrangement: exactly six of the eight bioactive extracts fall in cluster C1 and the remaining two form singletons S1 and S2.
- Both visualization artifacts are produced (UMAP scatter and TMAP tree) and render the overlay of the eight >80% T. cruzi inhibition extracts correctly.
- The MEMO matrix rows were sample-wise normalized before ordination — check that fingerprint vectors are comparable across samples prior to UMAP/TMAP.
- The UMAP and TMAP layouts are consistent with each other regarding the position of the bioactive extracts, since both operate on the same normalized MEMO distance structure.

## Limitations

- Cluster reproduction depends on the ordination method and normalization choices; the expected six-in-C1 / two-singletons arrangement was verified specifically on the sample-wise normalized MEMO matrix of the 1,600-extract dataset.
- TMAP is retained as an RT-agnostic layout alternative; results should be checked across both layouts rather than relying on a single projection.
- The memo-ms tooling requires a dedicated environment; compatibility notes indicate the Python version requirement was lowered from >3.8 to >3.7 for TMAP compatibility.

## Evidence

- [abstract] a method allowing to cluster large amounts of chemodiverse samples based on their LC-MS/MS profiles in a retention time agnostic manner: "we introduce MEMO— M S2 Bas E d Sa M ple Vect O rization—a method allowing to cluster large amounts of chemodiverse samples based on their LC-MS/MS profiles in a retention time agnostic manner"
- [intro] different filtering (remove peaks/losses from blanks for example) and visualization techniques (MDS/PCoA, TMAP, Heatmap, ...) can be used.: "different filtering (remove peaks/losses from
blanks for example) and visualization techniques (MDS/PCoA, TMAP,
Heatmap, ...) can be used."
- [methods] conda create --name memo python=3.8: "conda create --name memo python=3.8
   conda activate memo"
- [readme] The occurrence of MS2 peaks and neutral losses (to the precursor) in each sample is counted and used to generate an MS2 fingerprint of the sample.: "The occurence of MS2 peaks and neutral losses (to the precursor) in each sample is counted
and used to generate an *MS2 fingerprint* of the sample"
