---
name: stop-gradient-anti-collapse-training
description: Use when training a self-supervised contrastive encoder on augmented
  ion-image pairs from mass spectrometry imaging data and representations are collapsing
  to trivial constants.
when_to_use_negative:
- When the goal is supervised classification with annotated labels rather than self-supervised
  representation learning without manual annotation.
- When input is already a set of extracted low-dimensional feature vectors — the encoder/projection/prediction
  stack is designed for raw 2D ion images.
- When data are not mass spectrometry imaging ion images; the augmentation and Poisson-noise
  assumptions are MSI-specific.
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3474
tools:
- name: DeepION
  role: Framework implementing the SimSiam-based contrastive encoder, projection,
    and prediction modules with stop-gradient loss for ion-image representation
  repo: https://github.com/BioNet-XMU/DeepION
- name: ResNet18
  role: Backbone network for the parameter-sharing twin encoders producing 512-dimensional
    representations
  repo: https://github.com/BioNet-XMU/DeepION
- name: SimSiam
  role: Contrastive learning paradigm providing the stop-gradient anti-collapse loss
    design
  repo: https://github.com/BioNet-XMU/DeepION
- name: PyTorch
  role: Library used to implement and train the DeepION model
  repo: https://github.com/BioNet-XMU/DeepION
- name: torchvision
  role: Provides the pretrained ResNet18 backbone
  repo: https://github.com/BioNet-XMU/DeepION
- name: Adam optimizer
  role: Optimizes encoder and prediction module with lr 0.0003, beta1 0.5, beta2 0.99
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.3c05002
    title: 'DeepION: A Deep Learning-Based Low-Dimensional Representation Model of
      Ion Images for Mass Spectrometry Imaging'
  source_content_hash: sha256:66bdbec9169339d3dd4462805d1db847d26d85903922f1290b8f66b7b8ed5078
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BioNet-XMU/DeepION
  tools:
  - Adam optimizer
  - DeepION
  - PyTorch
  - Python
  - ResNet18
  - Resnet18
  - SimSiam
  - torchvision
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9375
n_claims: 16
---

# stop-gradient-anti-collapse-training

## Summary

Trains the DeepION contrastive model with a SimSiam-style asymmetric stop-gradient loss on parameter-sharing ResNet18 encoders, preventing collapse of the 512-dimensional ion-image representations learned from augmented MSI image pairs.

## When to use

Use when training a self-supervised contrastive encoder on augmented ion-image pairs from mass spectrometry imaging data and representations are collapsing to trivial constants.

## When NOT to use

- When the goal is supervised classification with annotated labels rather than self-supervised representation learning without manual annotation.
- When input is already a set of extracted low-dimensional feature vectors — the encoder/projection/prediction stack is designed for raw 2D ion images.
- When data are not mass spectrometry imaging ion images; the augmentation and Poisson-noise assumptions are MSI-specific.

## Inputs

- Two augmented views (T_COL or T_ISO) of each preprocessed ion image, as 2D matrices of shape [X*Y, P]
- Pretrained ResNet18 backbone weights (via torchvision)
- Preprocessed MSI matrix data, e.g. DATASET/Pos_brain_data_matrix.txt with shape [X*Y, P]

## Outputs

- 512-dimensional representation vectors r1, r2 per ion image
- Trained DeepION encoder, projection, and prediction module weights (PyTorch state dict)
- 20-dimensional reduced ion image representation vectors for downstream similarity search

## How to apply

Feed two augmented views of each ion image (produced by the T_COL or T_ISO augmentation strategies) through a pair of parameter-sharing twin ResNet18-based encoders, each yielding a 512-dimensional representation r1/r2. Project these via a three-layer MLP into h1/h2, then pass through a two-layer MLP prediction head to produce p1/p2. Apply the SimSiam contrastive loss with the stop-gradient operation: the encoder on x2 receives no gradient from h2 in the first loss term but receives gradients from h2 in the second term, and vice versa for x1 — this asymmetry blocks the collapse solution during optimization. Initialize all parameters from N(0, 0.02) and optimize with Adam (lr=0.0003, beta1=0.5, beta2=0.99). After training, reduce the 512-dim representations to a 20-dim vector via UMAP for downstream colocalization similarity measurement.

## Related tools

- **DeepION** (Framework implementing the SimSiam-based contrastive encoder, projection, and prediction modules with stop-gradient loss for ion-image representation) — https://github.com/BioNet-XMU/DeepION
- **ResNet18** (Backbone network for the parameter-sharing twin encoders producing 512-dimensional representations) — https://github.com/BioNet-XMU/DeepION
- **SimSiam** (Contrastive learning paradigm providing the stop-gradient anti-collapse loss design) — https://github.com/BioNet-XMU/DeepION
- **PyTorch** (Library used to implement and train the DeepION model) — https://github.com/BioNet-XMU/DeepION
- **torchvision** (Provides the pretrained ResNet18 backbone) — https://github.com/BioNet-XMU/DeepION
- **Adam optimizer** (Optimizes encoder and prediction module with lr 0.0003, beta1 0.5, beta2 0.99)

## Examples

```
python run.py --input_Matrix .../DATASET/Pos_brain_data_matrix.txt --input_PeakList .../DATASET/Pos_brain_data_peak.csv --input_shape 198 422 --mode COL --ion_mode positive --num 5 --output_file Pos_COL_result
```

## Evaluation signals

- Encoder output dimensionality is exactly 512 (d = 512) for each input view.
- Stop-gradient is applied asymmetrically: in the first loss term the encoder on x2 receives no gradient from h2, and in the second term the roles are reversed — verify by inspecting backward gradients.
- Loss value converges without representations collapsing to a constant vector (collapse prevented by the prediction head + stop-gradient).
- Training runs with the documented optimizer settings (Adam, lr 0.0003, beta1 0.5, beta2 0.99) and initialization N(0, 0.02) reach a stable loss.
- Post-training UMAP reduction yields a 20-dimensional output suitable for distance-based colocalization identification.

## Limitations

- Requires MSI-specific augmentation (Poisson noise, random missing values, intensity-dependent missing values) to be applied upstream; without it the contrastive objective is not aligned with the data distribution.
- The training assumes positive/negative ion-mode MSI experiments; switching modes requires retraining the encoder.
- No stated limitations or weaknesses of DeepION are given in the article's discussion section beyond its benefits.

## Evidence

- [methods] Twin encoders share parameters and use pretrained ResNet18: "It consists of twin networks that share identical parameters and adopt the pretrained Resnet18 as the backbone network"
- [readme] Stop-gradient loss prevents collapse: [evidence span withheld — anchor status: external_doc]
- [methods] SimSiam construction: "The proposed DeepION model is constructed based on the SimSiam model"
- [methods] Optimizer configuration: "The Adam optimizer is adopted to train the encoder and prediction module"
- [readme] Encoder output dimension: "Two augmented images are propagated through a pair of ResNet18-based encoders that shared parameters, then output two 512-dimensional representation vectors."
- [methods] Dimensionality reduction target: "The target of dimensionality reduction is to generate a dense vector for similarity measurement"
