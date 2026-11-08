---
name: collision-cross-section-prediction-integration
description: Use when constructing a 4D (m/z, RT, CCS, fragment ion) reference database
  for N-Me derivatized unsaturated sterol lipids, or when CCS values for derivatized
  lipids must be predicted and merged with RT and MS/MS predictions for LC-IM-MS/MS
  identification.
when_to_use_negative:
- The analyte class is not sterol lipids — the MS/MS script is 'Theoretically applicable
  to all molecules including C=C bond (only test sterol lipids)', so other lipid classes
  are unvalidated
- The input is already an experimental CCS/RT/MS/MS measurement table rather than
  candidate structures needing prediction
- A targeted analysis without the need for a 4D reference database (the workflow exists
  to power LC-IM-MS/MS based 4D sterolomics data processing and identification)
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3474
tools:
- name: Python
  role: Implementation language for the QCCAssisted4DSterol scripts that assemble
    the 4D database
  repo: github.com/Chen-micslab/QCCAssisted4DSterol
- name: Jupyter Notebook
  role: Hosts all functions of the QCCAssisted4DSterol workflow, including the 4D
    database assembly
  repo: github.com/Chen-micslab/QCCAssisted4DSterol
- name: RDkit
  role: Basis of the MS/MS calculation script that recognises double bond positions
    and generates N-Me fragment ions
  repo: github.com/Chen-micslab/QCCAssisted4DSterol
- name: scikit-learn
  role: API for the CCS prediction model (LASSO feature selection, cross-validated
    SVR)
  repo: github.com/Chen-micslab/QCCAssisted4DSterol
- name: retention_time_gnn
  role: Pretrained graph neural network used for retention time prediction of the
    derivatized sterol candidates
  repo: github.com/seokhokang/retention_time_gnn/
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/anie.202507483
    title: Quantum Chemistry Calculation‐Assisted Large‐Scale Collision Cross Section
      Prediction Empowers Derivatization‐Enhanced Multidimensional Metabolomics
  source_content_hash: sha256:c5dd8ab4e6042bac12e1bf99fa60e424a5807876bd8717e014f315b8a60ed6bb
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Jupyter Notebook
  - Python
  - RDkit
  - retention_time_gnn
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# collision-cross-section-prediction-integration

## Summary

Builds a large-scale 4D information database (m/z, retention time, CCS, fragment ions) of N-Me derivatized sterol lipids by combining quantum chemistry calculation-assisted SVR CCS prediction, pretrained graph neural network retention time prediction, and RDkit-based MS/MS fragment generation, yielding n = 4891 derivatized sterol lipid entries.

## When to use

Use when constructing a 4D (m/z, RT, CCS, fragment ion) reference database for N-Me derivatized unsaturated sterol lipids, or when CCS values for derivatized lipids must be predicted and merged with RT and MS/MS predictions for LC-IM-MS/MS identification.

## When NOT to use

- The analyte class is not sterol lipids — the MS/MS script is 'Theoretically applicable to all molecules including C=C bond (only test sterol lipids)', so other lipid classes are unvalidated
- The input is already an experimental CCS/RT/MS/MS measurement table rather than candidate structures needing prediction
- A targeted analysis without the need for a 4D reference database (the workflow exists to power LC-IM-MS/MS based 4D sterolomics data processing and identification)

## Inputs

- N-Me derivatized sterol lipid candidate structures (with double bond positions) as molecular representations processed by RDkit
- Quantum chemistry calculation-assisted feature set for CCS model training/prediction
- Pretrained GNN retention time prediction model from seokhokang/retention_time_gnn
- RDkit MS/MS calculation script for N-Me fragmentation patterns

## Outputs

- sterol_4d_database.csv — a table of 4891 derivatized sterol lipid entries, each carrying retention time, CCS, m/z, and fragment ion fields
- Predicted CCS values from the scikit-learn SVR model (LASSO feature selection, cross-validated hyperparameters)
- Predicted retention times from the pretrained graph neural network

## How to apply

First run the QCC-assisted CCS prediction workflow: use the scikit-learn SVR model with LASSO feature selection and cross-validated hyperparameters, trained on quantum chemistry calculation-assisted features, to predict CCS for N-Me derivatized sterol candidates. Next load retention time predictions from the pretrained graph neural network approach (seokhokang/retention_time_gnn), which transfers a GNN pretrained on METLIN-SMRT to target retention datasets. Then run the RDkit-based MS/MS calculation script, which recognises double bond positions and generates fragment ions according to N-Me fragmentation patterns for each sterol lipid. Combine the per-molecule m/z, RT, CCS, and fragment ion fields into a single table keyed by sterol lipid identity and write it to sterol_4d_database.csv, validating that all n = 4891 entries carry all four 4D fields.

## Related tools

- **Python** (Implementation language for the QCCAssisted4DSterol scripts that assemble the 4D database) — github.com/Chen-micslab/QCCAssisted4DSterol
- **Jupyter Notebook** (Hosts all functions of the QCCAssisted4DSterol workflow, including the 4D database assembly) — github.com/Chen-micslab/QCCAssisted4DSterol
- **RDkit** (Basis of the MS/MS calculation script that recognises double bond positions and generates N-Me fragment ions) — github.com/Chen-micslab/QCCAssisted4DSterol
- **scikit-learn** (API for the CCS prediction model (LASSO feature selection, cross-validated SVR)) — github.com/Chen-micslab/QCCAssisted4DSterol
- **retention_time_gnn** (Pretrained graph neural network used for retention time prediction of the derivatized sterol candidates) — github.com/seokhokang/retention_time_gnn/

## Examples

```
python run_transfer.py -t FEM_long
```

## Evaluation signals

- The assembled sterol_4d_database.csv contains exactly the reported n = 4891 derivatized sterol lipid entries
- Every entry carries all four 4D fields: retention time, CCS, m/z, and fragment ions
- The CCS model was trained with LASSO feature selection and cross-validated hyperparameters for the SVR model (scikit-learn API)
- Fragment ions were generated from N-Me fragmentation patterns with correctly recognized double bond positions
- RT predictions come from the referenced pretrained GNN (retention_time_gnn) rather than an ad hoc model

## Limitations

- The RDkit MS/MS script is only tested on sterol lipids despite being theoretically applicable to all molecules with C=C bonds
- RT prediction depends on transfer learning from a small training dataset via a pretrained GNN, so accuracy is bounded by the pretrained model's transferability
- No changelog found in the source repository, so version provenance of the scripts is untracked

## Evidence

- [abstract] create a large-scale, 4D information database of derivatized sterol lipids (n = 4891) by combining retention time and fragment ion prediction: "create a large-scale, 4D information database of derivatized sterol lipids (n = 4891) by combining retention time and fragment ion prediction"
- [methods] It employs LASSO for feature selection and uses cross-validation to select the best hyperparameters for the SVR model.: "It employs LASSO for feature selection and uses cross-validation to select the best hyperparameters for the SVR model."
- [readme] The RT prediction process referenced the ”Retention Time Prediction through Learning from a Small Training Data Set with a Pretrained Graph Neural Network“.: "The RT prediction process referenced the ”Retention Time Prediction through Learning from a Small Training Data Set with a Pretrained Graph Neural Network“."
- [readme] The script recognises double bond positions and generates MS/MS based on N-Me fragmentation patterns.: "The script recognises double bond positions and generates MS/MS based on N-Me fragmentation patterns."
- [abstract] quantum chemistry calculation-assisted machine learning strategies applicable to the accurate prediction of the CCS of derivatized sterols: "quantum chemistry calculation-assisted machine learning strategies applicable to the accurate prediction of the CCS of derivatized sterols"
