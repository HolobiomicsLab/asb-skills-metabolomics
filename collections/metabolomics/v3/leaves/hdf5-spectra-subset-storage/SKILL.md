---
name: hdf5-spectra-subset-storage
description: Use when a mined or filtered collection of tandem mass spectra (e.g.
  hundreds of millions of peak lists from .mzML files) must be persisted in a compact,
  deep-learning-ready binary format with per-spectrum LC–MS/MS metadata such as instrument
  type.
when_to_use_negative:
- When the input is already a curated spectral library in MGF/MSP format with annotations
  — the GeMS/HDF5 path targets unannotated repository-scale raw mined spectra, not
  small annotated reference libraries (e.g. MoNA, NIST20).
- When a downstream tool requires per-file mzML documents or vendor formats (e.g.
  for SIRIUS-style formula annotation pipelines) rather than a flattened spectral
  tensor container.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: HDF5
  role: Binary storage container used to serialize GeMS spectra and selected LC–MS/MS
    metadata in a compact deep-learning-ready format
- name: DreaMS
  role: Consumer of the stored GeMS HDF5 subsets for self-supervised pre-training
    and for constructing the DreaMS Atlas over 201 million spectra
  repo: https://github.com/pluskal-lab/DreaMS
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41587-025-02663-3
    title: Self-supervised learning of molecular representations from millions of
      tandem mass spectra using DreaMS
  source_content_hash: sha256:1efdecf32c1d51c48696ab15cbe78cf98a6b8eb887981da72da521ce88bce2c0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pluskal-lab/DreaMS
  tools:
  - DreaMS
  - HDF5
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# hdf5-spectra-subset-storage

## Summary

Stores large-scale MS/MS spectral collections (such as the GeMS subsets mined from MassIVE GNPS) together with selected LC–MS/MS metadata in a compact HDF5-based binary format designed for deep learning workloads.

## When to use

Use when a mined or filtered collection of tandem mass spectra (e.g. hundreds of millions of peak lists from .mzML files) must be persisted in a compact, deep-learning-ready binary format with per-spectrum LC–MS/MS metadata such as instrument type.

## When NOT to use

- When the input is already a curated spectral library in MGF/MSP format with annotations — the GeMS/HDF5 path targets unannotated repository-scale raw mined spectra, not small annotated reference libraries (e.g. MoNA, NIST20).
- When a downstream tool requires per-file mzML documents or vendor formats (e.g. for SIRIUS-style formula annotation pipelines) rather than a flattened spectral tensor container.

## Inputs

- Filtered MS/MS spectra collections (GeMS-A, GeMS-B, GeMS-C subsets of consecutively larger size and lower quality)
- Selected LC–MS/MS metadata per experiment/spectrum (e.g. instrument type)
- Source raw LC–MS/MS experiments from the MassIVE GNPS repository

## Outputs

- Compact HDF5-based binary file(s) containing the GeMS spectra and their metadata, ready for deep learning

## How to apply

After mining LC–MS/MS experiments (250,000 LC–MS/MS experiments from MassIVE GNPS) and filtering spectra into quality subsets via the QC algorithms (file-level checks on retention-time/MS ordering and instrument m/z accuracy using XICs, plus spectrum-level checks on peak count, precursor charge, m/z range and precursor-consistency of centroids), write the resulting subsets — GeMS-A (~42M), GeMS-B (~100M) and GeMS-C (~201M) — into a compact HDF5-based binary format. Include selected LC–MS/MS metadata per spectrum (e.g. instrument type) so downstream self-supervised pre-training (masked peak prediction, retention order prediction) and annotation tasks can consume the data without re-parsing raw .mzML. The nested-subset structure (A ⊂ B ⊂ C) lets training pipelines choose quality-vs-size tradeoffs: 97% of highest-quality GeMS-A spectra were acquired on Orbitrap instruments, whereas GeMS-C comprises 52% Orbitrap and 41% QTOF spectra. Validate the write by checking subset sizes (42, 100 and 201 million spectra) and by spot-checking that instrument-type fields and spectral peaks round-trip correctly through the HDF5 container.

## Related tools

- **HDF5** (Binary storage container used to serialize GeMS spectra and selected LC–MS/MS metadata in a compact deep-learning-ready format)
- **DreaMS** (Consumer of the stored GeMS HDF5 subsets for self-supervised pre-training and for constructing the DreaMS Atlas over 201 million spectra) — https://github.com/pluskal-lab/DreaMS

## Evaluation signals

- The three stored subsets contain approximately 42, 100 and 201 million spectra respectively (GeMS-A/B/C).
- Instrument-type metadata stored with the spectra matches the reported composition (GeMS-A: 97% Orbitrap; GeMS-C: 52% Orbitrap and 41% QTOF).
- A round-trip read of the HDF5 file reproduces the spectral peaks and precursor values of the source spectra without corruption.
- The subset nesting holds: GeMS-A ⊂ GeMS-B ⊂ GeMS-C with quality decreasing as size increases.
- The stored container can be loaded directly by a deep-learning training loop (e.g. DreaMS pre-training) without re-parsing raw .mzML files.

## Limitations

- The format is described as designed for deep learning on GeMS data; no standard interchange guarantees for arbitrary downstream tools.
- Only a subset of LC–MS/MS metadata is stored — full acquisition details (e.g. collision energies) are not preserved.
- Storage is tied to the mined positive-mode, singly charged precursor spectra used in this work; spectra beyond positive ionization mode or multiply charged precursor ions were not incorporated.

## Evidence

- [abstract] we stored the GeMS spectra, including selected LC–MS/MS metadata, in our compact HDF5-based binary format designed for deep learning: "we stored the GeMS spectra, including selected LC–MS/MS metadata, in our compact HDF5-based binary format designed for deep learning"
- [results] Quality criteria yield three subsets of consecutively larger size at the expense of quality: [evidence span withheld — anchor status: not_found]
- [methods] Algorithm 2 at the spectrum level checks peak count, precursor charge, m/z range: [evidence span withheld — anchor status: not_found]
- [discussion] the MIST CANOPUS benchmark is limited in size, covering 8,000 spectra from 7,000 unique molecules: "the MIST CANOPUS benchmark is limited in size, covering 8,000 spectra from 7,000 unique molecules"
