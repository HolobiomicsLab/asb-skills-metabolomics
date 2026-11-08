---
name: ms2-fingerprint-distance-matrix-construction
description: Use when samples were acquired under different LC methods (e.g. C18 vs
  C18 RT-shift) or show poor feature overlap, and you need RT-agnostic sample distances
  from MS2 spectra instead of an aligned feature table.
when_to_use_negative:
- Input is only a feature abundance table without MS2 spectra — compute Bray-Curtis
  directly with scikit-bio instead
- Retention-time information is the signal of interest, since MEMO is deliberately
  retention-time agnostic
- Samples contain few or no MS2 acquisitions, leaving fingerprints too sparse to align
  meaningfully
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: memo-ms
  role: 'Core package: SpectraDocuments count MS2 peaks/neutral losses per sample;
    MemoMatrix builds aligned/unaligned sample-sample distance matrices'
  repo: https://github.com/mandelbrot-project/memo_publication_examples
- name: matchms
  role: Underlying library used by MEMO for handling MS2 spectra
  repo: https://github.com/matchms/matchms
- name: spec2vec
  role: Underlying library used by MEMO for converting MS2 spectra into documents
  repo: https://github.com/iomega/spec2vec
- name: scikit-bio
  role: Computes Bray-Curtis dissimilarity from the feature table and runs PERMANOVA
    (999 permutations) on the distance matrices
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.3389/fbinf.2022.842964
    title: 'MEMO: Mass Spectrometry-Based Sample Vectorization to Explore Chemodiverse
      Datasets'
  source_content_hash: sha256:1c4698316323883efe2a1d08f8d1401d07e7947a0a170ccb7f73b05aae071d46
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mandelbrot-project/memo_publication_examples
  tools:
  - matchms
  - memo-ms
  - numpy
  - scikit-bio
  - spec2vec
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# ms2-fingerprint-distance-matrix-construction

## Summary

Constructs sample-sample distance matrices from LC-MS/MS fragmentation data by counting MS2 peaks and neutral losses per sample into MS2 fingerprints (via memo-ms/matchms/spec2vec), then aligning fingerprints to compare samples. This yields retention-time-agnostic distances suited to chemodiverse datasets, unlike feature-table Bray-Curtis.

## When to use

Use when samples were acquired under different LC methods (e.g. C18 vs C18 RT-shift) or show poor feature overlap, and you need RT-agnostic sample distances from MS2 spectra instead of an aligned feature table.

## When NOT to use

- Input is only a feature abundance table without MS2 spectra — compute Bray-Curtis directly with scikit-bio instead
- Retention-time information is the signal of interest, since MEMO is deliberately retention-time agnostic
- Samples contain few or no MS2 acquisitions, leaving fingerprints too sparse to align meaningfully

## Inputs

- LC-MS/MS spectra files (e.g. mzML) with MS2 fragmentation data per sample
- Sample metadata table with factors (e.g. 'experiment', 'composition')
- GNPS FBMN/Qemistree job artifacts from MassIVE deposit MSV000083306 (for the benchmark reproduction)

## Outputs

- Aligned sample-sample MEMO distance matrix
- Unaligned sample-sample MEMO distance matrix
- permanova_results.csv comparing pseudo-F values across MEMO, Qemistree UniFrac, CSCS, and Bray-Curtis distances

## How to apply

Load the MS2 spectra and sample metadata into memo-ms, using SpectraDocuments to count the occurrence of MS2 peaks and neutral losses (to the precursor) per sample — memo-ms is built on matchms and spec2vec for spectral handling and document conversion. Build a MemoMatrix and generate both an aligned and an unaligned sample-sample distance matrix; the alignment step merges fingerprint entries across samples. Optionally filter the fingerprints (e.g. remove peaks/losses originating from blanks) before computing distances. For benchmarking on the Qemistree evaluation dataset (27 samples, Q-Exactive, MassIVE MSV000083306), compare these matrices against Qemistree UniFrac/CSCS and feature-table Bray-Curtis distances using scikit-bio PERMANOVA (999 permutations) on 'experiment' and 'composition' factors.

## Related tools

- **memo-ms** (Core package: SpectraDocuments count MS2 peaks/neutral losses per sample; MemoMatrix builds aligned/unaligned sample-sample distance matrices) — https://github.com/mandelbrot-project/memo_publication_examples
- **matchms** (Underlying library used by MEMO for handling MS2 spectra) — https://github.com/matchms/matchms
- **spec2vec** (Underlying library used by MEMO for converting MS2 spectra into documents) — https://github.com/iomega/spec2vec
- **scikit-bio** (Computes Bray-Curtis dissimilarity from the feature table and runs PERMANOVA (999 permutations) on the distance matrices)

## Evaluation signals

- On the 27-sample Qemistree evaluation set, PERMANOVA pseudo-F for the 'experiment' factor must drop to <= 81.64 with MEMO distances (vs 113.55 for feature-table Bray-Curtis), indicating RT batch-effect mitigation
- Pseudo-F for the 'composition' factor must reach >= 11.10 (vs 4.72 for Qemistree/feature-table), confirming better sample discrimination
- Distance matrices must be symmetric with zero diagonal and identical sample ordering across aligned and unaligned versions
- Fingerprint counts must cover both MS2 peaks and neutral losses to the precursor; verify losses are not dropped during SpectraDocuments generation
- Results should be reproducible under scikit-bio permanova with a fixed seed over 999 permutations

## Limitations

- Fingerprint filtering can erase entries if implemented incorrectly (a classes.py bug fixed in memo changelog); verify filtering does not blank the matrix
- MEMO's RT-agnostic design discards retention-time information by construction, so it cannot separate samples differing only in RT behavior
- Requires MS2 fragmentation spectra; the article notes MEMO is aimed at chemodiverse samples, i.e. poor feature overlap or strong RT shift, different LC methods, or different mass spectrometers (Maxis Q-ToF vs Q-Exactive Orbitrap)

## Evidence

- [intro] MS2 fingerprint counting: "The occurence of MS2 peaks and neutral losses (to the precursor) in each sample is counted and used to generate an *MS2 fingerprint* of the sample."
- [methods] fingerprint alignment: "These fingerprints can in a second stage be aligned to compare different samples."
- [methods] matchms/spec2vec dependency: "MEMO is mainly built on `matchms`_ and `spec2vec`_ packages for handling the MS2 spectra and converting them into documents"
- [abstract] retention-time-agnostic clustering: "a method allowing to cluster large amounts of chemodiverse samples based on their LC-MS/MS profiles in a retention time agnostic manner"
- [readme] filtering and visualization options: "different filtering (remove peaks/losses from blanks for example) and visualization techniques (MDS/PCoA, TMAP, Heatmap, ...) can be used."
- [intro] scope: strong RT shift / instrument diversity: [evidence span withheld — anchor status: not_found]
