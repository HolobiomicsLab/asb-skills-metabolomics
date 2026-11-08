---
name: regularization-effect-quantification
description: Use when you need to quantify the embedding-space discrimination effect
  of JESTR's candidate-based regularization — i.e. measure whether candidate regularization
  reduces average target-candidate cosine similarity in a joint embedding space.
when_to_use_negative:
- When the goal is end-to-end candidate ranking (rank@1–20) rather than isolating
  the regularization effect — use cand_rank_canopus.py or JESTR.ipynb directly instead
- When regularization was not applied during training (aug_cands set to False), since
  there is no regularized configuration to contrast against
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: JESTR
  role: Embedding model whose regularized and non-regularized configurations are compared
    in the joint space
  repo: https://github.com/HassounLab/JESTR1/
- name: PyTorch
  role: Framework under which the JESTR model and cosine-similarity computation are
    executed
- name: rdkit
  role: Provides mol structures via mol_dict.pkl used to embed candidate molecules
- name: DGL
  role: Provides molgraph graphs via molgraph_dict.pkl for runtime-optimized embedding
- name: params.yaml
  role: Configuration file setting the pretrained model weight path and the aug_cands
    regularization flag
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btaf354
    title: 'JESTR: Joint Embedding Space Technique for Ranking candidate molecules
      for the annotation of untargeted metabolomics data'
  source_content_hash: sha256:69e1b5696acad6db7f74fd59cde458879d9b5086822dca0e7b0c0743cfe7563e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HassounLab/JESTR1
  tools:
  - DGL
  - JESTR
  - PyTorch
  - Python
  - params.yaml
  - rdkit
  - utils.py
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5625
n_claims: 16
---

# regularization-effect-quantification

## Summary

Quantifies how JESTR's candidate-based regularization reduces the average cosine similarity between target-molecule and candidate-molecule embeddings in the NPLIB1 test set, isolating the regularization effect on embedding-space discrimination from overall rank@1 gains.

## When to use

Use when you need to quantify the embedding-space discrimination effect of JESTR's candidate-based regularization — i.e. measure whether candidate regularization reduces average target-candidate cosine similarity in a joint embedding space.

## When NOT to use

- When the goal is end-to-end candidate ranking (rank@1–20) rather than isolating the regularization effect — use cand_rank_canopus.py or JESTR.ipynb directly instead
- When regularization was not applied during training (aug_cands set to False), since there is no regularized configuration to contrast against

## Inputs

- NPLIB1 test-set spectra (data_dict.pkl: m/z and intensity arrays, metadata, molecule SMILES and InChIKey)
- cand_dict.pkl: PubChem candidate lists keyed by target molecule InChIKey (test set)
- mol_dict.pkl and/or molgraph_dict.pkl (rdkit mol structures / DGL graphs keyed by InChIKey)
- split.pkl with keys 'train'/'valid'/'test' (lists of InChIKeys)
- Pretrained JESTR model weights (GPU-trained, nVidia A100 / CUDA 11.8)
- params.yaml (pretrained weight path, aug_cands flag, batch sizes, learning rates)

## Outputs

- Cosine-similarity distributions (mean and spread) of target-candidate pairs for regularized vs non-regularized model configurations
- Quantified reduction in average cosine similarity attributable to candidate regularization
- Side-by-side histogram/density plots reproducing Figure 5C

## How to apply

1. Load the NPLIB1 test set (data_dict.pkl, cand_dict.pkl, mol_dict.pkl/molgraph_dict.pkl, split.pkl) with dataset.py and utils.py. 2. Embed target molecules and their PubChem candidate molecules with the pretrained JESTR model (both regularized and non-regularized configurations) using models.py, pointing pretrained weights to the path set in params.yaml. 3. Compute cosine similarities between each target embedding and its candidate embeddings for both configurations. 4. Compute distribution statistics (mean, spread) for the with- vs without-regularization models and quantify the reduction in average similarity. 5. Plot the two distributions side by side (histogram/density) to reproduce Figure 5C. Validation: the regularized model's average target-candidate cosine similarity must be lower than the non-regularized model's, with the reduction consistent with the 5.72% rank@1 improvement reported from regularization across datasets.

## Related tools

- **JESTR** (Embedding model whose regularized and non-regularized configurations are compared in the joint space) — https://github.com/HassounLab/JESTR1/
- **PyTorch** (Framework under which the JESTR model and cosine-similarity computation are executed)
- **rdkit** (Provides mol structures via mol_dict.pkl used to embed candidate molecules)
- **DGL** (Provides molgraph graphs via molgraph_dict.pkl for runtime-optimized embedding)
- **params.yaml** (Configuration file setting the pretrained model weight path and the aug_cands regularization flag)

## Examples

```
python cand_rank_canopus.py  # then compute cosine sims between target and candidate embeddings via models.py, for both aug_cands=True and aug_cands=False
```

## Evaluation signals

- Average target-candidate cosine similarity of the regularized model must be lower than the non-regularized model's on the NPLIB1 test set
- The magnitude of the similarity reduction must be consistent with the reported 5.72% rank@1 improvement from regularization across datasets
- Distributions should be plotted side by side to reproduce Figure 5C
- Both configurations must use the same pretrained weight path from params.yaml so that the only variable is the regularization setting
- Side-by-side comparison must use the same NPLIB1 test set (split.pkl 'test' InChIKeys) for both model configurations

## Limitations

- GPU-trained pretrained weights are required — released weights are for GPU models (nVidia A100, CUDA 11.8); CPU-only environments are not covered
- Only the NPLIB1 dataset is publicly released; other datasets (e.g. MassSpecGym subsets) require appropriate licenses
- Some large NPLIB1 data files must be fetched from zenodo per the instructions in data/NPLIB1/*.txt
- The article's finding is specific to the NPLIB1 test set — the regularized/non-regularized contrast is reported there, with cross-dataset generalization only implied via the rank@1 gain

## Evidence

- [results] Regularization reduces cosine similarity in joint embedding space: [evidence span withheld — anchor status: not_found]
- [abstract] Regularization improves rank@1: "regularization with candidate molecules during training, boosting rank@1 performance by 5.72% across all datasets"
- [readme] params.yaml governs regularization and pretrained weights: "aug_cands - whether to do regularization or not"
- [readme] Pretrained weights and configuration location: "Pretrained model weights are available both from the [NPLIB1](https://github.com/HassounLab/JESTR1/tree/main/data/NPLIB1) as well as from the [MassSpecGym](https://github.com/HassounLab/JESTR1/tree/ma"
- [readme] Candidate dictionary source: "cand_dict.pkl - dictionary keys by target molecule InchiKey. Each entry is a list of candidates for that molecule downloaded from PubChem"
