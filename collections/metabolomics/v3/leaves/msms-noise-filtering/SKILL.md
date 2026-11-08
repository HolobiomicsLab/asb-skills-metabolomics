---
name: msms-noise-filtering
description: Use when raw MS2 fragmentation spectra paired to MS1 peaks contain noisy
  fragment or neutral-loss entries that must be removed before building a bag-of-fragments
  matrix or running topic-model inference.
when_to_use_negative:
- Input is already a cleaned, aligned feature table or count matrix — re-filtering
  would discard valid features.
- 'The goal is library spectral matching against MassBank or NIST: the 90% normalized-score
  matching path does not consume the noise-filtered motif vocabulary, and neutral
  losses are not supported by spectral matching tools.'
- Data are information-poor MS2 spectra (few metabolites fragmented, single fixed
  collision energy) where the MS2LDA preprocessing and inference chain underperforms.
tools:
- name: MS2LDA.Preprocessing.load_and_clean
  role: Core cleaning function that removes noise entries from fragment and neutral-loss
    lists of paired MS2 spectra
  repo: https://github.com/vdhooftcompmet/MS2LDA
- name: MS2LDA.Preprocessing.generate_corpus
  role: Aligns cleaned MS2 features across all spectra of each sample into the bag-of-fragments
    corpus including neutral losses
  repo: https://github.com/vdhooftcompmet/MS2LDA
- name: RMassBank
  role: Reliably pairs each MS1 peak with its corresponding MS2 fragmentation spectrum
    prior to cleaning
- name: XCMS
  role: Detects MS1 peaks in the raw beer-sample files that are subsequently paired
    to MS2 spectra
- name: PeakML/mzMatch
  role: Refines XCMS-detected features to the ~1,000 MS peaks per sample in both ionization
    modes
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1073/pnas.1608041113
    title: Topic modeling for untargeted substructure exploration in metabolomics
  source_content_hash: sha256:b9a0e19275e53df9159f8a5e3a885ab8c8cb02dc8843b31b336aad127a89e165
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/vdhooftcompmet/MS2LDA
  tools:
  - MS2LDA.Preprocessing.generate_corpus
  - MS2LDA.Preprocessing.load_and_clean
  - PeakML/mzMatch
  - RMassBank
  - XCMS
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# msms-noise-filtering

## Summary

Cleans paired MS1–MS2 fragmentation spectra by removing noise entries from fragment and neutral-loss lists using the MS2LDA preprocessing module (load_and_clean), so that only reliable MS2 features feed downstream corpus generation and LDA inference.

## When to use

Use when raw MS2 fragmentation spectra paired to MS1 peaks contain noisy fragment or neutral-loss entries that must be removed before building a bag-of-fragments matrix or running topic-model inference.

## When NOT to use

- Input is already a cleaned, aligned feature table or count matrix — re-filtering would discard valid features.
- The goal is library spectral matching against MassBank or NIST: the 90% normalized-score matching path does not consume the noise-filtered motif vocabulary, and neutral losses are not supported by spectral matching tools.
- Data are information-poor MS2 spectra (few metabolites fragmented, single fixed collision energy) where the MS2LDA preprocessing and inference chain underperforms.

## Inputs

- MS2 fragmentation spectra paired to MS1 peaks (output of XCMS + PeakML/mzMatch peak detection and RMassBank pairing, ~1,000 MS peaks per sample)
- Raw MS2 peak lists containing fragments and neutral losses with intensity values

## Outputs

- Cleaned MS2 spectra with noise entries removed from fragment and neutral-loss lists
- Aligned bag-of-fragments matrix (rows = MS2 features/fragments and neutral losses, columns = MS2 spectra) with integer occurrence counts

## How to apply

First ensure each MS1 peak is reliably paired with its MS2 spectrum (in the beer-extract workflow, MS1 peaks are detected with XCMS and refined with PeakML/mzMatch, yielding roughly 1,000 MS peaks per sample in both positive and negative ionization mode, and pairing is done with RMassBank). Then apply MS2LDA.Preprocessing.load_and_clean to each paired MS2 spectrum to remove noise entries from both the fragment list and the extracted neutral-loss list. Neutral losses are computed as part of this cleaning step so they can participate in the downstream motif vocabulary. The cleaned spectra are then passed to MS2LDA.Preprocessing.generate_corpus, which aligns MS2 features across all spectra of a sample into a bag-of-fragments representation. The final assembled matrix has fragments/losses as rows and MS2 spectra as columns, with intensity values transformed into integer occurrence counts suitable for LDA.

## Related tools

- **MS2LDA.Preprocessing.load_and_clean** (Core cleaning function that removes noise entries from fragment and neutral-loss lists of paired MS2 spectra) — https://github.com/vdhooftcompmet/MS2LDA
- **MS2LDA.Preprocessing.generate_corpus** (Aligns cleaned MS2 features across all spectra of each sample into the bag-of-fragments corpus including neutral losses) — https://github.com/vdhooftcompmet/MS2LDA
- **RMassBank** (Reliably pairs each MS1 peak with its corresponding MS2 fragmentation spectrum prior to cleaning)
- **XCMS** (Detects MS1 peaks in the raw beer-sample files that are subsequently paired to MS2 spectra)
- **PeakML/mzMatch** (Refines XCMS-detected features to the ~1,000 MS peaks per sample in both ionization modes)

## Examples

```
from MS2LDA.Preprocessing import load_and_clean, generate_corpus; cleaned = load_and_clean(paired_ms2_spectra); corpus = generate_corpus(cleaned)
```

## Evaluation signals

- The output CSV matrix exists, is well-formed, and contains only nonnegative integer values (intensities transformed into integer occurrence counts).
- Per-sample peak counts are consistent with the reported ~1,000 MS peaks per sample in both positive and negative ionization mode.
- After load_and_clean, fragment and neutral-loss lists contain no noise entries (spot-check low-intensity/random-m/z entries removed).
- Matrix dimensions match the expected layout: rows are MS2 features (fragments and losses), columns are MS2 spectra.
- Downstream LDA inference runs without errors on the integer count matrix, and extracted Mass2Motifs contain co-occurring fragments and neutral losses.

## Limitations

- MS2LDA works best for complex mixtures where a large number of metabolites are fragmented and information-rich MS2 spectra are available (e.g., generated by ramped or stepped collision energy).
- Neutral losses retained by this filtering step are not currently supported by spectral matching tools and had to be omitted in automated annotation.
- Cleaning quality depends on reliable MS1-to-MS2 peak pairing upstream; failed pairings propagate noise or missing spectra into the corpus.

## Evidence

- [other] Clean and filter the paired MS2 spectra with the MS2LDA preprocessing module (load_and_clean), removing noise entries from the fragment and neutral-loss lists.: [evidence span withheld — anchor status: not_found]
- [discussion] MS2LDA works best only for complex mixtures with information-rich MS2 spectra: [evidence span withheld — anchor status: not_found]
