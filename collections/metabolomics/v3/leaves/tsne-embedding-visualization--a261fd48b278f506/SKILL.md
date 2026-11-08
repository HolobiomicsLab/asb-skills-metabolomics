---
name: tsne-embedding-visualization
description: Use when you must build a structurally informed train/test split — e.g.
  testing how a spectral-prediction ranker generalizes to molecules dissimilar from
  its training set — by embedding binary fingerprints and grouping molecules into
  structural clusters.
when_to_use_negative:
- When the goal is a standard random train/test split — the paper's random-split evaluation
  already reports average rank 5.504, and this skill exists specifically to move away
  from that setup.
- When embeddings are already 2-D or low-dimensional and do not need t-SNE reduction
  before clustering.
- When the downstream dataset is not torchgeometric-based (data_trvate.py and data_tecand.py
  emit .pkl torchgeometric files specific to this pipeline).
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3474
tools:
- name: rdkit
  role: compute MACCS structural fingerprints for each molecule from the mol objects
    in mol_dict.pkl
- name: scikit-learn
  role: fit t-SNE to reduce MACCS fingerprints to two dimensions
- name: scipy
  role: UPGMA hierarchical clustering over the first two t-SNE dimensions
- name: PyTorch Geometric
  role: data_trvate.py / data_tecand.py write the realistic-split train and test candidate
    sets as torchgeometric .pkl files
- name: ESP (Ensemble Spectral Prediction)
  role: the downstream model whose realistic-split ranking metrics this split-based
    evaluation is designed to probe
  repo: https://github.com/HassounLab/ESP
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btae490
    title: An Ensemble Spectral Prediction (ESP) model for metabolite annotation
  source_content_hash: sha256:426dd593b04d53860ba295ca4b767952085554e53ba1db6aee54ab4fb8b33fae
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HassounLab/ESP
  tools:
  - ESP (Ensemble Spectral Prediction)
  - PyTorch Geometric
  - dgl
  - numpy
  - rdkit
  - scikit-learn
  - scipy
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# tsne-embedding-visualization

## Summary

Reduce high-dimensional molecular MACCS structural fingerprints to a 2-D t-SNE embedding with scikit-learn, then cluster the 2-D points with UPGMA hierarchical clustering in scipy, to construct a realistic (cluster-based) training/testing split that probes how annotation ranking performance degrades on less popular molecules.

## When to use

Use when you must build a structurally informed train/test split — e.g. testing how a spectral-prediction ranker generalizes to molecules dissimilar from its training set — by embedding binary fingerprints and grouping molecules into structural clusters.

## When NOT to use

- When the goal is a standard random train/test split — the paper's random-split evaluation already reports average rank 5.504, and this skill exists specifically to move away from that setup.
- When embeddings are already 2-D or low-dimensional and do not need t-SNE reduction before clustering.
- When the downstream dataset is not torchgeometric-based (data_trvate.py and data_tecand.py emit .pkl torchgeometric files specific to this pipeline).

## Inputs

- pos_train.csv (NPLIB1 dataset, labelled molecules in csv format)
- mol_dict.pkl (dictionary mapping InChIKeys to rdkit mol objects)
- MACCS structural fingerprints per molecule (rdkit)

## Outputs

- Two-dimensional t-SNE embedding of MACCS fingerprints (one 2-D point per molecule)
- 50 UPGMA clusters over the 2-D embedding, partitioned 29 train / 21 test
- Realistic-split train/validate/test torchgeometric files (torch_trvate_<res>bin.pkl)
- Realistic-split test candidate files (torch_tecand_<res>bin_te_cand<sz>_torchgeometric.pkl)

## How to apply

Start from a labelled molecular dataset such as the ESP NPLIB1 pos_train.csv and compute binary MACCS structural fingerprints for each molecule with rdkit (mol_dict.pkl maps InChIKeys to rdkit mol objects). Fit scikit-learn's t-SNE to reduce the MACCS fingerprints to exactly two dimensions. Run UPGMA hierarchical clustering (scipy) on the first two t-SNE dimensions to form 50 clusters, assigning 29 clusters to training and 21 clusters to testing so that structurally related molecules stay in the same side of the split. Regenerate the torchgeometric training/validation/test files with data_trvate.py and the test-candidate files with data_tecand.py using this cluster-based split, then train the MLP/GNN base models with train.py and the ESP ensemble with ens_train_canopus.py (mode 'canopus', correlation_mat_rank 100) on the realistic-split training set. Evaluate on the realistic-split test candidate set; the rationale is that cluster-based splitting removes structurally similar near-duplicates between train and test, exposing the degradation caused by testing on less popular molecules.

## Related tools

- **rdkit** (compute MACCS structural fingerprints for each molecule from the mol objects in mol_dict.pkl)
- **scikit-learn** (fit t-SNE to reduce MACCS fingerprints to two dimensions)
- **scipy** (UPGMA hierarchical clustering over the first two t-SNE dimensions)
- **PyTorch Geometric** (data_trvate.py / data_tecand.py write the realistic-split train and test candidate sets as torchgeometric .pkl files)
- **ESP (Ensemble Spectral Prediction)** (the downstream model whose realistic-split ranking metrics this split-based evaluation is designed to probe) — https://github.com/HassounLab/ESP

## Examples

```
python data_trvate.py && python data_tecand.py  # after computing MACCS fingerprints and running t-SNE + UPGMA clustering to build the realistic cluster-based split
```

## Evaluation signals

- The t-SNE embedding must be exactly 2-D, since UPGMA clustering is applied on the first two dimensions.
- The UPGMA clustering must yield 50 clusters, split as 29 train / 21 test.
- Regenerated realistic-split files must match the naming pattern torch_trvate_<res>bin.pkl and torch_tecand_<res>bin_te_cand<sz>_torchgeometric.pkl.
- Sanity check against reported numbers: ESP average rank should degrade from 5.504 on the random split to 16.532 on the realistic split when ranking against 100 random molecules from the candidate set.
- Confirm the same mol_dict.pkl InChIKey-to-mol-object mapping is used so fingerprints are computed on the same molecules as the original split.

## Limitations

- Realistic-split evaluation is only as good as the fingerprint choice — MACCS fingerprints and t-SNE dimensionality here are tuned for NPLIB1 and may need re-tuning for other datasets.
- UPGMA over 2-D t-SNE embeddings is sensitive to t-SNE randomness; different random seeds can give different cluster assignments.
- The 29/21 cluster train/test allocation is a fixed heuristic, not an optimized split ratio.
- Reproducibility: NIST-20 training data cannot be published under NIST license, so reproducing beyond the NPLIB1 realistic split is restricted.

## Evidence

- [methods] Load the NPLIB1 dataset (pos_train.csv) and compute MACCS structural fingerprints for each molecule using rdkit.: [evidence span withheld — anchor status: not_found]
- [methods] Reduce the fingerprints to two dimensions with t-SNE using scikit-learn.: [evidence span withheld — anchor status: not_found]
- [methods] UPGMA hierarchical clustering in scipy to form 50 clusters, assigning 29 clusters to training and 21 clusters to testing.: [evidence span withheld — anchor status: not_found]
- [results] ESP average rank degrades from 5.504 (random split) to 16.532 (realistic split): [evidence span withheld — anchor status: not_found]
- [readme] mol_dict.pkl is a dictionary mapping InChiKeys to rdkit mol objects.: "pos_train.csv is the NPLIB1 dataset in csv format. mol_dict.pkl is a dictionary mapping InChiKeys to rdkit mol objects."
