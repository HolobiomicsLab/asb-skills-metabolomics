---
name: ms2-peak-neutral-loss-counting
description: Use when you have LC-MS/MS peak lists (e.g. .mgf) and need to vectorize
  samples without feature alignment — especially for chemodiverse samples with poor
  feature overlap or strong RT shifts.
when_to_use_negative:
- 'When intensities matter to your analysis: MEMO discards MS1 feature intensity and
  uses only presence/absence of peaks and losses'
- When you need retention-time-aware sample comparison — MEMO is deliberately RT agnostic
  and will not preserve RT information
- When features are already aligned across samples and an intensity-based distance
  (e.g. Bray-Curtis on the feature table) is the intended metric
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0092
tools:
- name: memo-ms
  role: 'Primary package: provides import_data, SpectraDocuments and MemoMatrix classes
    for counting MS2 peaks and neutral losses and aligning fingerprints'
  repo: https://github.com/mandelbrot-project/memo_publication_examples
- name: matchms
  role: Underlying library used by memo-ms for handling the MS2 spectra
  repo: https://github.com/matchms/matchms
- name: spec2vec
  role: Underlying library used by memo-ms for converting MS2 spectra into documents
  repo: https://github.com/iomega/spec2vec
- name: numpy
  role: Dependency installed alongside memo-ms in the documented conda/pip setup
provenance:
  source_task_ids:
  - task_008
  - task_003
  source_papers:
  - doi: 10.3389/fbinf.2022.842964
    title: 'MEMO: Mass Spectrometry-Based Sample Vectorization to Explore Chemodiverse
      Datasets'
  source_content_hash: sha256:f912e82de90aa078caea82f13fe63884366bc53ab0254de821c85d396576d779
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 16
---

# ms2-peak-neutral-loss-counting

## Summary

Generates an MS2 fingerprint for each metabolomics sample by counting the occurrence of MS2 fragment peaks and neutral losses (relative to the precursor), as implemented in the memo-ms package, then aligns these fingerprints into a sample-by-feature MEMO matrix for retention-time-agnostic sample comparison.

## When to use

Use when you have LC-MS/MS peak lists (e.g. .mgf) and need to vectorize samples without feature alignment — especially for chemodiverse samples with poor feature overlap or strong RT shifts.

## When NOT to use

- When intensities matter to your analysis: MEMO discards MS1 feature intensity and uses only presence/absence of peaks and losses
- When you need retention-time-aware sample comparison — MEMO is deliberately RT agnostic and will not preserve RT information
- When features are already aligned across samples and an intensity-based distance (e.g. Bray-Curtis on the feature table) is the intended metric

## Inputs

- MS2 spectra peak list file (e.g. .mgf exported from GNPS FBMN)
- MZmine-exported aligned feature quantification table (.csv) when using the aligned-sample workflow
- memo-ms conda environment (Python 3.8, memo-ms installed with pip, built on matchms and spec2vec)

## Outputs

- Per-sample MS2 fingerprint documents (peak and neutral-loss occurrence counts, presence/absence)
- Aligned sample-by-feature MEMO count matrix (memo_matrix.csv)
- Downstream sample comparison outputs: MDS/PCoA, TMAP, Heatmap visualizations after optional blank filtering

## How to apply

Load the MS2 spectra (e.g. GNPS FBMN-exported .mgf) via memo_ms.import_data, then convert each spectrum into a document with memo_ms.SpectraDocuments, counting every MS2 peak and each neutral loss relative to the precursor per sample. Intensities are ignored — only occurrence/presence is recorded, so MS1 feature intensity values in an aligned quantification table are used solely to decide whether a feature was detected in a sample, never as weights. Align the per-sample MS2 fingerprints into a memo_ms.MemoMatrix sample-by-feature count matrix, which is retention-time agnostic and requires no prior feature alignment step. Export the result (e.g. memo_matrix.csv) and optionally filter it downstream (e.g. removing peaks/losses attributed to blanks) before visualization via MDS/PCoA, TMAP or Heatmap.

## Related tools

- **memo-ms** (Primary package: provides import_data, SpectraDocuments and MemoMatrix classes for counting MS2 peaks and neutral losses and aligning fingerprints) — https://github.com/mandelbrot-project/memo_publication_examples
- **matchms** (Underlying library used by memo-ms for handling the MS2 spectra) — https://github.com/matchms/matchms
- **spec2vec** (Underlying library used by memo-ms for converting MS2 spectra into documents) — https://github.com/iomega/spec2vec
- **numpy** (Dependency installed alongside memo-ms in the documented conda/pip setup)

## Examples

```
import memo_ms; spectra, feature_table = memo_ms.import_data('MSV000083306_spectra.mgf', 'aligned_quant_table.csv'); documents = memo_ms.SpectraDocuments(spectra); memo = memo_ms.MemoMatrix(documents); memo.export('memo_matrix.csv')
```

## Evaluation signals

- The exported memo_matrix.csv contains count/presence entries for every sample present in the input feature table
- For the large plant dataset (MassIVE MSV000087728, 1,920 samples / 906,509 MS/MS spectra), the matrix should be square with 1,920 rows and compute in under 10 min on a simple laptop
- Retention times do not influence the matrix — samples acquired with different LC methods or RT shifts can still be compared
- Fingerprints contain counts of both MS2 peaks and neutral losses relative to the precursor, with intensities ignored
- Environment reproduces the documented setup: conda python 3.8 environment, pip install memo-ms (version 0.1.3)

## Limitations

- MS1 feature intensities are not used; only a presence/absence / occurrence-count representation is produced, so quantitative abundance differences are lost
- Fingerprint filtering must be applied manually (e.g. remove peaks/losses from blanks); an early version had a bug where filtering in classes.py could erase data
- The 'memo ready' feature table import mode is documented without file format or dataset specification

## Evidence

- [intro] The occurence of MS2 peaks and neutral losses (to the precursor) in each sample is counted and used to generate an *MS2 fingerprint* of the sample.: "The occurence of MS2 peaks and neutral losses (to the precursor) in each sample is counted and used to generate an *MS2 fingerprint* of the sample."
- [methods] These fingerprints can in a second stage be aligned to compare different samples.: "These
fingerprints can in a second stage be aligned to compare different
samples."
- [readme] different filtering (remove peaks/losses from blanks for example) and visualization techniques (MDS/PCoA, TMAP, Heatmap, ...) can be used.: "different filtering (remove peaks/losses from
blanks for example) and visualization techniques (MDS/PCoA, TMAP,
Heatmap, ...) can be used."
- [methods] MEMO is mainly built on `matchms`_ and `spec2vec`_ packages for handling the MS2 spectra and converting them into documents: "MEMO is mainly built on `matchms`_ and `spec2vec`_ packages for handling
the MS2 spectra and converting them into documents"
- [other] In the aligned-sample workflow, the occurrence of each peak and loss within a given sample is counted using the feature intensity table to determine whether a feature is detected in the sample; the MS1 feature intensity is not used, only a presence/absence matrix.: [evidence span withheld — anchor status: not_found]
- [results] a MEMO matrix was computed in less than 10 min on a simple laptop, demonstrating MEMO's computational efficiency at large scale: [evidence span withheld — anchor status: not_found]
