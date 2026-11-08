---
name: train-test-chemical-similarity-stratification
description: 'Use when per-compound prediction quality must be explained by chemical
  similarity: you have train/test pickle splits and predicted-vs-reference cosine
  scores and want maximum Bulk Tanimoto similarity to the training set as the explanatory
  variable.'
when_to_use_negative:
- You only need an aggregate accuracy number — the paper's headline metrics are the
  average cosine similarities (0.691 positive / 0.478 negative), not the similarity-performance
  relationship.
- 'No per-compound predictions exist yet: this skill correlates existing predictions
  against references and does not train the model or generate spectra (run molnet_engine.train
  / pred_msms first).'
- Evaluating spectra from a different lab or instrument without the minor fine-tuning
  step — cosine scores would conflate instrument shift with chemical-similarity effects.
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: RDKit
  role: Converts every train/test molecule to a Bulk Tanimoto-comparable fingerprint
    and computes the maximum Tanimoto similarity of each test compound against all
    training compounds.
- name: molnetpack
  role: MolNet engine loads the qtof test pickle, predicts MS/MS to MGF via pred_msms(instrument='qtof'),
    and reports per-compound cosine similarities through molnet_engine.evaluate.
  repo: https://github.com/JosieHong/3DMolMS
- name: Pandas
  role: Assembles the per-compound table of maximum Tanimoto similarity and cosine
    similarity and saves it as a CSV.
- name: matplotlib
  role: Plots cosine similarity versus maximum training-set Tanimoto similarity for
    the correlation figure (Supplementary Fig. S3 analog).
- name: NumPy
  role: Vectorized handling of fingerprint arrays and correlation computation over
    per-compound scores.
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad354
    title: '3DMolMS: prediction of tandem mass spectra from 3D molecular conformations'
  source_content_hash: sha256:7cf35247a915ffa3868b68713bd3dd5d2b72eb8af99b40813a1f9290c966bea2
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
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# train-test-chemical-similarity-stratification

## Summary

Tests whether 3DMolMS MS/MS prediction accuracy depends on training-set coverage by correlating per-compound cosine similarity (predicted vs experimental spectra) with each test compound's maximum Bulk Tanimoto similarity to the training set.

## When to use

Use when per-compound prediction quality must be explained by chemical similarity: you have train/test pickle splits and predicted-vs-reference cosine scores and want maximum Bulk Tanimoto similarity to the training set as the explanatory variable.

## When NOT to use

- You only need an aggregate accuracy number — the paper's headline metrics are the average cosine similarities (0.691 positive / 0.478 negative), not the similarity-performance relationship.
- No per-compound predictions exist yet: this skill correlates existing predictions against references and does not train the model or generate spectra (run molnet_engine.train / pred_msms first).
- Evaluating spectra from a different lab or instrument without the minor fine-tuning step — cosine scores would conflate instrument shift with chemical-similarity effects.

## Inputs

- Training-split pickle produced by build_msms_dataset.py (e.g., data/qtof_all_train.pkl)
- Test-split pickle (e.g., data/qtof_all_test.pkl)
- Predicted spectra in MGF format (e.g., result/pred_qtof_test.mgf)
- Experimental reference MS/MS spectra for the test compounds (ground truth)

## Outputs

- Per-compound CSV table with maximum Tanimoto similarity and cosine similarity columns
- Scatter plot of cosine similarity versus maximum training-set Tanimoto similarity (matplotlib) with fitted correlation
- Evaluation artifacts from molnet_engine.evaluate: eval_results.csv (one cosine similarity per test compound) and eval_similarity_hist.png

## How to apply

Load the split pickles produced by build_msms_dataset.py for the instrument group (e.g., data/qtof_all_train.pkl and data/qtof_all_test.pkl), convert every molecule to a Bulk Tanimoto-comparable RDKit fingerprint, and for each test compound compute the maximum Tanimoto similarity against all training-set compounds. Obtain per-compound cosine similarities between the predicted spectra (e.g., result/pred_qtof_test.mgf written by molnet_engine.pred_msms with instrument='qtof') and the experimental reference spectra, e.g. via molnet_engine.evaluate. Assemble the maximum-Tanimoto and cosine-similarity columns in pandas and save the per-compound table as a CSV. Scatter-plot cosine similarity versus maximum training-set Tanimoto similarity with matplotlib and fit/report the correlation. Judge success by a positive trend — cosine similarity rises with training-set chemical similarity (Supplementary Fig. S3) — and by cosine averages consistent with the reported 0.691 (positive) and 0.478 (negative) ion modes.

## Related tools

- **RDKit** (Converts every train/test molecule to a Bulk Tanimoto-comparable fingerprint and computes the maximum Tanimoto similarity of each test compound against all training compounds.)
- **molnetpack** (MolNet engine loads the qtof test pickle, predicts MS/MS to MGF via pred_msms(instrument='qtof'), and reports per-compound cosine similarities through molnet_engine.evaluate.) — https://github.com/JosieHong/3DMolMS
- **Pandas** (Assembles the per-compound table of maximum Tanimoto similarity and cosine similarity and saves it as a CSV.)
- **matplotlib** (Plots cosine similarity versus maximum training-set Tanimoto similarity for the correlation figure (Supplementary Fig. S3 analog).)
- **NumPy** (Vectorized handling of fingerprint arrays and correlation computation over per-compound scores.)

## Examples

```
molnet_engine.evaluate(test_pkl="./data/qtof_all_test.pkl", pred_mgf="./output_msms.mgf", result_path="./eval_results.csv", plot_path="./eval_similarity_hist.png")
```

## Evaluation signals

- Mean/median per-compound cosine similarity should be consistent with the reported averages: 0.691 (positive) and 0.478 (negative) ion modes on QTOF data.
- Scatter of cosine similarity vs maximum training-set Tanimoto similarity should show a positive trend; confirm the fitted correlation has the expected sign.
- Every test compound in qtof_all_test.pkl must yield a finite maximum Tanimoto in [0, 1] and exactly one cosine score in the merged table — no NaNs, row count matches the filtered test set.
- molnet_engine.evaluate must emit one row per test compound in eval_results.csv plus eval_similarity_hist.png; check both files exist and the schema is per-compound.
- Cosine values must lie in [0, 1]; verify predicted spectra contain no peaks above the precursor m/z (no isotope envelope) before scoring.

## Limitations

- Applicability restricted to supported inputs: ≤ 300 atoms, atom types C, O, N, H, P, S, F, Cl, B, Br, I, and precursor types [M+H]+, [M-H]-, [M+H-H2O]+, [M+Na]+, [M+2H]2+; unsupported formats are automatically excluded during loading, so some test compounds get no cosine score.
- The model is trained on METLIN-SMRT, so the similarity-performance relationship is measured under those experimental conditions; spectra from other labs/instruments need minor fine-tuning on a small set before the trend is meaningful.
- Results are split- and instrument-group specific (e.g., qtof_all_test.pkl); do not extrapolate the correlation to other instrument groups or splits without recomputing maximum Tanimoto and cosine values.

## Evidence

- [supplementary] prediction performance is higher when more structurally similar compounds are present in the training set (Supplementary Fig. S3, measured via maximum Bulk Tanimoto similarity): [evidence span withheld — anchor status: not_found]
- [abstract] 3DMolMS predicted the spectra with the average cosine similarity of 0.691 and 0.478 with the experimental MS/MS spectra acquired in positive and negative ion modes, respectively: "3DMolMS predicted the spectra with the average cosine similarity of 0.691 and 0.478 with the experimental MS/MS spectra acquired in positive and negative ion modes, respectively"
- [methods] For each test compound, compute the maximum Tanimoto similarity against all training-set compounds.: [evidence span withheld — anchor status: not_found]
- [methods] The model is trained on METLIN-SMRT, so predictions are under the same experimental conditions as that dataset.: "The model is trained on METLIN-SMRT, so predictions are under the same experimental conditions as that dataset."
- [readme] Unsupported formats are automatically excluded during loading.: [evidence span withheld — anchor status: external_doc]
