---
name: lcms-file-level-quality-filtering
description: Use when building training corpora from repository-scale LC-MS/MS deposits
  (e.g. MassIVE GNPS .mzML/.mzXML experiments) and you need to drop low-quality files
  before spectrum-level QC.
when_to_use_negative:
- Input is already a curated spectral library such as MoNA or NIST20 rather than raw
  mined repository experiments
- You need spectrum-level filtering (peak count, precursor charge, m/z range, centroid-consistency
  checks) — this skill covers only file-level checks (Algorithm 1), not Algorithm
  2
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: HDF5
  role: Storage format for the mined GeMS spectra and selected LC-MS/MS metadata that
    the filtering pipeline reads and writes
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41587-025-02663-3
    title: Self-supervised learning of molecular representations from millions of
      tandem mass spectra using DreaMS
  source_content_hash: sha256:444ed66c898583c22bc2708072c1b370a43f086bbb47ac7a63bb206b450f0afd
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - HDF5
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# LC-MS file-level quality filtering

## Summary

Filter LC-MS/MS experiment files mined from MassIVE GNPS by file-level quality checks before spectrum-level filtering into GeMS-A/B/C subsets, using retention-time/MS-level ordering checks and an XIC-based instrument m/z-accuracy filter (Algorithm 1).

## When to use

Use when building training corpora from repository-scale LC-MS/MS deposits (e.g. MassIVE GNPS .mzML/.mzXML experiments) and you need to drop low-quality files before spectrum-level QC.

## When NOT to use

- Input is already a curated spectral library such as MoNA or NIST20 rather than raw mined repository experiments
- You need spectrum-level filtering (peak count, precursor charge, m/z range, centroid-consistency checks) — this skill covers only file-level checks (Algorithm 1), not Algorithm 2

## Inputs

- Collection of LC-MS/MS experiments mined from MassIVE GNPS (~250,000 experiments) stored in the compact HDF5-based GeMS binary format, including instrument-type metadata

## Outputs

- Quality-passed subset of LC-MS/MS experiments that feeds Algorithm 2 spectrum-level filtering into GeMS-A (~42M spectra), GeMS-B (~100M) and GeMS-C (~201M) subsets with instrument-type composition (e.g. GeMS-A 97% Orbitrap, GeMS-C 52% Orbitrap / 41% QTOF)

## How to apply

Start from the mined collection of 250,000 LC-MS/MS experiments stored in the HDF5-based GeMS format. First run file-level checks that retention times and MS levels are properly ordered within each experiment; then apply Algorithm 1, which filters files by LC-MS instrument accuracy using extracted-ion chromatograms (XICs). Files passing these checks feed the downstream spectrum-level Algorithm 2, whose incremental application yields the nested GeMS-A, GeMS-B (adds instrument m/z accuracy) and GeMS-C (adds number of high-intensity signals) subsets of roughly 42, 100 and 201 million spectra.

## Related tools

- **HDF5** (Storage format for the mined GeMS spectra and selected LC-MS/MS metadata that the filtering pipeline reads and writes)

## Evaluation signals

- Resulting subsets contain approximately 42, 100 and 201 million spectra respectively
- Instrument-type composition matches the reported composition (GeMS-A: 97% Orbitrap; GeMS-C: 52% Orbitrap, 41% QTOF)
- Files retained show consistent XIC-based instrument m/z accuracy (Algorithm 1)
- Retention times and MS levels pass ordering checks within each experiment

## Limitations

- Filtering trades size against quality: GeMS-A is the smallest but highest-quality subset, GeMS-C the largest but least stringent
- Instrument-composition skew means GeMS-A is dominated by Orbitrap data (97%), limiting QTOF coverage in the highest-quality tier

## Evidence

- [abstract] File-level ordering and instrument-accuracy filtering rationale: "we developed a pipeline of quality control algorithms allowing us to filter the collected spectra into three subsets—GeMS-A, GeMS-B and GeMS-C"
- [methods] Algorithm 1 described in source card: [evidence span withheld — anchor status: not_found]
- [results] Instrument composition outcome: [evidence span withheld — anchor status: not_found]
- [abstract] Storage format for filtered spectra: "we stored the GeMS spectra, including selected LC–MS/MS metadata, in our compact HDF5-based binary format designed for deep learning"
- [results] Subset sizes validation target: [evidence span withheld — anchor status: not_found]
