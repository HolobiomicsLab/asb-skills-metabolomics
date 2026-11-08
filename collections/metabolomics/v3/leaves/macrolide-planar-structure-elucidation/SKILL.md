---
name: macrolide-planar-structure-elucidation
description: Use when a fractionated marine natural product extract yields an unannotated
  macrolide candidate and you need a planar structure from 1D/2D NMR plus LC-MS2 molecular
  networking rather than full de novo stereochemical assignment.
when_to_use_negative:
- The goal is full stereochemical/absolute configuration assignment — this skill establishes
  planar structure only, via 1D/2D NMR and LC-MS2.
- The input is a pure, already-isolated compound with no fractionation needed — SMART
  2.0 mixture analysis and fraction prioritization are unnecessary.
- The extract has no cytotoxicity or bioactivity data to guide fraction prioritization
  and no reference spectra/networks for the expected molecular family.
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0154
tools:
- name: SMART 2.0
  role: CNN-based NMR mixture analysis tool used to annotate prioritized fractions
    and guide isolation of symplocolide A
  repo: https://github.com/mwang87/NP-Classifier
- name: molecular networking
  role: MS2-based molecular networking compares LC-MS2 network nodes against the swinholide
    A reference to prioritize candidates
- name: atomic sort
  role: Candidate ranking method applied to molecular network nodes during macrolide
    annotation
- name: tensorflow
  role: TensorFlow 2.3.0 converts the keras SMART 2.0 models into HDF5 TF2 models
    for serving
- name: docker
  role: Runs the dockerized SMART 2.0/NP Classifier model server locally
  repo: https://github.com/mwang87/NP-Classifier
- name: docker-compose
  role: Brings up the model server stack (docker network create nginx-net; make server-compose)
  repo: https://github.com/mwang87/NP-Classifier
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/jacs.9b13786
    title: A Convolutional Neural Network-Based Approach for the Rapid Annotation
      of Molecularly Diverse Natural Products
  source_content_hash: sha256:39b18fb951b0ab30791e89fb2b88129fc88aef77f271225aee0f58b08f20b25d
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
  - tensorflow
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# macrolide-planar-structure-elucidation

## Summary

Elucidates the planar structure of new macrolides (e.g., swinholide-like chimeras) by prioritizing fractions of a crude cyanobacterial extract with cytotoxicity data, then integrating SMART 2.0 NMR-based mixture annotation with MS2-based molecular networking and 1D/2D NMR evidence.

## When to use

Use when a fractionated marine natural product extract yields an unannotated macrolide candidate and you need a planar structure from 1D/2D NMR plus LC-MS2 molecular networking rather than full de novo stereochemical assignment.

## When NOT to use

- The goal is full stereochemical/absolute configuration assignment — this skill establishes planar structure only, via 1D/2D NMR and LC-MS2.
- The input is a pure, already-isolated compound with no fractionation needed — SMART 2.0 mixture analysis and fraction prioritization are unnecessary.
- The extract has no cytotoxicity or bioactivity data to guide fraction prioritization and no reference spectra/networks for the expected molecular family.

## Inputs

- Roughly fractionated crude marine cyanobacterial extract (e.g., environmental Symploca extract)
- 1H NMR spectra of each fraction (input to SMART 2.0)
- LC-MS2 data of prioritized fractions (for molecular networking)
- Reference compound structures/spectra (e.g., swinholide A) for molecular network comparison

## Outputs

- Fraction-prioritization table ranked by cancer cell cytotoxicity and SMART 2.0 annotations
- MS2-based molecular network with nodes matched to reference compounds
- Assigned planar structure of the isolated macrolide (e.g., symplocolide A as a swinholide A / luminaolide B hybrid)
- Single CSV record combining prioritization, SMART 2.0 annotations, and structure assignment

## How to apply

Roughly fractionate the crude extract and rank fractions by cancer cell cytotoxicity priorities. Process each fraction's 1H NMR spectra with SMART 2.0 (Small Molecule Accurate Recognition Technology), a convolutional neural network for NMR-based mixture analysis, to flag molecular families such as swinholide-like macrolides. In parallel, generate MS2-based molecular networking from LC-MS2 data of the prioritized fractions and compare network nodes against a reference compound (e.g., swinholide A), applying the 'atomic sort' method for candidate ranking. Isolate the top candidate and assign its planar structure by matching 1D/2D NMR chemical shifts and correlations plus LC-MS2 fragmentation against annotated relatives — in the source study this revealed symplocolide A as a structural hybrid between swinholide A and luminaolide B, alongside annotations of swinholide A and samholides A–I. Compile the fraction-prioritization table, SMART 2.0 annotation results, and the structure assignment into a single CSV record for validation.

## Related tools

- **SMART 2.0** (CNN-based NMR mixture analysis tool used to annotate prioritized fractions and guide isolation of symplocolide A) — https://github.com/mwang87/NP-Classifier
- **molecular networking** (MS2-based molecular networking compares LC-MS2 network nodes against the swinholide A reference to prioritize candidates)
- **atomic sort** (Candidate ranking method applied to molecular network nodes during macrolide annotation)
- **tensorflow** (TensorFlow 2.3.0 converts the keras SMART 2.0 models into HDF5 TF2 models for serving)
- **docker** (Runs the dockerized SMART 2.0/NP Classifier model server locally) — https://github.com/mwang87/NP-Classifier
- **docker-compose** (Brings up the model server stack (docker network create nginx-net; make server-compose)) — https://github.com/mwang87/NP-Classifier

## Examples

```
cd Classifier/models_folder/models && sh ./get_models.sh && docker network create nginx-net && make server-compose
```

## Evaluation signals

- The output CSV record reproduces the reported annotation of symplocolide A as a new chimeric swinholide-like macrolide, alongside annotation of swinholide A and samholides A–I.
- SMART 2.0 model server metadata reports input layer names 'input_2048' and 'input_4096' and output layer name 'output'.
- Planar structure assignment is corroborated by both 1D/2D NMR evidence and LC-MS2 fragmentation, not by a single modality.
- Molecular network nodes match against the swinholide A reference and candidate ranking uses the 'atomic sort' method.
- The isolated compound's planar structure is consistent with a structural hybrid between swinholide A and luminaolide B.

## Limitations

- SMART 2.0 requires specific model input/output layer names ('input_2048'/'input_4096'/'output'); if the model input names change, the code must be updated.
- The keras models must be converted with tensorflow version 2.3.0 specifically to produce HDF5 TF2 models.
- The approach addresses planar structure only; the paper's evidence concerns structure hybrid confirmation, not stereochemistry.
- Fraction prioritization depends on an external bioactivity axis (cancer cell cytotoxicity); without such data, prioritization guidance is lost.

## Evidence
