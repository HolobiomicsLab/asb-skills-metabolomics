---
name: metabolomics-sample-vectorization
description: Use when comparing large ensembles of LC-MS/MS samples (e.g. 1,920 plant
  extracts / 906,509 MS/MS spectra) that are chemodiverse, have strong RT shifts,
  or were acquired across different LC or MS platforms (Q-ToF vs Orbitrap).
when_to_use_negative:
- The analysis requires retention-time-aware sample comparison — MEMO is deliberately
  RT-agnostic and ignores chromatographic alignment.
- The dataset contains only MS1 features without MS2 fragmentation spectra — counting
  MS2 peaks and neutral losses to the precursor is impossible.
- The input is already a feature-aligned table from another pipeline — MEMO's stated
  advantage is that it operates directly on MS2 spectra without a prior feature alignment
  step, so re-running alignment is redundant.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: memo-ms
  role: Core package implementing MS2 fingerprint generation, MemoMatrix construction,
    and fingerprint alignment (built on matchms and spec2vec); installed via pip inside
    a Python 3.8 conda environment
  repo: https://github.com/mandelbrot-project/memo
- name: matchms
  role: Underlying library for importing and handling the MS2 spectra before fingerprint
    counting
  repo: https://github.com/matchms/matchms
- name: spec2vec
  role: Underlying library for converting MS2 spectra into documents used by memo_ms
    SpectraDocuments
  repo: https://github.com/iomega/spec2vec
- name: numpy
  role: Numerical dependency installed alongside memo-ms in the memo conda environment
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.3389/fbinf.2022.842964
    title: 'MEMO: Mass Spectrometry-Based Sample Vectorization to Explore Chemodiverse
      Datasets'
  source_content_hash: sha256:8a6dbb69a4482ffb7266ef28e87ca62e8131e10e4c89054b654da199831c4b02
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mandelbrot-project/memo
  tools:
  - Python
  - matchms
  - memo-ms
  - numpy
  - spec2vec
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# metabolomics-sample-vectorization

## Summary

Vectorizes metabolomics samples into MS2 fingerprints by counting MS2 peaks and neutral losses (to the precursor) per sample, then aligns these fingerprints sample-by-sample to enable RT-agnostic clustering of chemodiverse LC-MS/MS datasets without prior feature alignment.

## When to use

Use when comparing large ensembles of LC-MS/MS samples (e.g. 1,920 plant extracts / 906,509 MS/MS spectra) that are chemodiverse, have strong RT shifts, or were acquired across different LC or MS platforms (Q-ToF vs Orbitrap).

## When NOT to use

- The analysis requires retention-time-aware sample comparison — MEMO is deliberately RT-agnostic and ignores chromatographic alignment.
- The dataset contains only MS1 features without MS2 fragmentation spectra — counting MS2 peaks and neutral losses to the precursor is impossible.
- The input is already a feature-aligned table from another pipeline — MEMO's stated advantage is that it operates directly on MS2 spectra without a prior feature alignment step, so re-running alignment is redundant.

## Inputs

- MS2 peak list files per sample (e.g. downloaded from MassIVE accession MSV000087728 — 1,920 plant extract samples / 906,509 MS/MS spectra)
- Sample-level metadata for fingerprint alignment across batches (e.g. 19 acquisition batches profiled over a year)
- Python 3.8 conda environment with memo-ms 0.1.3, numpy, matchms, spec2vec

## Outputs

- Aligned MEMO fingerprint matrix exported as memo_matrix.csv (square sample-by-sample matrix, 1,920 x 1,920 for the plant dataset)
- computation_time.txt recording the wall-clock runtime of the matrix-building step
- Optionally filtered fingerprints (e.g. blanks removed) and visualizations (MDS/PCoA, TMAP, Heatmap)

## How to apply

Create a dedicated conda environment (`conda create --name memo python=3.8`) and install memo-ms 0.1.3 via pip; MEMO is built on matchms and spec2vec for handling the MS2 spectra. Load the MS2 peak lists through memo_ms import_data/SpectraDocuments directly from raw MS2 peak list files (e.g. from MassIVE accession MSV000087728), with no prior feature alignment step. Build the MemoMatrix (which replaces the former MemoContainer) by counting the occurrence of MS2 peaks and neutral losses to the precursor in each sample, generating an MS2 fingerprint per sample, then align fingerprints across all samples to produce the sample-by-sample matrix. After alignment, optional filtering (e.g. remove peaks/losses from blanks) and visualization (MDS/PCoA, TMAP, Heatmap) can be applied. Wall-clock time the matrix-building step: for the 1,920-sample plant dataset this ran in less than 10 min on a simple laptop.

## Related tools

- **memo-ms** (Core package implementing MS2 fingerprint generation, MemoMatrix construction, and fingerprint alignment (built on matchms and spec2vec); installed via pip inside a Python 3.8 conda environment) — https://github.com/mandelbrot-project/memo
- **matchms** (Underlying library for importing and handling the MS2 spectra before fingerprint counting) — https://github.com/matchms/matchms
- **spec2vec** (Underlying library for converting MS2 spectra into documents used by memo_ms SpectraDocuments) — https://github.com/iomega/spec2vec
- **numpy** (Numerical dependency installed alongside memo-ms in the memo conda environment)

## Examples

```
conda create --name memo python=3.8 && conda activate memo && pip install numpy && pip install memo-ms
```

## Evaluation signals

- memo_matrix.csv exists and is a square sample-by-sample matrix whose row count equals the reported sample count (1,920 rows for the plant extract dataset).
- computation_time.txt records a wall-clock runtime below the reported ~10 min for the 1,920-sample / 906,509-spectrum dataset on a laptop-class machine.
- Each sample's fingerprint contains counts of both MS2 peaks and neutral losses relative to the precursor, not peak intensities or retention times.
- No prior feature alignment step was run before fingerprint generation (a defining property of the method).
- Downstream clustering (e.g. MDS/PCoA) on the aligned matrix reproduces chemodiverse sample grouping consistent with state-of-the-art fragmentation-spectra metrics reported in the paper.

## Limitations

- MEMO is RT-agnostic by design, so it cannot capture retention-time-based discriminations; it is positioned for chemodiverse samples with poor feature overlap or strong RT shifts, not for RT-sensitive analyses.
- Package constraints must be respected: originally required Python >3.8, later lowered to >3.7 for TMAP compatibility, and filtering code in classes.py had a bug that erased data when filtering (fixed in 0.1.2-era changes).
- The <10 min runtime claim is hardware-dependent (reported on 'a simple laptop'); larger datasets may exceed it, so wall-clock timing should be recorded per run.

## Evidence

- [intro] The occurence of MS2 peaks and neutral losses (to the precursor) in each sample is counted and used to generate an *MS2 fingerprint* of the sample.: "The occurence of MS2 peaks and neutral losses (to the precursor) in each sample is counted and used to generate an *MS2 fingerprint* of the sample."
- [methods] MEMO is mainly built on `matchms`_ and `spec2vec`_ packages for handling the MS2 spectra: "MEMO is mainly built on `matchms`_ and `spec2vec`_ packages for handling the MS2 spectra"
- [methods] conda create --name memo python=3.8: "conda create --name memo python=3.8"
- [abstract] such performance was achieved without the requirement of a prior feature alignment step and in a significantly shorter computational time: "such performance was achieved without the requirement of a prior feature alignment step and in a significantly shorter computational time"
- [readme] These fingerprints can in a second stage be aligned to compare different samples.: "These fingerprints can in a second stage be aligned to compare different samples."
- [abstract] a method allowing to cluster large amounts of chemodiverse samples based on their LC-MS/MS profiles in a retention time agnostic manner: "a method allowing to cluster large amounts of chemodiverse samples based on their LC-MS/MS profiles in a retention time agnostic manner"
