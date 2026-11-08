---
name: mgf-ms2-spectra-loading-and-filtering
description: Use when input is .mgf MS2 spectra per sample (e.g. a GNPS/FBMN job export)
  and you need retention-time-agnostic sample fingerprints from counted peaks and
  losses instead of RT-aligned feature tables.
when_to_use_negative:
- Input is already an RT-aligned feature table from FBMN/MZmine — MEMO's benefit is
  bypassing feature alignment, so re-vectorizing an aligned table adds nothing
- Samples share strong feature overlap and were acquired on the same LC-MS platform
  with no RT shift — standard feature-based metrics may suffice and MEMO's word-count
  fingerprint loses intensity information
- Spectra have fewer than 10 peaks (min_peaks_required=10) or lack precursor mass
  needed to compute losses_from=10 to losses_to=200 — documents will be degenerate
  or empty
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: memo-ms
  role: pip-installed package providing memo_ms.import_data, SpectraDocuments word
    encoding, MemoMatrix assembly/alignment, and CSV export
  repo: https://github.com/mandelbrot-project/memo_publication_examples
- name: numpy
  role: installed alongside memo-ms for array handling of fingerprint vectors
- name: matchms
  role: underlying library used by MEMO for handling the MS2 spectra
  repo: https://github.com/matchms/matchms
- name: spec2vec
  role: provides the document/word encoding style ('peak@mz', 'loss@mz' tokens) used
    to convert spectra into documents
  repo: https://github.com/iomega/spec2vec
- name: conda
  role: creates the dedicated Python 3.8 environment (conda create --name memo python=3.8)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.3389/fbinf.2022.842964
    title: 'MEMO: Mass Spectrometry-Based Sample Vectorization to Explore Chemodiverse
      Datasets'
  source_content_hash: sha256:7716c614174c730ef055eb96f339bc49b407ca7566fada9c24699f3725068529
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mandelbrot-project/memo_publication_examples
  tools:
  - Python
  - conda
  - matchms
  - memo-ms
  - numpy
  - spec2vec
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# mgf-ms2-spectra-loading-and-filtering

## Summary

Load .mgf MS2 spectra files with memo_ms.import_data into SpectraDocuments, converting peaks and precursor neutral losses into spec2vec-style word tokens (peak@mz, loss@mz) that are counted per sample to build MEMO fingerprints without retention-time feature alignment.

## When to use

Use when input is .mgf MS2 spectra per sample (e.g. a GNPS/FBMN job export) and you need retention-time-agnostic sample fingerprints from counted peaks and losses instead of RT-aligned feature tables.

## When NOT to use

- Input is already an RT-aligned feature table from FBMN/MZmine — MEMO's benefit is bypassing feature alignment, so re-vectorizing an aligned table adds nothing
- Samples share strong feature overlap and were acquired on the same LC-MS platform with no RT shift — standard feature-based metrics may suffice and MEMO's word-count fingerprint loses intensity information
- Spectra have fewer than 10 peaks (min_peaks_required=10) or lack precursor mass needed to compute losses_from=10 to losses_to=200 — documents will be degenerate or empty

## Inputs

- .mgf MS2 spectra files (e.g. exported from a GNPS FBMN job such as MassIVE MSV000083306), one or more spectra per sample with sample identity in the .mgf filename metadata
- optional blank sample .mgf spectra for blank filtering

## Outputs

- memo_matrix.csv: aligned sample-by-word MEMO matrix whose rows are samples and whose columns are peak@/loss@ word tokens with occurrence counts
- per-sample MS2 fingerprint vectors (intermediate)

## How to apply

Create a Python 3.8 conda environment and pip install numpy and memo-ms (memo-ms is built on matchms and spec2vec for handling MS2 spectra and converting them into documents). Load the .mgf spectra with memo_ms.import_data, constructing SpectraDocuments with spec2vec-style word encoding using min_relative_intensity=0.01, min_peaks_required=10, losses_from=10, losses_to=200, and n_decimals=2, so fragment peaks become 'peak@mz' tokens and precursor neutral losses become 'loss@mz' tokens rounded to 2 decimals. Count the occurrence of each peak/loss word per .mgf filename to produce a per-sample MS2 fingerprint vector, then assemble the per-sample vectors into a MemoMatrix aligned on the union of word tokens. Optionally filter the matrix, e.g. removing words corresponding to peaks/losses found in blanks, and write the sample-by-word matrix to memo_matrix.csv. Rationale: counting peaks and losses yields an MS2 fingerprint that can be aligned across samples without a prior feature alignment step, enabling comparison of chemodiverse samples or data acquired with different LC methods or mass spectrometers.

## Related tools

- **memo-ms** (pip-installed package providing memo_ms.import_data, SpectraDocuments word encoding, MemoMatrix assembly/alignment, and CSV export) — https://github.com/mandelbrot-project/memo_publication_examples
- **numpy** (installed alongside memo-ms for array handling of fingerprint vectors)
- **matchms** (underlying library used by MEMO for handling the MS2 spectra) — https://github.com/matchms/matchms
- **spec2vec** (provides the document/word encoding style ('peak@mz', 'loss@mz' tokens) used to convert spectra into documents) — https://github.com/iomega/spec2vec
- **conda** (creates the dedicated Python 3.8 environment (conda create --name memo python=3.8))

## Examples

```
conda create --name memo python=3.8; conda activate memo; pip install numpy memo-ms; python -c "import memo_ms; spectra = memo_ms.import_data('MSV000083306_spectra.mgf', min_relative_intensity=0.01, min_peaks_required=10, losses_from=10, losses_to=200, n_decimals=2)"
```

## Evaluation signals

- memo_matrix.csv exists and parses as a numeric matrix
- matrix rows correspond exactly to the input samples (.mgf filenames), with no dropped or duplicated samples
- matrix columns are peak@/loss@ word tokens rounded to 2 decimals (n_decimals=2), consistent with losses bounded at 10–200 Da
- row values are non-negative integer-like occurrence counts of peaks and neutral losses per sample, matching the 'counted MS2 peaks and neutral losses' definition
- after blank filtering, no word tokens traceable exclusively to blanks remain in the matrix

## Limitations

- MEMO fingerprints discard relative/absolute peak intensities beyond the min_relative_intensity=0.01 filtering threshold, since only peak occurrence counts are kept
- Parameter choices (min_peaks_required=10, losses_from=10, losses_to=200, n_decimals=2) constrain which spectra and losses contribute; low-peak or out-of-range-loss spectra yield sparse documents
- Comparing samples analyzed on different chromatographic or mass spectrometer platforms works, but downstream interpretation still relies on filtering/visualization choices (remove blanks, MDS/PCoA, TMAP, Heatmap)
- mind loss filtering bug historically present: 'Changed classes.py to avoid erasing when filtering'

## Evidence

- [methods] import parameters: [evidence span withheld — anchor status: not_found]
- [intro] word encoding rationale: "The occurence of MS2 peaks and neutral losses (to the precursor) in each sample is counted and used to generate an *MS2 fingerprint* of the sample."
- [methods] alignment step: "These fingerprints can in a second stage be aligned to compare different samples."
- [intro] blank filtering: "different filtering (remove peaks/losses from blanks for example) and visualization techniques (MDS/PCoA, TMAP, Heatmap, ...) can be used."
- [methods] tool stack: "MEMO is mainly built on `matchms`_ and `spec2vec`_ packages for handling the MS2 spectra and converting them into documents"
- [readme] environment setup: "conda create --name memo python=3.8"
- [abstract] RT-agnostic scope: "such performance was achieved without the requirement of a prior feature alignment step and in a significantly shorter computational time"
