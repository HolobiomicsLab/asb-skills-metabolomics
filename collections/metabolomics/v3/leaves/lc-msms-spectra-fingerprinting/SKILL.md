---
name: lc-msms-spectra-fingerprinting
description: Use when comparing chemodiverse metabolomics samples with poor feature
  overlap, strong retention time shifts, or data acquired across different LC methods
  or mass spectrometers (Q-ToF vs Orbitrap), where feature alignment is impractical.
when_to_use_negative:
- When a retention-time-dependent feature alignment is already available and sample
  overlap is high — MEMO's advantage lies in chemodiverse samples with poor features
  overlap or strong RT shift
- When the analysis requires mass spectral structural relatedness such as cosine-score-based
  molecular networking distances (e.g. the CSCS metric), rather than peak/loss occurrence
  counting
edam_operation: http://edamontology.org/operation_2929
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: memo-ms
  role: 'Core package: SpectraDocuments import of MS2 spectra and MemoMatrix fingerprint
    counting/alignment (installed with pip install memo-ms, version 0.1.3)'
  repo: https://github.com/mandelbrot-project/memo_publication_examples
- name: matchms
  role: Backend library used by MEMO for handling the MS2 spectra
  repo: https://github.com/matchms/matchms
- name: spec2vec
  role: Backend library used by MEMO for converting MS2 spectra into documents
  repo: https://github.com/iomega/spec2vec
- name: numpy
  role: Dependency installed alongside memo-ms in the conda environment
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.3389/fbinf.2022.842964
    title: 'MEMO: Mass Spectrometry-Based Sample Vectorization to Explore Chemodiverse
      Datasets'
  source_content_hash: sha256:d478fd8f862e47aa99518ed03b28edfdca838b849460fb567fce7a6c900d610b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mandelbrot-project/memo_publication_examples
  tools:
  - Python
  - matchms
  - memo-ms
  - numpy
  - spec2vec
  edam_operation: http://edamontology.org/operation_2929
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# lc-msms-spectra-fingerprinting

## Summary

Vectorize LC-MS/MS metabolomics samples into MS2 fingerprints by counting the occurrence of MS2 peaks and neutral losses (to the precursor) per sample, then aligning the fingerprints to obtain a sample-by-sample comparison matrix without any prior feature alignment step.

## When to use

Use when comparing chemodiverse metabolomics samples with poor feature overlap, strong retention time shifts, or data acquired across different LC methods or mass spectrometers (Q-ToF vs Orbitrap), where feature alignment is impractical.

## When NOT to use

- When a retention-time-dependent feature alignment is already available and sample overlap is high — MEMO's advantage lies in chemodiverse samples with poor features overlap or strong RT shift
- When the analysis requires mass spectral structural relatedness such as cosine-score-based molecular networking distances (e.g. the CSCS metric), rather than peak/loss occurrence counting

## Inputs

- MS2 peak list files from an LC-MS/MS run (e.g. MassIVE accession MSV000087728: 1,920 samples / 906,509 MS/MS spectra)
- No prior feature-aligned feature table required — raw MS2 peak lists are sufficient

## Outputs

- Aligned sample-by-sample MEMO matrix (e.g. memo_matrix.csv; 1,920 x 1,920 for the plant extract dataset)
- Per-sample MS2 fingerprints (counts of MS2 peaks and neutral losses to the precursor)

## How to apply

Import the MS2 peak lists (e.g. from a MassIVE accession such as MSV000087728) using memo_ms import_data/SpectraDocuments, which is built on matchms and spec2vec for handling the MS2 spectra — no prior feature alignment is required. For each sample, count the occurrence of MS2 peaks and neutral losses (to the precursor) to generate an MS2 fingerprint. Align these fingerprints across all samples into a MemoMatrix (the MemoMatrix class replaces the former MemoContainer) whose entries compare samples pairwise. Optionally filter the aligned fingerprints (e.g. remove peaks/losses from blanks) before visualization via MDS/PCoA, TMAP, or heatmap. Export the aligned matrix (e.g. memo_matrix.csv) and verify its dimensions match the sample count; at the reported scale of 1,920 samples and 906,509 MS/MS spectra the matrix is 1,920 x 1,920 and computes in under 10 min on a simple laptop (memo-ms 0.1.3, Python 3.8).

## Related tools

- **memo-ms** (Core package: SpectraDocuments import of MS2 spectra and MemoMatrix fingerprint counting/alignment (installed with pip install memo-ms, version 0.1.3)) — https://github.com/mandelbrot-project/memo_publication_examples
- **matchms** (Backend library used by MEMO for handling the MS2 spectra) — https://github.com/matchms/matchms
- **spec2vec** (Backend library used by MEMO for converting MS2 spectra into documents) — https://github.com/iomega/spec2vec
- **numpy** (Dependency installed alongside memo-ms in the conda environment)

## Examples

```
conda create --name memo python=3.8; conda activate memo; pip install numpy; pip install memo-ms
```

## Evaluation signals

- memo_matrix.csv exists and is a square sample-by-sample matrix with 1,920 rows matching the reported sample count
- computation_time.txt contains the wall-clock runtime of the matrix-building step; reported benchmark is <10 min for 1,920 samples / 906,509 MS/MS spectra on a simple laptop
- Fingerprints are derived without any prior feature alignment step (the import accepts MS2 peak lists directly)
- Matrix dimensions are N x N where N equals the number of input samples (e.g. 1,920 x 1,920)
- Clustering/ordination of the aligned fingerprints should show performance comparable to state-of-the-art metrics that consider fragmentation spectra

## Limitations

- MEMO counts peak/loss occurrences only — it is retention time agnostic, so it will not capture RT-based information; suited to chemodiverse samples rather than tightly overlapping features
- memo-ms requires a dedicated conda environment (Python 3.8; later relaxed to >3.7 for TMAP compatibility) to avoid dependency clashes
- Filtering in classes.py previously risked erasing data when filtering (fixed in a code change); filter steps such as blank removal must be applied with care

## Evidence

- [methods] fingerprint counting method: "The occurrence of MS2 peaks and neutral losses (to the precursor) in each sample is counted and used to generate an *MS2 fingerprint* of the sample"
- [abstract] no feature alignment needed: "such performance was achieved without the requirement of a prior feature alignment step and in a significantly shorter computational time"
- [readme] matched backend libraries: "MEMO is mainly built on `matchms`_ and `spec2vec`_ packages for handling the MS2 spectra and converting them into documents"
- [intro] filtering and visualization options: "different filtering (remove peaks/losses from
blanks for example) and visualization techniques (MDS/PCoA, TMAP,
Heatmap, ...) can be used"
- [readme] environment setup: "conda create --name memo python=3.8"
- [intro] applicability scope: "MEMO suits particularly well to compare chemodiverse samples, ie with a
poor features overlap, or to compare samples with a strong RT shift"
