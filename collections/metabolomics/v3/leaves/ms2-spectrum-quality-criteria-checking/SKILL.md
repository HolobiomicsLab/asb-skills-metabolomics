---
name: ms2-spectrum-quality-criteria-checking
description: Use when building a large-scale MS/MS training corpus (e.g. from GeMS-style
  mining of 250,000 LC–MS/MS experiments) and redundant or low-quality spectra must
  be filtered into nested quality tiers.
when_to_use_negative:
- Input is already a curated spectral library (e.g. MoNA or NIST20) rather than raw
  unannotated repository-scale MS/MS data — library spectra do not need mining-based
  quality filtering.
- Only a single instrument's data is available; per-instrument m/z accuracy filtering
  (GeMS-B) is calibrated across the mixed Orbitrap/QTOF repository collection.
- The goal is one-vs-one spectral library matching rather than constructing a pre-training
  corpus of hundreds of millions of unannotated spectra.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: HDF5
  role: Binary storage format for the GeMS spectra and their LC–MS/MS metadata, designed
    to be compact for deep learning workflows
- name: DreaMS
  role: Downstream model whose self-supervised pre-training consumes the filtered
    GeMS subsets produced by this skill
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41587-025-02663-3
    title: Self-supervised learning of molecular representations from millions of
      tandem mass spectra using DreaMS
  source_content_hash: sha256:3f634df2df508232615a40f15cce2659f58fa896e064ab3f3207a7c854eb224f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DreaMS
  - HDF5
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# ms2-spectrum-quality-criteria-checking

## Summary

Quality-control pipeline that filters hundreds of millions of mined MS/MS spectra from MassIVE GNPS into the GeMS-A, GeMS-B and GeMS-C subsets using file-level (LC–MS/MS experiment) and spectrum-level checks, incrementally trading subset size for spectral quality.

## When to use

Use when building a large-scale MS/MS training corpus (e.g. from GeMS-style mining of 250,000 LC–MS/MS experiments) and redundant or low-quality spectra must be filtered into nested quality tiers.

## When NOT to use

- Input is already a curated spectral library (e.g. MoNA or NIST20) rather than raw unannotated repository-scale MS/MS data — library spectra do not need mining-based quality filtering.
- Only a single instrument's data is available; per-instrument m/z accuracy filtering (GeMS-B) is calibrated across the mixed Orbitrap/QTOF repository collection.
- The goal is one-vs-one spectral library matching rather than constructing a pre-training corpus of hundreds of millions of unannotated spectra.

## Inputs

- ~250,000 LC–MS/MS experiments mined from the MassIVE GNPS repository
- HDF5-based GeMS spectra files with selected LC–MS/MS metadata (e.g. instrument type)
- Extracted-ion chromatograms (XICs) used by Algorithm 1 for instrument m/z accuracy checks

## Outputs

- GeMS-A subset (~42 million highest-quality spectra; ~97% Orbitrap instruments)
- GeMS-B subset (~100 million spectra, adds instrument m/z accuracy criterion)
- GeMS-C subset (~201 million spectra, adds high-intensity signal count criterion; 52% Orbitrap / 41% QTOF)
- LSH clusters with random-sampled size caps (10 or 1,000 spectra per cluster)
- HDF5-formatted filtered spectra with LC–MS/MS metadata

## How to apply

Load the mined collection of ~250,000 LC–MS/MS experiments from MassIVE GNPS stored in the HDF5-based GeMS format. Apply file-level quality criteria first — checks such as retention-time and MS-level ordering of the LC–MS/MS experiment — then Algorithm 1, which filters files by LC–MS instrument m/z accuracy using extracted-ion chromatograms (XICs). Next apply Algorithm 2 at the spectrum level, checking peak count, precursor charge, m/z range and consistency of centroids with the precursor m/z. Incrementally stack criteria into three nested subsets: GeMS-A (basic criteria, ~42 million spectra), GeMS-B (adds instrument m/z accuracy, ~100 million) and GeMS-C (adds number of high-intensity signals, ~201 million), each larger but of consecutively lower quality. To address redundancy, cluster similar spectra with locality-sensitive hashing (LSH) and limit cluster sizes by random sampling (e.g. 10 or 1,000 spectra per cluster). Store the filtered spectra together with selected LC–MS/MS metadata such as instrument type in the compact HDF5-based binary format designed for deep learning.

## Related tools

- **HDF5** (Binary storage format for the GeMS spectra and their LC–MS/MS metadata, designed to be compact for deep learning workflows)
- **DreaMS** (Downstream model whose self-supervised pre-training consumes the filtered GeMS subsets produced by this skill)

## Evaluation signals

- Resulting subsets contain approximately 42, 100 and 201 million spectra for GeMS-A, GeMS-B and GeMS-C respectively
- Instrument-type composition matches expectations: ~97% Orbitrap in GeMS-A; GeMS-C 52% Orbitrap and 41% QTOF
- Quality of the subsets is inversely related to size (consecutively larger size at the expense of quality)
- Each spectrum in the subsets passes Algorithm 2 checks: peak count, precursor charge, m/z range and centroid–precursor m/z consistency
- Redundancy is controlled: clusters from LSH are capped by random sampling at 10 or 1,000 spectra per cluster

## Limitations

- Higher-quality subsets (GeMS-A) cover far fewer spectra (~42 M) than the full-filtered set (GeMS-C, ~201 M), so quality and coverage trade off directly
- The pipeline focuses on positive-mode, singly charged precursor spectra; negative-ionization-mode and multiply charged spectra are not handled
- Only tandem MS is considered — MS1 isotopic patterns and adduct distributions are disregarded
- Quality tiers depend on instrument m/z accuracy, so subsets are biased toward accurate instruments (e.g. Orbitrap-dominated GeMS-A)

## Evidence

- [results] Quality criteria yield three nested subsets of increasing size and decreasing quality: [evidence span withheld — anchor status: not_found]
- [results] Instrument composition of GeMS subsets: [evidence span withheld — anchor status: not_found]
- [methods] Filtering into GeMS-A/B/C tiers: [evidence span withheld — anchor status: not_found]
- [methods] Reported subset sizes: [evidence span withheld — anchor status: not_found]
- [methods] Spectrum-level Algorithm 2 checks: [evidence span withheld — anchor status: not_found]
- [abstract] HDF5 storage of GeMS with metadata: "we stored the GeMS spectra, including selected LC–MS/MS metadata, in our compact HDF5-based binary format designed for deep learning"
- [abstract] Quality control pipeline producing three subsets: "we developed a pipeline of quality control algorithms allowing us to filter the collected spectra into three subsets—GeMS-A, GeMS-B and GeMS-C"
