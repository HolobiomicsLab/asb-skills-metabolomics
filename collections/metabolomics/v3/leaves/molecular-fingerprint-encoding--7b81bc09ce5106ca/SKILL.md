---
name: molecular-fingerprint-encoding
description: Use when you have SMILES strings of natural product molecules and need
  NP-relevant structural class labels (pathway, superclass, class) predicted programmatically,
  or when testing whether specific structural moieties drive a classifier's predictions.
when_to_use_negative:
- Input is analytical data without structures (e.g., MS/MS spectra or peak lists)
  — the classifier consumes SMILES-derived counted Morgan fingerprints only
- You need non-NP cheminformatics classes (e.g., ClassyFire-style categories for non-natural-product
  compounds) rather than NPClassifier's NP-oriented pathway/superclass/class taxonomy
- You require quantitative structure-property regression rather than categorical class
  labels
edam_operation: http://edamontology.org/operation_3438
edam_topics:
- http://edamontology.org/topic_3332
- http://edamontology.org/topic_2258
tools:
- name: NPClassifier
  role: Deep-learning classifier mapping counted Morgan fingerprints to three-level
    NP class labels; provides the /classify?smiles= API
  repo: github.com/mwang87/NP-Classifier
- name: TensorFlow 2.3.0
  role: Framework used to convert keras models into HDF5 TF2 models and serve the
    feed-forward networks
- name: docker
  role: Used with docker-compose to bring up the local classification server
  repo: github.com/mwang87/NP-Classifier
- name: Python
  role: Language for encoding fingerprints, loading structure lists, and calling the
    classify endpoint
provenance:
  source_task_ids:
  - task_002
  - task_003
  source_papers:
  - doi: 10.1021/acs.jnatprod.1c00399
    title: 'NPClassifier: A Deep Neural Network-Based Structural Classification Tool
      for Natural Products'
  source_content_hash: sha256:8ab8692672b342c849a843a56dbdfa8438d3deea21f4c9151ada936513e6860c
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/mwang87/NP-Classifier
  tools:
  - NPClassifier
  - Python
  - TensorFlow
  - TensorFlow 2.3.0
  - docker
  - docker-compose
  edam_operation: http://edamontology.org/operation_3438
  edam_topics:
  - http://edamontology.org/topic_3332
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# molecular-fingerprint-encoding

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Encode natural product structures as counted Morgan fingerprints and pass them through NPClassifier's three feed-forward neural networks to obtain hierarchical pathway/superfather/class predictions, deployed locally via a dockerized TF-serving API.

## When to use

Use when you have SMILES strings of natural product molecules and need NP-relevant structural class labels (pathway, superclass, class) predicted programmatically, or when testing whether specific structural moieties drive a classifier's predictions.

## When NOT to use

- Input is analytical data without structures (e.g., MS/MS spectra or peak lists) — the classifier consumes SMILES-derived counted Morgan fingerprints only
- You need non-NP cheminformatics classes (e.g., ClassyFire-style categories for non-natural-product compounds) rather than NPClassifier's NP-oriented pathway/superclass/class taxonomy
- You require quantitative structure-property regression rather than categorical class labels

## Inputs

- SMILES strings of small molecules (e.g., from the Natural Products Atlas structure list)
- Downloaded NPClassifier model files (keras models converted to HDF5 TF2 models)
- Perturbed analog structures (e.g., depsipeptide ester-to-amide variants, flavanone keto/hydroxy variants)

## Outputs

- Hierarchical classification JSON per molecule: pathway, superclass, and class predicted labels
- Count tables aggregating predictions per organism or per structural series (e.g., npatlas_pathway_class_counts.csv)

## How to apply

Convert each molecule's SMILES into a counted Morgan fingerprint, which is the input representation NPClassifier's networks consume; the TF-serving models expose input layers named 'input_2048' and 'input_4096', reflecting the two fingerprint bit sizes supported. Run the fingerprint through the three feed-forward DNNs that classify at the three hierarchy levels (pathway, superclass, class); the models were built in TensorFlow 2.3.0 with hyperparameters tuned via Keras Tuner's Hyperband algorithm, dropout on fully connected layers, and cosine scoring between binarized label vectors during training. Deploy locally by downloading the HDF5 TF2 model files ('sh ./get_models.sh' inside Classifier/models_folder/models) and building the dockerized server ('docker network create nginx-net; make server-compose'). Then query the '/classify?smiles=<>' endpoint per structure, optionally adding the cached flag to reuse prior classifications for speed. Verify output by checking /model/metadata for unchanged layer names and by confirming predicted labels on known controls behave sensibly.

## Related tools

- **NPClassifier** (Deep-learning classifier mapping counted Morgan fingerprints to three-level NP class labels; provides the /classify?smiles= API) — github.com/mwang87/NP-Classifier
- **TensorFlow 2.3.0** (Framework used to convert keras models into HDF5 TF2 models and serve the feed-forward networks)
- **docker** (Used with docker-compose to bring up the local classification server) — github.com/mwang87/NP-Classifier
- **Python** (Language for encoding fingerprints, loading structure lists, and calling the classify endpoint)

## Examples

```
import requests; labels = requests.get('http://localhost/classify', params={'smiles':'Cn1cnc2c1c(=O)n(C)c(=O)n2C','cached':'true'}).json()
```

## Evaluation signals

- GET /model/metadata returns metadata whose input layer names are 'input_2048' and 'input_4096' and whose output layer name is 'output'
- Predictions fall within NPClassifier's trained taxonomy of 3 Pathways and 62 Classes (external test sets of 3000 entities for pathways and 6200 compounds for classes)
- Structural perturbation control reproduces expected class shifts: ester-to-amide in a cyclic depsipeptide shifts prediction to cyclic peptides; 4-keto→3-hydroxy-4-keto in a flavanone shifts to dihydroflavonols; 3-hydroxy-4-keto→3-hydroxy shifts to flavan-3-ols
- On the NP Atlas analysis, aggregated counts reproduce ~3000 fungal vs ~300 bacterial terpenoids and the reported PKS superclass distribution split
- Repeated classification with the cached flag returns identical labels to uncached calls

## Limitations

- Model conversion requires exactly tensorflow version 2.3.0; if model input names change, the server code must be updated
- Class predictions are sensitive to specific structural moieties — modifying a class-defining group (ester bonds, keto groups) flips the predicted class, so labels for modified analogs may not reflect biosynthetic reality
- Training data balance was imperfect and required supplementation from the ChEBI database to improve balance

## Evidence
