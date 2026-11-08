---
name: ms2-fingerprint-generation
description: Use when LC-MS/MS samples must be compared in a retention-time-agnostic
  way, e.g. across chemodiverse samples, different LC methods, or different mass spectrometers
  (Q-Exactive Orbitrap vs Maxis II Q-TOF), from MS2 peak lists such as MGF.
when_to_use_negative:
- When the analysis depends on retention time — MEMO is explicitly a RT-agnostic alignment
  of metabolomics samples, so RT-based clustering is out of scope
- When samples already share a well-overlapping, pre-aligned feature table and a feature-based
  metric (e.g. Bray-Curtis) suffices; MEMO is aimed at chemodiverse samples with poor
  feature overlap
- When per-feature spectral similarity (e.g. cosine scores) rather than sample-level
  occurrence counts is required — use molecular networking instead
edam_operation: http://edamontology.org/operation_3661
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3318
tools:
- name: memo-ms
  role: Imports MGF/feature tables into SpectraDocuments and FeatureTable objects,
    counts MS2 peaks and neutral losses per sample, and aggregates fingerprints into
    a MemoMatrix
  repo: https://github.com/mandelbrot-project/memo_publication_examples
- name: matchms
  role: Underlying package used by MEMO for handling MS2 spectra
  repo: https://github.com/matchms/matchms
- name: spec2vec
  role: Package used by MEMO for converting MS2 spectra into documents from which
    peak/loss occurrences are counted
  repo: https://github.com/iomega/spec2vec
- name: numpy
  role: Numerical backend for the count matrices and subsequent alignment/normalization
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.3389/fbinf.2022.842964
    title: 'MEMO: Mass Spectrometry-Based Sample Vectorization to Explore Chemodiverse
      Datasets'
  source_content_hash: sha256:309bfdda21b929835fb8970a0a59a680c692c853124a29ca4004f75682c4fba3
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
  edam_operation: http://edamontology.org/operation_3661
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# ms2-fingerprint-generation

## Summary

Generates a sample-level MS2 fingerprint by counting the occurrence of MS2 fragment peaks and neutral losses (to the precursor) in each sample's fragmentation spectra, then aggregates them into an aligned MemoMatrix for sample vectorization (MEMO).

## When to use

Use when LC-MS/MS samples must be compared in a retention-time-agnostic way, e.g. across chemodiverse samples, different LC methods, or different mass spectrometers (Q-Exactive Orbitrap vs Maxis II Q-TOF), from MS2 peak lists such as MGF.

## When NOT to use

- When the analysis depends on retention time — MEMO is explicitly a RT-agnostic alignment of metabolomics samples, so RT-based clustering is out of scope
- When samples already share a well-overlapping, pre-aligned feature table and a feature-based metric (e.g. Bray-Curtis) suffices; MEMO is aimed at chemodiverse samples with poor feature overlap
- When per-feature spectral similarity (e.g. cosine scores) rather than sample-level occurrence counts is required — use molecular networking instead

## Inputs

- MS2 peak list files (e.g. MGF) from LC-MS/MS acquisitions
- Feature quantification tables for the same samples
- Precursor m/z values per feature (needed to compute neutral losses to the precursor)

## Outputs

- Per-sample MS2 fingerprint (count vector of MS2 peaks and neutral losses to the precursor)
- Aligned MemoMatrix (samples x aligned peak/loss keys), optionally sample-wise normalized for cross-platform comparability

## How to apply

Load MS2 peak lists (MGF) and feature quantification tables (e.g. from MassIVE accession MSV000083306) with memo_ms.import_data into SpectraDocuments and FeatureTable objects; matchms/spec2vec handle the MS2 spectra and convert them into documents. For each sample, count the occurrence of MS2 peaks and neutral losses to the precursor to build the sample's MS2 fingerprint, aggregating counts into a MemoMatrix (memo-ms >= 0.1.2 MemoMatrix class replaces the former MemoContainer). When fingerprints come from different acquisition platforms, align the matrices on their shared peak/loss keys and apply sample-wise normalization so the count matrices are comparable. The rationale is that counting peak/loss occurrences yields a sample vector without requiring a prior feature-alignment step or retention-time information, enabling comparison across platforms and over long time periods.

## Related tools

- **memo-ms** (Imports MGF/feature tables into SpectraDocuments and FeatureTable objects, counts MS2 peaks and neutral losses per sample, and aggregates fingerprints into a MemoMatrix) — https://github.com/mandelbrot-project/memo_publication_examples
- **matchms** (Underlying package used by MEMO for handling MS2 spectra) — https://github.com/matchms/matchms
- **spec2vec** (Package used by MEMO for converting MS2 spectra into documents from which peak/loss occurrences are counted) — https://github.com/iomega/spec2vec
- **numpy** (Numerical backend for the count matrices and subsequent alignment/normalization)

## Examples

```
conda create --name memo python=3.8; conda activate memo; pip install memo-ms
```

## Evaluation signals

- Each fingerprint is a count vector: values must be non-negative integers (or normalized counts), with one vector per sample
- After alignment, the matrix columns correspond exactly to the union of shared peak/loss keys across platforms; unmatched keys must be handled consistently
- Sample-wise normalization must be applied before cross-platform (Q-Exactive Orbitrap vs Maxis II Q-TOF) comparison so matrices are comparable
- Downstream sanity check: PCoA/PERMANOVA on MEMO distances should reproduce platform pseudo-F ≈ 213.35 and composition pseudo-F ≈ 5.19, p = 0.001, on MSV000083306
- Fingerprint generation must be retention-time independent: identical spectra with shifted RTs yield identical fingerprints

## Limitations

- Filtering is expected before/after vectorization (e.g. remove peaks/losses present in blanks); the memo-ms changelog notes a classes.py fix to avoid erasing data when filtering
- The 'memo ready' feature table import lacks a specified file format/dataset specification, so feature-table ingestion paths are underspecified
- Python version requirement was lowered from >3.8 to >3.7 for TMAP compatibility — pin the environment carefully (conda create --name memo python=3.8)

## Evidence

- [methods] count MS2 peaks and neutral losses to generate an MS2 fingerprint of the sample: "The occurrence of MS2 peaks and neutral losses (to the precursor) in each sample is counted
and used to generate an *MS2 fingerprint* of the sample"
- [methods] fingerprints aligned to compare samples: "These
fingerprints can in a second stage be aligned to compare different
samples."
- [methods] matchms and spec2vec handle spectra: "MEMO is mainly built on `matchms`_ and `spec2vec`_ packages for handling
the MS2 spectra"
- [discussion] MemoMatrix replaces MemoContainer: "New `MemoMatrix` class, replacing the former `MemoContainer`"
- [readme] RT-agnostic vectorization description in README: "The occurence of MS2 peaks and neutral losses (to the precursor) in each sample is counted
and used to generate an *MS2 fingerprint* of the sample."
