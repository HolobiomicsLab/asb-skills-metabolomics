---
name: memo-matrix-alignment-across-instrument-platforms
description: Use when samples were acquired on different LC-MS/MS platforms or LC
  methods, or with strong RT shift/poor feature overlap, and you need sample-level
  clustering without prior feature alignment.
when_to_use_negative:
- Input is already a fully aligned feature table from a single platform with good
  feature overlap — feature-table metrics (e.g., Bray-Curtis, or CSCS via q2-cscs)
  are more appropriate there
- The analysis requires retention-time-resolved information — MEMO is explicitly a
  retention time agnostic alignment and discards RT structure
- Only a handful of samples or a single instrument is involved — the cross-platform
  alignment machinery and platform-effect PERMANOVA add no value
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3174
tools:
- name: memo-ms
  role: Builds per-sample MS2 fingerprints (counts of MS2 peaks and neutral losses
    to the precursor) into a MemoMatrix and imports MGF/feature tables via memo_ms.import_data
  repo: https://github.com/mandelbrot-project/memo_publication_examples
- name: matchms
  role: Underlying package (with spec2vec) for handling MS2 spectra and converting
    them into documents
  repo: https://github.com/matchms/matchms
- name: spec2vec
  role: Underlying package used with matchms for handling the MS2 spectra and converting
    them into documents
  repo: https://github.com/iomega/spec2vec
- name: scikit-bio
  role: Provides PCoA ordination and PERMANOVA (pseudo-F, p-values with 999+ permutations)
    on the MEMO distance matrix
- name: numpy
  role: Numerical manipulation of the aligned/normalized MEMO matrices
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.3389/fbinf.2022.842964
    title: 'MEMO: Mass Spectrometry-Based Sample Vectorization to Explore Chemodiverse
      Datasets'
  source_content_hash: sha256:b4d0ea00c73e699cc256d563f5567a95a6a6db91f26e875e5db657a963ea58c1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mandelbrot-project/memo_publication_examples
  tools:
  - matchms
  - memo-ms
  - numpy
  - scikit-bio
  - spec2vec
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# memo-matrix-alignment-across-instrument-platforms

## Summary

Vectorize metabolomics samples as MS2 peak/neutral-loss count fingerprints (MS2 fingerprints) with memo-ms, align the resulting MemoMatrix objects across acquisitions made on different instruments (e.g., Q-Exactive Orbitrap vs Maxis II Q-TOF), and validate cross-platform comparability via PCoA and PERMANOVA on the MEMO distance matrix.

## When to use

Use when samples were acquired on different LC-MS/MS platforms or LC methods, or with strong RT shift/poor feature overlap, and you need sample-level clustering without prior feature alignment.

## When NOT to use

- Input is already a fully aligned feature table from a single platform with good feature overlap — feature-table metrics (e.g., Bray-Curtis, or CSCS via q2-cscs) are more appropriate there
- The analysis requires retention-time-resolved information — MEMO is explicitly a retention time agnostic alignment and discards RT structure
- Only a handful of samples or a single instrument is involved — the cross-platform alignment machinery and platform-effect PERMANOVA add no value

## Inputs

- MS2 peak list files (MGF) for both platforms (Q-Exactive Orbitrap and Maxis II Q-TOF), e.g. from MassIVE dataset MSV000083306
- Feature quantification tables (sample x feature) for both platforms
- Sample metadata labels: experiment (platform) and composition annotations for PERMANOVA grouping

## Outputs

- Cross-platform-aligned, sample-wise-normalized MEMO matrix (samples x shared MS2 peak/loss keys)
- Pairwise MEMO distance/similarity matrix between samples
- PCoA ordination coordinates colored by experiment and by composition
- PERMANOVA results (pseudo-F and p-values) written to pseudo_f_verification.csv

## How to apply

Retrieve the MS2 peak lists (MGF) and feature quantification tables from MassIVE (here MSV000083306) and load them with memo_ms.import_data into SpectraDocuments and FeatureTable objects. For each platform, count the occurrence of MS2 peaks and neutral losses (to the precursor) per sample to build MS2 fingerprints and aggregate them into a MemoMatrix. Align the platform-specific matrices on their shared peak/loss keys and apply sample-wise normalization so Q-Exactive and Q-TOF matrices become comparable — MEMO requires no prior feature alignment step. Compute the pairwise MEMO similarity/distance matrix, ordinate samples with PCoA (scikit-bio), coloring by experiment and by composition, and run scikit-bio PERMANOVA twice (experiment labels, composition labels) with 999+ permutations. Filter fingerprints (e.g., remove peaks/losses from blanks) before ordination if blank contamination is present.

## Related tools

- **memo-ms** (Builds per-sample MS2 fingerprints (counts of MS2 peaks and neutral losses to the precursor) into a MemoMatrix and imports MGF/feature tables via memo_ms.import_data) — https://github.com/mandelbrot-project/memo_publication_examples
- **matchms** (Underlying package (with spec2vec) for handling MS2 spectra and converting them into documents) — https://github.com/matchms/matchms
- **spec2vec** (Underlying package used with matchms for handling the MS2 spectra and converting them into documents) — https://github.com/iomega/spec2vec
- **scikit-bio** (Provides PCoA ordination and PERMANOVA (pseudo-F, p-values with 999+ permutations) on the MEMO distance matrix)
- **numpy** (Numerical manipulation of the aligned/normalized MEMO matrices)

## Examples

```
conda create --name memo python=3.8; conda activate memo; pip install memo-ms numpy; conda install -c conda-forge scikit-bio
```

## Evaluation signals

- PERMANOVA pseudo-F for the experiment (platform) factor reproduces 213.35 with p = 0.001 (999+ permutations)
- PERMANOVA pseudo-F for composition reproduces 5.19 with p = 0.001, resolved on PC2/PC3 of the PCoA
- The aligned matrix contains only peak/loss keys shared between both platforms after cross-platform key alignment
- Each sample row is sample-wise normalized so Q-Exactive and Q-TOF rows are on comparable scales
- Ordination output coordinates and PERMANOVA statistics land in pseudo_f_verification.csv matching the reported values

## Limitations

- MEMO is retention time agnostic, so RT-based quality checks or RT-separated isomer distinctions are out of scope
- Cross-platform comparability depends on shared peak/loss keys between platforms; very chemodiverse samples with poor overlap reduce effective alignment
- Blank-derived peaks/losses must be filtered (e.g., remove peaks/losses from blanks) before ordination or they will bias the fingerprints

## Evidence

- [abstract] cross-platform comparison validated: "MEMO thus allows the comparison of vast ensembles of samples, even when analyzed over long periods of time, and on different chromatographic or mass spectrometry platforms."
- [methods] fingerprint construction: "The occurrence of MS2 peaks and neutral losses (to the precursor) in each sample is counted
and used to generate an *MS2 fingerprint* of the sample"
- [intro] platform scope: [evidence span withheld — anchor status: not_found]
- [methods] alignment step: "These
fingerprints can in a second stage be aligned to compare different
samples."
- [methods] environment setup: "conda create --name memo python=3.8
   conda activate memo"
- [readme] visualization utilities: "different filtering (remove peaks/losses from
blanks for example) and visualization techniques (MDS/PCoA, TMAP,
Heatmap, ...) can be used."
- [methods] tooling basis: "MEMO is mainly built on `matchms`_ and `spec2vec`_ packages for handling
the MS2 spectra"
