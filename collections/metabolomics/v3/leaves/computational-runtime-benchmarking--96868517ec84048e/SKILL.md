---
name: computational-runtime-benchmarking
description: Use when validating that a sample-vectorization or matrix-building step
  (e.g., MEMO MS2 fingerprinting of a 1,600+ extract dataset) completes within reported
  computational-time bounds on commodity hardware.
when_to_use_negative:
- The dataset is small (e.g., a handful of samples) where a runtime benchmark is not
  the analysis goal and MEMO's scale-efficiency claim is not being tested
- The analysis goal is clustering-quality comparison against state-of-the-art metrics
  (MDS/PCoA, TMAP, Heatmap evaluation) rather than computational-time measurement
- Input data lacks MS2 fragmentation spectra (MS1-only features), since MEMO fingerprints
  are built from counted MS2 peaks and neutral losses to the precursor
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: memo-ms
  role: package whose MemoMatrix computation is the object of the timing benchmark
    (version 0.1.3 installed via pip)
  repo: https://github.com/mandelbrot-project/memo_publication_examples
- name: numpy
  role: dependency installed alongside memo-ms in the benchmark environment
- name: matchms
  role: underlying package used by memo_ms for handling the MS2 spectra during import
    and fingerprinting
  repo: https://github.com/matchms/matchms
- name: spec2vec
  role: underlying package used for converting MS2 spectra into documents within memo_ms
  repo: https://github.com/iomega/spec2vec
- name: Python 3.8
  role: pinned interpreter version of the conda environment in which the timed computation
    is run
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.3389/fbinf.2022.842964
    title: 'MEMO: Mass Spectrometry-Based Sample Vectorization to Explore Chemodiverse
      Datasets'
  source_content_hash: sha256:9b1481c9c4ed4c1a37ac9b649c2a03b1e41810aa6fd6ebd10fb8f91b5f1cd72f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mandelbrot-project/memo_publication_examples
  tools:
  - Python
  - Python 3.8
  - matchms
  - memo-ms
  - numpy
  - spec2vec
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# computational-runtime-benchmarking

## Summary

Benchmark the wall-clock runtime of MEMO matrix computation — MS2 peak/neutral-loss fingerprinting followed by fingerprint alignment — on a large LC-MS/MS cohort (e.g., 1,920 plant extracts / 906,509 MS/MS spectra) to demonstrate computational efficiency at scale (reported <10 min on a simple laptop).

## When to use

Use when validating that a sample-vectorization or matrix-building step (e.g., MEMO MS2 fingerprinting of a 1,600+ extract dataset) completes within reported computational-time bounds on commodity hardware.

## When NOT to use

- The dataset is small (e.g., a handful of samples) where a runtime benchmark is not the analysis goal and MEMO's scale-efficiency claim is not being tested
- The analysis goal is clustering-quality comparison against state-of-the-art metrics (MDS/PCoA, TMAP, Heatmap evaluation) rather than computational-time measurement
- Input data lacks MS2 fragmentation spectra (MS1-only features), since MEMO fingerprints are built from counted MS2 peaks and neutral losses to the precursor

## Inputs

- MS2 peak list files from MassIVE accession MSV000087728 (1,920 plant extract samples, 906,509 MS/MS spectra)
- conda environment memo with Python 3.8, memo-ms 0.1.3, numpy
- memo_ms import_data / SpectraDocuments spectra objects built on matchms and spec2vec

## Outputs

- computation_time.txt containing the wall-clock elapsed runtime of the matrix-building step
- memo_matrix.csv: aligned fingerprint matrix, square sample-by-sample (1,920 x 1,920 entries)

## How to apply

Set up the memo environment exactly as documented (conda env with Python 3.8, pip install numpy, pip install memo-ms v0.1.3), since runtime claims are tied to that stack. Import the MS2 spectra via memo_ms import_data/SpectraDocuments (built on matchms and spec2vec) from the MassIVE accession MSV000087728 peak lists, explicitly without any prior feature alignment step — the whole point of the benchmark is that MEMO achieves its performance without alignment. Build the MemoMatrix by counting MS2 peaks and neutral losses (to the precursor) per sample and aligning the fingerprints across all 1,920 samples, wrapping only the matrix-building step in a wall-clock timer that writes the elapsed time to computation_time.txt. Export the aligned fingerprint matrix as memo_matrix.csv and validate its dimensions against the reported scale. Judge success both by the recorded runtime (<10 min / 600 s expected) and by structural validation of the output matrix (1,920 x 1,920 sample-by-sample square matrix).

## Related tools

- **memo-ms** (package whose MemoMatrix computation is the object of the timing benchmark (version 0.1.3 installed via pip)) — https://github.com/mandelbrot-project/memo_publication_examples
- **numpy** (dependency installed alongside memo-ms in the benchmark environment)
- **matchms** (underlying package used by memo_ms for handling the MS2 spectra during import and fingerprinting) — https://github.com/matchms/matchms
- **spec2vec** (underlying package used for converting MS2 spectra into documents within memo_ms) — https://github.com/iomega/spec2vec
- **Python 3.8** (pinned interpreter version of the conda environment in which the timed computation is run)

## Examples

```
conda create --name memo python=3.8 && conda activate memo && pip install numpy && pip install memo-ms==0.1.3
```

## Evaluation signals

- computation_time.txt exists and contains a recorded wall-clock runtime below the reported bound of 10 minutes (600 s) for the 1,920-sample dataset
- memo_matrix.csv is a square sample-by-sample matrix with 1,920 rows matching the reported sample count of the MSV000087728 plant dataset
- the conda environment was created with python=3.8 and memo-ms version 0.1.3 per the documentation install commands
- no prior feature alignment step was performed before fingerprinting (the runtime claim is specific to MEMO's alignment-free workflow)
- the timer brackets only the MemoMatrix-building/alignment step, not the spectra download or environment setup

## Limitations

- The <10 min figure is hardware-dependent — reported for 'a simple laptop' — so reproductions on different machines may deviate and the hardware used must be recorded
- The dataset spans 1,920 samples acquired over a year in 19 batches; the runtime benchmark does not itself assess batch-related fingerprint variability
- The memo-ms changelog shows gaps (release 0.1.3 has a compare link but no changelog section describing its contents), so the exact pinned version matters for reproducibility

## Evidence

- [results] For the plant extract dataset of 1920 samples and 906,509 MS/MS spectra, a MEMO matrix was computed in less than 10 min on a simple laptop, demonstrating MEMO's computational efficiency at large scale.: [evidence span withheld — anchor status: not_found]
- [abstract] Performance without feature alignment: "such performance was achieved without the requirement of a prior feature alignment step and in a significantly shorter computational time"
- [methods] Environment setup commands: "conda create --name memo python=3.8
   conda activate memo"
- [readme] Fingerprint definition benchmarked: "The occurence of MS2 peaks and neutral losses (to the precursor) in each sample is counted
and used to generate an *MS2 fingerprint* of the sample"
