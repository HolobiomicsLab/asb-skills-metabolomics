---
name: ccs-based-false-positive-filtering
description: Use when a candidate list of molecular structures (e.g. PubChem hits
  within a 10 ppm mass window of a deprotonated/protonated ion) must be filtered using
  ion-mobility-derived CCS values.
when_to_use_negative:
- When the analyte forms dimers such as [2M+H]+ or [2M−H]− or other less common adducts
  or in-source fragments — current training sets only cover [M−H]−, [M+H]+, [M+Na]+,
  and [M+K]+.
- When no experimental CCS value is available for the unknown ion, since the ΔCCS
  filtering step compares predicted CCS against a measured CCS.
- When small-scale prediction in a non-Python environment is needed and local installation
  is impractical — the Google Colab version caps runs at 12 hours and disconnects
  after ~20 minutes idle.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: CCSP 2.0
  role: Open-source Jupyter notebook implementing linear SVR CCS prediction and ΔCCS-based
    candidate filtering with user-curated training sets
  repo: https://github.com/facundof2016/CCSP2.0
- name: Mordred
  role: Python package embedded in the CCSP 2.0 notebook used to compute the 1613
    molecular descriptors for training and candidate structures
- name: Jupyter Notebook
  role: Execution environment for the CCSP 2.0 notebook (Jupyter Lab version with
    Tk interface, or Google Colab variant)
  repo: https://github.com/facundof2016/CCSP2.0
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.2c03491
    title: 'CCS Predictor 2.0: An Open-Source Jupyter Notebook Tool for Filtering
      Out False Positives in Metabolomics'
  source_content_hash: sha256:56dca1d247c98bb07a0db8efe314463319c10fdb04e1282dd490b75c17bf5103
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/facundof2016/CCSP2.0
  tools:
  - CCSP 2.0
  - Jupyter Notebook
  - Mordred
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# ccs-based-false-positive-filtering

## Summary

Predict collision cross section (CCS) values with CCSP 2.0's linear support vector regression models and filter false-positive metabolomic annotation candidates using a ΔCCS threshold of 2.8% combined with a 10 ppm mass-error window.

## When to use

Use when a candidate list of molecular structures (e.g. PubChem hits within a 10 ppm mass window of a deprotonated/protonated ion) must be filtered using ion-mobility-derived CCS values.

## When NOT to use

- When the analyte forms dimers such as [2M+H]+ or [2M−H]− or other less common adducts or in-source fragments — current training sets only cover [M−H]−, [M+H]+, [M+Na]+, and [M+K]+.
- When no experimental CCS value is available for the unknown ion, since the ΔCCS filtering step compares predicted CCS against a measured CCS.
- When small-scale prediction in a non-Python environment is needed and local installation is impractical — the Google Colab version caps runs at 12 hours and disconnects after ~20 minutes idle.

## Inputs

- Training set of experimental CCS values (e.g. 93 [M−H]− lipid ions from the McLean CCS Compendium)
- Candidate structure list from PubChem within a 10 ppm mass-error window of the measured m/z
- Measured CCS value and m/z of the unknown ion (e.g. leukotriene E4 [M−H]−)

## Outputs

- Filtered candidate list (e.g. 4980 → 806 candidates remaining after ΔCCS 2.8% filtering)
- Filtering statistics: percent incorrect-structure filtering (36.1%), isomeric filtering (19.3%), retention of correct annotations (100%), AUC (~0.75–0.78)
- CSV of results (ccsp_lipid_filtering_results.csv)

## How to apply

Open the 'CCSP 2.0 - CCS Prediction in Jupyter' notebook (.ipynb) in Jupyter Lab (requires Sklearn v1.0.2+) and load a superclass-matched training set of experimental CCS values, e.g. 93 [M−H]− lipid ions from the McLean CCS Compendium. Compute the 1613 molecular descriptors for each training structure with the Mordred Python package embedded in the notebook, then fit the linear support vector regression model and confirm the median relative CCS prediction error is ≤1.25% for deprotonated adducts. Retrieve candidate structures (e.g. 4980 PubChem hits for leukotriene E4 [M−H]− within 10 ppm), predict their CCS values, and apply the ΔCCS 2.8% threshold to retain only candidates whose predicted CCS falls within 2.8% of the measured ion's CCS. Rationale: superclass-matched models improve annotation confidence because they retain 100% of correct annotations while filtering 36.1% of incorrect class-matched structures.

## Related tools

- **CCSP 2.0** (Open-source Jupyter notebook implementing linear SVR CCS prediction and ΔCCS-based candidate filtering with user-curated training sets) — https://github.com/facundof2016/CCSP2.0
- **Mordred** (Python package embedded in the CCSP 2.0 notebook used to compute the 1613 molecular descriptors for training and candidate structures)
- **Jupyter Notebook** (Execution environment for the CCSP 2.0 notebook (Jupyter Lab version with Tk interface, or Google Colab variant)) — https://github.com/facundof2016/CCSP2.0

## Examples

```
jupyter lab "CCSP 2.0 - CCS Prediction in Jupyter.ipynb"
```

## Evaluation signals

- Median relative CCS prediction error of the trained SVR model on the [M−H]− lipid test set is ≤1.25%.
- After applying ΔCCS 2.8% with 10 ppm mass error, the candidate count drops from 4980 to 806 for the leukotriene E4 [M−H]− case.
- 100% of correct annotations are retained while 36.1% of incorrect class-matched structures are filtered (19.3% isomeric filtering).
- Filtering AUC scores fall in the ~0.75–0.78 range.
- requirements.txt is satisfied (Sklearn v1.0.2 or later) before running the notebook.

## Limitations

- One drawback of most ML methods is the need to define the specific training set to be used prior to predicting CCS values.
- Current training sets allow prediction of the most common adduct ion types ([M−H]−, [M+H]+, [M+Na]+, [M+K]+) but not dimers such as [2M+H]+ or [2M−H]−, or other less common adducts or in-source fragments.
- Google Colab limits continuous operation to 12 hours and disconnects after ~20 minutes idle; large training/target sets require the locally hosted Jupyter Lab version.

## Evidence
