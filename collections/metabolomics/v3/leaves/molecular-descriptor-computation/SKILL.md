---
name: molecular-descriptor-computation
description: Use when you need to convert molecular structures from a training or
  target set into a large numerical descriptor matrix (e.g. 1613 Mordred descriptors)
  before feature selection and machine learning regression.
when_to_use_negative:
- You already have a final feature table or curated descriptor set — recomputing all
  1613 Mordred descriptors is unnecessary.
- You need CCS predictions for dimers such as [2 M + H]+ or [2 M − H]−, or other less
  common adducts or in-source fragments — current training sets and their descriptor
  pipeline do not support these.
- You lack structures for the target compounds — Mordred descriptors cannot be computed
  without molecular input structures.
edam_operation: http://edamontology.org/operation_3767
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3332
tools:
- name: Mordred
  role: Python package used to compute the library of 1613 molecular descriptors from
    compound structures as SVR input features
- name: CCSP 2.0
  role: Open-source Jupyter notebook that orchestrates descriptor computation, RFE
    feature selection, and downstream CCS prediction
  repo: https://github.com/facundof2016/CCSP2.0
- name: Jupyter Notebook
  role: Execution environment for the CCSP 2.0 notebook (Python 3, e.g. via Anaconda)
    in which Mordred descriptors are calculated
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.2c03491
    title: 'CCS Predictor 2.0: An Open-Source Jupyter Notebook Tool for Filtering
      Out False Positives in Metabolomics'
  source_content_hash: sha256:28e32b2fdd47fbd4981591aa360a4d3816fdb3b7f25a60b319909847877d2693
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/facundof2016/CCSP2.0
  tools:
  - CCSP 2.0
  - Jupyter Notebook
  - Mordred
  edam_operation: http://edamontology.org/operation_3767
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3332
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# molecular-descriptor-computation

## Summary

Compute the 1613-entry Mordred molecular descriptor library from compound structures to serve as regression features for machine learning models such as CCSP 2.0's linear-kernel SVR CCS predictors.

## When to use

Use when you need to convert molecular structures from a training or target set into a large numerical descriptor matrix (e.g. 1613 Mordred descriptors) before feature selection and machine learning regression.

## When NOT to use

- You already have a final feature table or curated descriptor set — recomputing all 1613 Mordred descriptors is unnecessary.
- You need CCS predictions for dimers such as [2 M + H]+ or [2 M − H]−, or other less common adducts or in-source fragments — current training sets and their descriptor pipeline do not support these.
- You lack structures for the target compounds — Mordred descriptors cannot be computed without molecular input structures.

## Inputs

- Molecular structures (e.g. SMILES/SDF from PubChem) for compounds in the McLean CCS Compendium training sets (170 [M−H]−, 155 [M+H]+, 138 [M+Na]+ adducts)
- McLean CCS Compendium reference spreadsheets partitioned by adduct type

## Outputs

- 1613-entry Mordred molecular descriptor matrix per compound structure
- RFE-reduced informative descriptor subset used as SVR regression features

## How to apply

Obtain the compound structures underlying the training set — in CCSP 2.0 these are the 170 [M−H]−, 155 [M+H]+, and 138 [M+Na]+ CCS measurements from the McLean CCS Compendium — and compute the full 1613-entry Mordred molecular descriptor library for each structure via the Mordred Python package inside the CCSP 2.0 Jupyter notebook. Because the descriptor space is much larger than the number of training compounds, apply recursive feature elimination (RFE) next to reduce the descriptor set to the informative subset used for regression. The resulting feature matrices are the input for per-adduct linear-kernel SVR models trained with GridSearchCV over repeated train/test splits. Descriptor computation must precede model training, since ML methods require the training set and its features to be defined prior to predicting CCS values.

## Related tools

- **Mordred** (Python package used to compute the library of 1613 molecular descriptors from compound structures as SVR input features)
- **CCSP 2.0** (Open-source Jupyter notebook that orchestrates descriptor computation, RFE feature selection, and downstream CCS prediction) — https://github.com/facundof2016/CCSP2.0
- **Jupyter Notebook** (Execution environment for the CCSP 2.0 notebook (Python 3, e.g. via Anaconda) in which Mordred descriptors are calculated)

## Evaluation signals

- Descriptor matrix contains the expected 1613 columns (the full Mordred library) for each of the 463 McLean CCS Compendium compounds.
- After RFE, the reduced descriptor subset yields median relative errors of ~1.25% ([M−H]−), ~1.73% ([M+H]+), and ~1.87% ([M+Na]+) with RMSEV of 4.998/5.127/6.612 Å² when fed to linear-kernel SVR models.
- Descriptors are computed for structures partitioned by adduct type before any model training, matching the notebook's embedded instructions.
- Sklearn version V1.0.2 or later is installed (per README requirements.txt) so descriptor/feature pipeline outputs are reproducible.
- Predicted CCS values on held-out test sets fall within a ΔCCS threshold of 2.8% that retains 100% of correct annotations.

## Limitations

- Training set must be defined prior to predicting CCS values, so descriptors for target sets cannot compensate for a missing or mismatched training set.
- Current training sets allow prediction only of the most common adduct ion types ([M − H]−, [M + H]+, [M + Na]+, [M + K]+) but not dimers such as [2 M + H]+ or [2 M − H]−, or other less common adducts or in-source fragments.
- Sklearn V1.0.2 or later is required; older versions may break the notebook's descriptor and regression pipeline.

## Evidence
