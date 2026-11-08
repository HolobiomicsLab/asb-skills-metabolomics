---
name: baseline-spectrum-prediction-comparison
description: 'Use when you must show a spectrum-prediction model significantly outperforms
  baselines on the same held-out compounds: predictions and baselines run under identical
  adducts ([M+H]+, [M-H]-) and collision energies, compared by cosine similarity.'
when_to_use_negative:
- The task is single-model spectrum generation, not head-to-head benchmarking against
  baselines — use the prediction skill directly instead
- Spectra come from instruments/precursor types outside the Q-TOF benchmark scope
  or the supported precursor set ([M+H]+, [M-H]-, [M+H-H2O]+, [M+Na]+, [M+2H]2+),
  so baseline predictions are not comparable under identical conditions
- The train/test split was not made at the non-stereochemical InChIKey skeleton level,
  so structural leakage would invalidate the cosine-similarity comparison
tools:
- name: 3DMolMS
  role: Primary model under evaluation; predicts MS/MS spectra from 3D conformations
    using released QTOF checkpoints
  repo: https://github.com/JosieHong/3DMolMS
- name: molnetpack
  role: PyPI package providing the MolNet engine's pred_msms and evaluate API for
    generating predictions and computing cosine similarity per compound
  repo: https://github.com/JosieHong/3DMolMS
- name: CFM-ID 4.0
  role: Competitive rule-based/combinatorial baseline whose predictions must be generated
    under the same adducts and collision energies
- name: NEIMS
  role: Retrained fingerprint-based baseline for the benchmark comparison
- name: MassFormer
  role: Retrained transformer-based baseline for the benchmark comparison
- name: SchNet
  role: 3D-graph neural network baseline for the benchmark comparison
- name: PointNet
  role: 3D point-cloud baseline for the benchmark comparison
- name: DGCNN
  role: Dynamic graph CNN baseline for the benchmark comparison
- name: SciPy
  role: Runs the Mann-Whitney U test comparing 3DMolMS's cosine-similarity distribution
    against each baseline
- name: matplotlib
  role: Renders diagnostic similarity histograms and the model-comparison figure
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad354
    title: '3DMolMS: prediction of tandem mass spectra from 3D molecular conformations'
  source_content_hash: sha256:9e63f62c6f10b218edf92ab15739c0f5a21284bc0c1a0b4c6930b48fd7cb171b
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5294117647058824
n_claims: 17
---

# baseline-spectrum-prediction-comparison

## Summary

Benchmarks a deep-learning MS/MS spectrum predictor (3DMolMS) against classical and learned baselines (CFM-ID 4.0, NEIMS, MassFormer, SchNet, PointNet, DGCNN) by computing per-compound cosine similarity to experimental Q-TOF spectra on a leak-free PCDL/NIST20 split and testing significance with a Mann-Whitney U test.

## When to use

Use when you must show a spectrum-prediction model significantly outperforms baselines on the same held-out compounds: predictions and baselines run under identical adducts ([M+H]+, [M-H]-) and collision energies, compared by cosine similarity.

## When NOT to use

- The task is single-model spectrum generation, not head-to-head benchmarking against baselines — use the prediction skill directly instead
- Spectra come from instruments/precursor types outside the Q-TOF benchmark scope or the supported precursor set ([M+H]+, [M-H]-, [M+H-H2O]+, [M+Na]+, [M+2H]2+), so baseline predictions are not comparable under identical conditions
- The train/test split was not made at the non-stereochemical InChIKey skeleton level, so structural leakage would invalidate the cosine-similarity comparison

## Inputs

- Held-out test pickles from the combined Agilent PCDL + NIST20 Q-TOF libraries (MGF converted, split at non-stereochemical InChIKey skeleton level)
- Predicted MS/MS spectra from 3DMolMS checkpoints (instrument=qtof)
- Predicted spectra from each baseline: CFM-ID 4.0, NEIMS, MassFormer, SchNet, PointNet, DGCNN (same adducts and collision energies)
- Experimental MS/MS ground-truth spectra for the test compounds

## Outputs

- Per-compound cosine similarity CSV per model (eval_results.csv via --result_path)
- Diagnostic similarity histograms/plots (--plot_path, e.g. eval_similarity_hist.png)
- Aggregate table of mean ± std cosine similarity per model per ion mode ([M+H]+ vs [M-H]-)
- Mann-Whitney U test significance results comparing 3DMolMS to each baseline
- Model-comparison figure and final benchmark table

## How to apply

First construct the evaluation partition from the Agilent DPCL and NIST20 spectral libraries (converted to MGF with build_msms_dataset.py --group qtof), splitting at the non-stereochemical InChIKey skeleton level so no structure leaks between train/val/test. Predict spectra for the held-out test pickles with the released 3DMolMS checkpoints (scripts/predict.py --task msms --instrument qtof, or the molnetpack API) and run each baseline (CFM-ID 4.0, retrained NEIMS, retrained MassFormer, SchNet, PointNet, DGCNN) under the same adducts and collision energies. Compute cosine similarity between each model's predictions and the experimental ground truth per compound (scripts/eval.py with --result_path for the CSV and --plot_path for diagnostic histograms, or molnet_engine.evaluate). Aggregate mean ± standard deviation cosine similarity per model per ion mode — the reference results are 0.691 (±0.361) for [M+H]+ and 0.478 (±0.379) for [M-H]- for 3DMolMS — then run a Mann-Whitney U test (SciPy) of 3DMolMS's similarity distribution against each baseline to establish significance. Assemble the final model-comparison table/figure from the aggregated statistics and U-test results.

## Related tools

- **3DMolMS** (Primary model under evaluation; predicts MS/MS spectra from 3D conformations using released QTOF checkpoints) — https://github.com/JosieHong/3DMolMS
- **molnetpack** (PyPI package providing the MolNet engine's pred_msms and evaluate API for generating predictions and computing cosine similarity per compound) — https://github.com/JosieHong/3DMolMS
- **CFM-ID 4.0** (Competitive rule-based/combinatorial baseline whose predictions must be generated under the same adducts and collision energies)
- **NEIMS** (Retrained fingerprint-based baseline for the benchmark comparison)
- **MassFormer** (Retrained transformer-based baseline for the benchmark comparison)
- **SchNet** (3D-graph neural network baseline for the benchmark comparison)
- **PointNet** (3D point-cloud baseline for the benchmark comparison)
- **DGCNN** (Dynamic graph CNN baseline for the benchmark comparison)
- **SciPy** (Runs the Mann-Whitney U test comparing 3DMolMS's cosine-similarity distribution against each baseline)
- **matplotlib** (Renders diagnostic similarity histograms and the model-comparison figure)

## Examples

```
results_df = molnet_engine.evaluate(test_pkl="./data/qtof_all_test.pkl", pred_mgf="./output_msms.mgf", result_path="./eval_results.csv", plot_path="./eval_similarity_hist.png")
```

## Evaluation signals

- Per-compound cosine similarity CSV is produced for every model and ion mode (e.g. eval_results.csv), not just aggregate means
- 3DMolMS aggregate cosine similarity on the Q-TOF PCDL/NIST20 test split falls near the published 0.691 (±0.361) for [M+H]+ and 0.478 (±0.379) for [M-H]-
- Mann-Whitney U test shows 3DMolMS significantly outperforms all baselines in positive ion mode, and NEIMS and CFM-ID in negative ion mode — negative-mode wins over all six baselines would signal a discrepancy
- The split was made at the non-stereochemical InChIKey skeleton level, so no structure appears in both train and test
- Predicted spectra contain no peaks above the precursor m/z (no isotope envelope), matching the model's documented behavior

## Limitations

- In negative ion mode, significance was established only against NEIMS and CFM-ID — 3DMolMS did not significantly outperform all baselines there
- The E(3) encoder is provably unable to model enantiomer differences (one output for both), so chiral compounds cannot be distinguished in the comparison
- Only compounds with ≤ 300 atoms of types C, O, N, H, P, S, F, Cl, B, Br, I and precursor types [M+H]+, [M-H]-, [M+H-H2O]+, [M+Na]+, [M+2H]2+ are supported; unsupported formats are automatically excluded during loading
- In v1.4.0 the QTOF, Orbitrap, RT, and CCS checkpoints were retrained and 'loading refuses any mismatch that would silently change predictions', so all baselines must be compared against the matching checkpoint version
- Predicted spectra never contain peaks above the precursor m/z — in particular, no isotope envelope

## Evidence

- [abstract] 3DMolMS predicted the spectra with the average cosine similarity of 0.691 and 0.478 with the experimental MS/MS spectra acquired in positive and negative ion modes, respectively: "3DMolMS predicted the spectra with the average cosine similarity of 0.691 and 0.478 with the experimental MS/MS spectra acquired in positive and negative ion modes, respectively"
- [readme] results_df = molnet_engine.evaluate(test_pkl=./data/qtof_all_test.pkl, pred_mgf=./output_msms.mgf, result_path=./eval_results.csv, plot_path=./eval_similarity_hist.png): "results_df = molnet_engine.evaluate(
    test_pkl="./data/qtof_all_test.pkl",
    pred_mgf="./output_msms.mgf",
    result_path="./eval_results.csv",
    plot_path="./eval_similarity_hist.png""
- [methods] Predicted spectra never contain peaks above the precursor m/z — in particular, no isotope envelope.: "Predicted spectra never contain peaks above the precursor m/z —
in particular, no isotope envelope."
- [readme] Checkpoints now embed the config they were trained under, and loading refuses any mismatch that would silently change predictions.: [evidence span withheld — anchor status: external_doc]
