---
name: msi-specific-data-augmentation-design
description: Use when training a self-supervised contrastive encoder (e.g., SimSiam
  with ResNet18 backbone) on MSI ion images and you must generate augmented view pairs
  whose noise model matches the Poisson-distributed ion abundance and random/intensity-dependent
  missing patterns of MSI data.
when_to_use_negative:
- When the input is a single image per sample and generic computer-vision augmentation
  suffices — this skill's T_COL/T_ISO exist specifically to mirror MSI's Poisson-distributed
  ion abundance and random/intensity-dependent missing values.
- 'When the goal is downstream similarity scoring rather than representation learning:
  augmentation feeds encoder training, but the identification step itself runs on
  the UMAP-reduced 20-dimensional vectors, not on the augmented images.'
- 'When the input is raw (unpreprocessed) MSI data: the augmentation module expects
  the preprocessed MSI matrix [X*Y, P] and peak list [P, 1] already prepared by preprocessing.'
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3869
- http://edamontology.org/topic_0091
tools:
- name: DeepION
  role: Framework in which the data augmentation module T (T_COL for COL mode, T_ISO
    for ISO mode) is embedded to generate augmented view pairs for contrastive learning
    of ion-image representations
  repo: https://github.com/gankLei-X/DeepION
- name: SimSiam
  role: Contrastive model whose twin networks consume the augmented image pairs; augmentation
    design directly drives what the shared-parameter ResNet18 encoders learn to be
    invariant to
  repo: https://github.com/gankLei-X/DeepION
- name: Resnet18
  role: Pretrained backbone of the twin encoders that processes each augmented view
    produced by T_COL/T_ISO into 512-dimensional representation vectors
  repo: https://github.com/gankLei-X/DeepION
- name: kornia
  role: Image-augmentation package used to implement the T_COL transforms (color jitter,
    filtering) within the DeepION augmentation module
  repo: https://github.com/gankLei-X/DeepION
- name: PyTorch
  role: Library in which DeepION (including the augmentation module and its contrastive
    training loop) is implemented
  repo: https://github.com/gankLei-X/DeepION
provenance:
  source_task_ids:
  - task_005
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.3c05002
    title: 'DeepION: A Deep Learning-Based Low-Dimensional Representation Model of
      Ion Images for Mass Spectrometry Imaging'
  source_content_hash: sha256:225771fc0ca75c21aa05393b8aea67aab98a594ee953bf36d38a5353e0d75b73
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gankLei-X/DeepION
  tools:
  - DeepION
  - METASPACE
  - Nvidia GTX 2080Ti
  - PCC
  - PyTorch
  - Python
  - R2
  - Resnet18
  - SimSiam
  - kornia
  - rMSI
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3869
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9545454545454546
n_claims: 22
---

# msi-specific-data-augmentation-design

## Summary

Design modality-specific data augmentation for contrastive learning on mass spectrometry imaging ion images: T_COL (color jitter, filtering, Poisson noise, random missing value) for colocalized-ion mode and T_ISO (adding intensity-dependent missing value) for isotope mode, so the SimSiam encoder learns meaningful ion-image representations without manual annotation.

## When to use

Use when training a self-supervised contrastive encoder (e.g., SimSiam with ResNet18 backbone) on MSI ion images and you must generate augmented view pairs whose noise model matches the Poisson-distributed ion abundance and random/intensity-dependent missing patterns of MSI data.

## When NOT to use

- When the input is a single image per sample and generic computer-vision augmentation suffices — this skill's T_COL/T_ISO exist specifically to mirror MSI's Poisson-distributed ion abundance and random/intensity-dependent missing values.
- When the goal is downstream similarity scoring rather than representation learning: augmentation feeds encoder training, but the identification step itself runs on the UMAP-reduced 20-dimensional vectors, not on the augmented images.
- When the input is raw (unpreprocessed) MSI data: the augmentation module expects the preprocessed MSI matrix [X*Y, P] and peak list [P, 1] already prepared by preprocessing.

## Inputs

- Preprocessed MSI matrix data with two-dimensional shape [X*Y, P], where X and Y are the pixel numbers of the horizontal and vertical coordinates and P is the number of ions (e.g., DATASET/Pos_brain_data_matrix.txt for a 198 x 422 rat brain section)
- Preprocessed MSI peak data with one-dimensional shape [P, 1] (e.g., DATASET/Pos_brain_data_peak.csv)
- The input MSI data shape (input_shape, e.g., 198 422)
- Mode selection: COL (regular colocalized ions from different molecules) or ISO (isotope ions from the same molecule)

## Outputs

- Two augmented ion-image views per original ion image, generated by T_COL (COL mode) or T_ISO (ISO mode)
- Two 512-dimensional representation vectors produced by the shared-parameter ResNet18-based encoders from the augmented views
- Augmented-view pairs used to train the SimSiam-style model with contrastive loss and stop-gradient, yielding learnable ion-image representations for downstream COL/ISO identification

## How to apply

Import the preprocessed MSI matrix with two-dimensional shape [X*Y, P] (e.g., rat brain section with input_shape 198 422) together with the one-dimensional peak list [P, 1]. Feed each original ion image into the augmentation module T to generate two augmented views: in COL mode apply T_COL, which composes color jitter, filtering, Poisson noise, and random missing value; in ISO mode apply T_ISO, which adds an intensity-dependent missing-value process on top of the COL transforms, because isotope ions from the same molecule share the same missing-value pattern tied to ion intensity. Propagate the two augmented images through the pair of parameter-sharing ResNet18-based encoders and maximize their similarity with the contrastive loss, using the stop-gradient operation to prevent collapse. Rationale: the augmentation strategy is designed by incorporating the characteristics of MSI data — the Poisson distribution of ion abundance and the random pattern of missing values — rather than borrowing generic image augmentations, which is what enables the learned 512-dimensional representations to yield meaningful 20-dimensional vectors (via UMAP) for colocalized-ion and isotope identification.

## Related tools

- **DeepION** (Framework in which the data augmentation module T (T_COL for COL mode, T_ISO for ISO mode) is embedded to generate augmented view pairs for contrastive learning of ion-image representations) — https://github.com/gankLei-X/DeepION
- **SimSiam** (Contrastive model whose twin networks consume the augmented image pairs; augmentation design directly drives what the shared-parameter ResNet18 encoders learn to be invariant to) — https://github.com/gankLei-X/DeepION
- **Resnet18** (Pretrained backbone of the twin encoders that processes each augmented view produced by T_COL/T_ISO into 512-dimensional representation vectors) — https://github.com/gankLei-X/DeepION
- **kornia** (Image-augmentation package used to implement the T_COL transforms (color jitter, filtering) within the DeepION augmentation module) — https://github.com/gankLei-X/DeepION
- **PyTorch** (Library in which DeepION (including the augmentation module and its contrastive training loop) is implemented) — https://github.com/gankLei-X/DeepION

## Examples

```
python run.py --input_Matrix .../DATASET/Pos_brain_data_matrix.txt --input_PeakList .../DATASET/Pos_brain_data_peak.csv --input_shape 198 422 --mode COL --ion_mode positive --num 5 --output_file Pos_COL_result
```

## Evaluation signals

- COL-mode benchmark: after training with T_COL augmentation, the COL mode of DeepION should achieve approximately 86.73% classification accuracy on the colocalized-ion task, outperforming the second-best SimSiam model by about 25%.
- ISO-mode benchmark: with the intensity-dependent missing-value augmentation, ISO mode should identify 249 monoisotope–isotope pairs at 75.90% accuracy in negative mode and 442 pairs at 92.76% accuracy in positive mode on the rat brain datasets.
- Isotope-detection sanity check: ISO mode should discover m/z 1547.8702 as the isotope of m/z 1544.8471, a pair that the PCC and R2 SIM-based baselines both fail to find.
- Representation shape check: each encoder output must be a 512-dimensional vector, subsequently reduced by UMAP to a 20-dimensional dense vector (the reduced dimension set as a trade-off between space–time complexity and information utilization).
- No-collapse check: the contrastive loss must be trained with the stop-gradient operation so the model does not produce collapse results during optimization.

## Limitations

- The augmentation design assumes the input MSI data have already been preprocessed into the [X*Y, P] matrix and [P, 1] peak list with the stated shape; raw instrument data are not handled.
- T_ISO's intensity-dependent missing value is additive and tied to ion intensity — it applies specifically to isotope ions from the same molecule and is not interchangeable with the COL-mode augmentation.
- The framework is implemented against specific pinned versions (python 3.5/3.6/3.7, scikit-learn 0.19.1, pytorch 1.8.2, umap 0.5.1, kornia 0.4.1, torchvision 0.11.3); deviating versions may alter augmentation behavior.
- The Poisson-noise and missing-value transforms assume the Poisson-distributed ion abundance and missing patterns characteristic of MSI; applying T_COL/T_ISO to non-MSI imaging data would misrepresent the noise model.

## Evidence

- [readme] T_COL composition: "the T_COL including color jitter, filtering, Poisson noise, and random missing value is carried in COL mode, while T_ISO introduces an additional process of intensity-dependent missing value in ISO"
- [abstract] MSI-specific augmentation rationale: "a unique data augmentation strategy is designed by considering the characteristics of MSI data, such as the Poisson distribution of ion abundance and a random pattern of missing values"
- [intro] Augmentation purpose for self-supervision: "we propose two data augmentation strategies especially for the COL and ISO mods by considering the characteristics of the Poisson distribution and missing patterns in MSI."
- [readme] Encoder consumes augmented views: "Two augmented images are propagated through a pair of ResNet18-based encoders that shared parameters, then output two 512-dimensional representation vectors"
- [methods] Stop-gradient anti-collapse: [evidence span withheld — anchor status: external_doc]
