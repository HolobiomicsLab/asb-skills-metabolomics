---
name: keras-neural-network-hyperparameter-tuning
description: Use when a QSAR/RT-prediction task requires fitting a Keras feed-forward
  neural network on centered-and-scaled chemical descriptors (e.g., Retip cesc()-scaled
  EnrichedIndex data) and ranking descriptor importance for HILIC or RPLC retention.
when_to_use_negative:
- Prediction targets lie outside the chemical landscape of the HILIC/RPLC training
  sets — the article warns results 'may be more erroneous', and complex high-molecular-weight
  lipids (triacylglycerols, phosphatidylcholines) were excluded from training
- 'The task needs narrow retention-time search windows: 1 min MAE errors preclude
  discriminating closely eluting compounds in typical 12 min UPLC methods'
- Only tree-based or AutoML learners are required (Random Forest, XGBoost, LightGBM,
  H2O autoML) rather than neural network descriptor-importance analysis
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: Retip
  role: R package providing fit.keras, cesc(), and getCD() for descriptor computation,
    scaling, Keras fitting, and importance extraction
  repo: https://github.com/olobion/Retip
- name: Keras
  role: High-level neural network API on TensorFlow used as the Retip fitting backend
- name: R
  role: Language in which the complete RT prediction workflow and Retip package are
    implemented
- name: reticulate
  role: Bridges R to Python 3.10, which is required to use the Keras model
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.9b05765
    title: 'Retip: Retention Time Prediction for Compound Annotation in Untargeted
      Metabolomics'
  source_content_hash: sha256:19ce1a003955e2b4918f8bcaf75b1a90b6754f36f81454af40fc2bc195860863
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/olobion/Retip
  tools:
  - Keras
  - R
  - Retip
  - reticulate
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# keras-neural-network-hyperparameter-tuning

## Summary

Train and tune Retip's Keras neural network for LC retention time (RT) prediction on HILIC (880 compounds) and RPLC (398 compounds) descriptor tables, using automated search over dense_unit, epochs, and dropout with a 0.2 validation split, then extract per-descriptor relative importance.

## When to use

Use when a QSAR/RT-prediction task requires fitting a Keras feed-forward neural network on centered-and-scaled chemical descriptors (e.g., Retip cesc()-scaled EnrichedIndex data) and ranking descriptor importance for HILIC or RPLC retention.

## When NOT to use

- Prediction targets lie outside the chemical landscape of the HILIC/RPLC training sets — the article warns results 'may be more erroneous', and complex high-molecular-weight lipids (triacylglycerols, phosphatidylcholines) were excluded from training
- The task needs narrow retention-time search windows: 1 min MAE errors preclude discriminating closely eluting compounds in typical 12 min UPLC methods
- Only tree-based or AutoML learners are required (Random Forest, XGBoost, LightGBM, H2O autoML) rather than neural network descriptor-importance analysis

## Inputs

- HILIC training set (880 compounds) with EnrichedIndex-named descriptors from Retip getCD() plus ChemAxon pKa values
- RPLC training set (398 compounds) with the same descriptor schema
- Retention time labels (minutes) for each compound

## Outputs

- Fitted Keras model per chromatographic mode (HILIC and RPLC) with tuned dense_unit, epochs, and dropout
- descriptor_importance.csv: ranked relative descriptor importances with percentage weights (e.g., HILIC XLogP 100%, ALogP 98%; RPLC XLogP 100%, Basic Group Count 90.6%)

## How to apply

Load the HILIC (880 compounds) and RPLC (398 compounds) training splits with descriptor data computed by Retip's getCD() plus ChemAxon pKa values, then center and scale both sets with Retip's cesc() function, which is required before neural network prediction. Fit the model with Retip's fit.keras wrapper around the Keras/TensorFlow backend, setting validation_split = 0.2 in place of cross-validation. Automatically tune the dense_unit, epochs, and dropout parameters while fixing batch_size and learning_rate manually. After fitting, extract relative descriptor importance: for HILIC expect XLogP (100%) and ALogP (98%) at the top, followed by largest π chain atoms (87.9%), nonrotatable bonds (60.77%), Kier-Hall κ2 (45.9%), and pKa (42.7%); for RPLC expect XLogP (100%) then Basic Group Count (90.6%), Total Polar Surface Area Efficiency, Valence Path 0, and Moreau-Broto ATS.C2.

## Related tools

- **Retip** (R package providing fit.keras, cesc(), and getCD() for descriptor computation, scaling, Keras fitting, and importance extraction) — https://github.com/olobion/Retip
- **Keras** (High-level neural network API on TensorFlow used as the Retip fitting backend)
- **R** (Language in which the complete RT prediction workflow and Retip package are implemented)
- **reticulate** (Bridges R to Python 3.10, which is required to use the Keras model)

## Examples

```
devtools::install_github('olobion/Retip')
```

## Evaluation signals

- Top-ranked HILIC descriptors match the reported ranking: XLogP at 100% and ALogP at 98%, followed by largest π chain atoms (87.9%), nonrotatable bonds (60.77%), Kier-Hall κ2 (45.9%), and pKa (42.7%)
- Top-ranked RPLC descriptors match: XLogP at 100% with Basic Group Count at 90.6%
- Small error differences between training, test, and validation sets indicate minimum overfitting
- Model MAE falls at approximately 0.78 min for HILIC and 0.57 min for RPLC

## Limitations

- Retention predictions were not tested for adjustment to small changes in chromatographic methods
- Prediction accuracy is generally limited by the number of compounds used for training and their chemical diversity; many chemicals in test and validation were not well represented by the training models
- None of the current models yield validation MAEs better than ±1 min, suggesting molecular descriptors themselves are not sufficiently complex
- Retip cannot be installed in a conda environment because rJava requires NVIDIA drivers; Keras fitting additionally requires python 3.10 via reticulate

## Evidence
