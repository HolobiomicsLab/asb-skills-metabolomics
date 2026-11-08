---
name: sample-wise-matrix-normalization
description: Use when MEMO MS2 fingerprint matrices built from different MS platforms
  or acquisition batches must be made comparable prior to distance computation, PCoA
  ordination, or PERMANOVA testing.
when_to_use_negative:
- Input is a raw MGF or feature quantification table that has not yet been converted
  into MS2 fingerprints and aligned on shared peak/loss keys — build the MemoMatrix
  first.
- Comparison requires retention-time-dependent or feature-alignment-based metrics
  rather than retention time agnostic MS2 fingerprint counts.
- The dataset consists of a single platform with no cross-platform or cross-batch
  comparability requirement, where simpler within-platform normalization may suffice.
edam_operation: http://edamontology.org/operation_3436
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: memo-ms
  role: Builds per-sample MS2 fingerprints (peak and neutral-loss counts) and aggregates
    them into a MemoMatrix; its aligned matrix is the object being sample-wise normalized
  repo: https://github.com/mandelbrot-project/memo_publication_examples
- name: numpy
  role: Performs the sample-wise (column-wise) division of the aligned MEMO matrix
    counts
- name: scikit-bio
  role: Consumes the normalized MEMO distance matrix for PCoA ordination and PERMANOVA
    pseudo-F testing
- name: matchms
  role: Underlying package used by MEMO for handling MS2 spectra and converting them
    into documents
  repo: https://github.com/matchms/matchms
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.3389/fbinf.2022.842964
    title: 'MEMO: Mass Spectrometry-Based Sample Vectorization to Explore Chemodiverse
      Datasets'
  source_content_hash: sha256:e968b284c519a1a6fe4864b7017651ff1aa5e3b45c0810f3763c4d15a7ed02f5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mandelbrot-project/memo_publication_examples
  tools:
  - matchms
  - memo-ms
  - numpy
  - scikit-bio
  edam_operation: http://edamontology.org/operation_3436
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# sample-wise-matrix-normalization

## Summary

Sample-wise normalization scales aligned MEMO MS2 fingerprint matrices (counts of MS2 peaks and neutral losses to the precursor per sample) so each sample contributes comparable signal. This is essential when comparing samples acquired on different MS platforms (Q-Exactive Orbitrap vs Maxis II Q-TOF) before computing MEMO distances for PCoA and PERMANOVA.

## When to use

Use when MEMO MS2 fingerprint matrices built from different MS platforms or acquisition batches must be made comparable prior to distance computation, PCoA ordination, or PERMANOVA testing.

## When NOT to use

- Input is a raw MGF or feature quantification table that has not yet been converted into MS2 fingerprints and aligned on shared peak/loss keys — build the MemoMatrix first.
- Comparison requires retention-time-dependent or feature-alignment-based metrics rather than retention time agnostic MS2 fingerprint counts.
- The dataset consists of a single platform with no cross-platform or cross-batch comparability requirement, where simpler within-platform normalization may suffice.

## Inputs

- Aligned MEMO matrix (shared MS2 peak/neutral-loss keys x samples) built from MS2 peak lists (MGF) via memo-ms
- Platform-specific MemoMatrix objects (e.g. Q-Exactive Orbitrap and Maxis II Q-TOF acquisitions of MSV000083306)
- Sample metadata labels for experiment (platform) and composition used downstream in PERMANOVA

## Outputs

- Sample-wise-normalized aligned MEMO matrix (each sample's counts scaled to a common total)
- Pairwise MEMO distance/similarity matrix suitable for PCoA and PERMANOVA
- Ordination coordinates and PERMANOVA pseudo-F statistics written to pseudo_f_verification.csv

## How to apply

First build per-sample MS2 fingerprints by counting the occurrence of MS2 peaks and neutral losses to the precursor with memo-ms, then align the platform-specific MEMO matrices on their shared peak/loss keys. Apply normalization sample-wise, i.e. divide each sample's fingerprint counts by that sample's total count (column-wise division of the aligned matrix, e.g. with numpy/pandas) so samples with different total spectral content become comparable. After normalization, compute the pairwise MEMO distance/similarity matrix and ordinate with scikit-bio PCoA, coloring by experiment and composition. Finally run PERMANOVA (999+ permutations) with experiment labels and composition labels; for MSV000083306 the expected results are pseudo-F = 213.35 for the platform (experiment) effect and pseudo-F = 5.19 for composition, both with p = 0.001, confirming the normalized cross-platform matrix resolves platform dominance on PC1 and composition on PC2/PC3.

## Related tools

- **memo-ms** (Builds per-sample MS2 fingerprints (peak and neutral-loss counts) and aggregates them into a MemoMatrix; its aligned matrix is the object being sample-wise normalized) — https://github.com/mandelbrot-project/memo_publication_examples
- **numpy** (Performs the sample-wise (column-wise) division of the aligned MEMO matrix counts)
- **scikit-bio** (Consumes the normalized MEMO distance matrix for PCoA ordination and PERMANOVA pseudo-F testing)
- **matchms** (Underlying package used by MEMO for handling MS2 spectra and converting them into documents) — https://github.com/matchms/matchms

## Examples

```
memo_aligned_norm = memo_aligned.div(memo_aligned.sum(axis=0), axis=1)  # sample-wise normalize aligned MEMO matrix before PCoA/PERMANOVA
```

## Evaluation signals

- After normalization, each sample column of the aligned MEMO matrix sums to the same total (e.g. 1), verified with a column-sum check.
- PERMANOVA on the normalized MEMO distances reproduces pseudo-F = 213.35 (experiment/platform effect) and pseudo-F = 5.19 (composition) with p = 0.001 using 999+ permutations.
- PCoA of the normalized matrix is dominated by MS platform along the first axis and resolves composition on PC2/PC3, matching the published Figure 3 pattern.
- The distance matrix is symmetric with a zero diagonal and covers all samples from both platforms (Q-Exactive Orbitrap and Maxis II Q-TOF).

## Limitations

- The article's method counts raw occurrences of MS2 peaks and neutral losses; without sample-wise normalization these raw counts are biased by platform-dependent spectral content and sample size, so unnormalized cross-platform comparison is unreliable.
- Filtering decisions (e.g. removing peaks/losses from blanks) alter fingerprint content and must be applied consistently across platforms before normalization; a memo release note reports 'Changed classes.py to avoid erasing when filtering', indicating filtering behavior is version-sensitive.
- The alignment step requires shared peak/loss keys between platforms; chemodiverse samples with poor feature overlap may yield sparse aligned matrices.

## Evidence

- [intro] fingerprints counted per sample then aligned: "The occurence of MS2 peaks and neutral losses (to the precursor) in each sample is counted and used to generate an *MS2 fingerprint* of the sample. These fingerprints can in a second stage be aligned"
- [abstract] MEMO enables cross-platform comparison: "MEMO thus allows the comparison of vast ensembles of samples, even when analyzed over long periods of time, and on different chromatographic or mass spectrometry platforms."
- [intro] filtering and visualization of aligned samples: "different filtering (remove peaks/losses from
blanks for example) and visualization techniques (MDS/PCoA, TMAP,
Heatmap, ...) can be used."
- [readme] README fingerprint definition: "The occurence of MS2 peaks and neutral losses (to the precursor) in each sample is counted
and used to generate an *MS2 fingerprint* of the sample"
- [intro] MEMO suits cross-instrument comparison: "MEMO suits particularly well to compare chemodiverse samples, ie with a
poor features overlap, or to compare samples with a strong RT shift,
acquired using different LC methods or even different mass"
