---
name: gnps-fbmn-output-processing
description: Use when the inputs are an MZmine join-aligner output (aligned feature
  intensity CSV) plus its GNPS FBMN-exported .mgf spectra file, and the goal is an
  RT-agnostic sample-by-feature presence/absence MEMO matrix rather than an intensity-based
  comparison.
when_to_use_negative:
- When MS1 feature intensities must inform the analysis — the MEMO matrix intentionally
  ignores intensity and uses only presence/absence occurrence counting
- When retention-time information is essential to distinguish samples — MEMO is explicitly
  RT-agnostic and cannot resolve RT-shift-based differences
- When the data are raw vendor files or spectra without MS2 fragmentation — the workflow
  requires MS2 peak/neutral-loss content and the MZmine-aligned CSV + GNPS FBMN .mgf
  pair
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: memo-ms
  role: 'Core package: memo_ms.import_data loads the aligned CSV + .mgf pair, SpectraDocuments
    counts MS2 peaks/neutral losses per sample, MemoMatrix aligns fingerprints and
    exports memo_matrix.csv'
  repo: https://github.com/mandelbrot-project/memo
- name: matchms
  role: Underlying library MEMO is built on for handling the MS2 spectra read from
    the GNPS FBMN .mgf file
  repo: https://github.com/matchms/matchms
- name: spec2vec
  role: Underlying library MEMO is built on for converting MS2 spectra into documents
    of peaks and losses
  repo: https://github.com/iomega/spec2vec
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.3389/fbinf.2022.842964
    title: 'MEMO: Mass Spectrometry-Based Sample Vectorization to Explore Chemodiverse
      Datasets'
  source_content_hash: sha256:ec578baebbf7015185e9d1c06bbad3f92047a583d5f7a49409289eb8bd8c8e7f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mandelbrot-project/memo
  tools:
  - matchms
  - memo-ms
  - numpy
  - spec2vec
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# gnps-fbmn-output-processing

## Summary

Convert GNPS FBMN workflow outputs — an MZmine-exported aligned feature quantification table (.csv) and its GNPS FBMN-exported .mgf MS/MS spectra file — into a MEMO matrix of per-sample MS2 peak and neutral-loss occurrence counts for retention-time-agnostic sample comparison.

## When to use

Use when the inputs are an MZmine join-aligner output (aligned feature intensity CSV) plus its GNPS FBMN-exported .mgf spectra file, and the goal is an RT-agnostic sample-by-feature presence/absence MEMO matrix rather than an intensity-based comparison.

## When NOT to use

- When MS1 feature intensities must inform the analysis — the MEMO matrix intentionally ignores intensity and uses only presence/absence occurrence counting
- When retention-time information is essential to distinguish samples — MEMO is explicitly RT-agnostic and cannot resolve RT-shift-based differences
- When the data are raw vendor files or spectra without MS2 fragmentation — the workflow requires MS2 peak/neutral-loss content and the MZmine-aligned CSV + GNPS FBMN .mgf pair

## Inputs

- MZmine-exported aligned feature quantification table (.csv) from the GNPS FBMN workflow (e.g. from join-aligner with 40 ppm m/z and 0.2 min RT tolerance)
- GNPS FBMN-exported .mgf MS/MS spectra file corresponding to the same feature table
- Demo dataset reference: MSV000083306

## Outputs

- Per-sample MS2 fingerprint documents (MS2 peak and neutral-loss occurrence counts, presence/absence)
- Aligned sample-by-feature MEMO count/presence matrix
- memo_matrix.csv export containing one row per sample in the input feature table

## How to apply

First ensure the feature table was produced by MZmine's join-aligner (the demo dataset MSV000083306 used m/z tolerance 40 ppm and RT tolerance 0.2 min), exported as an aligned feature quantification .csv alongside its GNPS FBMN-exported .mgf. Load both with memo_ms.import_data, then convert the MS2 spectra into per-sample documents with memo_ms.SpectraDocuments, which counts the occurrence of each MS2 peak and each neutral loss (relative to the precursor) per sample and records presence/absence while deliberately ignoring MS1 feature intensities. Assemble the per-sample MS2 fingerprints into a memo_ms.MemoMatrix, which aligns them into a shared sample-by-feature count matrix without requiring a prior feature-alignment step. Export the result to memo_matrix.csv and confirm it contains count/presence entries for every sample present in the input feature table.

## Related tools

- **memo-ms** (Core package: memo_ms.import_data loads the aligned CSV + .mgf pair, SpectraDocuments counts MS2 peaks/neutral losses per sample, MemoMatrix aligns fingerprints and exports memo_matrix.csv) — https://github.com/mandelbrot-project/memo
- **matchms** (Underlying library MEMO is built on for handling the MS2 spectra read from the GNPS FBMN .mgf file) — https://github.com/matchms/matchms
- **spec2vec** (Underlying library MEMO is built on for converting MS2 spectra into documents of peaks and losses) — https://github.com/iomega/spec2vec

## Examples

```
conda create --name memo python=3.8; conda activate memo; pip install memo-ms; import memo_ms; specs, feat_table = memo_ms.import_data("feature_quant_table.csv", "gnps_fbmn_spectra.mgf"); docs = memo_ms.SpectraDocuments(specs)
```

## Evaluation signals

- memo_matrix.csv contains a row (count/presence entry) for every sample present in the input aligned feature quantification table
- All matrix values are occurrence counts or presence/absence entries, not MS1 feature intensities
- The matrix columns cover both MS2 peak occurrences and neutral losses computed relative to each precursor
- The source aligned table traces to MZmine join-aligner settings (40 ppm m/z tolerance, 0.2 min RT tolerance) matching the demo MSV000083306 setup
- Fingerprints are unchanged by retention-time shifts or row reordering, confirming the alignment is RT-agnostic

## Limitations

- MS1 feature intensity is not used; the workflow counts only presence/absence of MS2 peaks and losses, so quantitative information is lost
- The aligned-table route still depends on a prior MZmine feature detection and join-aligner step (e.g. 40 ppm / 0.2 min), so errors in that alignment propagate
- Because alignment is retention-time agnostic, samples differing only by RT behavior may appear identical in the MEMO matrix

## Evidence

- [intro] is a method allowing a Retention Time (RT) agnostic alignment of metabolomics samples using the fragmentation spectra (MS2) of their consituents.: "is a method allowing a Retention Time (RT) agnostic alignment of metabolomics samples using the fragmentation spectra (MS2) of their consituents."
- [methods] The occurrence of MS2 peaks and neutral losses (to the precursor) in each sample is counted: "The occurrence of MS2 peaks and neutral losses (to the precursor) in each sample is counted"
- [abstract] such performance was achieved without the requirement of a prior feature alignment step and in a significantly shorter computational time: "such performance was achieved without the requirement of a prior feature alignment step and in a significantly shorter computational time"
- [readme] MEMO source code is available on [Github](https://github.com/mandelbrot-project/memo). It also available as a python package through [pip installation](https://pypi.org/project/memo-ms/).: [evidence span withheld — anchor status: not_found]
