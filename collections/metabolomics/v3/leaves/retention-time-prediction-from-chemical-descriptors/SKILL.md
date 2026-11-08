---
name: retention-time-prediction-from-chemical-descriptors
description: Use when SMILES structures need predicted retention times (HILIC or RPLC)
  to rank or filter candidate structures in untargeted metabolomics, or when batch-predicting
  RTs for a large compound database.
when_to_use_negative:
- Predicting RTs for complex, high molecular weight lipids such as triacylglycerols
  or phosphatidylcholines, which were largely excluded from the training set — results
  may be more erroneous outside the training chemical landscape
- Needing narrow retention time search windows to discriminate closely eluting compounds
  in typical 12 min UPLC methods, where 1 min prediction errors preclude such discrimination
- Adjusting retention time predictions for small changes in chromatographic methods,
  which was explicitly beyond the scope of the Retip study
tools:
- name: Retip
  role: R package providing descriptor computation (getCD), scaling (cesc), and the
    five/six ML models (Keras, random forest, BRNN, XGBoost, LightGBM, H2O autoML)
    for RT prediction
  repo: https://github.com/olobion/Retip
- name: Retiplib
  role: Companion package bundling SMILES from the public biochemical databases used
    for library-wide RT prediction
  repo: https://github.com/olobion/Retiplib
- name: Keras
  role: Best-performing model in Retip; used to generate HILIC (MAE 0.78 min) and
    RPLC (MAE 0.57 min) predictions for the 320,919 structures
- name: MS-FINDER
  role: Consumes Retip-predicted RTs (v3.24+/3.30) to rank and exclude candidate structures,
    e.g., a 68% reduction in isomer candidates on mouse blood plasma
- name: MS-DIAL
  role: Mass spectrometry software into which Retip RT predictions are integrated
    for compound annotation
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.9b05765
    title: 'Retip: Retention Time Prediction for Compound Annotation in Untargeted
      Metabolomics'
  source_content_hash: sha256:be9e9f81348c9378a36d9d934dc50616f265e9a5c807b0d2746dcd863dbfcbb6
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/olobion/Retip
  tools:
  - Keras
  - MS-DIAL
  - MS-FINDER
  - R
  - Retip
  - Retiplib
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# retention-time-prediction-from-chemical-descriptors

## Summary

Predict HILIC and reversed-phase LC retention times for small molecules from SMILES structures using the Retip R package's machine learning models (Keras neural network performed best), with RT predictions used to rank or exclude structure candidates in untargeted metabolomics annotation.

## When to use

Use when SMILES structures need predicted retention times (HILIC or RPLC) to rank or filter candidate structures in untargeted metabolomics, or when batch-predicting RTs for a large compound database.

## When NOT to use

- Predicting RTs for complex, high molecular weight lipids such as triacylglycerols or phosphatidylcholines, which were largely excluded from the training set — results may be more erroneous outside the training chemical landscape
- Needing narrow retention time search windows to discriminate closely eluting compounds in typical 12 min UPLC methods, where 1 min prediction errors preclude such discrimination
- Adjusting retention time predictions for small changes in chromatographic methods, which was explicitly beyond the scope of the Retip study

## Inputs

- SMILES structure strings for small molecules (e.g., from HMDB, DrugBank, FooDB, STOFF-IDENT and other bundled databases)
- Pre-trained Retip HILIC and RPLC Keras model artifacts
- CDK molecular descriptor matrix (286 descriptors) computed per structure

## Outputs

- Structured table (e.g., retip_rt_predictions_320919.csv) with compound identifier, SMILES, HILIC-predicted RT, and RPLC-predicted RT
- RT-annotated library files deployable in MS-FINDER v3.24+/3.30 for ranking or excluding candidate structures

## How to apply

Install the Retip and Retiplib R packages via devtools::install_github('olobion/Retip') and ('olobion/Retiplib'), requiring R 4.4.0, Java JDK, and python3/reticulate for Keras. Collect SMILES from the bundled biochemical databases (HMDB, KNApSAcK, ChEBI, DrugBank, SMPDB, YMDB, T3DB, FooDB, NANPDB, STOFF, BMDB, LipidMAPS, Urine, Saliva, Feces, ECMDB, CSF, Serum, PubChem.1, PlantCyc, UNPD, BLEXP, NPA, COCONUT) and compute the CDK molecular descriptors for each structure with the Retip getCD() function. Center and scale the descriptor matrix with the cesc() function as required for neural-network input. Predict RTs with the pre-trained HILIC and RPLC Keras models (chosen because Keras outperformed random forest, BRNN, XGBoost, and LightGBM with minimum overfitting; expected MAE 0.78 min HILIC and 0.57 min RPLC) via RT.spell()/getRT.smile(). Flag compounds outside the training chemical space (e.g., triacylglycerols or phosphatidylcholines, which were excluded from training) as out-of-domain, and apply predictions as a filter with about a 1 min window for excluding false positives in MS-FINDER/MS-DIAL annotation workflows.

## Related tools

- **Retip** (R package providing descriptor computation (getCD), scaling (cesc), and the five/six ML models (Keras, random forest, BRNN, XGBoost, LightGBM, H2O autoML) for RT prediction) — https://github.com/olobion/Retip
- **Retiplib** (Companion package bundling SMILES from the public biochemical databases used for library-wide RT prediction) — https://github.com/olobion/Retiplib
- **Keras** (Best-performing model in Retip; used to generate HILIC (MAE 0.78 min) and RPLC (MAE 0.57 min) predictions for the 320,919 structures)
- **MS-FINDER** (Consumes Retip-predicted RTs (v3.24+/3.30) to rank and exclude candidate structures, e.g., a 68% reduction in isomer candidates on mouse blood plasma)
- **MS-DIAL** (Mass spectrometry software into which Retip RT predictions are integrated for compound annotation)

## Examples

```
devtools::install_github('olobion/Retip')
```

## Evaluation signals

- Output contains exactly one prediction row per compound with non-negative HILIC and RPLC RT values
- Spot-checked predictions agree with RT values shipped in MS-FINDER 3.24+ within the reported tolerances (0.78 min MAE HILIC, 0.57 min MAE RPLC)
- Small error differences between training, test, and validation sets, indicating minimum overfitting for the chosen Keras model
- Descriptor matrix was centered and scaled with cesc() before Keras prediction (unscaled descriptors invalidate neural-network predictions)
- When used as an annotation filter, candidate-structure reduction in MS-FINDER isomer search is observed (paper reports 68% on mouse blood plasma)

## Limitations

- Molecular descriptors alone appear insufficiently complex to explain LC buffer, ionic-strength, and adsorption/desorption interactions; no current models achieve validation MAEs better than ±1 min
- 1 min filter windows excluded about 25% of true positives, and predictions are not yet adequate for narrow retention time search windows
- Prediction accuracy is limited by the number of training compounds and their chemical diversity; many chemicals were not well represented by the training models
- Installation constraints: Retip 2.0 requires R 4.4.0, Java JDK (rJava), and python3 for Keras, and cannot be installed in a conda environment because rJava requires NVIDIA drivers

## Evidence
