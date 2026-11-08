---
name: retention-time-prediction-integration
description: Use when building multidimensional (4D) LC-IM-MS/MS identification databases
  where computationally predicted retention times must be supplied for candidate molecules
  lacking experimental RT measurements, e.g. derivatized sterol lipid entries.
when_to_use_negative:
- When experimental retention times are already available for the analytes — measured
  RT should be preferred over GNN predictions.
- When the analysis concerns non-sterol metabolites without C=C bonds, since the N-Me
  derivatization and fragmentation logic in the accompanying workflow was only tested
  on sterol lipids.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: retention_time_gnn
  role: Pretrained graph neural network (PyTorch + DGL) used to predict retention
    times via transfer learning from a small training dataset
  repo: https://github.com/seokhokang/retention_time_gnn/
- name: RDKit
  role: Builds molecular graphs/features of sterol candidates for the GNN RT predictor;
    also generates fragment ions in the wider 4D workflow
- name: QCCAssisted4DSterol
  role: Collection of Python/Jupyter scripts that assembles the 4D sterol database
    into which the RT predictions are integrated
  repo: https://github.com/Chen-micslab/QCCAssisted4DSterol
- name: Python
  role: Runtime environment for the RT prediction scripts (run_pretrain.py, run_transfer.py)
    and the database assembly scripts
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/anie.202507483
    title: Quantum Chemistry Calculation‐Assisted Large‐Scale Collision Cross Section
      Prediction Empowers Derivatization‐Enhanced Multidimensional Metabolomics
  source_content_hash: sha256:070edf29925d96a27ef79a709e7675c544b0594a9aeb7943e0d5ca786b308723
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/seokhokang/retention_time_gnn
  tools:
  - Jupyter Notebook
  - Python
  - QCCAssisted4DSterol
  - RDKit
  - retention_time_gnn
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# retention-time-prediction-integration

## Summary

Generates chromatographic retention time predictions for N-Me derivatized sterol lipids using a pretrained graph neural network (GNN) approach, and integrates the RT values as one of the four 4D fields (m/z, RT, CCS, fragment ions) in the derivatized sterol lipid database.

## When to use

Use when building multidimensional (4D) LC-IM-MS/MS identification databases where computationally predicted retention times must be supplied for candidate molecules lacking experimental RT measurements, e.g. derivatized sterol lipid entries.

## When NOT to use

- When experimental retention times are already available for the analytes — measured RT should be preferred over GNN predictions.
- When the analysis concerns non-sterol metabolites without C=C bonds, since the N-Me derivatization and fragmentation logic in the accompanying workflow was only tested on sterol lipids.

## Inputs

- Sterol lipid candidate structures (SMILES/SD files parsed with RDKit to molecular graphs)
- Source retention time training dataset (e.g. METLIN-SMRT small molecule dataset)
- Target RT datasets from PredRet or MoNA databases for transfer learning

## Outputs

- Per-molecule retention time predictions for derivatized sterol lipids
- sterol_4d_database.csv — combined table of m/z, RT, CCS, and fragment ions keyed by sterol lipid identity (n = 4891 entries)

## How to apply

Load the candidate sterol lipid structures and generate molecular graphs with RDKit, then use the pretrained GNN model from the seokhokang/retention_time_gnn repository (PyTorch + DGL implementation) to predict retention times via transfer learning from a small training set, as described in 'Retention Time Prediction through Learning from a Small Training Data Set with a Pretrained Graph Neural Network'. Apply run_pretrain.py on a source dataset (e.g. METLIN-SMRT) and run_transfer.py on the target dataset to adapt the pretrained model. Combine the per-molecule RT predictions with m/z, predicted CCS (scikit-learn SVR with LASSO feature selection), and RDKit-generated N-Me fragment ions into a single table keyed by sterol lipid identity. Validate that the assembled database contains n = 4891 derivatized sterol lipid entries and that every entry carries all four 4D fields.

## Related tools

- **retention_time_gnn** (Pretrained graph neural network (PyTorch + DGL) used to predict retention times via transfer learning from a small training dataset) — https://github.com/seokhokang/retention_time_gnn/
- **RDKit** (Builds molecular graphs/features of sterol candidates for the GNN RT predictor; also generates fragment ions in the wider 4D workflow)
- **QCCAssisted4DSterol** (Collection of Python/Jupyter scripts that assembles the 4D sterol database into which the RT predictions are integrated) — https://github.com/Chen-micslab/QCCAssisted4DSterol
- **Python** (Runtime environment for the RT prediction scripts (run_pretrain.py, run_transfer.py) and the database assembly scripts)

## Examples

```
python run_pretrain.py; python run_transfer.py -t FEM_long
```

## Evaluation signals

- The assembled database contains exactly n = 4891 derivatized sterol lipid entries.
- Every entry in sterol_4d_database.csv carries all four 4D fields: m/z, retention time, CCS, and fragment ions.
- RT predictions are finite, positive values within plausible LC gradient ranges and are keyed consistently to the same sterol lipid identities used for CCS and fragment ion fields.
- The GNN transfer-learning pipeline (run_pretrain.py followed by run_transfer.py -t <target>) completes without error on the chosen target dataset.

## Limitations

- The RT prediction approach relies on a pretrained GNN and transfer learning from a small training dataset; prediction quality depends on the similarity of the target analytes to the training compounds.
- The MS/MS/fragmentation logic of the broader workflow is 'theoretically applicable to all molecules including C=C bond (only test sterol lipids)', so results for non-sterol molecules are untested.
- RT prediction is one component of the 4D database; without complementary CCS (SVR) and fragment ion (RDKit N-Me pattern) predictions, identifications cannot be made at the isomer level.

## Evidence

- [readme] Pytorch implementation of the model described in the paper [Retention Time Prediction by Learning from Small Training Dataset with Pre-Trained Graph Neural Network]: [evidence span withheld — anchor status: not_found]
