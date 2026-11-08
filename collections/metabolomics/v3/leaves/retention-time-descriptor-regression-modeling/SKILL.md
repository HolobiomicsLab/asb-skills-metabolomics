---
name: retention-time-descriptor-regression-modeling
description: Use when you have SMILES/InChI structures or a compound list with measured
  LC retention times (HILIC or reversed-phase) and need RT predictions or descriptor-importance
  rankings to filter annotation candidates.
when_to_use_negative:
- Predicting retention times for compounds outside the chemical landscape of the training
  set — results may be more erroneous
- Adjusting predictions for small changes in chromatographic methods — the study explicitly
  did not test this
- Complex, high molecular weight lipids such as triacylglycerols or phosphatidylcholines,
  which were largely excluded from training
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3474
tools:
- name: Retip
  role: R package providing getCD() descriptor computation, cesc() centering/scaling,
    fit.keras model training, and descriptor importance extraction
  repo: https://github.com/olobion/Retip
- name: Keras
  role: High-level neural networks API used as the top-performing RT regression model
    (MAE 0.78 min HILIC, 0.57 min RPLC)
- name: MS-FINDER
  role: Downstream consumer of Retip RT predictions to filter isomer candidate structures
    (68% candidate reduction)
- name: R
  role: Environment for the complete retention time prediction workflow
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.9b05765
    title: 'Retip: Retention Time Prediction for Compound Annotation in Untargeted
      Metabolomics'
  source_content_hash: sha256:1cf479ce2dfe541583eae1a10b5858d3daa83096c6fb066933d70c2738023dea
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/olobion/Retip
  tools:
  - Keras
  - MS-FINDER
  - R
  - Retip
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# retention-time-descriptor-regression-modeling

## Summary

Train and interpret machine learning regression models (Keras, random forest, BRNN, XGBoost, LightGBM) in the Retip R package to predict HPLC retention times of small molecules from chemical descriptors, on the Fiehn HILIC (981 metabolites) and RPLC datasets.

## When to use

Use when you have SMILES/InChI structures or a compound list with measured LC retention times (HILIC or reversed-phase) and need RT predictions or descriptor-importance rankings to filter annotation candidates.

## When NOT to use

- Predicting retention times for compounds outside the chemical landscape of the training set — results may be more erroneous
- Adjusting predictions for small changes in chromatographic methods — the study explicitly did not test this
- Complex, high molecular weight lipids such as triacylglycerols or phosphatidylcholines, which were largely excluded from training

## Inputs

- Compound lists with structures (SMILES/InChI) and measured retention times from HILIC or RPLC LC-MS runs (e.g., Fiehn HILIC set of 981 primary metabolites and biogenic amines)
- Molecular descriptors computed via Retip's getCD() (CDK descriptors incl. XLogP, ALogP, Kier-Hall κ2, Basic Group Count) plus ChemAxon pKa values

## Outputs

- Fitted retention time prediction models with validation MAE (0.78 min HILIC, 0.57 min RPLC for Keras)
- descriptor_importance.csv with ranked descriptors and percentage importance weights
- Predicted RT values used as ±1 min filters for candidate structures (e.g., in MS-FINDER isomer search)

## How to apply

Load the training sets (880 HILIC and 398 RPLC compounds) with EnrichedIndex-named descriptors computed by Retip's getCD() plus ChemAxon pKa values, then center and scale both descriptor matrices with Retip's cesc() function as required for neural network prediction. Train Keras models with Retip's fit.keras using validation_split = 0.2 instead of cross-validation, auto-tuning dense_unit, epochs, and dropout while setting batch_size and learning_rate manually. Extract relative descriptor importance per model and rank them: for HILIC XLogP (100%), ALogP (98%), largest π chain atoms (87.9%), nonrotatable bonds (60.77%), Kier-Hall κ2 (45.9%), pKa (42.7%); for RPLC XLogP (100%) followed by Basic Group Count (90.6%). Write ranked importances with percentage weights to descriptor_importance.csv and use predicted RT windows (MAE ~1 min thresholds) to exclude unlikely structure candidates.

## Related tools

- **Retip** (R package providing getCD() descriptor computation, cesc() centering/scaling, fit.keras model training, and descriptor importance extraction) — https://github.com/olobion/Retip
- **Keras** (High-level neural networks API used as the top-performing RT regression model (MAE 0.78 min HILIC, 0.57 min RPLC))
- **MS-FINDER** (Downstream consumer of Retip RT predictions to filter isomer candidate structures (68% candidate reduction))
- **R** (Environment for the complete retention time prediction workflow)

## Examples

```
devtools::install_github('olobion/Retip')
```

## Evaluation signals

- Reproduced HILIC importance ranking must match the paper: XLogP at 100% and ALogP at 98% at the top
- Reproduced RPLC importance ranking must show XLogP at 100% and Basic Group Count at 90.6%
- Keras validation MAE should be near the reported values: 0.78 min for HILIC and 0.57 min for RPLC
- Error differences between training, test, and validation sets should remain small (minimum overfitting, as reported)
- descriptor_importance.csv rows carry percentage weights consistent with the ranked order

## Limitations

- None of the current models yield validation MAEs better than ±1 min; such 1 min windows excluded about 25% of true positives
- For typical 12 min UPLC methods, 1 min prediction errors preclude discriminating closely eluting compounds
- Molecular descriptors may not fully explain complex interactions between LC buffer systems, microenvironments, and adsorption/desorption processes
- Prediction accuracy is limited by the number of compounds used for training and their chemical diversity
- Retip cannot be installed in conda environments because rJava requires NVIDIA drivers (README)

## Evidence
