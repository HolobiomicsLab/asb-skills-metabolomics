---
name: retention-time-pairing-from-adjacent-chromatographic-peaks
description: Use when building pairwise training data from a reference retention-time
  dataset (e.g., 'dataset_83') where ordered molecule pairs (Ci eluting before Cj)
  are needed to train a ranking model like RankNet.
when_to_use_negative:
- Input compounds lie outside the conventional elemental restrictions of C, H, N,
  O, P, and S, since ROASMI's chemical space is limited to those elements.
- The chromatographic system is HILIC rather than RPLC — ROASMI currently applies
  only to RPLC systems and HILIC models with generalizability remain future work.
- A retention-order (RankNet) training setup is not needed, e.g., when only predicting
  retention scores with the pre-trained ROASMI_1–ROASMI_5 ensemble.
edam_operation: http://edamontology.org/operation_3925
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0089
tools:
- name: ROASMI
  role: Framework whose training pipeline (code/ROASMI_train.py) performs RT pairing,
    batching, and embedding generation
  repo: https://github.com/FangYuan717/ROASMI
- name: chemprop
  role: Provides the D-MPNN directed message passing encoder used as the molecular
    embedding module that converts molecular graphs into continuous vectors
  repo: https://github.com/chemprop/chemprop
- name: PyTorch
  role: Backend framework on which the D-MPNN embedding and training run (GPU-enabled
    version recommended)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s13321-025-00968-8
    title: 'ROASMI: accelerating small molecule identification by repurposing retention
      data'
  source_content_hash: sha256:95cff8379baf9985610d6f7b9686095da47eb4aab181404d67e793cc31070712
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
  edam_operation: http://edamontology.org/operation_3925
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0089
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 14
---

# retention-time-pairing-from-adjacent-chromatographic-peaks

## Summary

Constructs training inputs for ROASMI's retention-order model by pairing adjacent compounds whose RTs fall within a 30-second interval, labeling elution order, and encoding the pairs via a chemprop-based D-MPNN molecular embedding module.

## When to use

Use when building pairwise training data from a reference retention-time dataset (e.g., 'dataset_83') where ordered molecule pairs (Ci eluting before Cj) are needed to train a ranking model like RankNet.

## When NOT to use

- Input compounds lie outside the conventional elemental restrictions of C, H, N, O, P, and S, since ROASMI's chemical space is limited to those elements.
- The chromatographic system is HILIC rather than RPLC — ROASMI currently applies only to RPLC systems and HILIC models with generalizability remain future work.
- A retention-order (RankNet) training setup is not needed, e.g., when only predicting retention scores with the pre-trained ROASMI_1–ROASMI_5 ensemble.

## Inputs

- Reference retention-time records from 'dataset_83' (RPLC reference set, default retraining setup)
- Isomeric SMILES strings per compound retrieved from PubChem
- Retention time (RT) values defining adjacent-elution relationships

## Outputs

- Ordered molecule-pair batches of size 64 labeled with elution order, saved as 'molecule_pair_batches.pt'
- Continuous molecular embeddings per compound from the D-MPNN encoder, saved as 'molecule_embeddings.npy'
- Trained ROASMI retention-order model artifacts (input to RankNet)

## How to apply

Load the reference RT records (default 'dataset_83' per code/ROASMI_train.py) and retrieve each compound's isomeric SMILES from PubChem. Sort compounds by retention time and define a training pair (Ci, Cj) whenever molecule Ci elutes before Cj within a 30-s retention interval, yielding ordered pairs labeled with elution direction. Group the resulting pairs into training batches of size 64 (the default batch size in TrainArgs in args.py). Feed each batch of SMILES through the chemprop-based D-MPNN directed message passing encoder, which iteratively aggregates atom and bond features and sums them into a single molecular embedding per molecule. Save the pair batches and embeddings (e.g., 'molecule_pair_batches.pt' and 'molecule_embeddings.npy') for downstream RankNet training; validate that all saved batches are non-empty and that the D-MPNN module runs on the SMILES without error.

## Related tools

- **ROASMI** (Framework whose training pipeline (code/ROASMI_train.py) performs RT pairing, batching, and embedding generation) — https://github.com/FangYuan717/ROASMI
- **chemprop** (Provides the D-MPNN directed message passing encoder used as the molecular embedding module that converts molecular graphs into continuous vectors) — https://github.com/chemprop/chemprop
- **PyTorch** (Backend framework on which the D-MPNN embedding and training run (GPU-enabled version recommended))

## Examples

```
python code/ROASMI_train.py  # default reference set 'dataset_83'; outputs molecule_pair_batches.pt and molecule_embeddings.npy
```

## Evaluation signals

- All saved molecule-pair batches are non-empty before proceeding to RankNet training.
- The D-MPNN module runs on the batch SMILES strings without error, matching the paper's default retraining setup (script runs on code/ROASMI_train.py).
- Each generated pair (Ci, Cj) satisfies elution order Ci before Cj with |RT(Ci) − RT(Cj)| within the 30-s retention interval.
- Embedding output arrays ('molecule_embeddings.npy') contain continuous vectors of consistent dimensionality across all compounds.
- Batch size equals 64, matching the default in TrainArgs in args.py.

## Limitations

- Molecular pairs with very close chromatographic peaks may have retention orders inconsistent across experiments; such indistinguishable pairs (e.g., 4-hydroxybenzoic acid vs 2,5-dihydroxybenzoic acid) should be excluded from subsequent identification.
- RT outliers resulting from slight changes in experimental conditions are unresolved and require further mechanistic study.
- The 30-s interval is a dataset-specific pairing threshold; transferability to other chromatographic methods is not established.

## Evidence

- [methods] defines a training pair (Ci, Cj) as a molecule Ci eluting before Cj within a 30-s retention interval: [evidence span withheld — anchor status: not_found]
- [methods] Group the resulting pairs into training batches of batch size 64 (the default in TrainArgs in args.py).: [evidence span withheld — anchor status: not_found]
- [readme] A directed message transfer neural network (D-MPNN) is used to learn directly from the structure of compounds: "A directed message transfer neural network (D-MPNN) is used to learn directly from the structure of compounds"
- [discussion] Such pairs should be excluded from the subsequent small molecule identification process.: "Such pairs should be excluded from the subsequent small molecule identification process."
