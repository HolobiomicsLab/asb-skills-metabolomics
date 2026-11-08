---
name: joint-embedding-similarity-ranking
description: Use when you have a query tandem MS spectrum plus a candidate molecule
  list (e.g. PubChem candidates in cand_dict.pkl) and need to rank candidates by structural
  match, reporting rank@k metrics such as rank@5 and rank@20.
when_to_use_negative:
- The goal is de novo structure elucidation or in-silico fragmentation rather than
  re-ranking a supplied candidate list — JESTR ranks candidates from PubChem and does
  not generate structures (unlike SIRIUS or CFM-ID, which it outperforms by 31% and
  238% on MassSpecGym subsets).
- The dataset under test is not NPLIB1 or MassSpecGym and you cannot access it under
  the appropriate license — other datasets are withheld due to licensing agreements.
- Only CPU compute is available — the released weights are GPU-trained (nVidia A100,
  CUDA 11.8) and the environment must be set up for GPU.
edam_operation: http://edamontology.org/operation_3926
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: JESTR
  role: Core joint-embedding model whose pretrained weights embed spectra and candidate
    molecules for cosine-similarity ranking
  repo: https://github.com/HassounLab/JESTR1/
- name: PyTorch
  role: Framework under which all JESTR code and inference runs
  repo: https://github.com/HassounLab/JESTR1/
- name: DGL
  role: Provides molecule graph representations (molgraph_dict.pkl) used to embed
    candidate molecules
- name: rdkit
  role: Parses candidate molecule structures (mol_dict.pkl) for embedding
- name: cand_rank_canopus.py
  role: Script that runs candidate ranking for the complete NPLIB1 dataset
  repo: https://github.com/HassounLab/JESTR1/
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btaf354
    title: 'JESTR: Joint Embedding Space Technique for Ranking candidate molecules
      for the annotation of untargeted metabolomics data'
  source_content_hash: sha256:0338a0d5f46cf754411e19fe97cf5440fd81aceb46cefb82913f77276bd8f89c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HassounLab/JESTR1
  tools:
  - DGL
  - JESTR
  - PyTorch
  - Python
  - cand_rank_canopus.py
  - params.yaml
  - rdkit
  edam_operation: http://edamontology.org/operation_3926
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# joint-embedding-similarity-ranking

## Summary

Rank candidate molecule structures for an unannotated tandem MS spectrum by embedding the query spectrum and each candidate molecule into a joint space (JESTR) and ranking candidates by cosine similarity between spectrum and molecule embeddings.

## When to use

Use when you have a query tandem MS spectrum plus a candidate molecule list (e.g. PubChem candidates in cand_dict.pkl) and need to rank candidates by structural match, reporting rank@k metrics such as rank@5 and rank@20.

## When NOT to use

- The goal is de novo structure elucidation or in-silico fragmentation rather than re-ranking a supplied candidate list — JESTR ranks candidates from PubChem and does not generate structures (unlike SIRIUS or CFM-ID, which it outperforms by 31% and 238% on MassSpecGym subsets).
- The dataset under test is not NPLIB1 or MassSpecGym and you cannot access it under the appropriate license — other datasets are withheld due to licensing agreements.
- Only CPU compute is available — the released weights are GPU-trained (nVidia A100, CUDA 11.8) and the environment must be set up for GPU.

## Inputs

- data_dict.pkl — spectra dictionary indexed by spectra id with m/z and intensity arrays, metadata, and molecule SMILES/InChIKey
- split.pkl — train/valid/test split lists of InChIKeys
- cand_dict.pkl — PubChem candidate lists keyed by target molecule InChIKey (test set)
- mol_dict.pkl / molgraph_dict.pkl — InChIKey to rdkit mol structure and DGL graph mappings
- Pretrained JESTR model weights (from data/NPLIB1 or the MassSpecGym subset)

## Outputs

- Ranked candidate molecule lists per query spectrum (ordered by cosine similarity of embeddings)
- rank@5 and rank@20 metrics on the test set (e.g. written to ranking_metrics.csv)

## How to apply

Load the NPLIB1 spectra (data_dict.pkl, split.pkl, cand_dict.pkl) with dataset.py and utils.py under PyTorch/DGL/rdkit, and load the pretrained JESTR weights from data/NPLIB1 or the MassSpecGym subset. Embed the query spectrum and every candidate molecule, then compute cosine similarity between the spectrum embedding and each candidate embedding and sort candidates by that score. Note the model is trained with a dot product (CMSSP's CE loss) yet ranked by cosine similarity — this mismatch is deliberate and was shown to improve rank@5 by 22.5% and rank@20 by 36.8% over the dot-product ranking baseline. Run the whole NPLIB1 test set with cand_rank_canopus.py, which uses settings from params.yaml (exp dataset name, pretrained weight path, batch_size). Validate by comparing your rank@5/rank@20 values against the reported improvements of +22.5% and +36.8% relative to the default (dot-product) ranking function.

## Related tools

- **JESTR** (Core joint-embedding model whose pretrained weights embed spectra and candidate molecules for cosine-similarity ranking) — https://github.com/HassounLab/JESTR1/
- **PyTorch** (Framework under which all JESTR code and inference runs) — https://github.com/HassounLab/JESTR1/
- **DGL** (Provides molecule graph representations (molgraph_dict.pkl) used to embed candidate molecules)
- **rdkit** (Parses candidate molecule structures (mol_dict.pkl) for embedding)
- **cand_rank_canopus.py** (Script that runs candidate ranking for the complete NPLIB1 dataset) — https://github.com/HassounLab/JESTR1/

## Examples

```
python cand_rank_canopus.py
```

## Evaluation signals

- Cosine-similarity ranking must beat the dot-product ranking: expected improvements of +22.5% rank@5 and +36.8% rank@20 over the CMSSP model on NPLIB1
- Each test molecule's candidate ranking is derived from cosine similarity between the spectrum embedding and candidate embeddings — verify similarity scores are normalized cosine values, not raw dot products
- The test split comes from split.pkl (key 'test' InChIKeys) and candidates from cand_dict.pkl — confirm test molecules match their candidate dictionary entries by InChIKey
- Ranking metrics written to ranking_metrics.csv should contain both rank@5 and rank@20 for the two ranking functions being compared
- Pretrained weights must be the GPU-trained NPLIB1 or MassSpecGym checkpoints specified in params.yaml ('pretrained' parameter)

## Limitations

- Only the NPLIB1 dataset and pretrained weights are publicly released; other datasets (including full MassSpecGym subsets) are under licensing agreements that prohibit public release
- Model was trained and tested on an nVidia A100 GPU with CUDA 11.8; released weights are for GPU-trained models and a GPU environment is required
- Candidates are limited to those downloaded from PubChem for each target molecule; JESTR cannot propose structures outside the candidate list
- Package versions are only documented in jestr_requirements.txt; tool versions beyond that are not stated, though newer package versions may work

## Evidence

- [abstract] JESTR embeds molecules and their corresponding spectra in a joint space and ranks candidate structures by cosine similarity between embeddings of the query spectrum and each candidate.: "JESTR leverages the insight that molecules and their corresponding spectra are views of the same data and effectively embeds their representations in a joint space."
- [other] Cosine similarity ranking improves rank@5 by 22.5% and rank@20 by 36.8% over dot-product ranking on NPLIB1.: "Using cosine similarity for ranking surpasses using the dot product and improves rank@5 and rank@20 performance over the CMSSP model by 22.5% and 36.8%"
- [readme] cand_rank_canopus.py ranks candidates for the complete NPLIB1 dataset.: "To rank candidates for the complete NPLIB1 dataset, use the command: 

python cand_rank_canopus.py."
- [readme] cand_dict.pkl holds PubChem candidates for test molecules.: "cand_dict.pkl - dictionary keys by target molecule InchiKey. Each entry is a list of candidates for that molecule downloaded from PubChem. This dictionary is for the test set"
- [readme] Pretrained weights are available for NPLIB1 and MassSpecGym.: "Pretrained model weights are available both from the [NPLIB1](https://github.com/HassounLab/JESTR1/tree/main/data/NPLIB1) as well as from the [MassSpecGym](https://github.com/HassounLab/JESTR1/tree/ma"
