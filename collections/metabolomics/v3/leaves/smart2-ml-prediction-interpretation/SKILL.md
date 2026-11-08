---
name: smart2-ml-prediction-interpretation
description: Use when you have uncharacterized natural-product fractions or purified
  compounds with 1D/2D NMR spectra (optionally LC-MS/MS) and need automated class-level
  annotation or mixture deconvolution before full structure elucidation.
when_to_use_negative:
- Input is a resolved 2D structure/SMILES only with no NMR data — use the NP Classifier
  /classify?smiles=<> API instead of the NMR-based SMART 2.0 classifier
- The goal is full absolute-structure elucidation rather than rapid class-level annotation
  — SMART 2.0 prioritizes speed of annotation over complete stereochemical/structural
  assignment
- Analysis of an arbitrary synthetic-compound library outside the natural-product
  chemical space the model was trained on
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: SMART 2.0
  role: NMR-based convolutional neural network classifier that assigns natural-product
    class probabilities to spectra and supports mixture analysis
- name: molecular networking
  role: MS2-based molecular networking used alongside SMART 2.0 and cytotoxicity to
    prioritize fractions and provide corroborating spectral evidence
- name: atomic sort
  role: Recently appearing comparison method used to benchmark SMART 2.0's cyclic-peptide
    classifications
- name: docker
  role: Runtime requirement to bring up the local NP Classifier serving stack
  repo: https://github.com/mwang87/NP-Classifier
- name: docker-compose
  role: Orchestrates the dockerized server via 'make server-compose' after creating
    the nginx-net network
  repo: https://github.com/mwang87/NP-Classifier
- name: tensorflow 2.3.0
  role: Used to convert keras models into HDF5 TF2 models when downloading NP Classifier
    models
  repo: https://github.com/mwang87/NP-Classifier
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/jacs.9b13786
    title: A Convolutional Neural Network-Based Approach for the Rapid Annotation
      of Molecularly Diverse Natural Products
  source_content_hash: sha256:c7d5c14de59c0285411b911daa6917670fa48319ed08d6f50c6ed77307dcb052
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/mwang87/NP-Classifier
  tools:
  - SMART 2.0
  - atomic sort
  - docker
  - docker-compose
  - molecular networking
  - tensorflow 2.3.0
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# smart2-ml-prediction-interpretation

## Summary

Applies SMART 2.0, a convolutional neural network trained on NMR data, to assign natural-product structural classes to raw 1D/2D NMR spectra (with LC-MS/MS support), enabling rapid annotation of molecularly diverse natural products such as cyclic peptides beyond swinholide-like macrolides.

## When to use

Use when you have uncharacterized natural-product fractions or purified compounds with 1D/2D NMR spectra (optionally LC-MS/MS) and need automated class-level annotation or mixture deconvolution before full structure elucidation.

## When NOT to use

- Input is a resolved 2D structure/SMILES only with no NMR data — use the NP Classifier /classify?smiles=<> API instead of the NMR-based SMART 2.0 classifier
- The goal is full absolute-structure elucidation rather than rapid class-level annotation — SMART 2.0 prioritizes speed of annotation over complete stereochemical/structural assignment
- Analysis of an arbitrary synthetic-compound library outside the natural-product chemical space the model was trained on

## Inputs

- 1D/2D NMR spectra of purified natural products or fractions
- LC-MS/MS spectra (supporting evidence and MS2-based molecular networking)
- Pre-fractionated natural-product extracts (e.g., marine cyanobacterium Symploca extract)
- SMILES strings of known structures for programmatic NP Classifier queries

## Outputs

- Predicted natural-product class probabilities per spectrum
- Top SMART 2.0 class assignment per compound
- Per-compound annotation/characterization table (e.g., cyclic_peptide_classifications.csv with columns: compound name, SMART 2.0 class assignment, supporting spectral evidence)

## How to apply

Collect the case-study spectra (here, cyclic-peptide spectra from the article's second case study) and run each through the SMART 2.0 NMR-based convolutional neural network classifier to obtain predicted natural-product class probabilities. Record the top SMART 2.0 class assignment per compound, then cross-check each assignment against supporting 1D/2D NMR and LC-MS/MS evidence — the same validation used to confirm symplocolide A as a structural hybrid between swinholide A and luminaolide B. Assemble the per-compound annotation table (compound name, SMART 2.0 class assignment, supporting spectral evidence) and benchmark against alternative approaches such as the 'atomic sort' method. For programmatic follow-up classification from resolved structures, the NP Classifier API can be queried from SMILES strings; models are served via TensorFlow Serving behind a dockerized stack (docker and docker-compose, tensorflow 2.3.0 to convert keras models to HDF5 TF2 models, with input layers 'input_2048'/'input_4096' and output layer 'output').

## Related tools

- **SMART 2.0** (NMR-based convolutional neural network classifier that assigns natural-product class probabilities to spectra and supports mixture analysis)
- **molecular networking** (MS2-based molecular networking used alongside SMART 2.0 and cytotoxicity to prioritize fractions and provide corroborating spectral evidence)
- **atomic sort** (Recently appearing comparison method used to benchmark SMART 2.0's cyclic-peptide classifications)
- **docker** (Runtime requirement to bring up the local NP Classifier serving stack) — https://github.com/mwang87/NP-Classifier
- **docker-compose** (Orchestrates the dockerized server via 'make server-compose' after creating the nginx-net network) — https://github.com/mwang87/NP-Classifier
- **tensorflow 2.3.0** (Used to convert keras models into HDF5 TF2 models when downloading NP Classifier models) — https://github.com/mwang87/NP-Classifier

## Examples

```
curl 'http://localhost:8080/classify?smiles=<SMILES>'  # programmatic NP Classifier classification; SMART 2.0 NMR spectra are scored via the CNN served behind /model/metadata
```

## Evaluation signals

- Each SMART 2.0 top-class assignment is cross-checked against supporting 1D/2D NMR and LC-MS/MS evidence from the same sample
- Output table contains all three required columns (compound name, SMART 2.0 class assignment, supporting spectral evidence) with one row per cyclic-peptide compound
- Class assignments agree with independently derived annotations (e.g., symplocolide A confirmed as a structural hybrid between swinholide A and luminaolide B by 1D/2D NMR and LC-MS/MS)
- Served model metadata exposes input layers named 'input_2048' and 'input_4096' and output layer 'output' via the /model/metadata endpoint
- Benchmark results against the 'atomic sort' method show consistent class assignments for structurally novel cyclic peptides

## Limitations

- No changelog is available for the repository, making model-version tracking difficult
- If the served model input layer names change, the serving code must be updated accordingly (input names are hard-coded as 'input_2048' and 'input_4096')
- The classifier assigns class-level annotations, not full structures — assignments still require confirmation by traditional 1D/2D NMR and LC-MS/MS analysis

## Evidence


Output layer's name should be "output""
