---
name: mzmine-feature-table-import
description: Use when the input is a MZmine join-aligned feature table (.csv, produced
  with m/z tolerance 40 ppm and RT tolerance 0.2 min) plus its corresponding GNPS
  FBMN .mgf spectra file, and the goal is to build a MEMO MS2 fingerprint matrix from
  the aligned samples (e.g. MSV000083306 demo data).
when_to_use_negative:
- When quantification depends on MS1 feature intensities — MEMO's aligned-sample workflow
  uses only a presence/absence matrix and ignores MS1 intensity values.
- When the spectra are not MZmine/GNPS FBMN exports or the table is not an aligned
  feature quantification .csv, since this import path assumes the MZmine join-aligner
  output format.
- When a MEMO matrix is already built — this skill is the import/front-end step, not
  the fingerprint-alignment or downstream filtering/visualization step.
tools:
- name: memo-ms
  role: Python package whose import_data routine loads the MZmine aligned .csv feature
    table and the GNPS FBMN .mgf spectra file to seed MS2 fingerprint generation
  repo: https://github.com/mandelbrot-project/memo_publication_examples
- name: matchms
  role: Underlying package used by MEMO for handling the imported MS2 spectra
  repo: https://github.com/matchms/matchms
- name: spec2vec
  role: Underlying package used by MEMO for converting the imported spectra into documents
  repo: https://github.com/iomega/spec2vec
- name: numpy
  role: Dependency installed alongside memo-ms for the matrix/array handling behind
    the fingerprint table
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.3389/fbinf.2022.842964
    title: 'MEMO: Mass Spectrometry-Based Sample Vectorization to Explore Chemodiverse
      Datasets'
  source_content_hash: sha256:8d13a9d4341533bef2aa3ea452b4a6a3a5f3a6a034ce652cc5cca8ab3b0d76a1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mandelbrot-project/memo_publication_examples
  tools:
  - matchms
  - memo-ms
  - numpy
  - spec2vec
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# mzmine-feature-table-import

## Summary

Loads an MZmine-exported aligned feature quantification table (.csv) together with its GNPS FBMN-exported .mgf MS2 spectra file into the memo_ms package (import_data), providing the aligned-sample entry point for MS2-based sample vectorization where only feature presence/absence — not MS1 intensity — is used.

## When to use

Use when the input is a MZmine join-aligned feature table (.csv, produced with m/z tolerance 40 ppm and RT tolerance 0.2 min) plus its corresponding GNPS FBMN .mgf spectra file, and the goal is to build a MEMO MS2 fingerprint matrix from the aligned samples (e.g. MSV000083306 demo data).

## When NOT to use

- When quantification depends on MS1 feature intensities — MEMO's aligned-sample workflow uses only a presence/absence matrix and ignores MS1 intensity values.
- When the spectra are not MZmine/GNPS FBMN exports or the table is not an aligned feature quantification .csv, since this import path assumes the MZmine join-aligner output format.
- When a MEMO matrix is already built — this skill is the import/front-end step, not the fingerprint-alignment or downstream filtering/visualization step.

## Inputs

- MZmine-exported aligned feature quantification table (.csv) from a join-aligner step (m/z tolerance 40 ppm, RT tolerance 0.2 min)
- Corresponding GNPS FBMN-exported MS2 spectra file (.mgf)
- Reference dataset: MSV000083306 demo dataset used in the MEMO publication examples

## Outputs

- Loaded spectra and feature-table objects ready for memo_ms.SpectraDocuments conversion
- Per-sample MS2 fingerprint documents (peak and neutral-loss occurrence counts, presence/absence)
- memo_ms.MemoMatrix object exported as memo_matrix.csv — a sample-by-feature count/presence matrix covering every sample in the input feature table

## How to apply

Load the MZmine-exported aligned feature quantification .csv and its paired GNPS FBMN .mgf spectra file via memo_ms.import_data (e.g. the MSV000083306 demo dataset). The aligned table should have been generated upstream by MZmine's join-aligner step using m/z tolerance 40 ppm and RT tolerance 0.2 min. The import serves to define the per-sample feature set: during downstream MemoMatrix construction, the feature intensity table is consulted only to determine whether each feature is detected in a sample, and the occurrence of each MS2 peak and each neutral loss (relative to the precursor) is counted while MS1 intensities are ignored. The loaded per-sample MS2 documents are then aligned into a shared sample-by-feature count matrix that is retention-time agnostic. Because MEMO relies on matchms and spec2vec for handling MS2 spectra and converting them into documents, install the stack in a dedicated conda environment (conda create --name memo python=3.8; note the Python requirement was later relaxed to >3.7 for TMAP compatibility).

## Related tools

- **memo-ms** (Python package whose import_data routine loads the MZmine aligned .csv feature table and the GNPS FBMN .mgf spectra file to seed MS2 fingerprint generation) — https://github.com/mandelbrot-project/memo_publication_examples
- **matchms** (Underlying package used by MEMO for handling the imported MS2 spectra) — https://github.com/matchms/matchms
- **spec2vec** (Underlying package used by MEMO for converting the imported spectra into documents) — https://github.com/iomega/spec2vec
- **numpy** (Dependency installed alongside memo-ms for the matrix/array handling behind the fingerprint table)

## Examples

```
from memo_ms import import_data  # load MZmine aligned feature table (.csv) + GNPS FBMN .mgf spectra, e.g. MSV000083306 demo dataset
```

## Evaluation signals

- The exported memo_matrix.csv contains a count/presence entry for every sample present in the input MZmine aligned feature table (row-set equality check).
- Matrix values are occurrence counts / presence indicators, not MS1 intensities — verify that no intensity-scale values from the feature table leaked into the matrix.
- Columns correspond to MS2 peaks and neutral losses (relative to the precursor), matching the fingerprint definition of the method.
- The alignment is retention-time agnostic: the resulting matrix should be unaffected by RT shifts between samples, consistent with MEMO's design goal.
- Sample identities in memo_matrix.csv match the sample columns of the MZmine aligned table exactly (no dropped or renamed samples).

## Limitations

- The MS1 feature intensity is not used — only a presence/absence occurrence matrix — so this skill cannot support intensity-weighted comparisons.
- Requires the table to have been aligned upstream in MZmine (join-aligner, m/z 40 ppm, RT 0.2 min); MEMO's core strength is that no prior feature alignment is needed, but this aligned-table import path presumes one exists.
- The 'memo ready' feature table import is mentioned in the changelog without file format or dataset specification, so format expectations may need to be verified against the demo dataset (MSV000083306).
- TMAP compatibility requires Python >3.7; earlier releases pinned >3.8, which can constrain environment setup.

## Evidence

- [readme] ms2 peak/loss counting defines fingerprints: "The occurence of MS2 peaks and neutral losses (to the precursor) in each sample is counted and used to generate an *MS2 fingerprint* of the sample."
- [discussion] memo-ready feature table import exists: "possibility to import "memo ready" feature table"
- [methods] dedicated conda environment for install: "conda create --name memo python=3.8
   conda activate memo"
