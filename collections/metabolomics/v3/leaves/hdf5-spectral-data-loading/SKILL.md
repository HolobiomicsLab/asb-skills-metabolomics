---
name: hdf5-spectral-data-loading
description: Use when the input is a GeMS or DreaMS Zenodo HDF5 deposit of MS/MS spectra
  (e.g., the GeMS-A10 subset for pre-training analyses) and spectra plus metadata
  must be read into PyTorch/numpy pipelines.
when_to_use_negative:
- Input spectra are raw instrument files (.mzML/.mgf) not yet converted into the DreaMS
  HDF5 format — conversion/mining must happen first
- The goal is interactive spectral-library browsing or GNPS molecular networking rather
  than feeding tensors to the DreaMS transformer
- Spectra fall outside the paper's scope (e.g., negative ionization mode or multiply
  charged precursors), which the current DreaMS training data do not cover
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: HDF5
  role: Binary storage format in which GeMS spectra and LC–MS/MS metadata are persisted
    for deep-learning loading
- name: pluskal-lab/DreaMS
  role: Official repository with the loading utilities, requirements, and tutorials
    for reading GeMS HDF5 files and the pre-trained model
  repo: github.com__pluskal-lab__DreaMS
provenance:
  source_task_ids:
  - task_004
  - task_006
  source_papers:
  - doi: 10.1038/s41587-025-02663-3
    title: Self-supervised learning of molecular representations from millions of
      tandem mass spectra using DreaMS
  source_content_hash: sha256:32ae5f25231d5333a41b2e460ad6d1ebcdfc631ff47c6fa0b72f72698660ee13
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - HDF5
  - MIST
  - PyTorch
  - RDKit
  - pluskal-lab/DreaMS
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# hdf5-spectral-data-loading

## Summary

Load tandem mass spectra (MS/MS) and LC–MS/MS metadata from the DreaMS paper's compact HDF5-based binary format, which stores the GeMS datasets (GeMS-A/B/C subsets mined from MassIVE GNPS) and the model/fine-tuning Zenodo deposits for downstream deep-learning analyses.

## When to use

Use when the input is a GeMS or DreaMS Zenodo HDF5 deposit of MS/MS spectra (e.g., the GeMS-A10 subset for pre-training analyses) and spectra plus metadata must be read into PyTorch/numpy pipelines.

## When NOT to use

- Input spectra are raw instrument files (.mzML/.mgf) not yet converted into the DreaMS HDF5 format — conversion/mining must happen first
- The goal is interactive spectral-library browsing or GNPS molecular networking rather than feeding tensors to the DreaMS transformer
- Spectra fall outside the paper's scope (e.g., negative ionization mode or multiply charged precursors), which the current DreaMS training data do not cover

## Inputs

- GeMS dataset file in the DreaMS compact HDF5-based binary format (e.g., GeMS-A10 subset prepared from MassIVE GNPS)
- DreaMS Zenodo deposits (records 10997887 and 13843034) containing model weights and MS/MS spectra paired with molecular structures
- DreaMS repository checkout providing the HDF5 loading utilities

## Outputs

- In-memory MS/MS spectra (m/z–intensity peak lists) with selected LC–MS/MS metadata (precursor m/z, retention)
- Held-out spectra ready for embedding extraction (1,024-dimensional DreaMS vectors) or masked-peak/retention-order pre-training

## How to apply

Clone the pluskal-lab/DreaMS repository and install its environment (`pip install -r requirements.txt`), then open the GeMS HDF5 file with a reader compatible with the DreaMS package. The files store raw MS/MS peak lists (m/z and intensity) together with selected LC–MS/MS metadata (e.g., precursor m/z, retention time) in a compact binary layout explicitly designed for deep learning rather than general-purpose spectral browsing. For pre-training/representation analyses, load the GeMS-A10 subset (a quality-controlled GeMS-A cluster sample of ~10 spectra per LSH cluster prepared from the MassIVE GNPS repository); for fine-tuning/retrieval experiments, load the spectra-plus-structure pairs from Zenodo records 10997887 and 13843034. Decision point: choose GeMS-A/B/C based on quality-control criteria such as instrument m/z accuracy and number of high-intensity signals — GeMS-A is the highest-quality subset and is the one used for the published pre-training and clustering results.

## Related tools

- **HDF5** (Binary storage format in which GeMS spectra and LC–MS/MS metadata are persisted for deep-learning loading)
- **pluskal-lab/DreaMS** (Official repository with the loading utilities, requirements, and tutorials for reading GeMS HDF5 files and the pre-trained model) — github.com__pluskal-lab__DreaMS

## Evaluation signals

- Loaded spectra round-trip to the expected m/z peak arrays and precursor metadata fields defined by the DreaMS HDF5 schema
- The selected subset corresponds to one of the documented quality tiers (GeMS-A, GeMS-B, or GeMS-C) filtered by instrument m/z accuracy and number of high-intensity signals
- Loaded held-out spectra yield 1,024-dimensional real-valued DreaMS embeddings, and downstream k=3 clustering in the 1,023-D PCA subspace approaches the reported 0.865 accuracy
- GeMS-A10 loading reproduces spectra grouped by 2D InChI key as used in the paper's structural-clustering analysis

## Limitations

- The paper trained on only a subset of available mass spectra; GeMS files do not cover all MassIVE GNPS data
- Training data exclude spectra beyond positive ionization mode or singly charged precursor ions, and disregard MS1 isotopic patterns and adduct distributions
- The MIST CANOPUS fine-tuning benchmark shipped with the deposits is limited in size (8,000 spectra from 7,000 unique molecules)

## Evidence

- [abstract] we stored the GeMS spectra, including selected LC–MS/MS metadata, in our compact HDF5-based binary format designed for deep learning: "we stored the GeMS spectra, including selected LC–MS/MS metadata, in our compact HDF5-based binary format designed for deep learning"
- [abstract] quality control filtering into GeMS subsets: "we developed a pipeline of quality control algorithms allowing us to filter the collected spectra into three subsets—GeMS-A, GeMS-B and GeMS-C"
- [abstract] mining pipeline scale: "Our mining pipeline consists of five main steps (Fig. 2a). First, we collected 250,000 LC–MS/MS experiments"
- [methods] environment setup prerequisite: "# Install requirements
pip install -r requirements.txt"
