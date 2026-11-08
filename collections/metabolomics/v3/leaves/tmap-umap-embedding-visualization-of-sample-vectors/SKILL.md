---
name: tmap-umap-embedding-visualization-of-sample-vectors
description: Use when a MEMO sample-by-MS2-fingerprint matrix has been aligned and
  you must visualize whether samples cluster retention-time-agnostically, including
  across different LC gradients and MS instruments.
when_to_use_negative:
- The input is raw LC-MS/MS spectra or an unaligned feature table — MEMO fingerprints
  must first be computed and aligned before embedding.
- The analysis goal requires retention-time-aware or feature-alignment-based sample
  comparison rather than RT-agnostic MS2-fingerprint clustering.
- Only two or very few samples are available, where MDS/PCoA or heatmap-style views
  may be more informative than UMAP/TMAP embeddings.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: memo-ms
  role: Builds the MemoMatrix of aligned MS2 sample fingerprints that is projected
    by UMAP/TMAP
  repo: https://github.com/mandelbrot-project/memo_publication_examples
- name: matchms
  role: Handles the MS2 spectra underlying the fingerprints before embedding
  repo: https://github.com/matchms/matchms
- name: spec2vec
  role: Converts MS2 spectra into documents used to generate the sample fingerprints
  repo: https://github.com/iomega/spec2vec
- name: scikit-bio
  role: Installed in the memo conda environment supporting ordination/visualization
    of aligned samples
- name: numpy
  role: Dependency installed before memo-ms for matrix handling
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.3389/fbinf.2022.842964
    title: 'MEMO: Mass Spectrometry-Based Sample Vectorization to Explore Chemodiverse
      Datasets'
  source_content_hash: sha256:2d7b22fd7b8d9d920ac4101d4967f688bc8566f38d8a6b695790adcd6b6073e8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mandelbrot-project/memo_publication_examples
  tools:
  - Python
  - matchms
  - memo-ms
  - numpy
  - scikit-bio
  - spec2vec
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# tmap-umap-embedding-visualization-of-sample-vectors

## Summary

Projection of aligned MS2-based sample vectors (MEMO matrix rows) into 2D embeddings using UMAP and TMAP to visually assess sample clustering across chemodiverse LC-MS/MS datasets.

## When to use

Use when a MEMO sample-by-MS2-fingerprint matrix has been aligned and you must visualize whether samples cluster retention-time-agnostically, including across different LC gradients and MS instruments.

## When NOT to use

- The input is raw LC-MS/MS spectra or an unaligned feature table — MEMO fingerprints must first be computed and aligned before embedding.
- The analysis goal requires retention-time-aware or feature-alignment-based sample comparison rather than RT-agnostic MS2-fingerprint clustering.
- Only two or very few samples are available, where MDS/PCoA or heatmap-style views may be more informative than UMAP/TMAP embeddings.

## Inputs

- Aligned MEMO matrix (MemoMatrix of sample MS2-fingerprint vectors: MS2 peak and neutral-loss counts)
- Optionally blank-filtered fingerprint table
- Sample metadata labels for plotting (e.g. species, organ, year, instrument)

## Outputs

- UMAP 2D embedding of samples
- TMAP visualization of samples
- Interactive HTML plot files showing cluster positions of samples

## How to apply

First generate per-sample MS2 fingerprints by counting MS2 peaks and neutral losses to the precursor with matchms/spec2vec, then align them retention-time-agnostically into a MemoMatrix using memo-ms — no prior feature alignment step is needed. Optionally filter the matrix (e.g. remove peaks/losses attributed to blanks) before embedding. Project the aligned matrix with UMAP and with TMAP; TMAP requires the memo conda environment built with Python 3.8 (the package requirement was lowered from >3.8 to >3.7 for TMAP compatibility). Export the embeddings as interactive plots (the publication examples repo serves them as interactive HTML) and inspect whether samples expected to be chemically similar — e.g. Waltheria indica 2014/2018 extracts and the trypanocidal Melochia umbellata green stems extract — co-locate despite being acquired on different instruments (QE-Plus vs QE-Focus) and gradients (40 min vs 8 min).

## Related tools

- **memo-ms** (Builds the MemoMatrix of aligned MS2 sample fingerprints that is projected by UMAP/TMAP) — https://github.com/mandelbrot-project/memo_publication_examples
- **matchms** (Handles the MS2 spectra underlying the fingerprints before embedding) — https://github.com/matchms/matchms
- **spec2vec** (Converts MS2 spectra into documents used to generate the sample fingerprints) — https://github.com/iomega/spec2vec
- **scikit-bio** (Installed in the memo conda environment supporting ordination/visualization of aligned samples)
- **numpy** (Dependency installed before memo-ms for matrix handling)

## Examples

```
conda create --name memo python=3.8; conda activate memo; pip install numpy; pip install memo-ms
```

## Evaluation signals

- The merged MEMO matrix contains the appended sample rows/columns (e.g. the three Waltheria indica 2014/2018 samples added to the plant dataset matrix).
- Both UMAP and TMAP plots show chemically related samples co-clustering: all three Waltheria indica extracts cluster with the trypanocidal Melochia umbellata green stems extract.
- Clustering is achieved retention-time-agnostically, without any prior feature alignment step.
- The environment uses Python 3.8 (memo conda env) so that TMAP imports/run successfully.
- Embeddings are exported as interactive plots comparable to those published at mandelbrot-project.github.io/memo_publication_examples/.

## Limitations

- TMAP compatibility constrains the Python version: the required version was changed from >3.8 to >3.7; the recommended setup is a dedicated conda env with Python 3.8.
- Fingerprint filtering bugs existed historically — classes.py was changed 'to avoid erasing when filtering', so verify filtering does not drop legitimate peaks/losses.
- Batch/platform differences (40 min vs 8 min gradients, QE-Plus vs QE-Focus) are tolerated but should be checked visually; MEMO's advantage is RT-agnostic comparison of chemodiverse samples, not guaranteed resolution of any two samples.

## Evidence

- [abstract] we introduce MEMO— M S2 Bas E d Sa M ple Vect O rization—a method allowing to cluster large amounts of chemodiverse samples based on their LC-MS/MS profiles in a retention time agnostic manner: "we introduce MEMO— M S2 Bas E d Sa M ple Vect O rization—a method allowing to cluster large amounts of chemodiverse samples based on their LC-MS/MS profiles in a retention time agnostic manner"
- [intro] different filtering (remove peaks/losses from
blanks for example) and visualization techniques (MDS/PCoA, TMAP,
Heatmap, ...) can be used.: "different filtering (remove peaks/losses from
blanks for example) and visualization techniques (MDS/PCoA, TMAP,
Heatmap, ...) can be used."
- [discussion] Changed required version of python from >3.8 to >3.7 for TMAP compatibility: "Changed required version of python from >3.8 to >3.7 for TMAP compatibility"
- [readme] The occurence of MS2 peaks and neutral losses (to the precursor) in each sample is counted: "The occurence of MS2 peaks and neutral losses (to the precursor) in each sample is counted"
- [readme] This repository contains code to reproduce MEMO analyses. The preprint is available on [Biorxiv](https://www.biorxiv.org/content/10.1101/2021.12.24.474089v1).: [evidence span withheld — anchor status: not_found]
