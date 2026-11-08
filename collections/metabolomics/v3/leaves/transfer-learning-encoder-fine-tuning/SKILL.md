---
name: transfer-learning-encoder-fine-tuning
description: Use when a pretrained 3DMolMS encoder (e.g. molnet_pre_geobond.pt) should
  be transferred to a new property-prediction head (CCS, RT) instead of training from
  scratch, and you must decide between frozen-encoder and full fine-tune.
when_to_use_negative:
- You only need standard MS/MS, RT, or CCS predictions from existing checkpoints —
  use molnet_engine.pred_msms/pred_rt/pred_ccs directly instead of fine-tuning.
- The downstream dataset is too small for the head alone to converge — the frozen-encoder
  default may underfit, and full fine-tuning (freeze_encoder=False) is the explicit
  alternative.
- Your target spectra or properties fall outside the model's stated input scope (atom
  number ≤ 300, atom types C/O/N/H/P/S/F/Cl/B/Br/I, supported precursor types).
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: molnetpack
  role: Python API whose MolNet.train(task='ccs', resume_path=..., transfer=True,
    freeze_encoder=...) performs encoder-only weight transfer and head retraining
  repo: https://pypi.org/project/molnetpack/
- name: 3DMolMS
  role: Source of the pretrained molecular encoder and of the from-scratch baseline
    used to quantify the transfer benefit
  repo: https://github.com/JosieHong/3DMolMS
- name: PyTorch
  role: Deep-learning backend on which the frozen/unfrozen encoder fine-tuning runs
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad354
    title: '3DMolMS: prediction of tandem mass spectra from 3D molecular conformations'
  source_content_hash: sha256:54aeb2b988c45326308a8b9910e7f1e853d19d226edcb4e65691363ac696f6d0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - 3DMolMS
  - NumPy
  - Pandas
  - PyTorch
  - Python+
  - RDKit
  - molnetpack
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 14
---

# transfer-learning-encoder-fine-tuning

## Summary

Adapts the molecular encoder pretrained by 3DMolMS on MS/MS spectra prediction to a downstream chemical-property task (e.g. CCS or RT) by loading only the encoder.* weights and training a fresh task head, improving over training from scratch.

## When to use

Use when a pretrained 3DMolMS encoder (e.g. molnet_pre_geobond.pt) should be transferred to a new property-prediction head (CCS, RT) instead of training from scratch, and you must decide between frozen-encoder and full fine-tune.

## When NOT to use

- You only need standard MS/MS, RT, or CCS predictions from existing checkpoints — use molnet_engine.pred_msms/pred_rt/pred_ccs directly instead of fine-tuning.
- The downstream dataset is too small for the head alone to converge — the frozen-encoder default may underfit, and full fine-tuning (freeze_encoder=False) is the explicit alternative.
- Your target spectra or properties fall outside the model's stated input scope (atom number ≤ 300, atom types C/O/N/H/P/S/F/Cl/B/Br/I, supported precursor types).

## Inputs

- Pretrained 3DMolMS encoder checkpoint (./check_point/molnet_pre_geobond.pt)
- Task-specific training/validation pickles (e.g. CCS pickles built from the AllCCS dataset with encoding_etkdgv3.yml and the molnet_ccs_tl.yml six-adduct layout)
- Held-out external test set (e.g. non-overlapping Bush Lab BushCCS measurements)
- PyTorch CUDA-enabled environment

## Outputs

- Transfer-learned task checkpoint (e.g. ./check_point/molnet_ccs_tl.pt) with fresh task head on transferred encoder weights
- From-scratch baseline checkpoint (./check_point/molnet_ccs.pt) for comparison
- Evaluation metrics (R², MAE) for both variants on the external test set

## How to apply

Build the task-specific pickles first (for CCS: experimental AllCCS values only, ETKDGv3 conformers, covalent bond graph, six-adduct one-hot layout, 80/10/10 split on non-stereochemical InChKey skeletons to prevent structure leakage). Train the from-scratch baseline with MolNet.train(task='ccs') and save the scratch checkpoint. Then call train with both resume_path pointing to the pretrained encoder checkpoint (molnet_pre_geobond.pt) and transfer=True: this loads only the encoder.* weights while the head starts fresh, and the encoder is frozen by default; pass freeze_encoder=False for a full fine-tune when the downstream dataset is large enough to justify updating the encoder. Evaluate both variants on a held-out, non-overlapping external test set (e.g. Bush Lab BushCCS) with MolNet.evaluate, computing R² and MAE, and confirm the transfer-learning model matches the Table 4 reference values and beats the from-scratch baseline.

## Related tools

- **molnetpack** (Python API whose MolNet.train(task='ccs', resume_path=..., transfer=True, freeze_encoder=...) performs encoder-only weight transfer and head retraining) — https://pypi.org/project/molnetpack/
- **3DMolMS** (Source of the pretrained molecular encoder and of the from-scratch baseline used to quantify the transfer benefit) — https://github.com/JosieHong/3DMolMS
- **PyTorch** (Deep-learning backend on which the frozen/unfrozen encoder fine-tuning runs)

## Examples

```
molnet_engine.train(task='ccs', train_data='./data/ccs_train.pkl', valid_data='./data/ccs_val.pkl', checkpoint_path='./check_point/molnet_ccs_tl.pt', resume_path='./check_point/molnet_pre_geobond.pt', transfer=True)
```

## Evaluation signals

- The transfer-learned CCS model's R² and MAE on the non-overlapping BushCCS test set must match the Table 4 reference values and outperform the from-scratch baseline.
- Inspection of the loaded checkpoint must confirm only encoder.* weights were restored (the task head starts fresh under transfer=True).
- With the default frozen encoder, encoder weights must be identical before and after training; with freeze_encoder=False they must differ.
- The dataset split must show no overlap of stereochemical InChKey skeletons between train/val/test, verifying no structure leakage inflates metrics.
- Only experimental CCS measurements (not AllCCS predicted values) may remain in the training pickles.

## Limitations

- The encoder is frozen by default during transfer; forgetting freeze_encoder=False limits adaptation on sufficiently large downstream datasets.
- AllCCS-predicted CCS values must be discarded to avoid circular evaluation; only experimental measurements are used for training.
- v1.4.0 checkpoints embed the config they were trained under, and loading refuses any mismatch that would silently change predictions — checkpoint/config version mismatch will block transfer.
- The E(3) encoder cannot model enantiomer differences, so transferred representations cannot support chiral-separation tasks without SE(3) mode.

## Evidence

- [readme] Fine-tune from a pretrained encoder (frozen by default; pass freeze_encoder=False to train everything): "Fine-tune from a pretrained encoder (frozen by default;
# pass freeze_encoder=False to train everything)"
- [readme] The learned representation transfers to RT/CCS: "The molecular representation it learns transfers to related tasks such as retention time (RT) and collision cross section (CCS) prediction."
- [methods] Transfer-learning call from the article workflow: [evidence span withheld — anchor status: not_found]
- [abstract] Transferred representation improves chemical-property prediction: "the molecular representation learned by 3DMolMS from MS/MS spectra prediction can be adapted to enhance the prediction of chemical properties such as the elution time in the liquid chromatography and"
