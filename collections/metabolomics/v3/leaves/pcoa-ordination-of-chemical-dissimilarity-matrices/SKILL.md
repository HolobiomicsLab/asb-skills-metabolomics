---
name: pcoa-ordination-of-chemical-dissimilarity-matrices
description: Use when you already have (or can compute) sample-by-sample dissimilarity
  matrices from LC-MS/MS data — e.g. a GNPS/MZmine feature table, aligned MS2 fingerprints,
  or MEMO matrices — and need to check for batch splits (e.
when_to_use_negative:
- You need spectrum-level similarity scores between individual MS2 spectra rather
  than sample-level ordination (use matchms/spec2vec similarity scoring instead)
- Samples are not chemodiverse and share near-complete feature overlap with no RT
  shift — the batch-mitigation advantage of unaligned MEMO fingerprints is not the
  point of comparison
- No MS2 fragmentation data or MS2 fingerprints are available (MEMO requires MS2 peaks
  and neutral losses to build fingerprints)
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_2258
tools:
- name: memo-ms
  role: Produces the aligned and unaligned MEMO matrices (MemoMatrix/SpectraDocuments)
    from MS2 fingerprints and provides the visualization module used to render the
    ordinations
  repo: https://github.com/mandelbrot-project/memo_publication_examples
- name: scikit-bio
  role: Provides the PCoA (classical multidimensional scaling) implementation run
    on the numpy dissimilarity matrices
- name: numpy
  role: Array representation of the pairwise dissimilarity matrices passed to scikit-bio's
    PCoA
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.3389/fbinf.2022.842964
    title: 'MEMO: Mass Spectrometry-Based Sample Vectorization to Explore Chemodiverse
      Datasets'
  source_content_hash: sha256:dcb49797abc2447399e14395d5ac134f1ae43a102c47a7d582ed86e0fc9a0662
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mandelbrot-project/memo_publication_examples
  tools:
  - memo-ms
  - numpy
  - scikit-bio
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# pcoa-ordination-of-chemical-dissimilarity-matrices

## Summary

Runs classical multidimensional scaling (PCoA) on pairwise chemical dissimilarity matrices — feature-table based metrics, aligned MEMO fingerprint matrices, and unaligned (retention-time agnostic) MEMO matrices — to visualize how LC-MS/MS samples cluster and whether acquisition-batch effects dominate the ordination.

## When to use

Use when you already have (or can compute) sample-by-sample dissimilarity matrices from LC-MS/MS data — e.g. a GNPS/MZmine feature table, aligned MS2 fingerprints, or MEMO matrices — and need to check for batch splits (e.g. samples acquired before vs after an instrument sensitivity change) or compare clustering across metrics.

## When NOT to use

- You need spectrum-level similarity scores between individual MS2 spectra rather than sample-level ordination (use matchms/spec2vec similarity scoring instead)
- Samples are not chemodiverse and share near-complete feature overlap with no RT shift — the batch-mitigation advantage of unaligned MEMO fingerprints is not the point of comparison
- No MS2 fragmentation data or MS2 fingerprints are available (MEMO requires MS2 peaks and neutral losses to build fingerprints)

## Inputs

- Pairwise sample dissimilarity matrix computed from a feature table (e.g. plant extract demo feature table)
- Aligned MEMO fingerprint matrix (from memo-ms MemoMatrix/SpectraDocuments after feature alignment)
- Unaligned MEMO fingerprint matrix (retention-time agnostic)
- Sample metadata with acquisition dates/batch labels (pre vs post 2017-10-26) for coloring and annotation

## Outputs

- PCoA sample coordinate table per dissimilarity matrix
- Ordination figures comparing batch separation across metrics (assembled as pcoa_plant_extracts.png)
- Qualitative comparison: batch split severity in feature-table PCoA vs MEMO PCoAs

## How to apply

Compute pairwise dissimilarity matrices for each representation under comparison: one metric from the feature table, one from aligned MEMO fingerprints (MemoMatrix/SpectraDocuments classes produced via memo-ms, which requires a prior feature alignment step), and one from unaligned MEMO fingerprints computed retention-time-agnostically. Feed each dissimilarity matrix as a numpy array into scikit-bio's PCoA implementation (classical multidimensional scaling). Project samples onto the first PCoA axes and color/annotate them by acquisition batch — in the MEMO paper's plant extract demo, pre vs post 2017-10-26, a date after which RT shift and MS sensitivity change occurred. Export the PCoA sample coordinates as a table and render the ordinations with memo-ms's visualization module, assembling figures (e.g. pcoa_plant_extracts.png) for side-by-side metric comparison. Rationale: MEMO was introduced precisely because feature-table metrics suffer from poor feature overlap and RT shifts, while MEMO clusters chemodiverse samples retention-time agnostically and mitigates such batch effects.

## Related tools

- **memo-ms** (Produces the aligned and unaligned MEMO matrices (MemoMatrix/SpectraDocuments) from MS2 fingerprints and provides the visualization module used to render the ordinations) — https://github.com/mandelbrot-project/memo_publication_examples
- **scikit-bio** (Provides the PCoA (classical multidimensional scaling) implementation run on the numpy dissimilarity matrices)
- **numpy** (Array representation of the pairwise dissimilarity matrices passed to scikit-bio's PCoA)

## Examples

```
pip install numpy && pip install memo-ms && conda install -c conda-forge scikit-bio
```

## Evaluation signals

- Feature-table PCoA reproduces the reported sample split of plant extracts analyzed before 2017-10-03 vs after 2017-10-26 (RT shift and MS sensitivity change)
- PCoA on both aligned and unaligned MEMO matrices shows visibly reduced batch-driven clustering compared with the feature-table PCoA, matching Supplementary Figure S3
- Each PCoA consumes a square, symmetric dissimilarity matrix covering the same sample set, supplied to scikit-bio PCoA as a numpy array
- PCoA sample coordinates are exported as a table and the ordinations render as the assembled pcoa_plant_extracts.png
- Interactive/reproducible outputs match those published in the memo_publication_examples repository

## Limitations

- Aligned MEMO fingerprints require a prior feature alignment step, whereas the unaligned MEMO variant avoids it and runs in significantly shorter computational time
- Interpretation requires sample metadata (acquisition dates) to annotate the batch split; without it the ordination cannot be linked to batch effects
- Interactive HTML plots and exact reproduction depend on cloning memo_publication_examples and creating the documented conda environment (memo, Python 3.8)

## Evidence

- [abstract] we introduce MEMO— M S2 Bas E d Sa M ple Vect O rization—a method allowing to cluster large amounts of chemodiverse samples based on their LC-MS/MS profiles in a retention time agnostic manner: "we introduce MEMO— M S2 Bas E d Sa M ple Vect O rization—a method allowing to cluster large amounts of chemodiverse samples based on their LC-MS/MS profiles in a retention time agnostic manner"
- [intro] different filtering (remove peaks/losses from blanks for example) and visualization techniques (MDS/PCoA, TMAP, Heatmap, ...) can be used.: [evidence span withheld — anchor status: not_found]
- [abstract] such performance was achieved without the requirement of a prior feature alignment step and in a significantly shorter computational time: "such performance was achieved without the requirement of a prior feature alignment step and in a significantly shorter computational time"
- [readme] This repository contains code to reproduce MEMO analyses. The preprint is available on Biorxiv: [evidence span withheld — anchor status: not_found]
