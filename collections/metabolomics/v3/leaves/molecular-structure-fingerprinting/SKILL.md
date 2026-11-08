---
name: molecular-structure-fingerprinting
description: Use when you need to test whether per-compound spectral prediction accuracy
  depends on how structurally similar each test compound is to the training set, e.g.
  via maximum Bulk Tanimoto similarity on RDKit fingerprints.
when_to_use_negative:
- Input is already a feature/embedding table (e.g. 3DMolMS molecular embeddings from
  molnet_engine.save_features()) rather than raw structures needing fingerprint-based
  similarity
- The question is cross-instrument generalization after fine-tuning on a small set
  of spectra, not the dependence of accuracy on training-set chemical similarity
- Compounds contain atom types outside the supported set (C, O, N, H, P, S, F, Cl,
  B, Br, I) or exceed 300 atoms, so RDKit-fingerprintable structures cannot be assumed
  comparable
tools:
- name: RDKit
  role: Converts each molecule to a fingerprint comparable under Bulk Tanimoto similarity
- name: molnetpack
  role: Produces predicted MS/MS MGF and per-compound cosine similarities via MolNet.evaluate
    against the test pickle
  repo: https://github.com/JosieHong/3DMolMS
- name: Pandas
  role: Assembles and saves the per-compound table of Tanimoto and cosine similarity
- name: matplotlib
  role: Plots cosine similarity versus maximum training-set Tanimoto similarity and
    fits the correlation
- name: NumPy
  role: Numerical handling of fingerprint and similarity arrays
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad354
    title: '3DMolMS: prediction of tandem mass spectra from 3D molecular conformations'
  source_content_hash: sha256:4902759549f3ca3b5bde0426ecd71cbbe58cb7c624fd35d5b0d36dc82be77bf9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JosieHong/3DMolMS
  tools:
  - NumPy
  - Pandas
  - RDKit
  - matplotlib
  - molnetpack
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# molecular-structure-fingerprinting

## Summary

Computes structural (chemical) similarity between test and training compounds using RDKit fingerprints and Bulk Tanimoto scores, then correlates per-compound 3DMolMS MS/MS prediction accuracy (cosine similarity) with maximum training-set similarity (Supplementary Fig. S3 of the qtof evaluation).

## When to use

Use when you need to test whether per-compound spectral prediction accuracy depends on how structurally similar each test compound is to the training set, e.g. via maximum Bulk Tanimoto similarity on RDKit fingerprints.

## When NOT to use

- Input is already a feature/embedding table (e.g. 3DMolMS molecular embeddings from molnet_engine.save_features()) rather than raw structures needing fingerprint-based similarity
- The question is cross-instrument generalization after fine-tuning on a small set of spectra, not the dependence of accuracy on training-set chemical similarity
- Compounds contain atom types outside the supported set (C, O, N, H, P, S, F, Cl, B, Br, I) or exceed 300 atoms, so RDKit-fingerprintable structures cannot be assumed comparable

## Inputs

- Training pickle for the qtof instrument group (data/qtof_all_train.pkl from build_msms_dataset.py)
- Test pickle (data/qtof_all_test.pkl)
- Predicted spectra in MGF format (result/pred_qtof_test.mgf)
- Experimental reference MS/MS spectra for the test compounds

## Outputs

- Per-compound CSV table of maximum training-set Bulk Tanimoto similarity and prediction cosine similarity (pandas DataFrame saved as CSV)
- Scatter plot of cosine similarity versus maximum Tanimoto similarity with a fitted correlation (matplotlib, cf. Supplementary Fig. S3)

## How to apply

Load the training and test pickles (data/qtof_all_train.pkl, data/qtof_all_test.pkl) produced by build_msms_dataset.py for the qtof instrument group and convert every molecule to an RDKit fingerprint comparable under Bulk Tanimoto. For each test compound, compute the maximum Tanimoto similarity against all training-set compounds. Obtain per-compound cosine similarities by comparing predicted spectra (e.g. result/pred_qtof_test.mgf from molnet_engine.pred_msms) against the experimental reference spectra. Assemble a per-compound pandas table of maximum Tanimoto similarity vs. cosine similarity, save it as CSV, and plot cosine similarity versus maximum training-set Tanimoto similarity with matplotlib, fitting/reporting the correlation. The expected trend is that prediction cosine similarity rises with training-set chemical similarity, with average cosine values around 0.691 (positive) and 0.478 (negative) ion modes.

## Related tools

- **RDKit** (Converts each molecule to a fingerprint comparable under Bulk Tanimoto similarity)
- **molnetpack** (Produces predicted MS/MS MGF and per-compound cosine similarities via MolNet.evaluate against the test pickle) — https://github.com/JosieHong/3DMolMS
- **Pandas** (Assembles and saves the per-compound table of Tanimoto and cosine similarity)
- **matplotlib** (Plots cosine similarity versus maximum training-set Tanimoto similarity and fits the correlation)
- **NumPy** (Numerical handling of fingerprint and similarity arrays)

## Examples

```
results_df = molnet_engine.evaluate(test_pkl="./data/qtof_all_test.pkl", pred_mfg="./output_msms.mgf", result_path="./eval_results.csv")
```

## Evaluation signals

- All Tanimoto values fall in [0, 1], with 1.0 expected for test compounds also present in the training set
- Correlation between maximum training-set Tanimoto and prediction cosine similarity is positive, matching the reported trend in Supplementary Fig. S3
- Mean cosine similarities agree with the published averages: 0.691 (positive ion mode) and 0.478 (negative ion mode)
- Row count of the per-compound CSV equals the number of compounds in data/qtof_all_test.pkl (no missing predictions in result/pred_qtof_test.mgf)
- Predicted spectra used for cosine scoring contain no peaks above the precursor m/z (no isotope envelope), consistent with 3DMolMS output

## Limitations

- 3DMolMS loading supports only atom types C, O, N, H, P, S, F, Cl, B, Br, I, at most 300 atoms, and precursor types [M+H]+, [M-H]-, [M+H-H2O]+, [M+Na]+, [M+2H]2+; unsupported formats are excluded during loading
- Cosine similarities are computed against spectra acquired under the training conditions (METLIN-SMRT for the pretrained model); qtof/orbitrap checkpoints define the applicable instrument conditions
- The analysis shows a correlation with maximum Bulk Tanimoto similarity but the article provides no per-bin thresholds defining what counts as 'high' similarity

## Evidence

- [supplementary] prediction performance is higher when more structurally similar compounds are present in the training set (Supplementary Fig. S3, measured via maximum Bulk Tanimoto similarity): [evidence span withheld — anchor status: not_found]
- [readme] results_df = molnet_engine.evaluate(
    test_pkl="./data/qtof_all_test.pkl",: "results_df = molnet_engine.evaluate(
    test_pkl="./data/qtof_all_test.pkl","
- [abstract] 3DMolMS predicted the spectra with the average cosine similarity of 0.691 and 0.478 with the experimental MS/MS spectra acquired in positive and negative ion modes, respectively: "3DMolMS predicted the spectra with the average cosine similarity of 0.691 and 0.478 with the experimental MS/MS spectra acquired in positive and negative ion modes, respectively"
- [methods] The model is trained on METLIN-SMRT, so predictions are under the same experimental conditions as that dataset.: "The model is trained on METLIN-SMRT, so predictions are under the same experimental conditions as that dataset."
