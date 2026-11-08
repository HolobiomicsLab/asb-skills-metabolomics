---
name: natural-product-structure-classification
description: Use when you have SMILES strings for natural products (e.g. NP Atlas
  entries tagged bacterial vs fungal) and need pathway/superclass/class labels to
  aggregate counts or compare biosynthetic distributions across organism groups.
when_to_use_negative:
- The input is not a chemical structure (no SMILES) — NPClassifier requires a counted
  Morgan fingerprint derived from a SMILES string and cannot classify textual organism
  or activity metadata directly.
- You need consistent cross-platform chemical ontology classes for compounds outside
  natural product space — the article benchmarked against ClassyFire using Dictionary
  of Natural Products classes, and NPClassifier's ontology is natural-product specific.
- You cannot run docker/docker-compose or install tensorflow 2.3.0 — model conversion
  to HDF5 TF2 and the serving stack depend on those exact versions per the README.
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: NPClassifier
  role: 'Core classification tool: three feed-forward DNNs predict Pathway, Superclass,
    and Class from counted Morgan fingerprints, exposed via /classify?smiles=<>'
  repo: github.com/mwang87/NP-Classifier
- name: TensorFlow 2.3.0
  role: Converts the released Keras models into HDF5 TF2 models and serves inference
    through the dockerized TensorFlow-Serving stack
- name: docker / docker-compose
  role: 'Deploy the local classification server: docker network create nginx-net,
    then make server-compose'
  repo: github.com/mwang87/NP-Classifier
- name: Python
  role: Load SMILES lists, iterate over /classify queries, aggregate predictions into
    count tables
provenance:
  source_task_ids:
  - task_003
  - task_004
  - task_002
  source_papers:
  - doi: 10.1021/acs.jnatprod.1c00399
    title: 'NPClassifier: A Deep Neural Network-Based Structural Classification Tool
      for Natural Products'
  source_content_hash: sha256:aedc834c89cee5cc610745e08f077d30c11cd6805721d3582bb26cb851f4b6a2
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/mwang87/NP-Classifier
  tools:
  - MAIP
  - NPClassifier
  - Python
  - TensorFlow
  - TensorFlow 2.3.0
  - docker
  - docker / docker-compose
  - docker-compose
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# natural-product-structure-classification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Classify natural product structures hierarchically (Pathway, Superclass, Class) from SMILES using NPClassifier's three feed-forward DNNs over counted Morgan fingerprints, served locally via a dockerized TensorFlow-Serving API.

## When to use

Use when you have SMILES strings for natural products (e.g. NP Atlas entries tagged bacterial vs fungal) and need pathway/superclass/class labels to aggregate counts or compare biosynthetic distributions across organism groups.

## When NOT to use

- The input is not a chemical structure (no SMILES) — NPClassifier requires a counted Morgan fingerprint derived from a SMILES string and cannot classify textual organism or activity metadata directly.
- You need consistent cross-platform chemical ontology classes for compounds outside natural product space — the article benchmarked against ClassyFire using Dictionary of Natural Products classes, and NPClassifier's ontology is natural-product specific.
- You cannot run docker/docker-compose or install tensorflow 2.3.0 — model conversion to HDF5 TF2 and the serving stack depend on those exact versions per the README.

## Inputs

- SMILES strings of natural product structures (e.g. NP Atlas structure list tagged with organism origin: bacterial vs fungal)
- Perturbed/modified structures (e.g. depsipeptide ester-to-amide variants, flavanone keto/hydroxy variants) as SMILES

## Outputs

- Per-structure predicted labels at three hierarchy levels: Pathway, Superclass, Class
- Aggregated count table (e.g. npatlas_pathway_class_counts.csv) of per-pathway and per-superclass counts per organism group
- Subgroup-filtered classification tables for downstream analyses (e.g. spirotetronate macrolide sets for MAIP scoring)

## How to apply

Download the pretrained models from github.com/mwang87/NP-Classifier (models converted from Keras to HDF5 TF2 with tensorflow 2.3.0), create a docker network and bring up the server with docker-compose ('make server-compose'). Verify deployment via the /model/metadata endpoint, checking that input layer names are 'input_2048' and 'input_4096' and the output layer name is 'output'. For each structure, call /classify?smiles=<> (optionally with a cached flag for speed); the server converts the SMILES to a counted Morgan fingerprint and runs three feed-forward networks — one per hierarchy level — trained with binary-encoded categories and cosine scoring. Split predictions by organism origin or structural group, aggregate per-pathway and per-superclass counts into a table such as npatlas_pathway_class_counts.csv, and validate against known distribution landmarks (e.g. >3000 fungal vs ~300 bacterial terpenoids in the NP Atlas).

## Related tools

- **NPClassifier** (Core classification tool: three feed-forward DNNs predict Pathway, Superclass, and Class from counted Morgan fingerprints, exposed via /classify?smiles=<>) — github.com/mwang87/NP-Classifier
- **TensorFlow 2.3.0** (Converts the released Keras models into HDF5 TF2 models and serves inference through the dockerized TensorFlow-Serving stack)
- **docker / docker-compose** (Deploy the local classification server: docker network create nginx-net, then make server-compose) — github.com/mwang87/NP-Classifier
- **Python** (Load SMILES lists, iterate over /classify queries, aggregate predictions into count tables)

## Examples

```
curl "http://localhost/classify?smiles=CC(=O)Oc1ccccc1C(=O)O"
```

## Evaluation signals

- The /model/metadata endpoint is reachable and the model input layer names are 'input_2048' and 'input_4096' with output layer name 'output'.
- Structural-perturbation check: ester-to-amide replacement in a cyclic depsipeptide shifts the predicted Class to cyclic peptides; 4-keto→3-hydroxy-4-keto in a flavanone shifts to dihydroflavonols; 3-hydroxy-4-keto→3-hydroxy shifts to flavan-3-ols.
- NP Atlas validation: the aggregated count table shows over 3000 terpenoids from fungi versus around 300 from bacteria, matching the MIBiG BGC ratio.
- PKS superclass distributions match the reported pattern: macrolides and polycyclic aromatic polyketides dominant in bacteria; cyclic polyketides, chromones, aromatic polyketides, and naphthalene derivatives dominant in fungi.
- Every queried SMILES returns labels at all three hierarchy levels (Pathway, Superclass, Class) with no missing predictions.

## Limitations

- Model conversion requires exactly tensorflow version 2.3.0; other versions may fail to produce valid HDF5 TF2 models.
- If the TensorFlow-Serving model input names change from 'input_2048'/'input_4096' (output 'output'), the server code must be updated accordingly.
- The models were trained on 73607 labeled natural products (64/16/20 train/val/test split with Class stratification), so coverage of rare or novel scaffold classes may be limited.

## Evidence


Output layer's name should be "output""
