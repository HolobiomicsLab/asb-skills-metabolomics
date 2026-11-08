---
name: retention-order-supervised-training-pipeline
description: Use when raw reference retention-time records (e.g. dataset_83 for the
  default RPLC retraining setup) must be converted into pairwise elution-order training
  data with D-MPNN molecular embeddings for retraining or building a RankNet retention
  model.
when_to_use_negative:
- The goal is scoring/annotating already-identified peaks rather than retraining —
  use ROASMI_predict.py and Identify.py instead of this training pipeline
- 'The chromatographic system is HILIC: the article states ROASMI currently applies
  only to RPLC systems, and HILIC models with generalizability are future work'
- Compounds fall outside the elemental restrictions of C, H, N, O, P, and S that the
  model's application domain covers
edam_topics:
- http://edamontology.org/topic_2258
- http://edamontology.org/topic_3465
tools:
- name: ROASMI
  role: Provides the training pipeline (code/ROASMI_train.py), pairing logic, and
    RankNet retention prediction module for supervised elution-order learning
  repo: https://github.com/FangYuan717/ROASMI
- name: chemprop
  role: 'The molecular embedding module: a chemprop extension implementing the D-MPNN
    directed message passing encoder that converts molecular graphs into continuous
    vectors'
  repo: https://github.com/chemprop/chemprop
- name: PyTorch
  role: Deep-learning backend for running the D-MPNN embedding and RankNet training
    (GPU-enabled install recommended for cuda >= 8.0)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s13321-025-00968-8
    title: 'ROASMI: accelerating small molecule identification by repurposing retention
      data'
  source_content_hash: sha256:9dded48e1a3b085eac817196430389cf16d290b353bf614ac708b68cdc7201dc
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
  - http://edamontology.org/topic_2258
  - http://edamontology.org/topic_3465
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7575757575757576
n_claims: 33
---

# retention-order-supervised-training-pipeline

## Summary

The ROASMI training-input construction pipeline: build ordered molecule pairs from retention-time records (pairs labeled by elution order within a 30-s RT interval), encode each compound's isomeric SMILES with a chemprop-based D-MPNN, and train a RankNet retention-order model on the resulting embeddings.

## When to use

Use when raw reference retention-time records (e.g. dataset_83 for the default RPLC retraining setup) must be converted into pairwise elution-order training data with D-MPNN molecular embeddings for retraining or building a RankNet retention model.

## When NOT to use

- The goal is scoring/annotating already-identified peaks rather than retraining — use ROASMI_predict.py and Identify.py instead of this training pipeline
- The chromatographic system is HILIC: the article states ROASMI currently applies only to RPLC systems, and HILIC models with generalizability are future work
- Compounds fall outside the elemental restrictions of C, H, N, O, P, and S that the model's application domain covers

## Inputs

- Reference retention-time records (e.g. 'dataset_83' for the default RPLC pH ~2.7 retraining setup)
- Isomeric SMILES strings for each compound (retrieved from PubChem)
- Retrieval interval threshold (30 s) and batch size (64, default in TrainArgs in args.py)

## Outputs

- Ordered molecule pairs labeled with elution order (Ci elutes before Cj within a 30-s RT interval)
- Molecule-pair training batches (e.g. saved as 'molecule_pair_batches.pt')
- D-MPNN molecular embeddings per compound (e.g. saved as 'molecule_embeddings.npy')
- Trained RankNet retention-order model(s) (ROASMI ensemble models)

## How to apply

Load the reference RT records (default 'dataset_83' per code/ROASMI_train.py) and retrieve each compound's isomeric SMILES from PubChem. Construct pairwise retention sequences by pairing adjacent molecules whose RTs differ within a 30-s retention interval, yielding ordered pairs (Ci, Cj) labeled with elution order (Ci elutes before Cj). Group pairs into training batches of batch size 64 (the default in TrainArgs in args.py), then feed each batch's SMILES through the D-MPNN directed message passing encoder, which iteratively aggregates atom and bond features and sums them into a single molecular embedding per molecule. Train the RankNet retention prediction module on these embeddings to learn elution orders; note the paper trains an ensemble (ROASMI_1–ROASMI_5) so prediction variance can quantify uncertainty. Save the generated molecule-pair batches and embeddings (e.g. 'molecule_pair_batches.pt' and 'molecule_embeddings.npy').

## Related tools

- **ROASMI** (Provides the training pipeline (code/ROASMI_train.py), pairing logic, and RankNet retention prediction module for supervised elution-order learning) — https://github.com/FangYuan717/ROASMI
- **chemprop** (The molecular embedding module: a chemprop extension implementing the D-MPNN directed message passing encoder that converts molecular graphs into continuous vectors) — https://github.com/chemprop/chemprop
- **PyTorch** (Deep-learning backend for running the D-MPNN embedding and RankNet training (GPU-enabled install recommended for cuda >= 8.0))

## Examples

```
python code/ROASMI_train.py
```

## Evaluation signals

- All saved molecule-pair batches are non-empty (validation criterion from the paper's workflow)
- The D-MPNN module runs on the batch SMILES strings without error under the paper's default retraining setup (script_runs on code/ROASMI_train.py)
- Each training pair is a genuine adjacent elution pair whose RT difference falls within the 30-s retention interval, with correct elution-order labels
- Molecular embeddings are single continuous vectors per compound (D-MPNN atom/bond messages summed into one molecular embedding)
- Training uses the paper's default batch size of 64 as defined in TrainArgs in args.py

## Limitations

- Retention order of very close chromatographic peaks (within the pairing interval) may not be consistent across experiments, and such indistinguishable pairs should be excluded from downstream identification
- The trained models apply only to RPLC systems (eluent pH ~2.7); HILIC model construction with generalizability is future work
- ROASMI is limited to conventional compounds within elemental restrictions of C, H, N, O, P, and S
- RT outliers resulting from slight experimental condition changes remain unresolved and require mechanistic study

## Evidence

- [readme] D-MPNN encoder basis: "A directed message transfer neural network (D-MPNN) is used to learn directly from the structure of compounds, allowing prediction of compounds in new chemical spaces"
- [methods] RankNet retention module: "The retention prediction module. A Ranking Neural Network (RankNet) was used to learn elution orders"
- [methods] Retraining command: "To retrain ROASMI, run: python code/ROASMI_train.py"
- [methods] Default reference/test datasets: "retrain two HILIC models using 'dataset_83' as the reference set and 'dataset_84' as the test set"
- [readme] Ensemble uncertainty rationale: "The ensemble approach allowed quantifying model uncertainty using the variance of the retention order predictions across the trained models."
