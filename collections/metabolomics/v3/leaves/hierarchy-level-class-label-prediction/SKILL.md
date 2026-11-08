---
name: hierarchy-level-class-label-prediction
description: Use when you have a natural product SMILES string and need automated
  hierarchical structural classification into Pathway/Superclass/Class labels without
  rule-based systems like ClassyFire.
when_to_use_negative:
- The classification scheme you require is not in NPClassifier's 62 Classes / three
  Pathways vocabulary (e.g. needing ClassyFire's broader ChemOnt classes) — the paper
  benchmarked against ClassyFire for classes included in both platforms
- Input is not a molecular structure but raw spectra, a feature table, or text — this
  skill consumes SMILES/fingerprints only
- You need interpretability of the classification rules rather than predictions —
  the paper instead probes feature dependence via structural perturbation (ester-to-amide,
  4-keto to 3-hydroxy-4-keto changes)
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3473
- http://edamontology.org/topic_2258
tools:
- name: NPClassifier
  role: Deep neural network classification tool that maps counted Morgan fingerprints
    to hierarchical NP class labels
  repo: https://github.com/mwang87/NP-Classifier
- name: TensorFlow 2.3.0
  role: Framework used to train the three feed-forward networks and to convert keras
    models into HDF5 TF2 models
- name: Keras Tuner
  role: Hyperband algorithm used to optimize hidden units, learning rates, regularization,
    and dropout rate
- name: docker / docker-compose
  role: Deploy the dockerized TensorFlow serving server locally to expose the /classify
    API
  repo: https://github.com/mwang87/NP-Classifier
- name: ClassyFire
  role: External comparison classifier for classes included in both platforms, chosen
    from the Dictionary of Natural Products
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.jnatprod.1c00399
    title: 'NPClassifier: A Deep Neural Network-Based Structural Classification Tool
      for Natural Products'
  source_content_hash: sha256:edfedb851b32965703a762346e055487443dd8644fb17236cf12b5c94e2bb06c
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/mwang87/NP-Classifier
  repo_url: https://github.com/mwang87/NP-Classifier
  tools:
  - ClassyFire
  - Keras Tuner
  - NPClassifier
  - Python
  - TensorFlow 2.3.0
  - docker / docker-compose
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3473
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# hierarchy-level-class-label-prediction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Predicts natural product chemical class labels at three hierarchy levels (Pathway, Superclass, Class) by encoding a molecule's SMILES as a counted Morgan fingerprint and passing it through NPClassifier's three feed-forward neural networks, enabling structure-to-property linking during NP discovery.

## When to use

Use when you have a natural product SMILES string and need automated hierarchical structural classification into Pathway/Superclass/Class labels without rule-based systems like ClassyFire.

## When NOT to use

- The classification scheme you require is not in NPClassifier's 62 Classes / three Pathways vocabulary (e.g. needing ClassyFire's broader ChemOnt classes) — the paper benchmarked against ClassyFire for classes included in both platforms
- Input is not a molecular structure but raw spectra, a feature table, or text — this skill consumes SMILES/fingerprints only
- You need interpretability of the classification rules rather than predictions — the paper instead probes feature dependence via structural perturbation (ester-to-amide, 4-keto to 3-hydroxy-4-keto changes)

## Inputs

- Natural product structure as a SMILES string
- Counted Morgan fingerprint (2048/4096-bit) derived from the SMILES

## Outputs

- Predicted class labels at three hierarchy levels (Pathway, Superclass, Class) per molecule
- Optional: table mapping each structure (or perturbed structure) to its predicted labels

## How to apply

Encode the input structure as a counted Morgan fingerprint and pass it through three feed-forward neural networks that classify at the three levels of hierarchy; each unique category at the three levels was binary-encoded and cosine scoring was used to measure similarity between binarized vectors during training. The models are the released keras models from github.com/mwang87/NP-Classifier, converted to HDF5 TensorFlow 2.3.0 models; input layer names must be 'input_2048' and 'input_4096' and the output layer 'output'. Classification can also be done programmatically from SMILES via the dockerized TensorFlow serving API at /classify?smiles=<>, optionally with a cached flag for speed. The models were trained on 73,607 labeled natural products (64% train / 16% validation / 20% test, class-stratified), with hyperparameters (hidden units, learning rates, regularization, dropout) optimized via Hyperband in Keras Tuner.

## Related tools

- **NPClassifier** (Deep neural network classification tool that maps counted Morgan fingerprints to hierarchical NP class labels) — https://github.com/mwang87/NP-Classifier
- **TensorFlow 2.3.0** (Framework used to train the three feed-forward networks and to convert keras models into HDF5 TF2 models)
- **Keras Tuner** (Hyperband algorithm used to optimize hidden units, learning rates, regularization, and dropout rate)
- **docker / docker-compose** (Deploy the dockerized TensorFlow serving server locally to expose the /classify API) — https://github.com/mwang87/NP-Classifier
- **ClassyFire** (External comparison classifier for classes included in both platforms, chosen from the Dictionary of Natural Products)

## Examples

```
curl 'http://localhost/classify?smiles=CC(=O)OC1=CC=CC=C1C(=O)O&cached=true'
```

## Evaluation signals

- Verify model metadata via /model/metadata and confirm input layer names are 'input_2048' and 'input_4096' and output layer name is 'output'
- On external test sets, predictions should match the externally prepared evaluation sets (3000 chemical entities for three Pathways, 6200 compounds for 62 Classes)
- Structural-perturbation sanity check: replacing ester bonds with amide bonds in a cyclic depsipeptide should shift the predicted Class to cyclic peptides
- Flavonoid perturbation check: 4-keto → 3-hydroxy-4-keto in a flavanone should shift prediction to dihydroflavonols; 3-hydroxy-4-keto → simple 3-hydroxy should shift to flavan-3-ols
- Repeated /classify calls with the cached flag should return identical labels to non-cached calls

## Limitations

- Requires TensorFlow version 2.3.0 specifically to convert the keras models into HDF5 TF2 models; other versions may fail
- If the model input layer names change, the serving code must be updated accordingly (input_2048 / input_4096, output)
- Data set balance was imperfect: additional compounds from the ChEBI database were added specifically to improve data set balance, so rare classes may be less reliable
- Classification depends on specific class-related structural features, so modifying key structural moieties can change the predicted class (as shown by depsipeptide and flavonoid perturbations)

## Evidence


Output layer's name should be "output""
- [results] Structural perturbation shifts predicted class: [evidence span withheld — anchor status: not_found]
