---
name: hilic-versus-rplc-descriptor-mechanism-interpretation
description: Use when you have fitted Retip Keras RT-prediction models (HILIC and/or
  RPLC descriptor tables from Retip getCD() plus ChemAxon pKa) and need to rank descriptor
  importances to explain retention mechanisms per chromatographic mode.
when_to_use_negative:
- You need accurate retention predictions for new compounds rather than model interpretation
  — importance ranking does not improve MAE, which remains about ±1 min (0.78 min
  HILIC, 0.57 min RPLC).
- Your compounds fall outside the chemical landscape of the Fiehn HILIC/RPLC training
  sets (e.g. complex high-molecular-weight lipids such as triacylglycerols or phosphatidylcholines,
  which were largely excluded) — the learned descriptor importances may not transfer.
- You are using Retip's other algorithms (Random Forest, BRNN, XGBoost, LightGBM,
  H2O autoML) — this workflow's importance extraction and validation rankings are
  specific to the Keras models.
edam_operation: http://edamontology.org/operation_3791
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3399
tools:
- name: Retip
  role: R package supplying getCD() descriptor computation, cesc() centering/scaling,
    and fit.keras Keras training used to build the models whose importances are ranked
  repo: https://github.com/olobion/Retip
- name: Keras
  role: High-level neural network API (via Tensorflow) whose fitted models are the
    source of the extracted relative descriptor importances
- name: R
  role: Language in which the full Retip workflow (descriptor computation, scaling,
    Keras fitting, importance extraction, CSV export) is executed
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.9b05765
    title: 'Retip: Retention Time Prediction for Compound Annotation in Untargeted
      Metabolomics'
  source_content_hash: sha256:78339b6dfcea653784d13935492fe327f247280fffebb9d195a6bedb44a581af
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/olobion/Retip
  tools:
  - Keras
  - R
  - Retip
  edam_operation: http://edamontology.org/operation_3791
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3399
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# hilic-versus-rplc-descriptor-mechanism-interpretation

## Summary

Interpret trained Retip Keras retention-time models by extracting and ranking relative chemical-descriptor importances separately for the HILIC (880-compound Fiehn training set) and RPLC (398-compound training set) models, revealing which molecular properties (e.g. XLogP, ALogP, Basic Group Count, pKa) drive retention on each chromatographic mode.

## When to use

Use when you have fitted Retip Keras RT-prediction models (HILIC and/or RPLC descriptor tables from Retip getCD() plus ChemAxon pKa) and need to rank descriptor importances to explain retention mechanisms per chromatographic mode.

## When NOT to use

- You need accurate retention predictions for new compounds rather than model interpretation — importance ranking does not improve MAE, which remains about ±1 min (0.78 min HILIC, 0.57 min RPLC).
- Your compounds fall outside the chemical landscape of the Fiehn HILIC/RPLC training sets (e.g. complex high-molecular-weight lipids such as triacylglycerols or phosphatidylcholines, which were largely excluded) — the learned descriptor importances may not transfer.
- You are using Retip's other algorithms (Random Forest, BRNN, XGBoost, LightGBM, H2O autoML) — this workflow's importance extraction and validation rankings are specific to the Keras models.

## Inputs

- HILIC training descriptor table (880 compounds) from Retip getCD() with ChemAxon pKa, EnrichedIndex-named columns
- RPLC training descriptor table (398 compounds) from Retip getCD() with ChemAxon pKa, EnrichedIndex-named columns
- Fitted Keras model objects produced by Retip fit.keras

## Outputs

- descriptor_importance.csv with ranked descriptors and percentage importance weights per chromatographic mode (HILIC and RPLC)
- Ranked descriptor lists (e.g. XLogP, ALogP, largest π chain atoms, nonrotatable bonds, Kier-Hall κ2, pKa for HILIC; XLogP, Basic Group Count, Total Polar Surface Area Efficiency, Valence Path 0, Moreau-Broto ATS.C2 for RPLC)

## How to apply

Load the HILIC training set (880 compounds) and RPLC training set (398 compounds) with EnrichedIndex-named descriptor data computed by Retip's getCD() plus ChemAxon pKa values, then center and scale both data sets with Retip's cesc() function, which is required for neural network prediction. Train the Keras models with Retip's fit.keras on each training split using validation_split = 0.2 (instead of cross-validation), letting dense_unit, epochs, and dropout tune automatically while batch_size and learning_rate are set manually. After fitting, extract relative descriptor importance from each fitted Keras model and rank descriptors per mode: for HILIC the expected order is XLogP (100%), ALogP (98%), largest π chain atoms (87.9%), nonrotatable bonds (60.77%), Kier-Hall κ2 (45.9%), pKa (42.7%); for RPLC it is XLogP (100%) followed by Basic Group Count (90.6%), Total Polar Surface Area Efficiency, Valence Path 0, and Moreau-Broto ATS.C2. Write the ranked importances with their percentage weights to descriptor_importance.csv. Interpret mechanistically: the octanol/water partition coefficient dominating both modes confirms prior models' findings that lipophilicity drives both RPLC and HILIC molecule/column interactions, while Basic Group Count prominence in RPLC reflects ionic/silanol interactions.

## Related tools

- **Retip** (R package supplying getCD() descriptor computation, cesc() centering/scaling, and fit.keras Keras training used to build the models whose importances are ranked) — https://github.com/olobion/Retip
- **Keras** (High-level neural network API (via Tensorflow) whose fitted models are the source of the extracted relative descriptor importances)
- **R** (Language in which the full Retip workflow (descriptor computation, scaling, Keras fitting, importance extraction, CSV export) is executed)

## Examples

```
desc <- getCD(HTMLTable); ds <- cesc(desc); fit.keras(ds, validation_split = 0.2, batch_size = 20, learning_rate = 0.01)
```

## Evaluation signals

- Reproduced HILIC ranking has XLogP at 100% and ALogP at 98% as the top two descriptors.
- Reproduced RPLC ranking has XLogP at 100% and Basic Group Count at 90.6%.
- descriptor_importance.csv contains percentage weights matching reported values for lower-ranked descriptors (largest π chain atoms 87.9%, nonrotatable bonds 60.77%, Kier-Hall κ2 45.9%, pKa 42.7% for HILIC).
- Keras models were trained with validation_split = 0.2 and cesc()-scaled inputs, with small error differences between training, test, and validation sets indicating minimum overfitting.
- Model prediction quality consistent with Keras MAE of 0.78 min (HILIC) and 0.57 min (RPLC).

## Limitations

- Descriptor importances reflect models whose validation MAEs are not better than ±1 min, suggesting molecular descriptors themselves are not sufficiently complex to fully explain LC buffer systems, ionic strengths, microenvironments, and adsorption/desorption processes.
- Importance rankings may not generalize to chemicals not well represented by the training models or to predictions outside the chemical landscape of the training set.
- Prediction accuracy is generally limited by the number of compounds used for training and their chemical diversity; high-molecular-weight lipids (triacylglycerols, phosphatidylcholines) were largely excluded from training.

## Evidence

- [results] For HILIC, the most predictive descriptor was XLogP (100%) and ALogP (98%), followed by largest π chain atoms (87.9%): [evidence span withheld — anchor status: not_found]
- [results] for RPLC, XLogP (100%) was most important, followed by Basic Group Count (90.6%): [evidence span withheld — anchor status: not_found]
