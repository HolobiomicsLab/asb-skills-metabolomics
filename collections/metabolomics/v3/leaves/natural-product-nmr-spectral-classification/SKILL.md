---
name: natural-product-nmr-spectral-classification
description: Use when you have 1D NMR spectra of crude extracts, fractions, or purified
  natural products (e.g., Symploca fractions, cyclic-peptide case-study spectra) and
  need rapid NP class-level annotation, prioritization, or benchmarking against the
  'atomic sort' method.
when_to_use_negative:
- You already have SMILES structures for the compounds — classify them via the NP
  Classifier /classify?smiles=<> API instead of the NMR spectral classifier
- Only LC-MS/MS data is available with no 1D NMR spectra — SMART 2.0 is NMR-based,
  so MS2-only inputs belong in MS2-based molecular networking
- The goal is full planar-structure elucidation rather than class-level annotation
  — SMART 2.0 class calls still required confirmation by 1D/2D NMR and LC-MS/MS in
  the case study
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: SMART 2.0
  role: NMR-based convolutional neural network ('Small Molecule Accurate Recognition
    Technology') that predicts natural-product class probabilities from 1D NMR spectra
    of fractions and purified compounds
  repo: https://github.com/mwang87/NP-Classifier
- name: MS2-based molecular networking
  role: complementary MS/MS prioritization run alongside NMR-based SMART 2.0 to guide
    fractionation of the Symploca extract
- name: atomic sort
  role: recently appearing benchmarking method against which SMART 2.0's cyclic-peptide
    annotations were compared
- name: docker-compose
  role: brings up the dockerized NP Classifier serving stack on the nginx-net network
    via 'make server-compose'
  repo: https://github.com/mwang87/NP-Classifier
- name: tensorflow 2.3.0
  role: converts the keras classifier models into HDF5 TF2 models for tensorflow serving
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/jacs.9b13786
    title: A Convolutional Neural Network-Based Approach for the Rapid Annotation
      of Molecularly Diverse Natural Products
  source_content_hash: sha256:e84f7fb702242286863474b114903e912f0ba4aacf21214a8413ebc14c390b11
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/mwang87/NP-Classifier
  tools:
  - MS2-based molecular networking
  - SMART 2.0
  - atomic sort
  - docker-compose
  - tensorflow 2.3.0
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# natural-product-nmr-spectral-classification

## Summary

An NMR-based deep learning skill in which the SMART 2.0 convolutional neural network ('Small Molecule Accurate Recognition Technology') assigns natural-product class annotations directly from 1D NMR spectra. It accelerates fractionation prioritization and annotation of molecularly diverse natural products, from swinholide-like macrolides to structurally novel cyclic peptides.

## When to use

Use when you have 1D NMR spectra of crude extracts, fractions, or purified natural products (e.g., Symploca fractions, cyclic-peptide case-study spectra) and need rapid NP class-level annotation, prioritization, or benchmarking against the 'atomic sort' method.

## When NOT to use

- You already have SMILES structures for the compounds — classify them via the NP Classifier /classify?smiles=<> API instead of the NMR spectral classifier
- Only LC-MS/MS data is available with no 1D NMR spectra — SMART 2.0 is NMR-based, so MS2-only inputs belong in MS2-based molecular networking
- The goal is full planar-structure elucidation rather than class-level annotation — SMART 2.0 class calls still required confirmation by 1D/2D NMR and LC-MS/MS in the case study

## Inputs

- 1D NMR spectra of crude extracts, fractions, or purified natural products (e.g., the cyclic-peptide case-study spectra from the article's second case study)
- Fractionated marine cyanobacterium (Symploca) extract spectra for prioritization
- NP Classifier model files (keras models converted to HDF5 TF2) fetched via get_models.sh
- Supporting 1D/2D NMR and LC-MS/MS data for cross-checking annotations
- SMILES strings of known structures (for the companion /classify?smiles=<> API)

## Outputs

- Per-compound natural-product class probability vector from the SMART 2.0 NMR-based CNN, with a top class assignment per compound
- cyclic_peptide_classifications.csv with columns: compound name, SMART 2.0 class assignment, supporting spectral evidence
- Model metadata response from /model/metadata confirming serving layer names

## How to apply

Download the NP Classifier models (cd Classifier/models_folder/models; sh ./get_models.sh) using python with tensorflow 2.3.0 so the keras models are converted into HDF5 TF2 models, then bring up the dockerized server on the nginx-net network via 'docker network create nginx-net' and 'make server-compose'. Verify serving through the /model/metadata endpoint — input layers must be named "input_2048" and "input_4096" and the output layer "output". Run each 1D NMR spectrum (e.g., each cyclic-peptide spectrum from the article's second case study) through the SMART 2.0 NMR-based CNN classifier to obtain predicted natural-product class probabilities and record the top class per compound; these calls simultaneously guide fractionation prioritization alongside cancer cell cytotoxicity and MS2-based molecular networking. Cross-check each top assignment against the supporting spectral evidence (1D/2D NMR and LC-MS/MS data described in the case study) and benchmark the approach against the recently appearing 'atomic sort' method. Assemble the per-compound table (compound name, SMART 2.0 class assignment, supporting spectral evidence) and write it to cyclic_peptide_classifications.csv; for already-elucidated structures, SMILES strings can be classified programmatically via /classify?smiles=<> with the cached flag for speed.

## Related tools

- **SMART 2.0** (NMR-based convolutional neural network ('Small Molecule Accurate Recognition Technology') that predicts natural-product class probabilities from 1D NMR spectra of fractions and purified compounds) — https://github.com/mwang87/NP-Classifier
- **MS2-based molecular networking** (complementary MS/MS prioritization run alongside NMR-based SMART 2.0 to guide fractionation of the Symploca extract)
- **atomic sort** (recently appearing benchmarking method against which SMART 2.0's cyclic-peptide annotations were compared)
- **docker-compose** (brings up the dockerized NP Classifier serving stack on the nginx-net network via 'make server-compose') — https://github.com/mwang87/NP-Classifier
- **tensorflow 2.3.0** (converts the keras classifier models into HDF5 TF2 models for tensorflow serving)

## Examples

```
cd Classifier/models_folder/models && sh ./get_models.sh && make server-compose
```

## Evaluation signals

- Before inference, /model/metadata reports input layers named "input_2048" and "input_4096" and an output layer named "output"
- Each row of cyclic_peptide_classifications.csv carries a top SMART 2.0 class assignment plus a cited piece of supporting spectral evidence (1D/2D NMR and/or LC-MS/MS)
- SMART 2.0 top-class calls on the cyclic-peptide spectra agree with, or are benchmarked against, the 'atomic sort' method
- Annotations are consistent with manual structural interpretation — e.g., symplocolide A confirmed as a structural hybrid between swinholide A and luminaolide B by 1D/2D NMR and LC-MS2 analysis
- The dockerized server responds on the nginx-net network after 'make server-compose' with models converted from keras to HDF5 TF2

## Limitations

- Requires python with tensorflow version 2.3.0 specifically to convert the keras models into HDF5 TF2 models
- If the model input layer names change, the serving code must be updated accordingly (per the README)
- Deployment depends on docker and docker-compose (nginx-net network, make server-compose)
- SMART 2.0 yields class-level annotation, not planar structures — structural confirmation still needs 1D/2D NMR and LC-MS/MS
- Query logging retains which structures were classified (README privacy note); no changelog was found for the repository

## Evidence

- [abstract] first application of the novel NMR-based machine learning tool 'Small Molecule Accurate Recognition Technology' (SMART 2.0): [evidence span withheld — anchor status: not_found]
- [abstract] the recently appearing 'atomic sort' method: [evidence span withheld — anchor status: not_found]
