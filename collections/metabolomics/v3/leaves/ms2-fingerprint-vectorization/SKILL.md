---
name: ms2-fingerprint-vectorization
description: Use when metabolomics samples cannot be compared by RT-aligned features
  — chemodiverse samples with poor feature overlap, strong RT shifts, different LC
  gradients, or different MS instruments (e.g. Maxis Q-ToF vs Q-Exactive Orbitrap)
  — and a sample-level MS2 fingerprint matrix is needed.
when_to_use_negative:
- Samples already align well by retention time and share substantial feature overlap,
  where standard RT-based feature alignment and ordination suffice — MEMO is stated
  to suit particularly well to chemodiverse samples with poor feature overlap or strong
  RT shifts.
- The analysis requires structurally annotated, compound-level chemical similarity
  (e.g. cosine-score-based metrics like CSCS) rather than raw occurrence counts of
  MS2 peaks and neutral losses.
- Spectra contain too few peaks to be informative — the document encoding requires
  min_peaks_required=10, so sparse MS2 files yield weak fingerprints.
edam_operation: http://edamontology.org/operation_3219
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: memo-ms
  role: 'Core package: import_data loads .mgf spectra, builds SpectraDocuments, and
    assembles/aligns per-sample fingerprints into a MemoMatrix'
  repo: https://github.com/mandelbrot-project/memo_publication_examples
- name: matchms
  role: Handling and filtering of the MS2 spectra underlying the fingerprint computation
  repo: https://github.com/matchms/matchms
- name: spec2vec
  role: Converts MS2 spectra into documents, providing the word-encoding style used
    for peak/loss tokens
  repo: https://github.com/iomega/spec2vec
- name: numpy
  role: Dependency installed before memo-ms for the fingerprint vector computations
- name: scikit-bio
  role: Downstream distance/ordination computations on the aligned MEMO matrix
provenance:
  source_task_ids:
  - task_006
  - task_007
  source_papers:
  - doi: 10.3389/fbinf.2022.842964
    title: 'MEMO: Mass Spectrometry-Based Sample Vectorization to Explore Chemodiverse
      Datasets'
  source_content_hash: sha256:2b257dc45b5f4d340d76eb443107b36f4f482577c38c64ef18f4aacf5b3fcbce
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
  edam_operation: http://edamontology.org/operation_3219
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# ms2-fingerprint-vectorization

## Summary

Vectorizes LC-MS/MS samples by counting occurrences of MS2 peaks and neutral losses to the precursor as peak@mz / loss@mz word tokens, producing retention-time-agnostic MEMO sample vectors that are aligned into a sample-by-word matrix without prior feature alignment.

## When to use

Use when metabolomics samples cannot be compared by RT-aligned features — chemodiverse samples with poor feature overlap, strong RT shifts, different LC gradients, or different MS instruments (e.g. Maxis Q-ToF vs Q-Exactive Orbitrap) — and a sample-level MS2 fingerprint matrix is needed.

## When NOT to use

- Samples already align well by retention time and share substantial feature overlap, where standard RT-based feature alignment and ordination suffice — MEMO is stated to suit particularly well to chemodiverse samples with poor feature overlap or strong RT shifts.
- The analysis requires structurally annotated, compound-level chemical similarity (e.g. cosine-score-based metrics like CSCS) rather than raw occurrence counts of MS2 peaks and neutral losses.
- Spectra contain too few peaks to be informative — the document encoding requires min_peaks_required=10, so sparse MS2 files yield weak fingerprints.

## Inputs

- .mgf MS2 spectra files with per-file sample metadata (e.g. from a GNPS FBMN job / MassIVE accession MSV000083306 or MSV000088521)
- existing MEMO fingerprint matrix (for appending new sample vectors)
- blank sample spectra (for optional blank filtering of peaks/losses)

## Outputs

- Per-sample MS2 fingerprint vectors (counted peak@mz / loss@mz word occurrences)
- Aligned sample-by-word MEMO matrix written to memo_matrix.csv
- UMAP / TMAP / MDS-PCoA / heatmap embeddings and plots of the aligned samples

## How to apply

Create a dedicated conda environment (conda create --name memo python=3.8; pip install numpy; pip install memo-ms) and load the .mgf MS2 spectra (e.g. from a GNPS FBMN job such as MassIVE MSV000083306) with memo_ms.import_data, building SpectraDocuments with spec2vec-style word encoding using min_relative_intensity=0.01, min_peaks_required=10, losses_from=10, losses_to=200, and n_decimals=2 so fragments and precursor losses become rounded word tokens. Count occurrence of each peak@mz and loss@mz word per sample, using the .mgf filename metadata to attribute spectra to samples, then assemble and align the per-sample fingerprint vectors into a MemoMatrix — deliberately bypassing retention-time-based feature alignment. Optionally apply blank filtering (remove words corresponding to peaks/losses found in blanks) before merging. Visualize the aligned matrix with MDS/PCoA, UMAP, TMAP, or heatmap; e.g. append new sample vectors (such as Waltheria indica 2014 QE-Plus 40-min-gradient data alongside 2018 QE-Focus 8-min-gradient data) to an existing MEMO matrix and confirm they cluster with the expected reference extract. Finally write the aligned sample-by-word MEMO matrix to memo_matrix.csv.

## Related tools

- **memo-ms** (Core package: import_data loads .mgf spectra, builds SpectraDocuments, and assembles/aligns per-sample fingerprints into a MemoMatrix) — https://github.com/mandelbrot-project/memo_publication_examples
- **matchms** (Handling and filtering of the MS2 spectra underlying the fingerprint computation) — https://github.com/matchms/matchms
- **spec2vec** (Converts MS2 spectra into documents, providing the word-encoding style used for peak/loss tokens) — https://github.com/iomega/spec2vec
- **numpy** (Dependency installed before memo-ms for the fingerprint vector computations)
- **scikit-bio** (Downstream distance/ordination computations on the aligned MEMO matrix)

## Examples

```
conda create --name memo python=3.8 && conda activate memo && pip install numpy && pip install memo-ms
```

## Evaluation signals

- memo_matrix.csv exists and parses as a matrix whose rows correspond exactly to the input samples and whose columns are peak/loss word tokens (peak@mz, loss@mz with n_decimals=2).
- No retention-time-based feature alignment step was performed; the matrix is aligned on word tokens, consistent with the paper's RT-agnostic description.
- Appended vectors from different platforms still cluster with their expected reference: Waltheria indica 2014 (QE-Plus, 40 min gradient) and 2018 (QE-Focus, 8 min gradient) extracts co-cluster with the trypanocidal Melochia umbellata green stems extract in both UMAP and TMAP.
- Blank-attributed words are absent after optional filtering, without erasing non-blank words (a filtering erasing bug was fixed in classes.py).
- Word counts reflect summed peak and loss occurrences across all of a sample's MS2 spectra, aggregating documents per .mgf filename.

## Limitations

- Fingerprints are occurrence counts of peaks/losses only; chemical identity and spectral intensity structure beyond the min_relative_intensity=0.01 threshold are reduced to word tokens.
- Because alignment is RT-agnostic and word-based, chemically distinct compounds sharing identical binned fragment/loss tokens are conflated in the matrix.
- A filtering bug that erased data in classes.py existed in earlier versions; use the fixed memo release when applying blank filtering.
- Comparison relies on sufficient MS2 peak counts per sample (min_peaks_required=10); low-quality or peak-poor spectra yield unreliable vectors.

## Evidence

- [readme] Fingerprint counting defines the vectorization: "The occurence of MS2 peaks and neutral losses (to the precursor) in each sample is counted and used to generate an *MS2 fingerprint* of the sample."
- [abstract] Platform and time-span agnostic comparison: "MEMO thus allows the comparison of vast ensembles of samples, even when analyzed over long periods of time, and on different chromatographic or mass spectrometry platforms."
- [methods] Filtering and visualization options after alignment: "different filtering (remove peaks/losses from blanks for example) and visualization techniques (MDS/PCoA, TMAP, Heatmap, ...) can be used."
- [readme] Intended scope: chemodiverse and RT-shifted samples: "MEMO suits particularly well to compare chemodiverse samples, ie with a poor features overlap, or to compare samples with a strong RT shift,"
- [discussion] Filtering erasing bug fixed in classes.py: "Changed classes.py to avoid erasing when filtering"
