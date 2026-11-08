---
name: metabolomics-sample-fingerprint-generation
description: Use when you have LC-MS/MS samples (e.g. an MZmine-exported aligned feature
  table plus a GNPS FBMN-exported .mgf spectra file) and need a retention-time-agnostic
  sample-vs-sample comparison matrix built from MS2 peak/loss occurrence counts.
when_to_use_negative:
- Analysis requires quantitative use of MS1 feature intensities — MEMO fingerprints
  record only peak/loss occurrence (presence/absence), ignoring intensities.
- Comparison depends on retention-time behavior (e.g. detecting RT shifts) — MEMO
  is deliberately retention-time agnostic.
- Samples were compared where a well-behaved feature overlap exists and standard feature-table
  metrics suffice — MEMO targets chemodiverse samples with poor feature overlap.
edam_operation: http://edamontology.org/operation_2945
tools:
- name: memo-ms
  role: 'Core package: imports the MZmine feature table + mgf spectra, builds SpectraDocuments
    (peak/loss counts) and the aligned MemoMatrix'
  repo: https://github.com/mandelbrot-project/memo
- name: matchms
  role: Handling of the MS2 spectra underlying fingerprint construction
  repo: https://github.com/matchms/matchms
- name: spec2vec
  role: Converting MS2 spectra into documents for fingerprint generation
  repo: https://github.com/iomega/spec2vec
- name: numpy
  role: Numerical dependency for the matrix computations in the MEMO install
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.3389/fbinf.2022.842964
    title: 'MEMO: Mass Spectrometry-Based Sample Vectorization to Explore Chemodiverse
      Datasets'
  source_content_hash: sha256:68756fc181756ba5c4b1f224602eb94828e6b884011f7a07e5343587a9ea6bd3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mandelbrot-project/memo
  tools:
  - matchms
  - memo-ms
  - numpy
  - spec2vec
  edam_operation: http://edamontology.org/operation_2945
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# metabolomics-sample-fingerprint-generation

## Summary

Generates MS2-based sample fingerprints by counting the occurrence of MS2 peaks and neutral losses (to the precursor) per sample, then aligns them into a shared sample-by-feature MEMO matrix. This provides a retention-time-agnostic vectorization of LC-MS/MS samples that works without a prior feature alignment step.

## When to use

Use when you have LC-MS/MS samples (e.g. an MZmine-exported aligned feature table plus a GNPS FBMN-exported .mgf spectra file) and need a retention-time-agnostic sample-vs-sample comparison matrix built from MS2 peak/loss occurrence counts.

## When NOT to use

- Analysis requires quantitative use of MS1 feature intensities — MEMO fingerprints record only peak/loss occurrence (presence/absence), ignoring intensities.
- Comparison depends on retention-time behavior (e.g. detecting RT shifts) — MEMO is deliberately retention-time agnostic.
- Samples were compared where a well-behaved feature overlap exists and standard feature-table metrics suffice — MEMO targets chemodiverse samples with poor feature overlap.

## Inputs

- MZmine-exported aligned feature quantification table (.csv)
- GNPS FBMN-exported MS/MS spectra file (.mgf)
- Sample metadata/annotations (optional, for downstream filtering and visualization)

## Outputs

- Per-sample MS2 fingerprint documents (SpectraDocuments: counts of MS2 peaks and neutral losses to precursor)
- Aligned sample-by-feature MEMO matrix (memo_matrix.csv) with count/presence entries for every input sample

## How to apply

Load the MZmine-exported aligned feature quantification table (.csv, e.g. produced with join-aligner at 40 ppm m/z and 0.2 min RT tolerance) together with its GNPS FBMN-exported .mgf MS/MS spectra file using memo_ms.import_data. Convert the MS2 spectra into per-sample documents with memo_ms.SpectraDocuments, counting the occurrence of each MS2 peak and each neutral loss relative to the precursor per sample; MS1 feature intensities are not used — only a presence/absence (count) representation. Build a memo_ms.MemoMatrix object by aligning the per-sample MS2 fingerprints into a shared sample-by-feature count matrix, which is retention-time agnostic and requires no prior feature alignment. Export the aligned MEMO matrix to memo_matrix.csv and verify it contains count/presence entries for every sample in the input feature table; downstream filtering (e.g. removing peaks/losses from blanks) can then be applied.

## Related tools

- **memo-ms** (Core package: imports the MZmine feature table + mgf spectra, builds SpectraDocuments (peak/loss counts) and the aligned MemoMatrix) — https://github.com/mandelbrot-project/memo
- **matchms** (Handling of the MS2 spectra underlying fingerprint construction) — https://github.com/matchms/matchms
- **spec2vec** (Converting MS2 spectra into documents for fingerprint generation) — https://github.com/iomega/spec2vec
- **numpy** (Numerical dependency for the matrix computations in the MEMO install)

## Examples

```
import memo_ms; samples = memo_ms.import_data('features_quant.csv', 'spectra.mgf')
```

## Evaluation signals

- memo_matrix.csv contains a count/presence entry for every sample in the input feature table.
- Matrix values are occurrence counts (presence/absence) of MS2 peaks and neutral losses, not MS1 feature intensities.
- Matrix columns correspond to observed MS2 peaks and neutral losses computed relative to each precursor.
- The matrix is built without requiring a prior MZmine feature-alignment step; samples need not share aligned features.
- Environment reproduces the paper's setup: a dedicated conda env (e.g. python=3.8) with pip-installed memo-ms and numpy.

## Limitations

- MS1 feature intensity is ignored — the fingerprint is presence/absence/count based, so intensity-based quantitative differences between samples are lost.
- Retention-time information is not used; the method cannot resolve RT-dependent effects by design.
- The 'memo ready' feature table import is mentioned in the changelog without file-format or dataset specification.
- A changelog bugfix notes 'Changed classes.py to avoid erasing when filtering', indicating filtering could previously erase data in the tool.

## Evidence

- [readme] The occurence of MS2 peaks and neutral losses (to the precursor) in each sample is counted and used to generate an *MS2 fingerprint* of the sample.: "The occurence of MS2 peaks and neutral losses (to the precursor) in each sample is counted and used to generate an *MS2 fingerprint* of the sample."
- [intro] RT-agnostic alignment: "is a method allowing a Retention Time (RT) agnostic alignment of metabolomics samples using the fragmentation spectra (MS2) of their consituents."
- [methods] environment setup: "conda create --name memo python=3.8"
- [abstract] no prior alignment needed: "such performance was achieved without the requirement of a prior feature alignment step and in a significantly shorter computational time"
- [discussion] memo ready import caveat: "possibility to import "memo ready" feature table"
