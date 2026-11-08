---
name: spectral-library-split-evaluation
description: Use when benchmarking MS/MS prediction models on held-out compounds from
  Q-TOF spectral libraries (Agilent PCDL, NIST20) and reporting cosine similarity
  per ion mode with significance tests against baselines.
when_to_use_negative:
- Input spectra are acquired on instruments other than Q-TOF without fine-tuning —
  the released checkpoints predict under METLIN-SMRT/QTOF or Orbitrap conditions,
  and predictions never contain peaks above the precursor m/z
- The task is single-model inspection rather than comparison against baselines — use
  direct prediction (pred_msms) and spectral plotting instead of the split benchmark
  protocol
- Compounds fall outside the supported input scope (atom number > 300, unsupported
  atom types, or adducts outside [M+H]+, [M-H]-, [M+H-H2O]+, [M+Na]+, [M+2H]2+)
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3474
tools:
- name: 3DMolMS
  role: Primary prediction model evaluated; provides released QTOF checkpoints and
    the evaluate() routine for cosine-similarity scoring against ground truth
  repo: https://github.com/JosieHong/3DMolMS
- name: molnetpack
  role: PyPI package wrapping 3DMolMS; MolNet engine loads test pickles, predicts
    MS/MS, and evaluates similarity with result and plot paths
  repo: https://pypi.org/project/molnetpack/
- name: SciPy
  role: Mann-Whitney U test comparing 3DMolMS's cosine-similarity distribution against
    each baseline
- name: CFM-ID 4.0
  role: Baseline MS/MS prediction method retrained and evaluated under the same adducts
    and collision energies
- name: NEIMS
  role: Baseline MS/MS prediction method retrained for the benchmark comparison
- name: MassFormer
  role: Baseline MS/MS prediction method retrained for the benchmark comparison
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad354
    title: '3DMolMS: prediction of tandem mass spectra from 3D molecular conformations'
  source_content_hash: sha256:66eea6ff6d9c9e7ff77d3e61e7dbb4a4f8835164306d9b5389c374a2d8d8ec27
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JosieHong/3DMolMS
  tools:
  - 3DMolMS
  - CFM-ID 4.0
  - DGCNN
  - MassFormer
  - NEIMS
  - NumPy
  - Pandas
  - PointNet
  - PyTorch
  - RDKit
  - SchNet
  - SciPy
  - matplotlib
  - molnetpack
  edam_operation: http://edamontology.org/operation_3216
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6111111111111112
n_claims: 18
---

# spectral-library-split-evaluation

## Summary

Benchmark protocol for evaluating MS/MS spectral prediction models (e.g. 3DMolMS) against experimental spectra from compound-level splits of spectral libraries such as Agilent PCDL and NIST20 Q-TOF, using cosine similarity and statistical significance testing.

## When to use

Use when benchmarking MS/MS prediction models on held-out compounds from Q-TOF spectral libraries (Agilent PCDL, NIST20) and reporting cosine similarity per ion mode with significance tests against baselines.

## When NOT to use

- Input spectra are acquired on instruments other than Q-TOF without fine-tuning — the released checkpoints predict under METLIN-SMRT/QTOF or Orbitrap conditions, and predictions never contain peaks above the precursor m/z
- The task is single-model inspection rather than comparison against baselines — use direct prediction (pred_msms) and spectral plotting instead of the split benchmark protocol
- Compounds fall outside the supported input scope (atom number > 300, unsupported atom types, or adducts outside [M+H]+, [M-H]-, [M+H-H2O]+, [M+Na]+, [M+2H]2+)

## Inputs

- MGF-format spectral libraries (Agilent DPCL, NIST20/NIST23 Q-TOF) with adduct and collision-energy metadata
- Test-split pickle files (e.g. qtof_all_test.pkl) with molecular records and 3D conformations
- Predicted spectra in MGF format from 3DMolMS and baseline models (CFM-ID 4.0, NEIMS, MassFormer, SchNet, PointNet, DGCNN)
- Experimental MS/MS ground-truth spectra for held-out compounds

## Outputs

- Per-compound cosine similarity CSV (eval_results.csv) for each model and ion mode
- Similarity distribution histogram plots (eval_similarity_hist.png)
- Aggregated mean ± standard deviation cosine similarity per model per ion mode ([M+H]+ vs [M-H]-)
- Mann-Whitney U test significance results comparing 3DMolMS against each baseline
- Final model-comparison benchmark table and figure

## How to apply

Build the evaluation partition from the Agilent DPCL and NIST20 spectral libraries converted to MGF (build_msms_dataset.py --group qtof), splitting at the non-stereochemical InChIKey skeleton level so no structure leaks between train/val/test. Predict MS/MS spectra for the held-out test pickles with the released 3DMolMS checkpoints (scripts/predict.py --task msms --instrument qtof) and generate baseline predictions (CFM-ID 4.0, retrained NEIMS, MassFormer, SchNet, PointNet, DGCNN) under the same adducts ([M+H]+, [M-H]-) and collision energies. Compute cosine similarity between each model's predictions and the experimental ground truth with scripts/eval.py, writing per-compound results via --result_path and diagnostic plots via --plot_path; the molnetpack MolNet engine exposes this as evaluate(test_pkl=..., pred_mgf=..., result_path=..., plot_path=...). Aggregate mean and standard deviation of cosine similarity per model and per ion mode, then run a Mann-Whitney U test comparing 3DMolMS's similarity distribution against each baseline to establish significance. Judge success by reproducing the reported performance range: average cosine similarity of ~0.691 in positive ion mode and ~0.478 in negative ion mode for 3DMolMS.

## Related tools

- **3DMolMS** (Primary prediction model evaluated; provides released QTOF checkpoints and the evaluate() routine for cosine-similarity scoring against ground truth) — https://github.com/JosieHong/3DMolMS
- **molnetpack** (PyPI package wrapping 3DMolMS; MolNet engine loads test pickles, predicts MS/MS, and evaluates similarity with result and plot paths) — https://pypi.org/project/molnetpack/
- **SciPy** (Mann-Whitney U test comparing 3DMolMS's cosine-similarity distribution against each baseline)
- **CFM-ID 4.0** (Baseline MS/MS prediction method retrained and evaluated under the same adducts and collision energies)
- **NEIMS** (Baseline MS/MS prediction method retrained for the benchmark comparison)
- **MassFormer** (Baseline MS/MS prediction method retrained for the benchmark comparison)

## Examples

```
molnet_engine.evaluate(test_pkl="./data/qtof_all_test.pkl", pred_mgf="./output_msms.mgf", result_path="./eval_results.csv", plot_path="./eval_similarity_hist.png")
```

## Evaluation signals

- Mean cosine similarity for 3DMolMS in [M+H]+ mode lands near 0.691 (±0.361) and in [M-H]- mode near 0.478 (±0.379) against experimental spectra
- Whitney U-test confirms 3DMolMS significantly outperforms all baselines in positive ion mode and NEIMS and CFM-ID in negative ion mode
- Split integrity check: no shared InChIKey skeletons between train/val/test partitions
- evaluate() writes a per-compound results CSV (result_path) and a similarity histogram (plot_path) without errors
- Predicted spectra contain no peaks above the precursor m/z (no isotope envelope)

## Limitations

- Predicted spectra never contain peaks above the precursor m/z — in particular, no isotope envelope
- The E(3) encoder is provably unable to model enantiomer differences; SE(3) chirality mode is required for chiral separation
- The model is trained on METLIN-SMRT, so predictions are under the same experimental conditions as that dataset unless fine-tuned on a small set of spectra from the target instrument
- Baseline results depend on retraining NEIMS and MassFormer under identical adducts and collision energies; significance in negative ion mode was not established against all baselines (SchNet, PointNet, DGCNN)

## Evidence

- [abstract] 3DMolMS predicted the spectra with the average cosine similarity of 0.691 and 0.478 with the experimental MS/MS spectra acquired in positive and negative ion modes, respectively: "3DMolMS predicted the spectra with the average cosine similarity of 0.691 and 0.478 with the experimental MS/MS spectra acquired in positive and negative ion modes, respectively"
- [readme] results_df = molnet_engine.evaluate(test_pkl="./data/qtof_all_test.pkl", pred_mgf="./output_msms.mgf", result_path="./eval_results.csv", plot_path="./eval_similarity_hist.png"): "results_df = molnet_engine.evaluate(
    test_pkl="./data/qtof_all_test.pkl",
    pred_mgf="./output_msms.mgf",
    result_path="./eval_results.csv",
    plot_path="./eval_similarity_hist.png",
)"
- [methods] 3DMolMS can generate a reference library of small-molecule MS/MS spectra for identification through MS/MS searching.: "3DMolMS can generate a reference library of small-molecule MS/MS spectra for identification through MS/MS searching."
- [readme] Supported inputs: Atom number ≤ 300; Atom types C, O, N, H, P, S, F, Cl, B, Br, I; Precursor types [M+H]+, [M-H]-, [M+H-H2O]+, [M+Na]+, [M+2H]2+: "| Atom number      | ≤ 300                                              |
| Atom types       | C, O, N, H, P, S, F, Cl, B, Br, I                  |
| Precursor types  | [M+H]+, [M-H]-, [M+H-H2O]+,"
