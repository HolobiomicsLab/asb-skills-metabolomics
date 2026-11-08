---
name: ms2-fingerprint-sample-metrics-interpretation
description: Use when an LC-MS/MS dataset shows an acquisition-batch split (e.g.,
  RT shift or MS sensitivity drift around a run date) and you must test whether MS2-fingerprint
  MEMO matrices mitigate it relative to feature-table PCoA.
when_to_use_negative:
- When the goal is to cluster chemodiverse samples in a retention-time-agnostic way
  from scratch, the fingerprint-generation/vectorization skill should be used instead
  — this skill assumes MEMO matrices already exist.
- When samples were acquired on identical platforms with no RT shift or sensitivity
  drift, a plain feature-table dissimilarity (e.g., Bray-Curtis) is adequate and the
  MEMO comparison is unnecessary.
- When the input is raw mzML spectra rather than MS2 fingerprint matrices — this skill
  starts from MemoMatrix/SpectraDocuments objects, not raw spectral files.
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: memo-ms
  role: Provides MemoMatrix/SpectraDocuments classes to build aligned and unaligned
    MS2 fingerprint matrices and a visualization module for rendering the ordinations
  repo: https://github.com/mandelbrot-project/memo_publication_examples
- name: scikit-bio
  role: Runs classical multidimensional scaling (PCoA) on each dissimilarity matrix,
    with numpy arrays as input
- name: numpy
  role: Holds the pairwise dissimilarity matrices as arrays passed into the scikit-bio
    PCoA call
- name: matchms
  role: Underlying package memo-ms uses for handling the MS2 spectra from which fingerprints
    are counted
  repo: https://github.com/matchms/matchms
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.3389/fbinf.2022.842964
    title: 'MEMO: Mass Spectrometry-Based Sample Vectorization to Explore Chemodiverse
      Datasets'
  source_content_hash: sha256:94e8a0b9e7ab36061f53c8e25f2daeb3c50e09ba7523a26a14520fc960f1e5d6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mandelbrot-project/memo_publication_examples
  tools:
  - matchms
  - memo-ms
  - numpy
  - scikit-bio
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# ms2-fingerprint-sample-metrics-interpretation

## Summary

Reproduces the plant-extract batch-effect analysis from the MEMO paper (Supplementary Figure S3) by comparing PCoA ordinations of a feature-table-based dissimilarity metric against PCoA on aligned and unaligned MEMO MS2-fingerprint matrices, showing that MEMO mitigates the pre/post 2017-10-26 acquisition batch split.

## When to use

Use when an LC-MS/MS dataset shows an acquisition-batch split (e.g., RT shift or MS sensitivity drift around a run date) and you must test whether MS2-fingerprint MEMO matrices mitigate it relative to feature-table PCoA.

## When NOT to use

- When the goal is to cluster chemodiverse samples in a retention-time-agnostic way from scratch, the fingerprint-generation/vectorization skill should be used instead — this skill assumes MEMO matrices already exist.
- When samples were acquired on identical platforms with no RT shift or sensitivity drift, a plain feature-table dissimilarity (e.g., Bray-Curtis) is adequate and the MEMO comparison is unnecessary.
- When the input is raw mzML spectra rather than MS2 fingerprint matrices — this skill starts from MemoMatrix/SpectraDocuments objects, not raw spectral files.

## Inputs

- Plant extract demo feature table (LC-MS/MS features from the MEMO paper dataset)
- Aligned MEMO matrix built from aligned MS2 fingerprints (MemoMatrix class, memo-ms)
- Unaligned MEMO matrix built retention-time-agnostically from MS2 fingerprints
- Sample metadata mapping each sample to its acquisition batch (pre vs post 2017-10-26)

## Outputs

- PCoA sample coordinate tables for all three dissimilarity matrices
- Side-by-side ordination figure pcoa_plant_extracts.png comparing feature-table vs MEMO-based PCoAs
- Batch-annotated ordination showing strong pre/post 2017-10-26 separation for the feature table and mitigation for both MEMO variants

## How to apply

Install memo-ms in a dedicated conda environment (python=3.8 per the article) alongside numpy and scikit-bio, then load the plant extract demo feature table plus the aligned and unaligned MEMO matrices built from MS2 fingerprints via the MemoMatrix/SpectraDocuments classes. Compute three pairwise dissimilarity matrices: one from the feature table, one from aligned MS2 fingerprints (which requires a prior feature alignment step), and one from unaligned fingerprints computed retention-time-agnostically. Run classical multidimensional scaling on each with scikit-bio's PCoA implementation, feeding numpy arrays, and project each sample onto the first PCoA axes, annotating samples by acquisition batch (pre 2017-10-03 vs post 2017-10-26). Optionally filter fingerprints first (e.g., remove peaks/losses from blanks), export the PCoA sample coordinates as a table, and render the figures with memo-ms's visualization module into pcoa_plant_extracts.png for side-by-side comparison with Supplementary Figure S3.

## Related tools

- **memo-ms** (Provides MemoMatrix/SpectraDocuments classes to build aligned and unaligned MS2 fingerprint matrices and a visualization module for rendering the ordinations) — https://github.com/mandelbrot-project/memo_publication_examples
- **scikit-bio** (Runs classical multidimensional scaling (PCoA) on each dissimilarity matrix, with numpy arrays as input)
- **numpy** (Holds the pairwise dissimilarity matrices as arrays passed into the scikit-bio PCoA call)
- **matchms** (Underlying package memo-ms uses for handling the MS2 spectra from which fingerprints are counted) — https://github.com/matchms/matchms

## Examples

```
conda create --name memo python=3.8 && conda activate memo && pip install numpy memo-ms && conda install -c conda-forge scikit-bio
```

## Evaluation signals

- Feature-table PCoA reproduces the reported batch split: samples before 2017-10-03 cluster apart from samples after 2017-10-26, driven by RT shift and MS sensitivity change
- Both MEMO-based PCoAs (aligned and unaligned fingerprints) show visibly reduced batch-driven clustering compared with the feature-table ordination, matching Supplementary Figure S3
- Each dissimilarity matrix is passed to scikit-bio PCoA as a numpy array and the sample coordinates export cleanly to a table
- The rendered figure is named pcoa_plant_extracts.png and permits side-by-side comparison of the three ordinations colored by acquisition batch

## Limitations

- The aligned MEMO variant requires a prior feature alignment step; the unaligned variant avoids this but is retention-time agnostic by construction
- Earlier memo versions used the MemoContainer class instead of MemoMatrix and required python >3.8 (later lowered to >3.7 for TMAP compatibility), so API and environment details vary by version
- Fingerprints may need filtering first (e.g., remove peaks/losses from blanks); a changelog notes filtering in classes.py was changed to avoid erasing data

## Evidence

- [abstract] such performance was achieved without the requirement of a prior feature alignment step and in a significantly shorter computational time: "such performance was achieved without the requirement of a prior feature alignment step and in a significantly shorter computational time"
- [abstract] we introduce MEMO— M S2 Bas E d Sa M ple Vect O rization—a method allowing to cluster large amounts of chemodiverse samples based on their LC-MS/MS profiles: "we introduce MEMO— M S2 Bas E d Sa M ple Vect O rization—a method allowing to cluster large amounts of chemodiverse samples based on their LC-MS/MS profiles"
- [methods] different filtering (remove peaks/losses from blanks for example) and visualization techniques (MDS/PCoA, TMAP, Heatmap, ...) can be used.: "different filtering (remove peaks/losses from
blanks for example) and visualization techniques (MDS/PCoA, TMAP,
Heatmap, ...) can be used."
- [methods] conda create --name memo python=3.8: "conda create --name memo python=3.8"
- [readme] This repository contains code to reproduce MEMO analyses.: [evidence span withheld — anchor status: not_found]
