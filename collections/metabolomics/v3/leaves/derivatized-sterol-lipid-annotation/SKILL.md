---
name: derivatized-sterol-lipid-annotation
description: Use when identifying unsaturated sterol lipids at the C=C isomer level
  in LC-IM-MS/MS tissue data, where measured CCS, RT, and MS/MS of N-Me derivatized
  sterols must be matched to a predicted 4D database.
when_to_use_negative:
- Analyzing native, non-derivatized sterol lipids or lipids lacking C=C bonds — the
  MS/MS script is built on N-Me fragmentation patterns and was only tested on sterol
  lipids
- Datasets without ion mobility/CCS measurements — the workflow is built around LC-IM-MS/MS
  4D sterolomics data processing
- Non-sterol metabolomics where QCC-assisted sterol-specific descriptors and N-Me
  derivatization do not apply
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: scikit-learn
  role: Implements LASSO feature selection and cross-validated SVR for the QCC-assisted
    CCS prediction stage
- name: RDkit
  role: Recognizes C=C double bond positions in N-Me derivatized sterols and generates
    theoretical MS/MS from N-Me fragmentation patterns
- name: retention_time_gnn
  role: Pretrained graph neural network referenced for retention time prediction in
    the 4D sterol lipid database
  repo: https://github.com/seokhokang/retention_time_gnn/
- name: QCCAssisted4DSterol
  role: Jupyter notebook collection hosting the MS/MS calculation, CCS prediction
    model training, and 4D sterolomics data processing/identification scripts
  repo: https://github.com/Chen-micslab/QCCAssisted4DSterol
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/anie.202507483
    title: Quantum Chemistry Calculation‐Assisted Large‐Scale Collision Cross Section
      Prediction Empowers Derivatization‐Enhanced Multidimensional Metabolomics
  source_content_hash: sha256:55a3655d371b763a48bbdf36f43a75c0df581e273368e9ac941c2749f1d3ce3b
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/seokhokang/retention_time_gnn
  tools:
  - Jupyter Notebook
  - Python
  - QCCAssisted4DSterol
  - RDkit
  - retention_time_gnn
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7368421052631579
n_claims: 19
---

# derivatized-sterol-lipid-annotation

## Summary

Annotates N-Me derivatized unsaturated sterol lipids at the isomer level by combining QCC-assisted predicted CCS, GNN-predicted retention time, and N-Me-fragmentation-predicted MS/MS into a 4D sterol lipid database (n = 4891) used to search tissue LC-IM-MS/MS data.

## When to use

Use when identifying unsaturated sterol lipids at the C=C isomer level in LC-IM-MS/MS tissue data, where measured CCS, RT, and MS/MS of N-Me derivatized sterols must be matched to a predicted 4D database.

## When NOT to use

- Analyzing native, non-derivatized sterol lipids or lipids lacking C=C bonds — the MS/MS script is built on N-Me fragmentation patterns and was only tested on sterol lipids
- Datasets without ion mobility/CCS measurements — the workflow is built around LC-IM-MS/MS 4D sterolomics data processing
- Non-sterol metabolomics where QCC-assisted sterol-specific descriptors and N-Me derivatization do not apply

## Inputs

- QCC-assisted training dataset of derivatized sterol lipid quantum chemistry descriptors and measured CCS values (from QCCAssisted4DSterol notebooks)
- Structures of N-Me derivatized unsaturated sterol lipids (double-bond positions recognized by the RDKit-based script)
- LC-IM-MS/MS tissue sample files organized under ./Search/tissue
- RT prediction datasets: METLIN-SMRT (figshare), PredRet targets (predret.org), MoNA targets (mona.fiehnlab.ucdavis.edu)

## Outputs

- ccs_predictions.csv with predicted CCS values alongside lipid identifiers
- Theoretical MS/MS spectra generated from N-Me fragmentation patterns
- Large-scale 4D information database of derivatized sterol lipids (n = 4891) combining retention time and fragment ion prediction
- Isomer-level sterol lipid identifications and tissue-specific distribution profiles

## How to apply

First predict CCS for N-Me derivatized unsaturated sterols: load the QCC-assisted training dataset of quantum chemistry descriptors and measured CCS values, apply LASSO (scikit-learn) to select informative descriptors, then run cross-validated grid search over SVR hyperparameters (kernel, C, epsilon, gamma) and fit the final SVR on the LASSO-selected subset, writing predictions with lipid identifiers to ccs_predictions.csv. In parallel, use the RDKit-based script to recognize double bond positions and generate theoretical MS/MS from N-Me fragmentation patterns, and transfer the pretrained retention time GNN (METLIN-SMRT pretraining, PredRet/MoNA target sets) to predict RT. Combine CCS, RT, and fragment ion predictions into the large-scale 4D information database of derivatized sterol lipids (n = 4891), then search LC-IM-MS/MS tissue sample files organized under ./Search/tissue to identify sterol lipids at the isomer level and quantify tissue-specific distributions (over 100 sterol lipids).

## Related tools

- **scikit-learn** (Implements LASSO feature selection and cross-validated SVR for the QCC-assisted CCS prediction stage)
- **RDkit** (Recognizes C=C double bond positions in N-Me derivatized sterols and generates theoretical MS/MS from N-Me fragmentation patterns)
- **retention_time_gnn** (Pretrained graph neural network referenced for retention time prediction in the 4D sterol lipid database) — https://github.com/seokhokang/retention_time_gnn/
- **QCCAssisted4DSterol** (Jupyter notebook collection hosting the MS/MS calculation, CCS prediction model training, and 4D sterolomics data processing/identification scripts) — https://github.com/Chen-micslab/QCCAssisted4DSterol

## Evaluation signals

- ccs_predictions.csv contains a CCS prediction entry for every sterol lipid in the derivatized set
- The cross-validated SVR converges on a best-performing hyperparameter configuration (kernel, C, epsilon, gamma) selected via grid search
- The combined 4D database reaches n = 4891 derivatized sterol lipid entries with CCS, RT, and fragment ion dimensions populated
- Tissue sample searches under ./Search/tissue follow the expected per-sample folder structure
- Identified sterol coverage is consistent with the paper's reported isomer-level quantitative map of over 100 sterol lipids

## Limitations

- The MS/MS calculation script is theoretically applicable to all molecules containing C=C bonds but was only tested on sterol lipids
- All functions are implemented in Jupyter notebooks in the QCCAssisted4DSterol repository, which has no changelog
- RT prediction depends on transfer learning from the external retention_time_gnn repository and its third-party datasets (METLIN-SMRT, PredRet, MoNA)

## Evidence

- [readme] The CCS prediction process in the paper is implemented using the scikit-learn API.: "The CCS prediction process in the paper is implemented using the scikit-learn API."
- [readme] LASSO and cross-validated SVR: "It employs LASSO for feature selection and uses cross-validation to select the best hyperparameters for the SVR model."
- [readme] RDKit MS/MS calculation: "The script  recognises double bond positions and generates MS/MS based on N-Me fragmentation patterns."
- [abstract] 4D database construction: "create a large-scale, 4D information database of derivatized sterol lipids (n = 4891) by combining retention time and fragment ion prediction"
- [abstract] Isomer-level sterolomics outcome: "a high-coverage unsaturated sterolomics at the isomer level was established on this basis, which quantitatively revealed the tissue-specific distribution patterns of over 100 sterol lipids"
