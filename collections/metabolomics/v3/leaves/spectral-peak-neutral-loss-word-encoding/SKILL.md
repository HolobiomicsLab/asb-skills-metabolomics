---
name: spectral-peak-neutral-loss-word-encoding
description: Use when MS2 spectra are available as .mgf files per sample and samples
  must be compared without RT-based feature alignment, e.g. chemodiverse datasets,
  RT-shifted runs, or different LC/MS platforms.
when_to_use_negative:
- The dataset is well-aligned by retention time and standard feature-table-based metrics
  suffice — MEMO was designed for retention time agnostic comparison and adds encoding
  overhead without benefit there.
- Only MS1 data are available — the encoding requires MS2 fragmentation spectra (peaks
  and neutral losses to the precursor).
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: memo-ms
  role: Python package providing memo_ms.import_data.SpectraDocuments to encode .mgf
    spectra into peak/loss word documents and MemoMatrix to assemble and align per-sample
    fingerprints
  repo: https://github.com/mandelbrot-project/memo_publication_examples
- name: matchms
  role: Underlying library MEMO builds on for handling and importing MS2 spectra
  repo: https://github.com/matchms/matchms
- name: spec2vec
  role: Provides the spectra-to-documents word-encoding concept (mass fragments and
    neutral losses as words) that MEMO's encoding mimics
  repo: https://github.com/iomega/spec2vec
- name: Python
  role: Runtime environment (conda env python=3.8) in which memo-ms and numpy are
    pip-installed
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.3389/fbinf.2022.842964
    title: 'MEMO: Mass Spectrometry-Based Sample Vectorization to Explore Chemodiverse
      Datasets'
  source_content_hash: sha256:45cbea0428c78d9fea9d3cfbfc8790ef24dee7fbad8a2e286b26d14668d73441
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
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# spectral-peak-neutral-loss-word-encoding

## Summary

A spec2vec-style encoding that converts MS2 spectra into 'documents' where peaks and precursor neutral losses become rounded word tokens (peak@mz, loss@mz), enabling retention-time-agnostic MEMO sample vectorization by counting word occurrences per sample.

## When to use

Use when MS2 spectra are available as .mgf files per sample and samples must be compared without RT-based feature alignment, e.g. chemodiverse datasets, RT-shifted runs, or different LC/MS platforms.

## When NOT to use

- The dataset is well-aligned by retention time and standard feature-table-based metrics suffice — MEMO was designed for retention time agnostic comparison and adds encoding overhead without benefit there.
- Only MS1 data are available — the encoding requires MS2 fragmentation spectra (peaks and neutral losses to the precursor).

## Inputs

- .mgf MS2 peak list files (e.g. GNPS FBMN export such as MassIVE MSV000083306), with per-file/sample metadata
- Optional: blank sample spectra for filtering peak/loss words

## Outputs

- Per-sample MS2 fingerprint vectors (counts of peak@/loss@ word tokens)
- Aligned sample-by-word MEMO matrix written to memo_matrix.csv

## How to apply

Load the .mgf MS2 spectra (e.g. from a MassIVE FBMN job) with memo_ms.import_data, building SpectraDocuments with min_relative_intensity=0.01, min_peaks_required=10, losses_from=10, losses_to=200, and n_decimals=2, so fragment peaks and precursor neutral losses become rounded word tokens. Group the encoded documents by the per-sample .mgf filename metadata and sum peak/loss word occurrences across each sample's spectra to produce an MS2 fingerprint (MEMO vector). Assemble and align the per-sample vectors into a MemoMatrix; optionally first remove words corresponding to peaks/losses found in blanks. Write the aligned sample-by-word matrix to memo_matrix.csv. The rationale is that counted peak/loss words are stable across retention-time shifts, avoiding the prior feature alignment step.

## Related tools

- **memo-ms** (Python package providing memo_ms.import_data.SpectraDocuments to encode .mgf spectra into peak/loss word documents and MemoMatrix to assemble and align per-sample fingerprints) — https://github.com/mandelbrot-project/memo_publication_examples
- **matchms** (Underlying library MEMO builds on for handling and importing MS2 spectra) — https://github.com/matchms/matchms
- **spec2vec** (Provides the spectra-to-documents word-encoding concept (mass fragments and neutral losses as words) that MEMO's encoding mimics) — https://github.com/iomega/spec2vec
- **Python** (Runtime environment (conda env python=3.8) in which memo-ms and numpy are pip-installed)

## Examples

```
from memo_ms import import_data; docs = import_data.SpectraDocuments('MSV000083306.mgf', min_relative_intensity=0.01, min_peaks_required=10, losses_from=10, losses_to=200, n_decimals=2)
```

## Evaluation signals

- memo_matrix.csv exists and parses as a matrix whose rows correspond exactly to the input samples (.mgf filenames).
- Columns of memo_matrix.csv are peak@/loss@ word tokens generated with n_decimals=2 rounding.
- Per-sample word counts equal summed peak and loss occurrences across that sample's spectra after filtering (min_relative_intensity=0.01, min_peaks_required=10, losses_from=10 to losses_to=200).
- Resulting sample clustering is retention time agnostic — reproducing MEMO's reported performance without a prior feature alignment step.
- Blank-derived peak/loss words are removed when the optional blank-filtering step is applied.

## Limitations

- Spectra with fewer than min_peaks_required=10 peaks are excluded from encoding (min_peaks_required parameter).
- Word tokens depend on n_decimals=2 rounding; precursor neutral losses are only encoded in the 10–200 range (losses_from/losses_to).
- The resulting fingerprint comparison is word-based, so chemodiverse samples with poor feature overlap are its strength but highly similar feature-rich samples may not gain over state-of-the-art spectral metrics.

## Evidence

- [abstract] abstract: "MEMO— M S2 Bas E d Sa M ple Vect O rization—a method allowing to cluster large amounts of chemodiverse samples based on their LC-MS/MS profiles"
- [readme] The occurence of MS2 peaks and neutral losses (to the precursor) in each sample is counted and used to generate an *MS2 fingerprint* of the sample.: "The occurence of MS2 peaks and neutral losses (to the precursor) in each sample is counted and used to generate an *MS2 fingerprint* of the sample."
- [methods] methods tooling quote: "MEMO is mainly built on `matchms`_ and `spec2vec`_ packages for handling"
- [readme] spec2vec encoding analogy: "Where Word2Vec learns relationships between words in sentences,
**spec2vec** does so for mass fragments and neutral losses in MS/MS spectra."
