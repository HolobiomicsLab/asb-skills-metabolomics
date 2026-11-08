---
name: chemical-descriptor-importance-extraction
description: 'Use when a fitted retention time prediction model (e.g. Retip Keras
  on HILIC or RPLC) needs interpretation: which molecular descriptors most strongly
  predict RT, with percentage importance weights.'
when_to_use_negative:
- When the goal is retention time prediction itself rather than model interpretation
  — descriptor importances do not replace the prediction pipeline and 1 min MAE precludes
  discriminating closely eluting compounds in typical 12 min UPLC methods
- When the target compound domain is outside the training set's chemical landscape
  (e.g. complex, high molecular weight lipids like triacylglycerols or phosphatidylcholines
  were largely excluded from training, so importances may not transfer)
- When a chromatographic method differs from the training method — the study did not
  test whether predictions or their descriptor importances can be adjusted for small
  changes in chromatographic methods
edam_operation: http://edamontology.org/operation_3658
edam_topics:
- http://edamontology.org/topic_3315
- http://edamontology.org/topic_0091
tools:
- name: Retip
  role: Supplies getCD() for descriptor computation, cesc() for centering/scaling,
    and fit.keras for training the Keras model whose descriptor importances are extracted
  repo: https://github.com/olobion/Retip
- name: Keras
  role: High-level neural networks API (via TensorFlow) whose fitted model yields
    the relative descriptor importances to be ranked
- name: R
  role: Runtime environment in which Retip is installed and the importance-extraction
    workflow is executed
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.9b05765
    title: 'Retip: Retention Time Prediction for Compound Annotation in Untargeted
      Metabolomics'
  source_content_hash: sha256:5dff161304c6a7fb6ef4d4b68cb335665e8530efcac88e95aae76b273e572958
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/olobion/Retip
  tools:
  - Keras
  - R
  - Retip
  edam_operation: http://edamontology.org/operation_3658
  edam_topics:
  - http://edamontology.org/topic_3315
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# chemical-descriptor-importance-extraction

## Summary

Extracts and ranks relative chemical-descriptor importance from machine learning models trained for retention time prediction, as done with Retip's Keras models on the HILIC and RPLC datasets. Reveals which descriptors (e.g. XLogP, ALogP, Basic Group Count) drive retention in each chromatographic mode.

## When to use

Use when a fitted retention time prediction model (e.g. Retip Keras on HILIC or RPLC) needs interpretation: which molecular descriptors most strongly predict RT, with percentage importance weights.

## When NOT to use

- When the goal is retention time prediction itself rather than model interpretation — descriptor importances do not replace the prediction pipeline and 1 min MAE precludes discriminating closely eluting compounds in typical 12 min UPLC methods
- When the target compound domain is outside the training set's chemical landscape (e.g. complex, high molecular weight lipids like triacylglycerols or phosphatidylcholines were largely excluded from training, so importances may not transfer)
- When a chromatographic method differs from the training method — the study did not test whether predictions or their descriptor importances can be adjusted for small changes in chromatographic methods

## Inputs

- EnrichedIndex-named chemical descriptor table computed by Retip's getCD() (e.g. 880 HILIC compounds, 398 RPLC compounds) augmented with ChemAxon pKa values
- Retention time labels for each compound in each chromatographic mode (HILIC and RPLC training splits)
- Fitted Retip Keras model (from fit.keras, tuned with dense_unit, epochs, dropout)

## Outputs

- descriptor_importance.csv: ranked descriptor importances with percentage weights per chromatographic mode (e.g. HILIC: XLogP 100%, ALogP 98%; RPLC: XLogP 100%, Basic Group Count 90.6%)

## How to apply

Train the retention time model on EnrichedIndex-named descriptor data computed by Retip's getCD() plus ChemAxon pKa values, after centering and scaling with Retip's cesc() function. For Keras, use Retip's fit.keras with validation_split = 0.2 instead of cross-validation, auto-tuning dense_unit, epochs, and dropout while batch_size and learning_rate are set manually. Then extract relative descriptor importances from each fitted model and rank them per chromatographic mode. For HILIC, expect XLogP at 100% and ALogP at 98% on top, followed by largest π chain atoms (87.9%), nonrotatable bonds (60.77%), Kier-Hall κ2 (45.9%), and pKa (42.7%); for RPLC, expect XLogP at 100% followed by Basic Group Count (90.6%). Write ranked importances with percentage weights to descriptor_importance.csv. The rationale is that descriptor importances reveal which chemical features dominate molecule/column interactions in each chromatographic mode, consistent with prior models finding the octanol/water partition coefficient highly important.

## Related tools

- **Retip** (Supplies getCD() for descriptor computation, cesc() for centering/scaling, and fit.keras for training the Keras model whose descriptor importances are extracted) — https://github.com/olobion/Retip
- **Keras** (High-level neural networks API (via TensorFlow) whose fitted model yields the relative descriptor importances to be ranked)
- **R** (Runtime environment in which Retip is installed and the importance-extraction workflow is executed)

## Examples

```
devtools::install_github('olobion/Retip'); devtools::install_github('olobion/Retiplib')
```

## Evaluation signals

- Top-ranked descriptor must match the paper's ranking: HILIC XLogP 100%, ALogP 98%, largest π chain atoms 87.9%, nonrotatable bonds 60.77%, Kier-Hall κ2 45.9%, pKa 42.7%
- RPLC ranking must show XLogP at 100% and Basic Group Count at 90.6%
- Importance values are percentages; the top descriptor should sum to 100% and remaining ranks decrease monotonically
- descriptor_importance.csv must be populated with ranked entries for both HILIC and RPLC modes
- Keras model diagnostics: small error differences between training, test, and validation sets, consistent with minimum overfitting and MAE of 0.78 min (HILIC) and 0.57 min (RPLC)

## Limitations

- Accuracy is limited by the number of compounds used for training and their chemical diversity
- If Retip is used for predictions outside the chemical landscape of the training set, results may be more erroneous
- Molecular descriptors by themselves seem to fail to fully explain the complex interactions between LC buffer systems (including ionic strengths) and microenvironments such as static aqueous phases
- Many chemicals in test and validation were not well represented by the training models
- It was beyond the scope of the study to test if retention time predictions could be adjusted for small changes in chromatographic methods

## Evidence

- [supplementary] HILIC descriptor ranking (XLogP, ALogP, largest π chain atoms, nonrotatable bonds, Kier-Hall κ2, pKa): [evidence span withheld — anchor status: not_found]
- [supplementary] Keras training protocol via Retip: [evidence span withheld — anchor status: not_found]
- [supplementary] Descriptor preprocessing requirement: [evidence span withheld — anchor status: not_found]
- [discussion] Prior importance pattern: "previous models30,32,33 found the octanol/water partition coefficient to be a highly important predictor for RPLC and HILIC molecule/column interactions, similar to our investigations"
- [discussion] Descriptor scope caveat: "molecular descriptors by themselves seem to fail to fully explain the complex interactions between LC buffer systems (including ionic strengths), microenvironments (such as static aqueous phases in"
- [discussion] Training diversity limitation: "The prediction accuracy is generally limited by the number of compounds used for training and their chemical diversity."
