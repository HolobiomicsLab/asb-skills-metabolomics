---
name: memo-matrix-samplewise-normalization
description: Use when MEMO MS2 fingerprints have been aligned into a matrix of chemodiverse
  samples (e.g. the 1,600-plant-extract T. cruzi dataset) and you need comparable
  sample vectors prior to UMAP/TMAP embedding or cluster membership verification.
when_to_use_negative:
- The analysis depends on retention-time structure — MEMO is explicitly RT-agnostic,
  so RT-shift or chromatographic-alignment questions need other approaches
- The input is raw MS1 feature abundances without MS2 fragmentation spectra — the
  fingerprint counts require MS2 peaks and neutral losses to the precursor
- Samples are not chemodiverse and share good feature overlap with no RT shift, where
  standard feature-alignment-based metrics suffice
tools:
- name: memo-ms
  role: Provides the MemoMatrix class holding the aligned MS2 fingerprints that are
    sample-wise normalized before ordination
  repo: https://github.com/mandelbrot-project/memo_publication_examples
- name: numpy
  role: Numeric backend used together with memo-ms for the row-wise sample normalization
    computation
- name: matchms
  role: Underlying package MEMO is built on for handling the MS2 spectra from which
    fingerprints and the matrix are derived
  repo: https://github.com/matchms/matchms
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.3389/fbinf.2022.842964
    title: 'MEMO: Mass Spectrometry-Based Sample Vectorization to Explore Chemodiverse
      Datasets'
  source_content_hash: sha256:ce3e1abfc7d9aa6962236988b86b61b3184eaa0f8fc63676116778194b4f9991
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mandelbrot-project/memo_publication_examples
  tools:
  - matchms
  - memo-ms
  - numpy
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# memo-matrix-samplewise-normalization

## Summary

Normalizes the rows of a MEMO matrix — an aligned table of MS2-fingerprint counts (MS2 peaks and neutral losses to precursor) — sample-wise so that each metabolomics sample vector is comparable before ordination (UMAP, TMAP, MDS/PCoA) or clustering.

## When to use

Use when MEMO MS2 fingerprints have been aligned into a matrix of chemodiverse samples (e.g. the 1,600-plant-extract T. cruzi dataset) and you need comparable sample vectors prior to UMAP/TMAP embedding or cluster membership verification.

## When NOT to use

- The analysis depends on retention-time structure — MEMO is explicitly RT-agnostic, so RT-shift or chromatographic-alignment questions need other approaches
- The input is raw MS1 feature abundances without MS2 fragmentation spectra — the fingerprint counts require MS2 peaks and neutral losses to the precursor
- Samples are not chemodiverse and share good feature overlap with no RT shift, where standard feature-alignment-based metrics suffice

## Inputs

- Aligned MEMO matrix (sample × MS2 peak/neutral-loss fingerprint counts) produced from LC-MS/MS data with memo-ms MemoMatrix
- Sample annotations/metadata (e.g. the 1,600 plant extract dataset with the eight bioactive T. cruzi-inhibiting extracts labeled)

## Outputs

- Sample-wise normalized MEMO matrix whose rows are directly comparable sample vectors
- Normalized matrix ready for UMAP 2D scatter projection, TMAP tree, MDS/PCoA, or heatmap visualization

## How to apply

MEMO generates each sample's MS2 fingerprint by counting the occurrence of MS2 peaks and neutral losses (to the precursor), then aligns fingerprints across samples into a MemoMatrix without any prior feature alignment step. Before ordination, apply sample-wise normalization to the rows of this matrix so each sample fingerprint contributes comparably regardless of total spectral counts — this is essential for chemodiverse samples with poor feature overlap or strong RT shift. Use the MemoMatrix class from the memo-ms package (installed via pip in a conda environment, Python 3.8) with numpy for the row-wise computation; scikit-bio and matchms/spec2vec underpin the spectral handling. After normalization, embed with UMAP and build a TMAP tree as retention-time-agnostic, structure-aware layouts. As a validation anchor, overlay the eight bioactive extracts (>80% T. cruzi growth inhibition) and check that six co-cluster (C1) and two remain singletons (S1, S2).

## Related tools

- **memo-ms** (Provides the MemoMatrix class holding the aligned MS2 fingerprints that are sample-wise normalized before ordination) — https://github.com/mandelbrot-project/memo_publication_examples
- **numpy** (Numeric backend used together with memo-ms for the row-wise sample normalization computation)
- **matchms** (Underlying package MEMO is built on for handling the MS2 spectra from which fingerprints and the matrix are derived) — https://github.com/matchms/matchms

## Evaluation signals

- Every row of the output matrix is sample-wise normalized, making fingerprint magnitudes comparable across all 1,600 samples before distance computation
- UMAP and TMAP run on the normalized matrix both render without errors and support overlaying the eight bioactive extracts
- Verification table reproduces the reported structure: six of the eight >80% T. cruzi-inhibiting extracts co-cluster in C1 and two appear as singletons S1 and S2
- No prior feature alignment step was required to obtain the normalized matrix (MEMO's stated advantage)
- The normalized matrix feeds the memo_ms visualization module to produce both the UMAP scatter and TMAP tree files

## Limitations

- TMAP compatibility constrains the environment: required Python version was lowered from >3.8 to >3.7
- A known memo code issue affected filtering: 'Changed classes.py to avoid erasing when filtering', so filtering operations on the matrix need care
- Being RT-agnostic, normalization operates on MS2 peak/neutral-loss counts only and discards chromatographic information

## Evidence

- [intro] Fingerprint generation then alignment: "The occurence of MS2 peaks and neutral losses (to the precursor) in each sample is counted and used to generate an *MS2 fingerprint* of the sample."
- [methods] Filtering and visualization options after alignment: "different filtering (remove peaks/losses from blanks for example) and visualization techniques (MDS/PCoA, TMAP,
Heatmap, ...) can be used."
- [abstract] RT-agnostic comparison enabling cross-platform normalization-driven clustering: "MEMO thus allows the comparison of vast ensembles of samples, even when analyzed over long periods of time, and on different chromatographic or mass spectrometry platforms."
- [intro] MEMO suits chemodiverse samples — rationale for sample-wise normalization before ordination: "MEMO suits particularly well to compare chemodiverse samples, ie with a
poor features overlap, or to compare samples with a strong RT shift"
- [discussion] MemoMatrix is the class holding the matrix being normalized: "New `MemoMatrix` class, replacing the former `MemoContainer`"
- [methods] Environment setup for running the normalization: "conda create --name memo python=3.8
   conda activate memo"
