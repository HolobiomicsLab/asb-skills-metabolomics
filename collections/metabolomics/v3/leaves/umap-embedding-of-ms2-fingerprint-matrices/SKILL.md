---
name: umap-embedding-of-ms2-fingerprint-matrices
description: Use when you have an aligned MEMO matrix of MS2 fingerprints (e.g. 1,600
  plant extracts) and need a 2D ordination to check whether biologically active samples
  co-cluster or appear as singletons.
when_to_use_negative:
- The samples are not chemodiverse but share strong feature overlap and were acquired
  under identical LC conditions — MEMO is positioned for poor feature overlap, RT
  shifts, or cross-platform (e.g. Maxis Q-ToF vs Q-Exactive Orbitrap) comparisons
- You need a quantitative distance matrix rather than a 2D visualization (use MDS/PCoA
  on the aligned fingerprints instead)
- The MS2 spectra have not yet been vectorized into an aligned MEMO matrix — fingerprint
  generation and alignment must precede UMAP embedding
edam_operation: http://edamontology.org/operation_2930
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: memo-ms
  role: Provides MemoMatrix/FeatureTable classes and the visualization module used
    to sample-wise normalize the MEMO matrix, compute the UMAP embedding, and build
    the TMAP tree
  repo: https://github.com/mandelbrot-project/memo_publication_examples
- name: numpy
  role: Numerical backend installed alongside memo-ms for handling the fingerprint
    matrix arrays
- name: matchms
  role: Underlying package memo is built on for handling the MS2 spectra that feed
    fingerprint generation
  repo: https://github.com/matchms/matchms
- name: spec2vec
  role: Underlying package memo is built on for converting MS2 spectra into documents
    during vectorization
  repo: https://github.com/iomega/spec2vec
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.3389/fbinf.2022.842964
    title: 'MEMO: Mass Spectrometry-Based Sample Vectorization to Explore Chemodiverse
      Datasets'
  source_content_hash: sha256:aa4991172ac62594cec1d2d012d2305e3c321d0c310627361cb2e9910970a53a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mandelbrot-project/memo_publication_examples
  tools:
  - matchms
  - memo-ms
  - numpy
  - spec2vec
  edam_operation: http://edamontology.org/operation_2930
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# umap-embedding-of-ms2-fingerprint-matrices

## Summary

Embed a sample-wise normalized MEMO matrix (MS2 peak/neutral-loss fingerprints) into 2D UMAP space using the memo_ms visualization module, optionally alongside an RT-agnostic TMAP tree, to expose sample clustering in chemodiverse LC-MS/MS datasets.

## When to use

Use when you have an aligned MEMO matrix of MS2 fingerprints (e.g. 1,600 plant extracts) and need a 2D ordination to check whether biologically active samples co-cluster or appear as singletons.

## When NOT to use

- The samples are not chemodiverse but share strong feature overlap and were acquired under identical LC conditions — MEMO is positioned for poor feature overlap, RT shifts, or cross-platform (e.g. Maxis Q-ToF vs Q-Exactive Orbitrap) comparisons
- You need a quantitative distance matrix rather than a 2D visualization (use MDS/PCoA on the aligned fingerprints instead)
- The MS2 spectra have not yet been vectorized into an aligned MEMO matrix — fingerprint generation and alignment must precede UMAP embedding

## Inputs

- Aligned MEMO matrix (MS2 fingerprint counts of peaks and neutral losses per sample)
- Sample annotation/metadata table (e.g. bioactivity labels such as >80% T. cruzi growth inhibition)
- FeatureTable and spectral data from the memo_publication_examples repository (or user-generated memo-ready feature table)

## Outputs

- 2D UMAP scatter projection of the normalized MEMO matrix with samples of interest overlaid
- TMAP tree layout of the samples as an RT-agnostic alternative visualization
- Cluster membership verification table (e.g. six actives in C1, two singletons S1/S2)
- Interactive HTML plot files (hosted via the memo_publication_examples GitHub pages)

## How to apply

Load the MEMO matrix and sample annotations from the memo_publication_examples demo repository using memo_ms FeatureTable and MemoMatrix classes. Normalize the MEMO matrix rows sample-wise so each MS2 fingerprint (counts of MS2 peaks and neutral losses to the precursor) is comparable before ordination. Embed the normalized matrix with UMAP via the memo_ms visualization module to obtain a 2D scatter projection, and optionally build a TMAP tree as a structure-aware, retention-time-agnostic alternative layout. Overlay the samples of interest — in the paper, the eight T. cruzi growth-inhibiting extracts with >80% inhibition from the 1,600-plant-extract dataset — onto both the UMAP scatter and the TMAP tree. Judge success by whether the overlay reproduces the expected organization: here, six active extracts co-clustered in cluster C1 and the remaining two forming singletons S1 and S2, with both visualization files produced and rendering the overlay correctly.

## Related tools

- **memo-ms** (Provides MemoMatrix/FeatureTable classes and the visualization module used to sample-wise normalize the MEMO matrix, compute the UMAP embedding, and build the TMAP tree) — https://github.com/mandelbrot-project/memo_publication_examples
- **numpy** (Numerical backend installed alongside memo-ms for handling the fingerprint matrix arrays)
- **matchms** (Underlying package memo is built on for handling the MS2 spectra that feed fingerprint generation) — https://github.com/matchms/matchms
- **spec2vec** (Underlying package memo is built on for converting MS2 spectra into documents during vectorization) — https://github.com/iomega/spec2vec

## Examples

```
conda create --name memo python=3.8; conda activate memo; pip install memo-ms numpy
```

## Evaluation signals

- Verification table reproduces the paper's reported arrangement: six of the eight bioactive extracts co-cluster in cluster C1, and the remaining two form singletons S1 and S2
- Both visualization files (UMAP scatter and TMAP tree) are produced and render the overlay of the eight >80% T. cruzi-inhibition extracts correctly
- MEMO matrix rows are sample-wise normalized before embedding (fingerprints comparable prior to ordination)
- Interactive HTML output matches the published plots available in the memo_publication_examples repository

## Limitations

- UMAP on chemodiverse fingerprints can be sensitive to normalization choice; the paper specifies sample-wise normalization of MEMO matrix rows before ordination
- TMAP compatibility constrained the Python version requirement (lowered from >3.8 to >3.7), so environment setup matters
- Clustering structure in a 2D UMAP projection is a visualization-level result; the reported C1/S1/S2 organization is specific to the 1,600-plant-extract dataset and its annotations

## Evidence

- [intro] Apply filtering (e.g. remove peaks/losses from blanks) and visualization techniques (MDS/PCoA, TMAP, Heatmap, ...).: "different filtering (remove peaks/losses from
blanks for example) and visualization techniques (MDS/PCoA, TMAP,
Heatmap, ...) can be used."
- [abstract] MEMO clusters chemodiverse samples in an RT-agnostic manner based on LC-MS/MS profiles.: "we introduce MEMO— M S2 Bas E d Sa M ple Vect O rization—a method allowing to cluster large amounts of chemodiverse samples based on their LC-MS/MS profiles in a retention time agnostic manner"
- [methods] MS2 fingerprints are generated by counting MS2 peaks and neutral losses, then aligned to compare samples.: "The occurrence of MS2 peaks and neutral losses (to the precursor) in each sample is counted
and used to generate an *MS2 fingerprint* of the sample"
- [discussion] Python version requirement was lowered to >3.7 for TMAP compatibility.: "Changed required version of python from >3.8 to >3.7 for TMAP compatibility"
- [readme] The publication-examples repository contains code and data to reproduce the MEMO paper analyses.: [evidence span withheld — anchor status: not_found]
