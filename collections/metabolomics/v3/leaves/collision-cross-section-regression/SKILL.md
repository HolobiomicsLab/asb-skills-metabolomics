---
name: collision-cross-section-regression
description: Use when experimental CCS values for small molecules must be predicted
  or modeled from structures (e.g. AllCCS/BushCCS data), especially to test whether
  an MS/MS-pretrained encoder improves CCS regression over training from scratch.
when_to_use_negative:
- The target is MS/MS spectra, retention time, or chirality — use the msms/rt/chiral
  tasks of MolNet instead of the CCS regression head
- The CCS values to learn are AllCCS's predicted (not experimental) measurements —
  these are discarded during dataset construction
- Molecules exceed 300 atoms or contain atom types outside C, O, N, H, P, S, F, Cl,
  B, Br, I
edam_operation: http://edamontology.org/operation_2938
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_3520
tools:
- name: molnetpack
  role: PyPI package providing MolNet.train(task='ccs'), pred_ccs, and evaluate for
    CCS regression and transfer learning
  repo: https://github.com/JosieHong/3DMolMS
- name: 3DMolMS
  role: Deep neural network whose learned molecular representation from MS/MS prediction
    is transferred to the CCS regression head
  repo: https://github.com/JosieHong/3DMolMS
- name: PyTorch
  role: Deep-learning backend for training and fine-tuning the CCS model
- name: RDKit
  role: Generates ETKDGv3 3D conformers used as encoder input
- name: AllCCS
  role: Source of experimental CCS training measurements (downloaded from http://allccs.zhulab.cn/)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad354
    title: '3DMolMS: prediction of tandem mass spectra from 3D molecular conformations'
  source_content_hash: sha256:0580cb30b752aa6ac1e63c13e7c4b29163e97abcf7fece4eabd99d69ae19f9fb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JosieHong/3DMolMS
  tools:
  - 3DMolMS
  - AllCCS
  - NumPy
  - Pandas
  - PyTorch
  - Python+
  - RDKit
  - molnetpack
  edam_operation: http://edamontology.org/operation_2938
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5625
n_claims: 16
---

# collision-cross-section-regression

## Summary

Predict collision cross section (CCS) values from 3D molecular conformations using the 3DMolMS/MolNet encoder, either from scratch or via transfer learning from an MS/MS-pretrained encoder, and evaluate with R² and MAE against experimental CCS benchmarks (AllCCS train, BushCCS test).

## When to use

Use when experimental CCS values for small molecules must be predicted or modeled from structures (e.g. AllCCS/BushCCS data), especially to test whether an MS/MS-pretrained encoder improves CCS regression over training from scratch.

## When NOT to use

- The target is MS/MS spectra, retention time, or chirality — use the msms/rt/chiral tasks of MolNet instead of the CCS regression head
- The CCS values to learn are AllCCS's predicted (not experimental) measurements — these are discarded during dataset construction
- Molecules exceed 300 atoms or contain atom types outside C, O, N, H, P, S, F, Cl, B, Br, I

## Inputs

- AllCCS dataset download (allccs_download.csv) with experimental CCS measurements
- ETKDGv3-generated 3D molecular conformations featurised per encoding_etkdgv3.yml
- Covalent bond graph attached per molecule
- CCS adduct one-hot layout from the `encoding:` section of molnet_ccs_tl.yml
- Pretrained MS/MS encoder checkpoint (molnet_pre_geobond.pt) for transfer learning

## Outputs

- Trained CCS regression checkpoints (e.g. ./check_point/molnet_ccs.pt and molnet_ccs_tl.pt)
- Per-compound CCS predictions (e.g. output_ccs.csv from pred_ccs)
- R² and MAE metrics on the held-out BushCCS test set for from-scratch vs 3DMolMS-TL variants

## How to apply

Download the AllCCS dataset from http://allccs.zhulab.cn/ and build the task dataset with `python scripts/build_rt_ccs_dataset.py --task ccs --workers 8`, keeping only experimental CCS measurements (discard AllCCS's predicted values), featurising ETKDGv3 conformers with the shared encoding_etkdgv3.yml config, attaching the covalent bond graph, and applying the CCS-specific six-adduct one-hot layout from the `encoding:` section of molnet_ccs_tl.yml. Split 80/10/10 on non-stereochemical InChKey skeletons to prevent structure leakage. Train the from-scratch baseline with `python scripts/train_rt_ccs.py --task ccs --gpu 0`, then fine-tune with transfer learning by passing resume_path='./check_point/molnet_pre_geobond.pt' and transfer=True, which loads only the encoder.* weights (head starts fresh; encoder frozen by default, pass freeze_encoder=False for a full fine-tune). Evaluate both variants on the held-out Bush Lab (BushCCS) test set with MolNet.evaluate, computing R² and MAE; the transfer-learning model (3DMolMS-TL) should outperform the from-scratch model, with the CCS improvement reported as very significant. Note the model supports six adducts ([M+H]+, [M-H]-, [M+H-H2O]+, [M+Na]+, [M+2H]2+ and related layouts) and molecules ≤ 300 atoms with atom types C, O, N, H, P, S, F, Cl, B, Br, I.

## Related tools

- **molnetpack** (PyPI package providing MolNet.train(task='ccs'), pred_ccs, and evaluate for CCS regression and transfer learning) — https://github.com/JosieHong/3DMolMS
- **3DMolMS** (Deep neural network whose learned molecular representation from MS/MS prediction is transferred to the CCS regression head) — https://github.com/JosieHong/3DMolMS
- **PyTorch** (Deep-learning backend for training and fine-tuning the CCS model)
- **RDKit** (Generates ETKDGv3 3D conformers used as encoder input)
- **AllCCS** (Source of experimental CCS training measurements (downloaded from http://allccs.zhulab.cn/))

## Examples

```
python scripts/train_rt_ccs.py --task ccs --gpu 0
```

## Evaluation signals

- Computed R² and MAE for the transfer-learning model must match the Table 4 CCS reference values
- 3DMolMS-TL must outperform the from-scratch 3DMolMS baseline on the held-out BushCCS test set
- Dataset split (80/10/10) must be performed on non-stereochemical InChKey skeletons so no structure leakage between train/val/test exists
- Only experimental CCS measurements should remain in the training pickles — AllCCS predicted values must be filtered out
- Transfer-learning checkpoint must contain encoder.* weights from molnet_pre_geobond.pt with a freshly initialized head

## Limitations

- The E(3) encoder cannot model enantiomer differences; SE(3) chirality mode is required if stereochemistry-sensitive CCS is needed
- By default the encoder is frozen during transfer; full fine-tuning requires explicitly passing freeze_encoder=False
- Predictions are restricted to molecules ≤ 300 atoms and the supported atom-type/adduct set

## Evidence

- [methods] The learned molecular representation transfers to retention time (RT) and collision cross section (CCS) prediction: "Its learned molecular representation also transfers to related tasks such as retention time (RT) and collision cross section (CCS) prediction."
- [readme] Transfer fine-tuning loads only encoder weights: "# Fine-tune from a pretrained encoder (frozen by default;
# pass freeze_encoder=False to train everything)"
- [abstract] The molecular representation learned by 3DMolMS can be adapted to enhance prediction of chemical properties such as elution time and collisional cross section: "the molecular representation learned by 3DMolMS from MS/MS spectra prediction can be adapted to enhance the prediction of chemical properties such as the elution time in the liquid chromatography and"
- [readme] 3DMolMS predicts CCS directly: [evidence span withheld — anchor status: external_doc]
- [other] Transfer-learning CCS task via MolNet.train: [evidence span withheld — anchor status: not_found]
