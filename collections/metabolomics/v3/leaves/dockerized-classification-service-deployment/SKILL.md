---
name: dockerized-classification-service-deployment
description: Use when a workflow needs repeat programmatic NPClassifier calls on many
  SMILES strings (e.g. batch classifying an NP Atlas structure list tagged bacterial
  vs fungal) and a local dockerized server is preferred over external APIs.
when_to_use_negative:
- The structures to classify are only available as InChI or MS/MS spectra rather than
  SMILES — the /classify endpoint takes SMILES only
- You need ClassyFire-style chemical taxonomy rather than the NPClassifier pathway/superclass/class
  ontology (the paper compares the two platforms but this service serves NPClassifier
  models)
- Only a handful of one-off classifications are needed and a local docker deployment
  (network creation, model conversion) adds unnecessary overhead
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3474
tools:
- name: docker
  role: Runtime required to bring up the containerized NPClassifier service locally
  repo: github.com/mwang87/NP-Classifier
- name: docker-compose
  role: Orchestrates the service stack via `make server-compose` on the created nginx-net
    network
  repo: github.com/mwang87/NP-Classifier
- name: TensorFlow 2.3.0
  role: Converts the keras models into HDF5 TF2 models so they can be served through
    TensorFlow Serving
  repo: github.com/mwang87/NP-Classifier
- name: NPClassifier
  role: 'The served model: three feed-forward DNNs predicting pathway, superclass,
    and class from counted Morgan fingerprints, exposed at /classify?smiles=<>'
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.jnatprod.1c00399
    title: 'NPClassifier: A Deep Neural Network-Based Structural Classification Tool
      for Natural Products'
  source_content_hash: sha256:07c0bdb2508f9a299db989c36b1a637a605c99e041c8a4f50a5e848d3c1591fd
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
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# dockerized-classification-service-deployment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Deploy NPClassifier locally as a dockerized TensorFlow Serving service (docker + docker-compose) so natural-product structures can be classified programmatically from SMILES strings via a /classify?smiles= HTTP endpoint producing pathway, superclass, and class predictions.

## When to use

Use when a workflow needs repeat programmatic NPClassifier calls on many SMILES strings (e.g. batch classifying an NP Atlas structure list tagged bacterial vs fungal) and a local dockerized server is preferred over external APIs.

## When NOT to use

- The structures to classify are only available as InChI or MS/MS spectra rather than SMILES — the /classify endpoint takes SMILES only
- You need ClassyFire-style chemical taxonomy rather than the NPClassifier pathway/superclass/class ontology (the paper compares the two platforms but this service serves NPClassifier models)
- Only a handful of one-off classifications are needed and a local docker deployment (network creation, model conversion) adds unnecessary overhead

## Inputs

- SMILES strings of natural product structures (e.g. NP Atlas structure list tagged with organism origin)
- Pretrained NPClassifier keras models fetched via get_models.sh

## Outputs

- HTTP classification responses from /classify?smiles=<> with predicted pathway, superclass, and class per structure
- Model metadata from /model/metadata confirming input_2048/input_4096 input layers and output layer
- A running dockerized TF-serving endpoint usable for batch classification (e.g. aggregated into npatlas_pathway_class_counts.csv downstream)

## How to apply

Clone mwang87/NP-Classifier and fetch the pretrained models with `sh ./get_models.sh` from Classifier/models_folder/models; python with TensorFlow 2.3.0 must be installed to convert the keras models into HDF5 TF2 models. Create a docker network (`docker network create nginx-net`) and bring the stack up with `make server-compose`, which serves the three feed-forward DNNs (pathway, superclass, class) through TensorFlow Serving. Verify deployment by hitting `/model/metadata` and confirming the input layer names are "input_2048" and "input_4096" with output layer name "output"; if these change, the serving code must be updated. Then classify structures programmatically via `/classify?smiles=<>`, optionally passing a cached flag in the params to reuse cached results and speed up repeated queries on the same structures.

## Related tools

- **docker** (Runtime required to bring up the containerized NPClassifier service locally) — github.com/mwang87/NP-Classifier
- **docker-compose** (Orchestrates the service stack via `make server-compose` on the created nginx-net network) — github.com/mwang87/NP-Classifier
- **TensorFlow 2.3.0** (Converts the keras models into HDF5 TF2 models so they can be served through TensorFlow Serving) — github.com/mwang87/NP-Classifier
- **NPClassifier** (The served model: three feed-forward DNNs predicting pathway, superclass, and class from counted Morgan fingerprints, exposed at /classify?smiles=<>)

## Examples

```
cd Classifier/models_folder/models && sh ./get_models.sh && docker network create nginx-net && make server-compose; curl 'http://localhost/classify?smiles=CC(=O)OC1=CC=CC=C1C(=O)O'
```

## Evaluation signals

- /model/metadata returns model metadata whose input layer names are "input_2048" and "input_4096" and output layer name is "output"
- A GET to /classify?smiles=<> returns a valid classification (pathway/superclass/class) for a known SMILES instead of an error
- Repeat queries with the cached flag return identical results faster, confirming the cache path works
- End-to-end: batch-classifying the NP Atlas bacterial vs fungal structure list and aggregating counts reproduces the reported terpenoid counts (over 3000 fungal vs around 300 bacterial) and the distinct PKS superclass distributions
- Docker stack comes up cleanly after `docker network create nginx-net` followed by `make server-compose` (production-integration CI badge passes)

## Limitations

- Model input layer names are hardcoded expectations — if the model input names change, the serving code must be updated (input_2048 / input_4096, output)
- TensorFlow version 2.3.0 specifically is required for the keras-to-HDF5 TF2 model conversion step
- Deployment requires creating a docker network (nginx-net) before make server-compose; omitting it breaks bring-up
- Query logs record which structures were classified (not which users queried them) — a privacy consideration for sensitive structure sets

## Evidence

- [readme] Local deployment requires docker and docker-compose with a created docker network: "We typically will deploy this locally. To bring everything up, you need docker and docker-compose."
- [readme] Keras models must be converted to HDF5 TF2 models using TensorFlow 2.3.0 before serving: "NOTE: Make sure you have python installed and tensorflow version 2.3.0 installed to convert the keras models into HDF5 TF2 models."
- [readme] Layer names are a deployment invariant checked via the metadata endpoint: "Input layers' names should be "input_2048" and "input_4096""
