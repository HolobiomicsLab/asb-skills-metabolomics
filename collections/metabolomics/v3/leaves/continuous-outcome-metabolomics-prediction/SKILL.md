---
name: continuous-outcome-metabolomics-prediction
description: Use when predicting a continuous phenotypic outcome (e.g. clinically
  measured bilirubin in serum) from multi-assay LC-MS metabolomics feature tables
  and you need cross-assay feature importance and significance calls.
when_to_use_negative:
- The outcome is a binary/categorical label (e.g. biological sex or disease status)
  — use MamsiPls classification mode with metric='auc' and .evaluate_class_model()
  instead.
- Data come from a single assay block only — the MB-PLS framework's rationale is modelling
  interblock relationships across multiple LC-MS assays.
- Feature tables have not been aligned across assay blocks or lack the assay-prefixed
  column names needed for cross-assay structural search.
edam_operation: http://edamontology.org/operation_2930
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: MAMSI
  role: Python framework providing MamsiPls (MB-PLS regression, LV estimation, regression
    evaluation, MB-VIP and MB-VIP permutation testing) for integrating multi-assay
    LC-MS metabolomics data against a continuous outcome.
  repo: https://github.com/kopeckylukas/py-mamsi
- name: mamsi
  role: PyPI package to install MAMSI; supplies MamsiPls class used to fit, cross-validate
    and interpret the regression model.
  repo: https://kopeckylukas.github.io/py-mamsi/
- name: mbpls 1.0.4
  role: Pinned dependency providing the underlying MB-PLS algorithm (NIPALS, standardise)
    used by MamsiPls.
- name: pandas
  role: Loads assay CSVs and metadata; pd.concat joins assay blocks and pd.read_csv
    reads the bilirubin outcome.
- name: numpy
  role: Masking significant features via np.where(p_vals < 0.01).
- name: scipy
  role: Dependency for statistical operations underlying permutation testing and CV.
- name: scikit-learn
  role: Provides train_test_split for the training/testing split and k-fold cross-validation
    machinery.
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.5c01327
    title: 'MAMSI: Integration of Multiassay Liquid Chromatography–Mass Spectrometry
      Metabolomics Data Using Multiview Machine Learning'
  source_content_hash: sha256:d4a5addb486e61b94786d4d468bcb439aaca897cca6ed15649629a468327c9b5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kopeckylukas/py-mamsi
  tools:
  - MAMSI
  - Python
  - mamsi
  - matplotlib
  - mbpls
  - mbpls 1.0.4
  - numpy
  - pandas
  - scikit-learn
  - scipy
  edam_operation: http://edamontology.org/operation_2930
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6875
n_claims: 16
---

# continuous-outcome-metabolomics-prediction

## Summary

Fit an MB-PLS regression model (MamsiPls, NIPALS, standardised) on multi-assay LC-MS metabolomics data to predict a continuous clinical outcome such as blood bilirubin, using cross-validated latent-variable estimation, MB-VIP feature importance, and permutation-based empirical p-values to identify significant predictors.

## When to use

Use when predicting a continuous phenotypic outcome (e.g. clinically measured bilirubin in serum) from multi-assay LC-MS metabolomics feature tables and you need cross-assay feature importance and significance calls.

## When NOT to use

- The outcome is a binary/categorical label (e.g. biological sex or disease status) — use MamsiPls classification mode with metric='auc' and .evaluate_class_model() instead.
- Data come from a single assay block only — the MB-PLS framework's rationale is modelling interblock relationships across multiple LC-MS assays.
- Feature tables have not been aligned across assay blocks or lack the assay-prefixed column names needed for cross-assay structural search.

## Inputs

- Multi-assay LC-MS feature intensity tables (CSV) with samples as rows and m/z/RT features as columns, one table per assay block (e.g. RPOS, RNEG, LPOS, LNEG), with assay-prefixed column names
- Metadata CSV containing sample identifiers and the continuous outcome variable (e.g. clinically measured blood bilirubin)

## Outputs

- Fitted MB-PLS regression model object (MamsiPls) with estimated number of latent variables (metric='q2')
- Regression performance metrics from .evaluate_regression_model() on the independent test set
- MB-VIP scores per feature across all blocks (optional plot)
- Empirical p-values per feature from permutation testing
- Selected significant feature table (p < 0.01) ready for downstream structural search/annotation

## How to apply

Load the MY Diabetes serum LC-MS intensity tables (e.g. RPOS, RNEG, LPOS, LNEG) and the metadata holding the continuous bilirubin outcome, then split samples into training and testing subsets (e.g. train_test_split with random_state=42). Fit an MB-PLS regression model with MamsiPls using the NIPALS algorithm and standardise=True across all assay blocks, and estimate the optimal number of latent variables with .estimate_lv() using k-fold cross-validation, metric='q2', max_components=10, n_splits=5. Evaluate the fitted regression model on the independent test set via .evaluate_regression_model(), then compute MB-VIP scores with .mb_vip(plot=True) to quantify each feature's contribution. Run .mb_vip_permtest() with n_permutations=10000 (ideally >100000) and n_jobs=-1 to obtain an empirical p-value per feature across all blocks, and select statistically significant features at the p < 0.01 threshold. This mirrors the article's demonstration where the workflow yielded 18 significant features forming two structural groups and three singletons, 15 of which were linked to bilirubin adducts/isotopologues and one to biliverdin.

## Related tools

- **MAMSI** (Python framework providing MamsiPls (MB-PLS regression, LV estimation, regression evaluation, MB-VIP and MB-VIP permutation testing) for integrating multi-assay LC-MS metabolomics data against a continuous outcome.) — https://github.com/kopeckylukas/py-mamsi
- **mamsi** (PyPI package to install MAMSI; supplies MamsiPls class used to fit, cross-validate and interpret the regression model.) — https://kopeckylukas.github.io/py-mamsi/
- **mbpls 1.0.4** (Pinned dependency providing the underlying MB-PLS algorithm (NIPALS, standardise) used by MamsiPls.)
- **pandas** (Loads assay CSVs and metadata; pd.concat joins assay blocks and pd.read_csv reads the bilirubin outcome.)
- **numpy** (Masking significant features via np.where(p_vals < 0.01).)
- **scipy** (Dependency for statistical operations underlying permutation testing and CV.)
- **scikit-learn** (Provides train_test_split for the training/testing split and k-fold cross-validation machinery.)

## Examples

```
from mamsi.mamsi_pls import MamsiPls; mamsipls = MamsiPls(n_components=6); mamsipls.fit([rpos_train, rneg_train, lpos_train, lneg_train], y_train); mamsipls.estimate_lv([rpos, rneg, lpos, lneg], y, metric='q2'); p_vals, null_vip = mamsipls.mb_vip_permtest([rpos, rneg, lpos, lneg], y, n_permutations=10000, return_scores=True)
```

## Evaluation signals

- The .estimate_lv() cross-validation (metric='q2', max_components=10, n_splits=5) should return a defensible optimal LV count (the article's classification example reported 6 LVs; for regression the q² curve should show a clear optimum).
- .evaluate_regression_model() on the held-out test set should produce non-trivial predictive performance; compare against single-assay PLS baselines, since the article reported MB-PLS outperformed single-assay models in regression tasks.
- MB-VIP scores should be computed across all blocks; block-importance values (e.g. via .block_importance()) should be interpretable and consistent with the predicted outcome.
- Permutation p-values from .mb_vip_permtest() should be empirical (return_scores=True gives null_VIP distributions); significant features at p < 0.01 should be a small, tractable subset.
- Downstream, MamsiStructSearch on the selected features should reproduce the article's signature scale: e.g. 18 significant features forming two structural groups and three singletons, with ~15 putative bilirubin/biliverdin annotations.

## Limitations

- Permutation testing is computationally expensive: the article used n_permutations=10000 in the tutorial but notes 'ideally >100000'; a dedicated HPC workflow is provided in the tutorials repository (sample_code/permtest_hpc/).
- The regression task assumes the response variable is numeric — for MAMSI's demonstration the outcome is clinically measured bilirubin; metadata must be loaded and encoded accordingly.
- Feature selection threshold (p < 0.01) is empirical; the article notes optional multiple-testing correction, and the resulting feature count (e.g. 18 for bilirubin) depends on the dataset and chosen threshold.
- Structural/cross-assay interpretation (adducts, isotopologues, cross-assay [M+H]+/[M-H]- links) relies on MamsiStructSearch tolerances (rt_win=5, ppm=10–15) and NPC ROI annotation files, which are specific to National Phenome Centre assays.

## Evidence

- [readme] MB-PLS regression workflow with MamsiPls, estimate_lv, evaluate_regression_model, mb_vip and mb_vip_permtest: "p_vals, null_vip = mamsipls.mb_vip_permtest([hpos, lpos, lneg], y, n_permutations=10000, return_scores=True)"
- [methods] Split dataset into training and testing subsets: "Split the dataset into training and testing subsets. The training subset will be used to fit the model, for cross-validation and for estimation of the number of latent variables."
- [methods] Permutation testing to obtain empirical p-values per feature: "Calculate empirical p-values for each feature by permuting the Y outcome variable"
- [methods] Select statistically significant features by p-value threshold: "mask = np.where(p_vals < 0.01)
selected = x.iloc[:, mask[0]]"
- [abstract] MB-PLS models outperformed single-assay models in both classification and regression tasks: "The MB-PLS models outperformed single-assay models in both classification and regression tasks, indicating that modeling interblock relationships enabled an improved estimate of phenotypic outcome."
- [intro] MAMSI is a Python framework designed for the integration of multi-assay mass spectrometry datasets.: "MAMSI is a Python framework designed for the integration of multi-assay mass spectrometry datasets."
