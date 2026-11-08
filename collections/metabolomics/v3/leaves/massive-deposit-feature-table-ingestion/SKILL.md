---
name: massive-deposit-feature-table-ingestion
description: Use when new samples (e.g. extracts profiled on a different LC gradient
  or MS instrument, such as 2014 vs 2018 Waltheria indica runs on QE-Plus vs QE-Focus)
  must be added to an existing MEMO matrix from publicly deposited MassIVE data.
when_to_use_negative:
- When a prior feature alignment step is expected/required — MEMO intentionally aligns
  fingerprints retention-time-agnostically without feature alignment
- When samples must be compared by retention-time-aligned features rather than MS2-peak/neutral-loss
  fingerprints (e.g. strong RT shift is actually the signal of interest)
- When only MS1-level feature intensities (no MS2 spectra) are available in the deposit,
  since fingerprints are built from MS2 peaks and neutral losses
edam_operation: http://edamontology.org/operation_2929
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: memo-ms
  role: Python package used to import MassIVE feature tables/spectra and align them
    into a MemoMatrix
  repo: https://github.com/mandelbrot-project/memo_publication_examples
- name: matchms
  role: Handling the MS2 spectra imported from the deposit before document conversion
  repo: https://github.com/matchms/matchms
- name: spec2vec
  role: Converting imported MS2 spectra into documents for fingerprint generation
  repo: https://github.com/iomega/spec2vec
- name: numpy
  role: Dependency installed alongside memo-ms in the dedicated environment
- name: scikit-bio
  role: Supporting ordination/visualization of the merged matrix (installed via conda-forge)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.3389/fbinf.2022.842964
    title: 'MEMO: Mass Spectrometry-Based Sample Vectorization to Explore Chemodiverse
      Datasets'
  source_content_hash: sha256:112ac271765bd859ae9864221d1667dffbf66b85d43209296139cc6de78c3f13
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
  edam_operation: http://edamontology.org/operation_2929
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# massive-deposit-feature-table-ingestion

## Summary

Ingest feature tables and MS2 spectra files from a MassIVE deposit (e.g. MSV000088521) into the MEMO sample-vectorization workflow using memo_ms.import_data, alongside existing plant dataset files, without any prior feature alignment step.

## When to use

Use when new samples (e.g. extracts profiled on a different LC gradient or MS instrument, such as 2014 vs 2018 Waltheria indica runs on QE-Plus vs QE-Focus) must be added to an existing MEMO matrix from publicly deposited MassIVE data.

## When NOT to use

- When a prior feature alignment step is expected/required — MEMO intentionally aligns fingerprints retention-time-agnostically without feature alignment
- When samples must be compared by retention-time-aligned features rather than MS2-peak/neutral-loss fingerprints (e.g. strong RT shift is actually the signal of interest)
- When only MS1-level feature intensities (no MS2 spectra) are available in the deposit, since fingerprints are built from MS2 peaks and neutral losses

## Inputs

- Feature tables and MS2 spectra files downloaded from a MassIVE accession (e.g. MSV000088521)
- Existing plant dataset feature tables/spectra files to merge with
- Blank files for optional blank filtering

## Outputs

- Merged MemoMatrix containing appended sample fingerprints (new sample columns/rows in the existing MEMO matrix)
- UMAP/TMAP embeddings showing positions of newly ingested samples relative to existing samples

## How to apply

Create a dedicated conda environment (conda create --name memo python=3.8, conda activate memo) and install numpy and memo-ms via pip. Download the feature tables/spectra files for the target samples from the MassIVE accession (e.g. MSV000088521: 2014 Waltheria indica aerial parts acquired on a Dionex Ultimate 3000 / Q-Exactive Plus with a modified 40 min gradient, plus the 2018 reprofiled roots and aerial parts on the Q-Exactive Focus with an 8 min gradient). Load these files with memo_ms.import_data together with the existing plant dataset files. Convert each sample's MS2 spectra into documents with matchms/spec2vec, counting MS2 peaks and neutral losses to the precursor to build SpectraDocuments MS2 fingerprints. Align the new fingerprints with existing fingerprints retention-time-agnostically via MemoMatrix, appending new sample columns/rows to the existing MEMO matrix; optionally apply blank filtering (removal of peaks/losses attributed to blanks) before merging, then project with UMAP/TMAP.

## Related tools

- **memo-ms** (Python package used to import MassIVE feature tables/spectra and align them into a MemoMatrix) — https://github.com/mandelbrot-project/memo_publication_examples
- **matchms** (Handling the MS2 spectra imported from the deposit before document conversion) — https://github.com/matchms/matchms
- **spec2vec** (Converting imported MS2 spectra into documents for fingerprint generation) — https://github.com/iomega/spec2vec
- **numpy** (Dependency installed alongside memo-ms in the dedicated environment)
- **scikit-bio** (Supporting ordination/visualization of the merged matrix (installed via conda-forge))

## Examples

```
conda create --name memo python=3.8; conda activate memo; pip install numpy; pip install memo-ms
```

## Evaluation signals

- The merged MEMO matrix contains the newly appended sample columns/rows (e.g. the three Waltheria indica samples from MSV000088521) alongside the original plant dataset samples
- Fingerprints count both MS2 peaks and neutral losses to the precursor for each ingested sample
- No retention-time-based feature alignment was performed before merging (alignment is RT-agnostic)
- Downstream UMAP/TMAP projection of the merged matrix renders without errors and positions ingested samples relative to existing ones (e.g. Waltheria indica 2014/2018 extracts clustering with the trypanocidal Melochia umbellata green stems extract despite 40 min vs 8 min gradients and QE-Plus vs QE-Focus instruments)
- The conda environment pins Python 3.8 and memo-ms installs successfully via pip

## Limitations

- Fingerprint import of 'memo ready' feature tables lacks file format/dataset specification (missing signal in the repo changelog)
- Python version compatibility matters: required version was changed from >3.8 to >3.7 for TMAP compatibility
- Filtering code was previously changed to avoid erasing data when filtering, so blank filtering must be applied carefully before merging
- Platform differences (different LC gradients and MS instruments) are tolerated but comparisons remain MS2-fingerprint-based rather than feature-level

## Evidence

- [methods] Loading MassIVE files with memo_ms.import_data: [evidence span withheld — anchor status: not_found]
- [methods] Install memo-ms with numpy via pip: "pip install numpy
   pip install memo-ms"
- [intro] Fingerprint generation from MS2 peaks and neutral losses: "The occurence of MS2 peaks and neutral losses (to the precursor) in each sample is counted and used to generate an *MS2 fingerprint* of the sample."
- [methods] Fingerprints aligned to compare different samples: "These
fingerprints can in a second stage be aligned to compare different
samples."
- [intro] Blank filtering example: "different filtering (remove peaks/losses from
blanks for example) and visualization techniques (MDS/PCoA, TMAP,
Heatmap, ...) can be used."
- [readme] README: MEMO is RT-agnostic alignment using MS2 spectra: "is a method allowing a Retention Time (RT) agnostic alignment of
metabolomics samples using the fragmentation spectra (MS2) of their
consituents."
- [abstract] MEMO enables cross-platform, long-period comparison: "MEMO thus allows the comparison of vast ensembles of samples, even when analyzed over long periods of time, and on different chromatographic or mass spectrometry platforms."
