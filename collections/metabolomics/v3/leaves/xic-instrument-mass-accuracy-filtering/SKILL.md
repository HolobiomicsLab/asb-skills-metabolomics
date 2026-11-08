---
name: xic-instrument-mass-accuracy-filtering
description: Use when you have a repository-scale collection of LC–MS/MS experiments
  (e.g. mzML files mined from MassIVE GNPS) and need to filter out files/spectra with
  unreliable precursor m/z before model training, grouping them into quality tiers.
when_to_use_negative:
- When the input is already a curated spectral library (e.g. MoNA or NIST20) with
  known-good reference spectra — the accuracy filter is designed for uncurated repository-scale
  mining output, not validated library entries.
- When analyzing a single small experiment where per-file m/z accuracy can be assessed
  directly by standard QC — the two-algorithm pipeline is aimed at repository-scale
  redundancy and quality triage.
- When downstream tasks require multiply charged or negative-mode spectra, since the
  GeMS mining and filtering pipeline targets predominantly positive-mode, singly charged
  metabolomics spectra.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: HDF5
  role: Binary storage format for persisting the quality-filtered GeMS subsets together
    with selected LC–MS/MS metadata for downstream deep learning
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41587-025-02663-3
    title: Self-supervised learning of molecular representations from millions of
      tandem mass spectra using DreaMS
  source_content_hash: sha256:2f2bb763e0b536a12c1812511977190e288c890da417b6dba89c51688de67591
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - HDF5
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# XIC instrument mass-accuracy filtering

## Summary

Estimates the m/z accuracy of each LC–MS/MS experiment by inspecting extracted-ion chromatograms (XICs) and uses the resulting instrument-level accuracy scores to partition ~700 million mined MassIVE GNPS spectra into the GeMS-A, GeMS-B and GeMS-C quality subsets.

## When to use

Use when you have a repository-scale collection of LC–MS/MS experiments (e.g. mzML files mined from MassIVE GNPS) and need to filter out files/spectra with unreliable precursor m/z before model training, grouping them into quality tiers.

## When NOT to use

- When the input is already a curated spectral library (e.g. MoNA or NIST20) with known-good reference spectra — the accuracy filter is designed for uncurated repository-scale mining output, not validated library entries.
- When analyzing a single small experiment where per-file m/z accuracy can be assessed directly by standard QC — the two-algorithm pipeline is aimed at repository-scale redundancy and quality triage.
- When downstream tasks require multiply charged or negative-mode spectra, since the GeMS mining and filtering pipeline targets predominantly positive-mode, singly charged metabolomics spectra.

## Inputs

- Collection of raw LC–MS/MS experiments from MassIVE GNPS (~250,000 experiments, e.g. mzML-derived files) mined into the GeMS format
- Per-experiment metadata such as instrument type and LC–MS/MS run information

## Outputs

- Three nested quality-filtered subsets: GeMS-A (~42M spectra), GeMS-B (~100M spectra), GeMS-C (~201M spectra)
- Per-spectrum and per-file quality annotations (instrument m/z accuracy, instrument type) persisted in an HDF5-based binary format for deep learning

## How to apply

Run Algorithm 1 at the file level: for each LC–MS/MS experiment, reconstruct XICs to estimate the instrument's m/z accuracy, since high-resolution instruments (Orbitrap, QTOF) show tighter extracted-peak m/z consistency than low-resolution devices. Combine this instrument-level accuracy score with file-level checks (retention-time and MS-level ordering) and spectrum-level checks from Algorithm 2 (peak count, precursor charge, m/z range, and consistency of centroids with precursor m/z). Apply the criteria incrementally: GeMS-A accepts spectra passing the basic checks, GeMS-B adds the instrument m/z-accuracy requirement, and GeMS-C further requires a minimum number of high-intensity signals, yielding ~42M, ~100M and ~201M spectra respectively. Store the surviving spectra together with selected LC–MS/MS metadata (e.g. instrument type) in a compact HDF5-based binary format designed for deep learning. Validate that the subsets are nested and that instrument composition matches expectations — e.g. 97% of GeMS-A spectra acquired on Orbitrap instruments versus 52% Orbitrap and 41% QTOF in GeMS-C.

## Related tools

- **HDF5** (Binary storage format for persisting the quality-filtered GeMS subsets together with selected LC–MS/MS metadata for downstream deep learning)

## Evaluation signals

- The three subsets should be strictly nested in size: GeMS-A ≈ 42M, GeMS-B ≈ 100M, GeMS-C ≈ 201M spectra.
- Instrument composition checks: ~97% of GeMS-A spectra should be Orbitrap-acquired; GeMS-C should contain roughly 52% Orbitrap and 41% QTOF spectra.
- Algorithm 1 should flag LC–MS/MS files whose XIC-derived m/z accuracy falls below the instrument accuracy threshold, and those files' spectra should be absent from GeMS-B/GeMS-C.
- Algorithm 2 should reject spectra with inconsistent centroids relative to precursor m/z, wrong precursor charge or m/z range, and insufficient peak counts.
- The resulting subsets must serialize into the compact HDF5-based GeMS format without loss of selected LC–MS/MS metadata fields.

## Limitations

- Filtering is restricted to spectra from positive ionization mode with singly charged precursor ions; other regimes are not covered by the pipeline.
- The pipeline considers only tandem mass spectra and ignores MS1 isotopic patterns and adduct distributions, so m/z accuracy estimated from XICs reflects a limited signal dimension.
- Even the highest-quality GeMS-A subset is only 97% Orbitrap, meaning a small fraction of lower-accuracy instruments remains, so absolute m/z accuracy is not guaranteed per spectrum.

## Evidence

- [results] Quality criteria yield three subsets of consecutively larger size at the expense of quality: 97% of GeMS-A spectra were acquired on Orbitrap instruments: [evidence span withheld — anchor status: not_found]
- [methods] Algorithm 1 filters files by LC–MS instrument accuracy using XICs: [evidence span withheld — anchor status: not_found]
- [methods] Algorithm 2 checks peak count, precursor charge, m/z range and the consistency of centroids with precursor m/z: [evidence span withheld — anchor status: not_found]
- [methods] Resulting subsets contain approximately 42, 100 and 201 million spectra respectively: [evidence span withheld — anchor status: not_found]
- [abstract] stored the GeMS spectra, including selected LC–MS/MS metadata, in our compact HDF5-based binary format: "we stored the GeMS spectra, including selected LC–MS/MS metadata, in our compact HDF5-based binary format designed for deep learning"
- [abstract] filter the collected spectra into three subsets—GeMS-A, GeMS-B and GeMS-C: "we developed a pipeline of quality control algorithms allowing us to filter the collected spectra into three subsets—GeMS-A, GeMS-B and GeMS-C"
- [discussion] our method is focused solely on tandem mass spectra, disregarding other important features such as MS1 isotopic patterns or adduct distributions: "our method is focused solely on tandem mass spectra, disregarding other important features such as MS1 isotopic patterns or adduct distributions"
