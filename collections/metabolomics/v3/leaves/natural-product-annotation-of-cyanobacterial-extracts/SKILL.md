---
name: natural-product-annotation-of-cyanobacterial-extracts
description: Use when a roughly fractionated natural-product extract has paired 1D
  NMR and LC-MS2 data per fraction and the goal is to prioritize fractions and assign
  natural-product classes (NP classes) to known and new metabolites without full isolation-first
  dereplication.
when_to_use_negative:
- Input is already purified single-compound material with full 2D NMR — manual structure
  elucidation, not mixture-level CNN annotation, is appropriate.
- Only LC-MS1 feature tables or MS2 data without accompanying 1D NMR spectra per fraction
  are available, since SMART 2.0 class prediction requires the 1D NMR input.
- No cytotoxicity or other prioritization readout exists and the goal is exhaustive
  isolation of all fractions rather than guided discovery of high-value fractions.
tools:
- name: SMART 2.0
  role: CNN applied to each fraction's 1D NMR data to produce natural-product class
    predictions used for prioritization and annotation
- name: molecular networking
  role: MS2-based molecular networking combined with SMART 2.0 predictions to prioritize
    and guide fraction interpretation
- name: NP-Classifier
  role: Dockerized model server (docker + docker-compose, tensorflow 2.3.0) exposing
    the /classify?smiles=<> API used for programmatic NP classification
  repo: github.com__mwang87__NP-Classifier
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/jacs.9b13786
    title: A Convolutional Neural Network-Based Approach for the Rapid Annotation
      of Molecularly Diverse Natural Products
  source_content_hash: sha256:a21048b80e6a0ea071bd01c95dd470c59ab5136d1718f4089406008e355b3ecb
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - NP-Classifier
  - SMART 2.0
  - molecular networking
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# natural-product-annotation-of-cyanobacterial-extracts

## Summary

An NMR- and MS2-guided annotation workflow for crude cyanobacterial (Symploca) extracts that uses the SMART 2.0 convolutional neural network on 1D NMR fractions plus MS2-based molecular networking to prioritize and annotate known and new metabolites (swinholide A, samholides A–I, symplocolide A).

## When to use

Use when a roughly fractionated natural-product extract has paired 1D NMR and LC-MS2 data per fraction and the goal is to prioritize fractions and assign natural-product classes (NP classes) to known and new metabolites without full isolation-first dereplication.

## When NOT to use

- Input is already purified single-compound material with full 2D NMR — manual structure elucidation, not mixture-level CNN annotation, is appropriate.
- Only LC-MS1 feature tables or MS2 data without accompanying 1D NMR spectra per fraction are available, since SMART 2.0 class prediction requires the 1D NMR input.
- No cytotoxicity or other prioritization readout exists and the goal is exhaustive isolation of all fractions rather than guided discovery of high-value fractions.

## Inputs

- Roughly fractionated environmental Symploca extract with 1D NMR spectra per fraction
- LC-MS2 data per fraction for MS2-based molecular networking
- Cancer-cell cytotoxicity readouts for fraction prioritization
- NP Classifier models (keras models convertible to HDF5 TF2, tensorflow 2.3.0)

## Outputs

- SMART 2.0 natural-product class predictions per fraction
- MS2 molecular-networking maps used for prioritization
- Structured annotation table (one row per compound: name, NP class, supporting NMR/LC-MS2 evidence), e.g. symploca_annotations.csv covering swinholide A, samholides A–I, new derivatives, and symplocolide A

## How to apply

Roughly fractionate the extract and acquire 1D NMR spectra plus LC-MS2 for each fraction. Run the SMART 2.0 CNN on each fraction's 1D NMR data to obtain automated natural-product class predictions, and run MS2-based molecular networking on the LC-MS2 data to visualize chemical families. Combine SMART 2.0 class predictions with molecular-networking clusters and (here) cancer-cell cytotoxicity to rank which fractions to interpret first, rather than pursuing all fractions equally. For each prioritized compound, cross-validate the SMART 2.0 classification against 1D/2D NMR and LC-MS2 evidence before finalizing the annotation — in this study that confirmed the planar structure of symplocolide A as a hybrid between swinholide A and luminaolide B. Compile a structured annotation table with one row per compound recording name, natural-product class, and supporting NMR/LC-MS2 evidence.

## Related tools

- **SMART 2.0** (CNN applied to each fraction's 1D NMR data to produce natural-product class predictions used for prioritization and annotation)
- **molecular networking** (MS2-based molecular networking combined with SMART 2.0 predictions to prioritize and guide fraction interpretation)
- **NP-Classifier** (Dockerized model server (docker + docker-compose, tensorflow 2.3.0) exposing the /classify?smiles=<> API used for programmatic NP classification) — github.com__mwang87__NP-Classifier

## Examples

```
docker network create nginx-net && make server-compose; curl '/classify?smiles=<>'
```

## Evaluation signals

- symploca_annotations.csv exists and contains entries for swinholide A, samholides A–I, and symplocolide A
- Annotation table matches the annotation set reported in the article (output_matches_reference against the reported annotations)
- SMART 2.0 class predictions for each prioritized fraction are consistent with 1D/2D NMR and LC-MS2 evidence
- Symplocolide A's planar structure is confirmed by 1D/2D NMR and LC-MS2 as a hybrid between swinholide A and luminaolide B
- Dockerized model server exposes input layers named 'input_2048' and 'input_4096' and output layer named 'output' when serving the classifier

## Limitations

- SMART 2.0 provides class-level automated predictions that still require cross-validation by 1D/2D NMR and LC-MS2 before structural confirmation.
- The NP Classifier server requires specific model internals (input layers 'input_2048' and 'input_4096', output layer 'output'); if model input names change, the code must be updated.
- Model conversion requires tensorflow 2.3.0 specifically to convert keras models into HDF5 TF2 models.

## Evidence


Output layer's name should be "output""
