---
name: batch-effect-assessment-in-lc-msms-metabolomics
description: Use when LC-MS/MS samples were acquired in multiple batches (e.g. before
  vs after a ~2017-10-26 instrument date) and you need to test whether a batch split
  in PCoA is driven by feature-table metrics and whether MS2-fingerprint-based MEMO
  dissimilarities reduce it.
when_to_use_negative:
- Samples were all acquired in a single batch with no RT shift or MS sensitivity change
  — no batch effect to assess and MEMO offers no advantage
- The analysis goal requires retention-time-resolved feature comparison rather than
  retention-time-agnostic sample clustering
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: memo-ms
  role: Generates MemoMatrix/SpectraDocuments MS2 fingerprints (aligned and unaligned)
    and provides the visualization module for ordination figures
  repo: https://github.com/mandelbrot-project/memo_publication_examples
- name: scikit-bio
  role: PCoA (classical multidimensional scaling) implementation run on numpy-array
    dissimilarity matrices
- name: numpy
  role: Array format for dissimilarity matrices passed into scikit-bio PCoA
- name: matchms
  role: Underlying package memo uses for handling the MS2 spectra
  repo: https://github.com/matchms/matchms
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.3389/fbinf.2022.842964
    title: 'MEMO: Mass Spectrometry-Based Sample Vectorization to Explore Chemodiverse
      Datasets'
  source_content_hash: sha256:1c5f7deb380b4a1bd7053b27e9907b1afed52263c309f5d722868b2227ebac68
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mandelbrot-project/memo_publication_examples
  tools:
  - matchms
  - memo-ms
  - numpy
  - scikit-bio
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# batch-effect-assessment-in-lc-msms-metabolomics

## Summary

Assess LC-MS/MS acquisition batch effects by running PCoA (classical MDS) on feature-table dissimilarities versus MEMO MS2-fingerprint dissimilarities, revealing RT-shift/MS-sensitivity drift that MEMO mitigates in a retention-time-agnostic manner.

## When to use

Use when LC-MS/MS samples were acquired in multiple batches (e.g. before vs after a ~2017-10-26 instrument date) and you need to test whether a batch split in PCoA is driven by feature-table metrics and whether MS2-fingerprint-based MEMO dissimilarities reduce it.

## When NOT to use

- Samples were all acquired in a single batch with no RT shift or MS sensitivity change — no batch effect to assess and MEMO offers no advantage
- The analysis goal requires retention-time-resolved feature comparison rather than retention-time-agnostic sample clustering

## Inputs

- Plant extract feature table (memo-ready) from LC-MS/MS acquisition in dated batches
- Aligned MEMO MS2 fingerprint matrix (requires prior feature alignment step)
- Unaligned MEMO MS2 fingerprint matrix (retention-time-agnostic, no feature alignment)
- Sample metadata with acquisition dates/batch labels (pre vs post 2017-10-26)

## Outputs

- Three pairwise dissimilarity matrices (feature-table, aligned MEMO, unaligned MEMO)
- PCoA sample coordinate tables for each dissimilarity matrix
- Side-by-side ordination figure (pcoa_plant_extracts.png) annotated by acquisition batch

## How to apply

Load the plant extract demo feature table plus aligned and unaligned MEMO matrices generated with memo-ms (MemoMatrix/SpectraDocuments classes); the unaligned variant is computed retention-time-agnostically and needs no prior feature alignment step. Compute pairwise dissimilarity matrices for the feature-table-based metric and for both MEMO variants, then run classical multidimensional scaling (PCoA) on each matrix with scikit-bio's PCoA implementation using numpy arrays. Project samples onto the first PCoA axes and color/annotate by acquisition batch (pre vs post 2017-10-26). The rationale, per the article, is that MEMO aligns samples via MS2 peaks and neutral-loss counts rather than RT-positioned features, so it suits datasets with strong RT shifts or changing MS sensitivity; export PCoA coordinates as a table and render ordinations with memo-ms's visualization module (pcoa_plant_extracts.png) for side-by-side comparison with Supplementary Figure S3.

## Related tools

- **memo-ms** (Generates MemoMatrix/SpectraDocuments MS2 fingerprints (aligned and unaligned) and provides the visualization module for ordination figures) — https://github.com/mandelbrot-project/memo_publication_examples
- **scikit-bio** (PCoA (classical multidimensional scaling) implementation run on numpy-array dissimilarity matrices)
- **numpy** (Array format for dissimilarity matrices passed into scikit-bio PCoA)
- **matchms** (Underlying package memo uses for handling the MS2 spectra) — https://github.com/matchms/matchms

## Examples

```
conda create --name memo python=3.8 && conda activate memo && pip install memo-ms numpy && conda install -c conda-forge scikit-bio
```

## Evaluation signals

- Feature-table PCoA reproduces the reported batch split: samples acquired before 2017-10-03 separate from those after 2017-10-26 (RT shift and MS sensitivity change)
- Both MEMO-based PCoAs (aligned and unaligned fingerprints) show reduced batch-driven clustering compared with the feature-table PCoA
- The reproduced pattern matches Supplementary Figure S3 of the MEMO paper when compared side-by-side
- PCoA coordinate tables export cleanly and each ordination is computed on the same sample set (consistent matrix dimensions across the three metrics)

## Limitations

- Aligned MEMO fingerprints require a prior feature alignment step; the unaligned variant avoids this but is the retention-time-agnostic choice
- Filtering of fingerprints (e.g. removing peaks/losses from blanks) may be needed and historically caused data erasure bugs in memo's classes.py filtering
- Batch-split conclusions rest on visual ordination comparison with the paper's Supplementary Figure S3, not a statistical batch-effect test

## Evidence

- [methods] Feature-table PCoA splits pre/post 2017-10-26 batch; MEMO mitigates: "conda create --name memo python=3.8
   conda activate memo"
- [abstract] MEMO clusters retention-time-agnostically across long periods: "we introduce MEMO— M S2 Bas E d Sa M ple Vect O rization—a method allowing to cluster large amounts of chemodiverse samples based on their LC-MS/MS profiles in a retention time agnostic manner"
- [intro] Fingerprint counting and alignment workflow: "The occurence of MS2 peaks and neutral losses (to the precursor) in each sample is counted and used to generate an *MS2 fingerprint* of the sample. These fingerprints can in a second stage be aligned"
- [methods] Visualization via PCoA/MDS on fingerprints: "visualization techniques (MDS/PCoA, TMAP,
Heatmap, ...) can be used"
- [readme] README: MEMO suits chemodiverse samples with RT shift across platforms: "MEMO suits particularly well to compare chemodiverse samples, ie with a
poor features overlap, or to compare samples"
