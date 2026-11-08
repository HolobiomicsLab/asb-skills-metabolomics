---
name: memo-matrix-alignment-across-samples
description: Use when LC-MS/MS samples lack a prior RT-based feature alignment — e.g.
  chemodiverse datasets, strong RT shifts, or different LC/MS platforms — and you
  need comparable sample-by-word vectors from raw MS2 spectra (.mgf).
when_to_use_negative:
- Input is already a curated, RT-aligned feature table where standard feature-based
  abundance metrics suffice.
- The analysis requires retention-time information itself rather than a retention-time-agnostic
  comparison of fragmentation spectra.
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: memo-ms
  role: 'Primary package: imports .mgf MS2 spectra into SpectraDocuments, counts peak/loss
    words into MS2 fingerprints, and aligns them into a MemoMatrix written to memo_matrix.csv.'
  repo: https://github.com/mandelbrot-project/memo_publication_examples
- name: matchms
  role: Underlying library MEMO is mainly built on for handling the MS2 spectra.
  repo: https://github.com/matchms/matchms
- name: spec2vec
  role: Underlying library used to convert MS2 spectra into documents (word tokens)
    in spec2vec style.
  repo: https://github.com/iomega/spec2vec
- name: numpy
  role: Dependency installed alongside memo-ms for matrix/array handling.
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.3389/fbinf.2022.842964
    title: 'MEMO: Mass Spectrometry-Based Sample Vectorization to Explore Chemodiverse
      Datasets'
  source_content_hash: sha256:efd56e11002f30d4593354f0e4ef6f61a2e6866e62a394997a3537cf9ebc82dc
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
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# memo-matrix-alignment-across-samples

## Summary

Build an MS2-based sample fingerprint matrix (MEMO) directly from unaligned MGF MS2 spectra by counting peak@mz and loss@mz word tokens per sample and aligning the per-sample vectors into a sample-by-word matrix, bypassing retention-time-based feature alignment.

## When to use

Use when LC-MS/MS samples lack a prior RT-based feature alignment — e.g. chemodiverse datasets, strong RT shifts, or different LC/MS platforms — and you need comparable sample-by-word vectors from raw MS2 spectra (.mgf).

## When NOT to use

- Input is already a curated, RT-aligned feature table where standard feature-based abundance metrics suffice.
- The analysis requires retention-time information itself rather than a retention-time-agnostic comparison of fragmentation spectra.

## Inputs

- .mgf MS2 spectra files (one per sample, from an FBMN job such as MassIVE MSV000083306)
- sample metadata mapping spectra to sample (.mgf filename)
- optional blank sample spectra for peaks/losses filtering

## Outputs

- memo_matrix.csv: aligned sample-by-word MEMO matrix whose rows are input samples and columns are peak/loss word tokens (e.g. peak@mz, loss@mz) with per-sample occurrence counts

## How to apply

Create a Python 3.8 conda environment and install numpy plus memo-ms with pip. Load the .mgf MS2 spectra (e.g. from the MassIVE MSV000083306 FBMN job) via memo_ms.import_data, building SpectraDocuments with spec2vec-style word encoding using min_relative_intensity=0.01, min_peaks_required=10, losses_from=10, losses_to=200, and n_decimals=2 so MS2 peaks and precursor neutral losses become rounded word tokens. Count occurrence of each peak/loss word per sample (.mgf filename metadata) to produce per-sample MS2 fingerprint vectors. Assemble and align the fingerprints into a MemoMatrix — the class replacing the former MemoContainer — optionally filtering words corresponding to peaks/losses found in blanks, then export the aligned sample-by-word MEMO matrix. The rationale is that counting peak and loss occurrences yields a retention-time-agnostic fingerprint that can be aligned across samples analyzed on different chromatographic or mass spectrometer platforms without a feature alignment step.

## Related tools

- **memo-ms** (Primary package: imports .mgf MS2 spectra into SpectraDocuments, counts peak/loss words into MS2 fingerprints, and aligns them into a MemoMatrix written to memo_matrix.csv.) — https://github.com/mandelbrot-project/memo_publication_examples
- **matchms** (Underlying library MEMO is mainly built on for handling the MS2 spectra.) — https://github.com/matchms/matchms
- **spec2vec** (Underlying library used to convert MS2 spectra into documents (word tokens) in spec2vec style.) — https://github.com/iomega/spec2vec
- **numpy** (Dependency installed alongside memo-ms for matrix/array handling.)

## Examples

```
conda create --name memo python=3.8; conda activate memo; pip install numpy memo-ms; python -c "from memo_ms import import_data; docs = import_data.Ms2Documents(mgf_path='MSV000083306.mgf', min_relative_intensity=0.01, min_peaks_required=10, losses_from=10, losses_to=200, n_decimals=2)"
```

## Evaluation signals

- memo_matrix.csv exists and parses as a numeric matrix.
- Matrix rows correspond exactly to the input samples (.mgf filenames).
- Matrix columns are peak/loss word tokens consistent with the encoding parameters (n_decimals=2 rounding of peak@mz and loss@mz values, losses within 10–200 range).
- Sample spectra with fewer than min_peaks_required=10 peaks are excluded per the SpectraDocuments parameters.
- Optionally filtered blanks: words corresponding to peaks/losses found in blanks are removed without erasing other entries.

## Limitations

- Counted peak/loss words discard retention time entirely, so the matrix cannot be used to recover RT-based information.
- Filtering behavior can erase data if mishandled — the discussion notes 'Changed classes.py to avoid erasing when filtering'.
- Requires Python 3.8 environment with numpy and memo-ms (later versions relaxed the requirement to >3.7 for TMAP compatibility).

## Evidence

- [intro] count peaks/losses into fingerprints and align: "The occurence of MS2 peaks and neutral losses (to the precursor) in each sample is counted and used to generate an *MS2 fingerprint* of the sample."
- [intro] alignment step: "These fingerprints can in a second stage be aligned to compare different samples."
- [methods] filtering options: "different filtering (remove peaks/losses from blanks for example) and visualization techniques (MDS/PCoA, TMAP, Heatmap, ...) can be used."
- [methods] encoding built on spec2vec/matchms: "MEMO is mainly built on `matchms`_ and `spec2vec`_ packages for handling
the MS2 spectra and converting them into documents"
- [discussion] MemoMatrix replaces MemoContainer: "New `MemoMatrix` class, replacing the former `MemoContainer`"
- [abstract] RT-agnostic alignment claim: "a method allowing to cluster large amounts of chemodiverse samples based on their LC-MS/MS profiles in a retention time agnostic manner"
