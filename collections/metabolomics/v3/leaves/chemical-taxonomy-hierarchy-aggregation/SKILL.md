---
name: chemical-taxonomy-hierarchy-aggregation
description: Use when a structure list (e.g. SMILES tagged by organism origin) must
  be classified with NPClassifier and summarized into per-pathway or per-superclass
  counts to compare chemical space across organism groups.
when_to_use_negative:
- When structures are already annotated with taxonomy labels from another source (e.g.
  ClassyFire or Dictionary of Natural Products classes) and no reclassification is
  needed
- When the goal is structural similarity clustering rather than hierarchical category
  counting — NPClassifier predicts fixed pathway/superclass/class labels, not pairwise
  similarities
edam_operation: http://edamontology.org/operation_3220
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_0091
tools:
- name: NPClassifier
  role: Deep-learning classifier that converts SMILES to counted Morgan fingerprints
    and predicts pathway, superclass, and class via three feed-forward DNNs
  repo: github.com/mwang87/NP-Classifier
- name: docker
  role: Runtime used to bring up the dockerized NPClassifier classification server
    locally
- name: docker-compose
  role: Orchestrates the dockerized NPClassifier server deployment ('make server-compose')
- name: tensorflow 2.3.0
  role: Converts keras models into HDF5 TF2 models; the deep learning framework behind
    the DNN classifiers
- name: Python
  role: Loads the NP Atlas structure list, queries the /classify?smiles= endpoint
    per structure, and aggregates counts into a CSV table
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.jnatprod.1c00399
    title: 'NPClassifier: A Deep Neural Network-Based Structural Classification Tool
      for Natural Products'
  source_content_hash: sha256:fad0bcd9ee1934c2709c8945a8459803cf3f689cabd07535c918a576b1b88342
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
  - docker
  - docker-compose
  - tensorflow 2.3.0
  edam_operation: http://edamontology.org/operation_3220
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# chemical-taxonomy-hierarchy-aggregation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Aggregates NPClassifier's three-level structural predictions (pathway, superclass, class) for a set of structures into per-taxonomy count tables, enabling organism-level comparisons such as bacterial-vs-fungal terpenoid and PKS superclass distributions in the Natural Products Atlas.

## When to use

Use when a structure list (e.g. SMILES tagged by organism origin) must be classified with NPClassifier and summarized into per-pathway or per-superclass counts to compare chemical space across organism groups.

## When NOT to use

- When structures are already annotated with taxonomy labels from another source (e.g. ClassyFire or Dictionary of Natural Products classes) and no reclassification is needed
- When the goal is structural similarity clustering rather than hierarchical category counting — NPClassifier predicts fixed pathway/superclass/class labels, not pairwise similarities

## Inputs

- Natural Products Atlas structure list: SMILES strings tagged with organism origin (bacterial vs fungal)
- Pretrained NPClassifier models (keras models converted to HDF5 TF2 models)

## Outputs

- Per-pathway and per-superclass count table split by organism group (e.g. npatlas_pathway_class_counts.csv)
- Three-level NPClassifier predictions (pathway, superclass, class) per structure

## How to apply

Deploy the NPClassifier server locally: download the pretrained models via the models folder script, convert Keras models to HDF5 TF2 models with TensorFlow 2.3.0, create a docker network, and bring up the dockerized TensorFlow-serving container with 'make server-compose' under docker-compose. Verify the served model by checking /model/metadata and confirming input layer names 'input_2048' and 'input_4096' with output layer 'output'. For each structure (SMILES tagged with organism origin, e.g. bacterial vs fungal), query the /classify?smiles= endpoint — optionally with the cached flag for speed — which converts each SMILES to a counted Morgan fingerprint and runs three feed-forward DNNs to predict pathway, superclass, and class. Split predictions by organism group, then aggregate per-pathway and per-superclass counts into a single count table (e.g. npatlas_pathway_class_counts.csv) for cross-group comparison of pathway-level chemical differences.

## Related tools

- **NPClassifier** (Deep-learning classifier that converts SMILES to counted Morgan fingerprints and predicts pathway, superclass, and class via three feed-forward DNNs) — github.com/mwang87/NP-Classifier
- **docker** (Runtime used to bring up the dockerized NPClassifier classification server locally)
- **docker-compose** (Orchestrates the dockerized NPClassifier server deployment ('make server-compose'))
- **tensorflow 2.3.0** (Converts keras models into HDF5 TF2 models; the deep learning framework behind the DNN classifiers)
- **Python** (Loads the NP Atlas structure list, queries the /classify?smiles= endpoint per structure, and aggregates counts into a CSV table)

## Examples

```
curl 'http://localhost:8080/classify?smiles=CC(=O)OC1=CC=CC=C1C(=O)O&cached=true'
```

## Evaluation signals

- The aggregated table reproduces the reported finding: over 3000 terpenoids from fungi versus around 300 from bacteria (ratio similar to BGCs in MIBiG)
- PKS superclass distributions match expectations: macrolides and polycyclic aromatic polyketides dominant in bacteria; cyclic polyketides, chromones, aromatic polyketides, and naphthalene derivatives dominant in fungi
- Server health check: /model/metadata responds and input layer names are 'input_2048' and 'input_4096' with output layer 'output'
- Every input SMILES yields predictions at all three hierarchy levels (pathway, superclass, class) with no missing entries
- Sum of per-pathway counts per organism group equals the total number of structures in that group

## Limitations

- NPClassifier outputs only its fixed ontology of pathways, superclasses, and classes — structures outside this ontology space may receive weak or misleading labels
- The local deployment requires TensorFlow version 2.3.0 specifically for model conversion; if model input layer names change, the serving code must be updated accordingly
- Classification without the cached flag re-serves each structure through TensorFlow, which is slower for large structure lists

## Evidence

- [abstract] DNN predicting pathway/superclass/class from counted Morgan fingerprints: "Here, we introduce NPClassifier, a deep-learning tool for the automated structural classification of NPs from their counted Morgan fingerprints."
- [discussion] DNNs classify at three hierarchy levels: "The DNN for the NPClassifier was composed of three different networks that classified the molecular structure at the three levels of hierarchy with feed-forward neural network architecture"
- [intro] API endpoint for programmatic classification from SMILES: "Classify programmatically ```/classify?smiles=<>```"
- [readme] Model conversion requires TensorFlow 2.3.0: "Make sure you have python installed and tensorflow version 2.3.0 installed to convert the keras models into HDF5 TF2 models."
- [readme] Expected input/output layer names for the served model: "Input layers' names should be "input_2048" and "input_4096" Output layer's name should be "output""
