---
name: massive-dataset-retrieval
description: Use when reproducing a published metabolomics analysis that cites a MassIVE
  accession (e.g. MSV000087728) and the MS2 peak-list files must be downloaded before
  sample vectorization or matrix building.
when_to_use_negative:
- The dataset is already locally available as a 'memo ready' feature table or aligned
  feature table — no MassIVE retrieval or MS2 peak counting is needed
- The analysis depends on retention-time alignment or requires prior feature alignment;
  MEMO is specifically an RT-agnostic MS2 fingerprinting method
- The target dataset is not the accession named in the source card — do not assume
  accession IDs across papers
edam_topics:
- http://edamontology.org/topic_3173
tools:
- name: memo-ms
  role: pip-installable MEMO package (version 0.1.3) whose import/SpectraDocuments
    and MemoMatrix classes consume the retrieved MS2 peak lists and produce the aligned
    sample-by-sample fingerprint matrix
  repo: https://github.com/mandelbrot-project/memo_publication_examples
- name: matchms
  role: Underlying library used by memo_ms for importing and handling the downloaded
    MS2 spectra
  repo: https://github.com/matchms/matchms
- name: spec2vec
  role: Underlying library used by memo_ms for converting MS2 spectra into documents
    prior to fingerprint generation
  repo: https://github.com/iomega/spec2vec
- name: conda
  role: Creates the isolated python=3.8 environment in which the retrieved dataset
    is processed reproducibly
- name: numpy
  role: Dependency installed via pip alongside memo-ms for matrix handling in the
    reproduction environment
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.3389/fbinf.2022.842964
    title: 'MEMO: Mass Spectrometry-Based Sample Vectorization to Explore Chemodiverse
      Datasets'
  source_content_hash: sha256:8e4cf5f5c794709cd8bb583958630600cfc8018c719e5ce8207495ece25eac9b
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# massive-dataset-retrieval

## Summary

Retrieves a large published LC-MS/MS dataset from MassIVE by accession ID (here MSV000087728: 1,920 plant extracts, 906,509 MS/MS spectra collected in 19 batches) so it can be reprocessed with the MEMO pipeline and its <10 min matrix-computation runtime can be reproduced on commodity hardware.

## When to use

Use when reproducing a published metabolomics analysis that cites a MassIVE accession (e.g. MSV000087728) and the MS2 peak-list files must be downloaded before sample vectorization or matrix building.

## When NOT to use

- The dataset is already locally available as a 'memo ready' feature table or aligned feature table — no MassIVE retrieval or MS2 peak counting is needed
- The analysis depends on retention-time alignment or requires prior feature alignment; MEMO is specifically an RT-agnostic MS2 fingerprinting method
- The target dataset is not the accession named in the source card — do not assume accession IDs across papers

## Inputs

- MassIVE accession identifier (MSV000087728 for the MEMO plant extract benchmark)
- Per-sample MS2 peak list files downloaded from the MassIVE accession
- Reproduction code repository (mandelbrot-project/memo_publication_examples)

## Outputs

- Working directory containing the retrieved MS2 peak list files for 1,920 samples
- Aligned MEMO fingerprint matrix exported as memo_matrix.csv (1,920 x 1,920 sample-by-sample)
- computation_time.txt recording the wall-clock runtime of the matrix build

## How to apply

Locate the accession in the source card or paper — for the MEMO plant extract benchmark this is MSV000087728, a 1,920-sample / 906,509-spectrum plant extract dataset profiled over a year in 19 batches — and download the per-sample MS2 peak list files from MassIVE into a working directory. Clone the authors' reproduction repository (mandelbrot-project/memo_publication_examples) to obtain the exact code and environment recipe. Create a dedicated conda environment with Python 3.8 and install the pinned package versions (memo-ms 0.1.3 via pip, numpy, scikit-bio via conda-forge) so the imported SpectraDocuments objects behave as in the publication. Feed the downloaded peak lists directly into memo_ms import/SpectraDocuments (built on matchms and spec2vec) without any prior feature alignment step, then time the MemoMatrix build with a wall-clock timer to check the reported <10 min laptop runtime. Validate the retrieval and downstream processing by confirming the aligned fingerprint matrix is a square 1,920 x 1,920 sample-by-sample matrix.

## Related tools

- **memo-ms** (pip-installable MEMO package (version 0.1.3) whose import/SpectraDocuments and MemoMatrix classes consume the retrieved MS2 peak lists and produce the aligned sample-by-sample fingerprint matrix) — https://github.com/mandelbrot-project/memo_publication_examples
- **matchms** (Underlying library used by memo_ms for importing and handling the downloaded MS2 spectra) — https://github.com/matchms/matchms
- **spec2vec** (Underlying library used by memo_ms for converting MS2 spectra into documents prior to fingerprint generation) — https://github.com/iomega/spec2vec
- **conda** (Creates the isolated python=3.8 environment in which the retrieved dataset is processed reproducibly)
- **numpy** (Dependency installed via pip alongside memo-ms for matrix handling in the reproduction environment)

## Examples

```
git clone https://github.com/mandelbrot-project/memo_publication_examples.git  # then download MassIVE accession MSV000087728 MS2 peak lists and run the MEMO matrix build
```

## Evaluation signals

- memo_matrix.csv exists and is a square sample-by-sample matrix with exactly 1,920 rows matching the reported sample count
- computation_time.txt contains a wall-clock runtime under 10 minutes for the matrix-building step
- memo-ms resolves to version 0.1.3 in the python=3.8 conda environment, matching the paper's install instructions
- The number of MS/MS spectra loaded (~906,509) is consistent with the reported dataset scale
- The matrix was computed without any prior feature alignment step, as the paper emphasizes

## Limitations

- The changelog provides no documented entry for release 0.1.3 (only a compare link), so behavior of the pinned version must be verified against the publication code rather than release notes
- The 'memo ready' feature table import is mentioned without file-format or dataset specification, so downstream feature-table inputs are underspecified
- The <10 min runtime claim is tied to 'a simple laptop' and this specific 1,920-sample dataset; larger datasets or weaker hardware may exceed it

## Evidence

- [readme] Reproduction repository grounds the retrieval-to-matrix workflow: [evidence span withheld — anchor status: not_found]
- [readme] Pinned install path for the processing environment: [evidence span withheld — anchor status: not_found]
- [methods] Conda environment setup used before processing the retrieved dataset: "conda create --name memo python=3.8"
- [methods] What is computed from the retrieved MS2 peak lists: "The occurrence of MS2 peaks and neutral losses (to the precursor) in each sample is counted"
- [intro] MEMO's defining property relevant to why the raw spectra (not aligned features) are retrieved: "is a method allowing a Retention Time (RT) agnostic alignment of metabolomics samples using the fragmentation spectra (MS2)"
