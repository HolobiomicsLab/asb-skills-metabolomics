---
name: batch-construction-for-pairwise-learning
description: Use when building training inputs for a pairwise (RankNet) retention-order
  model from RT records — i.e., you have retention-time datasets (e.g., dataset_83
  reference / dataset_84 test) and must generate ordered molecule pairs and D-MPNN
  embeddings.
when_to_use_negative:
- The task requires absolute retention-time prediction rather than pairwise elution-order
  ranking — ROASMI's RankNet learns relative order, not RT values
- Input peaks are already known to have indistinguishable retention (close-order pairs
  that the model cannot separate); per the article, 'Such pairs should be excluded
  from the subsequent small molecule identification process'
- The target chromatographic space is HILIC rather than RPLC — ROASMI currently applies
  only to RPLC systems, and HILIC models are future work
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3318
tools:
- name: ROASMI
  role: Provides code/ROASMI_train.py, which executes pair generation, batching, and
    D-MPNN encoding in the default retraining setup
  repo: https://github.com/FangYuan717/ROASMI
- name: chemprop
  role: Supplies the directed message passing neural network (D-MPNN) used as the
    molecular embedding module that converts molecular graphs into continuous vectors
  repo: https://github.com/chemprop/chemprop
- name: PyTorch
  role: Deep-learning framework underpinning the D-MPNN embedding and RankNet modules;
    a GPU-enabled version may be needed
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s13321-025-00968-8
    title: 'ROASMI: accelerating small molecule identification by repurposing retention
      data'
  source_content_hash: sha256:ad81af2899247b7538518d2b847358bbc7615c5ed3a822ca04c2565edf8a2321
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FangYuan717/ROASMI
  tools:
  - PyTorch
  - ROASMI
  - chemprop
  - conda
  - cuDNN
  - cuda
  - pip
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7647058823529411
n_claims: 17
---

# batch-construction-for-pairwise-learning

## Summary

Constructs ordered molecule-pair training inputs for ROASMI's retention-order model: compounds are encoded as isomeric SMILES (retrieved from PubChem), adjacent molecules within a 30-s retention interval are paired with elution-order labels, grouped into batches of 64, and converted to molecular embeddings by a chemprop-based D-MPNN.

## When to use

Use when building training inputs for a pairwise (RankNet) retention-order model from RT records — i.e., you have retention-time datasets (e.g., dataset_83 reference / dataset_84 test) and must generate ordered molecule pairs and D-MPNN embeddings.

## When NOT to use

- The task requires absolute retention-time prediction rather than pairwise elution-order ranking — ROASMI's RankNet learns relative order, not RT values
- Input peaks are already known to have indistinguishable retention (close-order pairs that the model cannot separate); per the article, 'Such pairs should be excluded from the subsequent small molecule identification process'
- The target chromatographic space is HILIC rather than RPLC — ROASMI currently applies only to RPLC systems, and HILIC models are future work

## Inputs

- Retention-time reference records (e.g., 'dataset_83' as the default retraining set in ROASMI_train.py)
- Isomeric SMILES strings retrieved from PubChem for each compound
- Retention interval threshold (30 s) and batch size (64, TrainArgs default in args.py)

## Outputs

- Ordered molecule-pair batches with elution-order labels (e.g., saved as 'molecule_pair_batches.pt')
- D-MPNN molecular embeddings per compound (e.g., saved as 'molecule_embeddings.npy')
- Retrained ROASMI RankNet model

## How to apply

Load the reference RT records (default: 'dataset_83' per code/ROASMI_train.py) and fetch each compound's isomeric SMILES from PubChem. Construct pairwise retention sequences by pairing adjacent molecules whose RTs differ within the 30-s retention interval, labeling each pair (Ci, Cj) with its elution order (Ci elutes before Cj). Group pairs into training batches of size 64 (TrainArgs default in args.py), then feed each batch's SMILES strings through the D-MPNN directed message-passing encoder, which iteratively aggregates atom and bond features and sums them into a single molecular embedding per molecule. Finally, save the molecule-pair batches and their embeddings to disk (e.g., 'molecule_pair_batches.pt' and 'molecule_embeddings.npy'). The rationale is that ranking-style learning from elution order pairs, rather than absolute RT regression, transfers across chromatographic methods where absolute RTs replicate poorly.

## Related tools

- **ROASMI** (Provides code/ROASMI_train.py, which executes pair generation, batching, and D-MPNN encoding in the default retraining setup) — https://github.com/FangYuan717/ROASMI
- **chemprop** (Supplies the directed message passing neural network (D-MPNN) used as the molecular embedding module that converts molecular graphs into continuous vectors) — https://github.com/chemprop/chemprop
- **PyTorch** (Deep-learning framework underpinning the D-MPNN embedding and RankNet modules; a GPU-enabled version may be needed)

## Examples

```
python code/ROASMI_train.py
```

## Evaluation signals

- All saved molecule-pair batches ('molecule_pair_batches.pt') are non-empty
- The D-MPNN module runs on the batch SMILES strings without error, matching the paper's default retraining setup (script_runs on code/ROASMI_train.py)
- Every training pair (Ci, Cj) satisfies the 30-s retention-interval constraint and carries a consistent elution-order label
- Batch size in TrainArgs (args.py) equals the default 64
- Embedding arrays in 'molecule_embeddings.npy' have consistent dimensionality across all molecules

## Limitations

- Pair labels inherit RT replicability problems: retention order of very close chromatographic peaks (within the 30-s interval) may not be consistent across experiments, so some pairs are effectively indistinguishable
- Compounds are restricted to conventional elements (C, H, N, O, P, S); broader chemical space is unexplored
- Pairs from slightly shifted experimental conditions can produce RT outliers that the model does not resolve

## Evidence

- [methods] retraining workflow: "To retrain ROASMI, run: python code/ROASMI_train.py"
- [methods] molecular embedding module: "A Ranking Neural Network (RankNet) was used to learn elution orders"
- [readme] D-MPNN description: "A directed message transfer neural network (D-MPNN) is used to learn directly from the structure of compounds"
- [readme] embedding module provenance: "The molecular embedding module. This module is an extension of chemprop"
- [discussion] indistinguishable pair exclusion: "Such pairs should be excluded from the subsequent small molecule identification process."
- [methods] reference/test split: "retrain two HILIC models using 'dataset_83' as the reference set and 'dataset_84' as the test set"
