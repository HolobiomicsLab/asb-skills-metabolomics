---
name: tmap-tree-visualization-of-metabolomics-samples
description: Use when samples have been vectorized into a MEMO MS2-fingerprint matrix
  and you need an RT-agnostic tree layout to explore clustering, overlay annotations,
  or cross-check UMAP/PCoA groupings of chemodiverse samples.
when_to_use_negative:
- Input is raw MS/MS spectra or an unaligned feature table — MEMO fingerprints must
  be counted and aligned first
- Samples are not chemodiverse and overlap well with retention-time alignment available
  — a standard feature-based ordination may be more appropriate
- You need intensity-aware distance metrics (e.g., Bray-Curtis on a feature table)
  rather than MS2-occurrence-based sample comparison
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0092
tools:
- name: memo-ms
  role: Provides the MemoMatrix class and the visualization module used to build the
    TMAP tree of samples from the normalized MEMO matrix
  repo: https://github.com/mandelbrot-project/memo_publication_examples
- name: matchms
  role: Underlying package used by MEMO for handling the MS2 spectra from which fingerprints
    are generated before TMAP rendering
  repo: https://github.com/matchms/matchms
- name: numpy
  role: Numerical backend installed alongside memo-ms for the sample-wise matrix normalization
    prior to TMAP/UMAP
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.3389/fbinf.2022.842964
    title: 'MEMO: Mass Spectrometry-Based Sample Vectorization to Explore Chemodiverse
      Datasets'
  source_content_hash: sha256:6e029187d29d24ecf2f9db28eea5f4d1dfb7d44c66575c404050d0c30e86e6b3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mandelbrot-project/memo_publication_examples
  tools:
  - matchms
  - memo-ms
  - numpy
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# TMAP tree visualization of metabolomics samples

## Summary

Builds a TMAP tree layout of metabolomics samples from an aligned MEMO matrix, offering a retention-time-agnostic, structure-aware alternative to UMAP/MDS/PCoA scatter ordination for exploring chemodiverse LC-MS/MS datasets.

## When to use

Use when samples have been vectorized into a MEMO MS2-fingerprint matrix and you need an RT-agnostic tree layout to explore clustering, overlay annotations, or cross-check UMAP/PCoA groupings of chemodiverse samples.

## When NOT to use

- Input is raw MS/MS spectra or an unaligned feature table — MEMO fingerprints must be counted and aligned first
- Samples are not chemodiverse and overlap well with retention-time alignment available — a standard feature-based ordination may be more appropriate
- You need intensity-aware distance metrics (e.g., Bray-Curtis on a feature table) rather than MS2-occurrence-based sample comparison

## Inputs

- Aligned, sample-wise normalized MEMO matrix (MS2 fingerprint count table) of metabolomics samples
- Sample annotation/metadata table (e.g., bioactivity labels such as >80% T. cruzi growth inhibition)
- MemoMatrix object created with memo_ms from the feature table or spectra files

## Outputs

- TMAP tree of samples with overlaid annotations (interactive HTML plot)
- UMAP 2D scatter projection of the same normalized MEMO matrix for comparison
- Cluster membership table identifying clusters and singletons (e.g., C1, S1, S2)

## How to apply

First generate the MS2 fingerprint of each sample by counting MS2 peaks and neutral losses to the precursor with memo_ms, then align the fingerprints into a MEMO matrix. Apply sample-wise normalization to the matrix rows so fingerprints are comparable before ordination. Embed the normalized matrix with UMAP via the memo_ms visualization module, and in parallel build a TMAP tree of the samples as an RT-agnostic, structure-aware alternative layout. Overlay sample annotations (e.g., the eight bioactive T. cruzi-inhibiting extracts, defined by >80% growth inhibition) onto both the UMAP scatter and the TMAP tree, then extract cluster memberships and verify the reported structure (six co-clustered in C1, two singletons S1/S2) is reproduced. Install memo-ms in a dedicated conda environment (python 3.8); note TMAP compatibility drove the package's Python requirement being changed from >3.8 to >3.7.

## Related tools

- **memo-ms** (Provides the MemoMatrix class and the visualization module used to build the TMAP tree of samples from the normalized MEMO matrix) — https://github.com/mandelbrot-project/memo_publication_examples
- **matchms** (Underlying package used by MEMO for handling the MS2 spectra from which fingerprints are generated before TMAP rendering) — https://github.com/matchms/matchms
- **numpy** (Numerical backend installed alongside memo-ms for the sample-wise matrix normalization prior to TMAP/UMAP)

## Examples

```
conda create --name memo python=3.8; conda activate memo; pip install memo-ms
```

## Evaluation signals

- TMAP tree renders successfully as an interactive HTML plot with sample annotations overlaid
- Cluster memberships extracted from the tree match the reported structure: six of eight active extracts co-clustered in C1, the remaining two as singletons S1 and S2
- TMAP and UMAP layouts applied to the same sample-wise normalized MEMO matrix are mutually consistent on the overlaid bioactive extracts
- memo-ms installed in a dedicated conda environment with a Python version compatible with TMAP (>3.7)
- All samples of the 1,600-extract dataset appear as nodes in the TMAP tree with no dropped samples after normalization

## Limitations

- TMAP compatibility constrains the Python version — the package changed its required version of python from >3.8 to >3.7 for TMAP compatibility
- TMAP is one of several visualization options (MDS/PCoA, TMAP, Heatmap); results should be interpreted alongside, not instead of, other ordinations
- The visualization is retention-time agnostic by design, so it will not reflect chromatographic (RT) differences between samples

## Evidence

- [intro] visualization techniques (MDS/PCoA, TMAP, Heatmap, ...) can be used.: "different filtering (remove peaks/losses from
blanks for example) and visualization techniques (MDS/PCoA, TMAP,
Heatmap, ...) can be used."
- [readme] MEMO allows RT-agnostic alignment using MS2 fragmentation spectra before visualization.: "is a method allowing a Retention Time (RT) agnostic alignment of
metabolomics samples using the fragmentation spectra (MS2) of their
consituents."
- [discussion] Python requirement changed for TMAP compatibility.: "Changed required version of python from >3.8 to >3.7 for TMAP compatibility"
- [methods] Install memo-ms in a dedicated conda environment.: "conda create --name memo python=3.8
   conda activate memo"
