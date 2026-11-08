---
name: multi-model-machine-learning-comparison
description: Use when you must select a regression model for retention time prediction
  from several candidates and need quantitative comparison of training/test/validation
  errors and true-positive coverage within ±1 min windows to justify the choice.
when_to_use_negative:
- You need predictions for compound classes absent from training data, e.g. complex
  high molecular weight lipids such as triacylglycerols or phosphatidylcholines, which
  were largely excluded from Retip models.
- You require narrow retention time search windows finer than ±1 min, e.g. to discriminate
  closely eluting isomers in typical 12 min UPLC methods.
- Your chromatographic method differs from the trained HILIC/RPLC setup and you have
  no matching training data to refit models.
edam_operation: http://edamontology.org/operation_3988
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_2269
tools:
- name: Retip
  role: R package providing the five-plus regression workflows, RT.spell() prediction,
    and molecular descriptors used to compare models
  repo: https://www.retip.app
- name: Keras
  role: Best-performing regressor with minimum overfitting; MAE 0.78 min HILIC, 0.57
    min RPLC
  repo: https://github.com/olobion/Retip
- name: XGBoost
  role: Extreme gradient boosting regressor benchmarked against Keras
- name: LightGBM
  role: Gradient boosting framework regressor benchmarked against Keras
- name: randomForest
  role: Decision tree ensemble regressor benchmarked against Keras
- name: brnn
  role: Bayesian-regularized neural network regressor benchmarked against Keras
- name: caret
  role: Training and cross-validation framework used by Retip to fit the candidate
    models
- name: ggplot2
  role: Generates violin plots of prediction-error distributions mirroring Figure
    2
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.9b05765
    title: 'Retip: Retention Time Prediction for Compound Annotation in Untargeted
      Metabolomics'
  source_content_hash: sha256:ab1895799a3825e06962f9298fe276f93fce13036d4e6d61e62d9a8ecb6c42ef
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Keras
  - LightGBM
  - R
  - Retip
  - XGBoost
  - brnn
  - caret
  - ggplot2
  - h2o
  - randomForest
  edam_operation: http://edamontology.org/operation_3988
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.92
n_claims: 25
---

# multi-model-machine-learning-comparison

## Summary

Benchmarks five machine learning regressors (random forest, Bayesian-regularized neural network, XGBoost, LightGBM, Keras) side by side for retention time prediction on the HILIC (981 compounds) and RPLC data sets, selecting Keras for lowest overfitting and mean absolute errors of 0.78 min (HILIC) and 0.57 min (RPLC).

## When to use

Use when you must select a regression model for retention time prediction from several candidates and need quantitative comparison of training/test/validation errors and true-positive coverage within ±1 min windows to justify the choice.

## When NOT to use

- You need predictions for compound classes absent from training data, e.g. complex high molecular weight lipids such as triacylglycerols or phosphatidylcholines, which were largely excluded from Retip models.
- You require narrow retention time search windows finer than ±1 min, e.g. to discriminate closely eluting isomers in typical 12 min UPLC methods.
- Your chromatographic method differs from the trained HILIC/RPLC setup and you have no matching training data to refit models.

## Inputs

- HILIC training/test/validation descriptor tables from the Fiehn hydrophilic interaction liquid chromatography data set (981 primary metabolites and biogenic amines)
- RPLC training/test/validation descriptor tables from the RIKEN PlaSMA database (852 secondary metabolites)
- Experimental retention times for each compound
- Fitted model objects: fit.rf, fit.brnn, fit.keras, fit.xgboost, fit.lightgbm

## Outputs

- Per-model mean absolute errors for HILIC and RPLC test sets (e.g. Keras 0.78 min HILIC, 0.57 min RPLC)
- Overfitting assessment as error differences between training, test, and validation sets per model
- True-positive coverage within ±1 min RT windows per model and chromatography (coverage_summary.csv)
- Violin plots of prediction-error distributions per model (violin_error_distributions.png)

## How to apply

Train each candidate regressor — randomForest, brnn, Keras (Tensorflow high-level API), xgboost, and lightgbm — on the same HILIC and reversed-phase training sets (HILIC: Fiehn data set of 981 primary metabolites and biogenic amines; RPLC: RIKEN PlaSMA database of 852 secondary metabolites) using identical molecular-descriptor features computed by Retip. Predict retention times on held-out test and validation splits with each fitted model and compute per-compound errors as predicted minus experimental RT. Report mean absolute errors per model and chromatography (Keras reached 0.78 min HILIC / 0.57 min RPLC) and quantify overfitting as error differences between training, test, and validation sets — Keras showed the minimum overfitting. Additionally compute the fraction of true positives whose absolute error falls within a ±1 min window (test-set averages: RP 87%, HILIC 65%, validation 76%) and visualise error distributions as violin plots, expecting symmetric errors for RP and slight positive-error skew for HILIC. Select the model balancing low MAE against generalisation gaps across splits.

## Related tools

- **Retip** (R package providing the five-plus regression workflows, RT.spell() prediction, and molecular descriptors used to compare models) — https://www.retip.app
- **Keras** (Best-performing regressor with minimum overfitting; MAE 0.78 min HILIC, 0.57 min RPLC) — https://github.com/olobion/Retip
- **XGBoost** (Extreme gradient boosting regressor benchmarked against Keras)
- **LightGBM** (Gradient boosting framework regressor benchmarked against Keras)
- **randomForest** (Decision tree ensemble regressor benchmarked against Keras)
- **brnn** (Bayesian-regularized neural network regressor benchmarked against Keras)
- **caret** (Training and cross-validation framework used by Retip to fit the candidate models)
- **ggplot2** (Generates violin plots of prediction-error distributions mirroring Figure 2)

## Examples

```
Rscript violin_error_distributions.R --models fit.rf,fit.brnn,fit.keras,fit.xgboost,fit.lightgbm --set hilic_test --window 1 --out violin_error_distributions.png
```

## Evaluation signals

- Mean absolute errors per model should match paper values within tolerance: Keras 0.78 min for HILIC and 0.57 min for RPLC; test/validation error differences should be small for the chosen model (minimum overfitting).
- Average true-positive coverage within ±1 min windows should equal reported averages: 65% HILIC test, 87% RPLC test, 76% validation.
- Violin plots should show symmetric error distributions for RP models and slight positive-error skew for HILIC models, mirroring Figure 2.
- Per-compound errors should be computed as predicted minus experimental RT, and the same molecular-descriptor feature set must be used for all models.
- Coverage summary table (coverage_summary.csv) should reconcile with the violin plot annotations and paper-reported averages.

## Limitations

- The octanol/water partition coefficient is a dominant predictor for both RPLC and HILIC interactions, so models are sensitive to descriptor coverage rather than architecture alone.
- None of the current models yield validation MAEs better than ±1 min; molecular descriptors appear insufficient to explain complex LC buffer/microenvironment interactions.
- Prediction accuracy is limited by the number of compounds used for training and their chemical diversity; errors grow outside the training-set chemical landscape.
- 1 min prediction windows exclude about 25% of true positives and preclude discriminating closely eluting compounds in 12 min UPLC methods.
- Predictions are useful for excluding unlikely candidates and ranking positional isomers, not for narrow retention time search windows.
- It was beyond the scope of the study to adjust retention time predictions for small changes in chromatographic methods.
- Complex high molecular weight lipids such as triacylglycerols or phosphatidylcholines were largely excluded from the training data.

## Evidence

- [results] five machine learning models' error distributions shown in Figure 2: [evidence span withheld — anchor status: not_found]
