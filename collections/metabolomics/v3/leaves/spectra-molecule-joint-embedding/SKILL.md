---
name: spectra-molecule-joint-embedding
description: Use when annotating untargeted metabolomics spectra by ranking a candidate
  molecule list (e.g. PubChem candidates keyed by InChIKey) for a query spectrum with
  no reference spectrum match, and you have pretrained spectra/molecule embedding
  weights available.
when_to_use_negative:
- If the goal is spectral-library matching against reference spectra (the task is
  candidate-list ranking, not spectrum-spectrum matching)
- If no candidate list is available or the user is not licensed to use datasets other
  than NPLIB1/MassSpecGym — other datasets are under licensing agreements that prohibit
  their public release
- If the environment cannot be set up for GPU; the released weights are for GPU-trained
  models and code was tested on nVidia A100 with CUDA 11.8
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_3474
tools:
- name: JESTR
  role: 'Main framework: two-stage contrastive + MLP training to embed spectra and
    molecules in a joint space and rank candidates by cosine similarity'
  repo: https://github.com/HassounLab/JESTR1/
- name: PyTorch
  role: Framework under which all JESTR code runs (training and inference)
- name: rdkit
  role: Provides molecule structures per InChIKey from mol_dict.pkl for molecular
    embedding input
- name: DGL
  role: Provides runtime-optimized molecule graph representations per InChIKey (molgraph_dict.pkl)
- name: SIRIUS
  role: Baseline used for pretrained-embedding comparison and performance benchmarking
    (outperformed by 31% on MassSpecGym subsets)
- name: CFM-ID
  role: Baseline used for pretrained-embedding comparison and performance benchmarking
    (outperformed by 238% on MassSpecGym subsets)
- name: params.yaml
  role: Configuration file setting dataset, batch sizes, epochs, aug_cands regularization
    flag, learning rates, early stopping and pretrained weight path
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btaf354
    title: 'JESTR: Joint Embedding Space Technique for Ranking candidate molecules
      for the annotation of untargeted metabolomics data'
  source_content_hash: sha256:7706aafc6485b916ccbf7833873de3d6a84bbb317ea26e8ac0be5200ccefddf9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HassounLab/JESTR1
  tools:
  - CFM-ID
  - DGL
  - JESTR
  - PyTorch
  - Python
  - SIRIUS
  - params.yaml
  - rdkit
  - utils.py
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 18
---

# spectra-molecule-joint-embedding

## Summary

Embeds mass spectra and their candidate molecules into a joint representation space and ranks candidates by cosine similarity between the query-spectrum embedding and each molecule embedding. It applies a candidate-regularization training strategy that reduces target-candidate embedding similarity and boosts rank@1. Code and pretrained weights are available for the NPLIB1 dataset.

## When to use

Use when annotating untargeted metabolomics spectra by ranking a candidate molecule list (e.g. PubChem candidates keyed by InChIKey) for a query spectrum with no reference spectrum match, and you have pretrained spectra/molecule embedding weights available.

## When NOT to use

- If the goal is spectral-library matching against reference spectra (the task is candidate-list ranking, not spectrum-spectrum matching)
- If no candidate list is available or the user is not licensed to use datasets other than NPLIB1/MassSpecGym — other datasets are under licensing agreements that prohibit their public release
- If the environment cannot be set up for GPU; the released weights are for GPU-trained models and code was tested on nVidia A100 with CUDA 11.8

## Inputs

- Query mass spectra (m/z and intensity arrays with metadata, molecule SMILES and InChIKey) from data_dict.pkl
- Per-target-molecule candidate lists downloaded from PubChem, keyed by InChIKey, from cand_dict.pkl (test) and cand_dict_train_updated.pkl (regularization)
- rdkit mol structures per InChIKey from mol_dict.pkl and DGL graphs from molgraph_dict.pkl
- train/valid/test InChIKey lists from split.pkl
- inchi_to_id_dict.pkl / inchi_to_id_dict_wneg.pkl mapping InChIKeys to spectra ids
- params.yaml configuration (exp, batch_size, num_epochs, aug_cands, contr_lr, final_lr, early_stopping, pretrained weight path)

## Outputs

- Ranked candidate list per query spectrum, ordered by cosine similarity between the spectrum embedding and each candidate molecule embedding
- Joint-space embeddings of spectra and molecules (e.g. as used in JESTR.ipynb ranking demo)
- rank@1–rank@20 performance statistics vs. baselines (SIRIUS, CFM-ID)
- Target-candidate cosine-similarity distributions quantifying the regularization effect (Figure 5C)

## How to apply

Load the target spectra and per-molecule candidate lists from data_dict.pkl and cand_dict.pkl (split by InChIKey via split.pkl) using dataset.py and utils.py; embed molecules from mol_dict.pkl (rdkit mol) or molgraph_dict.pkl (DGL graph) and their spectra with the pretrained weights in the path set by 'pretrained' in params.yaml (models.py model classes). Compute the cosine similarity between each query-spectrum embedding and each of its candidate embeddings, then rank candidates by descending similarity. For training from scratch, run two-stage training: contrastive training (train_contr.py) followed by final MLP training (train.py), optionally setting aug_cands=True in params.yaml to regularize with candidate molecules from cand_dict_train_updated.pkl — this boosts rank@1 by 5.72% across all datasets. The environment must be GPU-enabled (nVidia A100, CUDA 11.8) and packages pinned per jestr_requirements.txt.

## Related tools

- **JESTR** (Main framework: two-stage contrastive + MLP training to embed spectra and molecules in a joint space and rank candidates by cosine similarity) — https://github.com/HassounLab/JESTR1/
- **PyTorch** (Framework under which all JESTR code runs (training and inference))
- **rdkit** (Provides molecule structures per InChIKey from mol_dict.pkl for molecular embedding input)
- **DGL** (Provides runtime-optimized molecule graph representations per InChIKey (molgraph_dict.pkl))
- **SIRIUS** (Baseline used for pretrained-embedding comparison and performance benchmarking (outperformed by 31% on MassSpecGym subsets))
- **CFM-ID** (Baseline used for pretrained-embedding comparison and performance benchmarking (outperformed by 238% on MassSpecGym subsets))
- **params.yaml** (Configuration file setting dataset, batch sizes, epochs, aug_cands regularization flag, learning rates, early stopping and pretrained weight path)

## Examples

```
python cand_rank_canopus.py
```

## Evaluation signals

- Regularized vs non-regularized average target-candidate cosine similarity must be lower with regularization on the NPLIB1 test set (Figure 5C reduction)
- rank@1 improvement from candidate regularization should be ≈5.72% across datasets
- Average rank@1–20 uplift vs SIRIUS and CFM-ID on MassSpecGym subsets of ~31% and ~238% respectively; overall 55.5%–302.6% across four datasets
- Every target InChIKey in split.pkl test list must have a ranked candidate list drawn from cand_dict.pkl before scoring
- Ranking must be reproducible by running JESTR.ipynb for a demo or cand_rank_canopus.py for the full NPLIB1 dataset

## Limitations

- Only the NPLIB1 dataset and pretrained weights are released; MassSpecGym subsets and other datasets are under licensing agreements that prohibit their public release, so users must ensure datasets are available to them under appropriate licenses
- Models were trained and tested on GPU (nVidia A100, CUDA 11.8); released weights are for GPU-trained models — CPU-only environments may not reproduce results
- Some large NPLIB1 files are not checked into GitHub and must be downloaded from Zenodo per the data/NPLIB1/*.txt instructions
- Package versions are pinned in jestr_requirements.txt; newer versions may work but were not the tested configuration
- If not training with regularization (aug_cands=False), cand_dict_train_updated.pkl is not needed and can be skipped

## Evidence

- [abstract] Joint space ranking method: [evidence span withheld — anchor status: not_found]
- [abstract] Regularization gain: [evidence span withheld — anchor status: not_found]
- [abstract] Performance uplift: "On average, for rank@[1-20], JESTR outperforms other tools by 55.5%-302.6%."
- [readme] Dataset file layout: "data_dict.pkl - a dictionary of spectra information indexed by spectra id. Each entry is a dictionary for the spectra which contains the m/z amd intensity arrays, metadata and molecule SMILES and"
- [readme] Candidate file purpose: "cand_dict.pkl - dictionary keys by target molecule InchiKey. Each entry is a list of candidates for that molecule downloaded from PubChem. This dictionary is for the test set"
- [readme] Regularization flag: "aug_cands - whether to do regularization or not"
- [readme] Two-stage training: "train_contr.py - script to do contrastive training"
- [readme] Ranking command: "To rank candidates for the complete NPLIB1 dataset, use the command: 

python cand_rank_canopus.py."
- [readme] GPU environment: "The model was trained and tested on GPU nVidia A100 with CUDA 11.8."
