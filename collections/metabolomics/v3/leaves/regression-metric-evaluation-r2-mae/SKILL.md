---
name: regression-metric-evaluation-r2-mae
description: Use when comparing regression model variants (e.g. 3DMolMS-TL vs a from-scratch
  model vs ALLCCS/DeepCCS baselines) on held-out experimental CCS measurements, where
  performance must be quantified with R² and MAE rather than spectral cosine similarity.
when_to_use_negative:
- Evaluating MS/MS spectrum prediction quality, which is judged by average cosine
  similarity (0.691 positive / 0.478 negative ion mode), not R²/MAE
- The test data has no ground-truth experimental CCS/RT labels, so R² and MAE cannot
  be computed
- The goal is classification or spectral-peak filtering (filter_spec, filter_mol,
  check_atom) rather than regression performance comparison
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: molnetpack
  role: Provides the MolNet.evaluate API that computes R² and MAE on held-out test
    pickles and writes metric CSVs and plots; installed via pip install molnetpack.
  repo: https://github.com/JosieHong/3DMolMS
- name: PyTorch
  role: Backend framework for running the trained 3DMolMS models (from-scratch and
    transfer-learned) during evaluation on GPU/CPU device.
  repo: https://github.com/JosieHong/3DMolMS
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad354
    title: '3DMolMS: prediction of tandem mass spectra from 3D molecular conformations'
  source_content_hash: sha256:5a050988c3ec84c312dd669ba9b4bb86004fd167861beb746f088acfe4215ee2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JosieHong/3DMolMS
  tools:
  - 3DMolMS
  - NumPy
  - Pandas
  - PyTorch
  - Python+
  - RDKit
  - molnetpack
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# regression-metric-evaluation-r2-mae

## Summary

Quantify continuous-property regression performance (e.g. CCS prediction from transferred 3DMolMS molecular representations) by computing R² and MAE with the MolNet.evaluate API on held-out test sets, and comparing transfer-learned vs from-scratch model variants.

## When to use

Use when comparing regression model variants (e.g. 3DMolMS-TL vs a from-scratch model vs ALLCCS/DeepCCS baselines) on held-out experimental CCS measurements, where performance must be quantified with R² and MAE rather than spectral cosine similarity.

## When NOT to use

- Evaluating MS/MS spectrum prediction quality, which is judged by average cosine similarity (0.691 positive / 0.478 negative ion mode), not R²/MAE
- The test data has no ground-truth experimental CCS/RT labels, so R² and MAE cannot be computed
- The goal is classification or spectral-peak filtering (filter_spec, filter_mol, check_atom) rather than regression performance comparison

## Inputs

- Held-out test pickle (e.g. ./data/qtof_all_test.pkl or the BushCCS non-overlapping test set)
- Trained from-scratch model checkpoint (./check_point/molnet_ccs.pt)
- Transfer-learned model checkpoint (./check_point/molnet_ccs_tl.pt)
- Predicted outputs (e.g. ./output_msms.mgf or predicted CCS values)

## Outputs

- R² and MAE metrics per model variant
- Evaluation results CSV (result_path, e.g. ./eval_results.csv)
- Diagnostic plot of score distribution (plot_path, e.g. ./eval_similarity_hist.png)

## How to apply

After training both the from-scratch baseline (saved to ./check_point/molnet_ccs.pt) and the transfer-learned model (resume_path=./check_point/molnet_pre_geobond.pt, transfer=True), evaluate each on a held-out, non-overlapping test set such as the Bush Lab (BushCCS) data that does not overlap the 80/10/10 split on non-stereochemical InChKey skeletons used for training. Call MolNet.evaluate on the test pickle, computing R² and MAE per model, and write the metrics table plus a diagnostic plot. The key decision point is a paired comparison: the transfer-learning variant must show higher R² and lower MAE than the from-scratch model, with values matching the paper's Table 4 reference numbers for the CCS task. Judgment of correctness combines metric agreement with the published values and the invariant that no test InChKey skeleton leaks into training.

## Related tools

- **molnetpack** (Provides the MolNet.evaluate API that computes R² and MAE on held-out test pickles and writes metric CSVs and plots; installed via pip install molnetpack.) — https://github.com/JosieHong/3DMolMS
- **PyTorch** (Backend framework for running the trained 3DMolMS models (from-scratch and transfer-learned) during evaluation on GPU/CPU device.) — https://github.com/JosieHong/3DMolMS

## Examples

```
results_df = molnet_engine.evaluate(test_pkl="./data/qtof_all_test.pkl", pred_mgf="./output_msms.mgf", result_path="./eval_results.csv", plot_path="./eval_similarity_hist.png")
```

## Evaluation signals

- The computed R² and MAE for the transfer-learning model must match the Table 4 CCS reference values from the paper
- The 3DMolMS-TL variant must outperform the from-scratch baseline on both R² (higher) and MAE (lower)
- Evaluation must be on a held-out, non-overlapping Bush Lab (BushCCS) test set — no InChKey skeleton overlap with the 80/10/10 training split
- Only experimental CCS measurements were used as targets (AllCCS predicted values discarded), so metrics reflect experimental ground truth
- The evaluation function writes a results CSV whose rows cover every test compound, confirming no silent exclusion during loading

## Limitations

- AllCCS-predicted CCS values must be discarded during dataset building; evaluating against predicted (not experimental) CCS would inflate apparent performance
- The E(3) encoder cannot model enantiomer differences, so metrics conflate stereoisomers unless the SE(3) chirality mode is used
- Checkpoints embed their training config and loading refuses mismatches, so metrics are only valid when the checkpoint and evaluation config match

## Evidence

- [methods] The learned molecular representation transfers to retention time (RT) and collision cross section (CCS) prediction: "Its learned molecular representation also transfers to related tasks such as retention time (RT) and collision cross section (CCS) prediction."
- [other] 3DMolMS-TL improves over scratch, CCS most significant: [evidence span withheld — anchor status: not_found]
- [other] Validation: metrics must match Table 4 and outperform scratch: [evidence span withheld — anchor status: not_found]
- [readme] README evaluate call against ground truth: "results_df = molnet_engine.evaluate(
    test_pkl="./data/qtof_all_test.pkl",
    pred_mgf="./output_msms.mgf",
    result_path="./eval_results.csv",
    plot_path="./eval_similarity_hist.png",
)"
- [other] Split on non-stereochemical InChKey skeletons to prevent leakage: [evidence span withheld — anchor status: not_found]
