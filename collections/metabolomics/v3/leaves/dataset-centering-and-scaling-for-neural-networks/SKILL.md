---
name: dataset-centering-and-scaling-for-neural-networks
description: Use when training Keras or BRNN neural networks on chemical descriptor
  matrices for retention time prediction, where features on different scales (e.g.,
  XLogP vs. atom counts vs. pKa) must be centered and scaled first.
when_to_use_negative:
- When fitting tree-based Retip models (random forest, XGBoost, LightGBM) that do
  not require centered/scaled inputs for the neural-network-specific rationale to
  apply
- When predicting on compounds far outside the chemical landscape of the training
  set, since scaling cannot fix descriptor extrapolation errors the article warns
  about
- When the input is already a normalized/scaled feature table rather than raw getCD()
  descriptor output
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3398
- http://edamontology.org/topic_0154
tools:
- name: Retip
  role: Provides the cesc() centering/scaling function and getCD() descriptor computation
    applied before neural network training
  repo: https://github.com/olobion/Retip
- name: Keras
  role: Downstream neural network algorithm (via fit.keras) whose training requires
    the cesc()-scaled descriptor matrix
- name: R
  role: Environment in which the Retip cesc()/fit.keras workflow is executed
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.9b05765
    title: 'Retip: Retention Time Prediction for Compound Annotation in Untargeted
      Metabolomics'
  source_content_hash: sha256:15dea804092f77f2cd456f653dfc7a1371536874a0d6f755a105692dfcada482
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/olobion/Retip
  tools:
  - Keras
  - R
  - Retip
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3398
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# dataset-centering-and-scaling-for-neural-networks

## Summary

Center and scale molecular-descriptor training sets (e.g., Retip's HILIC 880-compound and RPLC 398-compound sets) with Retip's cesc() function before fitting neural network retention-time models such as Keras.

## When to use

Use when training Keras or BRNN neural networks on chemical descriptor matrices for retention time prediction, where features on different scales (e.g., XLogP vs. atom counts vs. pKa) must be centered and scaled first.

## When NOT to use

- When fitting tree-based Retip models (random forest, XGBoost, LightGBM) that do not require centered/scaled inputs for the neural-network-specific rationale to apply
- When predicting on compounds far outside the chemical landscape of the training set, since scaling cannot fix descriptor extrapolation errors the article warns about
- When the input is already a normalized/scaled feature table rather than raw getCD() descriptor output

## Inputs

- Molecular descriptor data frame from Retip getCD() with EnrichedIndex-named columns and ChemAxon pKa values (HILIC: 880 compounds; RPLC: 398 compounds)
- Retention time values paired to each compound (SMILES/InChI) in the training split

## Outputs

- Centered and scaled descriptor matrix ready for fit.keras neural network training
- Trained Keras retention-time models from which relative descriptor importances (e.g., descriptor_importance.csv) can be extracted

## How to apply

Compute chemical descriptors with Retip's getCD() (EnrichedIndex-named columns, augmented with ChemAxon pKa values) for each training set, such as the Fiehn HILIC set of 880 compounds and the RPLC set of 398 compounds. Before fitting, pass the descriptor data frame through Retip's cesc() function, which centers and scales every descriptor column; the article states this step is required for neural network prediction. Keep the cesc() transformation consistent between HILIC and RPLC splits so that descriptor importances extracted afterwards from the fitted Keras model (e.g., XLogP at 100%, ALogP at 98% for HILIC) are comparable. Only after scaling, train with fit.keras using validation_split = 0.2, automatically tuning dense_unit, epochs, and dropout while batch_size and learning_rate are set manually.

## Related tools

- **Retip** (Provides the cesc() centering/scaling function and getCD() descriptor computation applied before neural network training) — https://github.com/olobion/Retip
- **Keras** (Downstream neural network algorithm (via fit.keras) whose training requires the cesc()-scaled descriptor matrix)
- **R** (Environment in which the Retip cesc()/fit.keras workflow is executed)

## Examples

```
scaled_hilic <- Retip::cesc(hilic_descriptors); fit.keras(scaled_hilic, dense_unit = tuned, epochs = tuned, dropout = tuned, validation_split = 0.2, batch_size = manual, learning_rate = manual)
```

## Evaluation signals

- Reproduced Keras descriptor importances match the paper's ranking: XLogP at 100% and ALogP at 98% for HILIC; XLogP at 100% with Basic Group Count at 90.6% for RPLC
- Scaled descriptor columns have (approximately) zero mean and unit variance after cesc()
- Keras model MAE stays near the reported 0.78 min (HILIC) and 0.57 min (RPLC), with small error differences between training, test, and validation sets indicating minimal overfitting
- validation_split = 0.2 tuning of dense_unit, epochs, and dropout converges without the scaling step being skipped or applied inconsistently across the HILIC/RPLC splits

## Limitations

- Scaling does not compensate for molecular descriptors being insufficiently complex: none of the current models yield validation MAEs better than ±1 min, and descriptors fail to fully explain complex interactions between LC buffer systems, microenvironments, and adsorption/desorption processes
- Predictions on compounds outside the chemical landscape of the training set may be more erroneous, limiting the usefulness of the scaled model
- Prediction accuracy is generally limited by the number of compounds used for training and their chemical diversity

## Evidence

- [methods] Center and scale both data sets with Retip's cesc() function as required for neural network prediction.: [evidence span withheld — anchor status: not_found]
- [methods] Train Keras models with Retip's fit.keras on the HILIC and RPLC training splits, using validation_split = 0.2 instead of cross-validation.: [evidence span withheld — anchor status: not_found]
