---
name: lipid-ccs-prediction
description: Use when annotating lipid metabolites from ion-mobility MS data where
  candidate structures from PubChem must be ranked or filtered using measured/predicted
  CCS values, and a lipid-superclass [M−H]− training set is available or desired.
when_to_use_negative:
- The analyte is outside the lipid/lipid-like superclass or uses adduct types not
  covered by the training set (e.g., dimers such as [2M+H]+ or [2M−H]−, other less
  common adducts, or in-source fragments)
- No CCS measurements are available to curate a superclass-matched training set and
  only generic all-class prediction is needed — consider broader models (MetCCS, AllCCS)
  instead of tailored training
- Very large prediction jobs where the 12-hour/20-minute-idle limits of Google Colab
  would be hit — use the locally hosted Jupyter Lab notebook instead
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: CCSP 2.0
  role: Open-source Jupyter notebook implementing linear SVR CCS prediction from user-curated
    training sets; used to train the lipid model and filter candidates
  repo: https://github.com/facundof2016/CCSP2.0
- name: Jupyter Notebook
  role: Execution environment for the locally hosted CCSP 2.0 notebook (Jupyter Lab
    compatible, with Tk interface)
  repo: https://github.com/facundof2016/CCSP2.0
- name: Mordred
  role: Python package embedded in the notebook used to compute the 1613 molecular
    descriptors for training and candidate structures
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.2c03491
    title: 'CCS Predictor 2.0: An Open-Source Jupyter Notebook Tool for Filtering
      Out False Positives in Metabolomics'
  source_content_hash: sha256:f1e3d27d8dc4f069fb9890951196fb8868da7f82ff95bea0feb95ea2bd25c5ab
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/facundof2016/CCSP2.0
  tools:
  - CCSP 2.0
  - Jupyter Notebook
  - Mordred
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# lipid-ccs-prediction

## Summary

Train and apply a superclass-matched linear support vector regression model in CCSP 2.0 (a Jupyter notebook using 1613 Mordred molecular descriptors) to predict collision cross sections for lipid [M−H]− ions and filter false-positive annotation candidates at a ΔCCS threshold of 2.8%.

## When to use

Use when annotating lipid metabolites from ion-mobility MS data where candidate structures from PubChem must be ranked or filtered using measured/predicted CCS values, and a lipid-superclass [M−H]− training set is available or desired.

## When NOT to use

- The analyte is outside the lipid/lipid-like superclass or uses adduct types not covered by the training set (e.g., dimers such as [2M+H]+ or [2M−H]−, other less common adducts, or in-source fragments)
- No CCS measurements are available to curate a superclass-matched training set and only generic all-class prediction is needed — consider broader models (MetCCS, AllCCS) instead of tailored training
- Very large prediction jobs where the 12-hour/20-minute-idle limits of Google Colab would be hit — use the locally hosted Jupyter Lab notebook instead

## Inputs

- Superclass-matched CCS training measurements (e.g., 93 lipid [M−H]− ions from the McLean CCS Compendium)
- Molecular structures of training compounds (for Mordred descriptor computation)
- Accurate precursor m/z of the unknown ion and mass-error tolerance (e.g., 10 ppm)
- Candidate structures retrieved from PubChem within the mass-error window

## Outputs

- Trained linear SVR CCS prediction model for lipid [M−H]− adducts
- Predicted CCS values for each candidate structure
- Filtered candidate list after ΔCCS 2.8% thresholding
- Filtering statistics CSV (e.g., ccsp_lipid_filtering_results.csv) with retention/filtering percentages

## How to apply

Download the 'CCSP 2.0 - CCS Prediction in Jupyter' notebook (.ipynb) from github.com/facundof2016/CCSP2.0 (requires sklearn ≥1.0.2; see requirements.txt) and open it in Jupyter Lab. Load a superclass-matched training set — e.g., 93 lipid [M−H]− CCS measurements from the McLean CCS Compendium — compute the 1613 Mordred molecular descriptors for each training structure, and fit the linear support vector regression model; verify the median relative CCS prediction error is ≤1.25% for deprotonated adducts. Retrieve candidate structures from PubChem within a 10 ppm mass-error window (e.g., 4980 candidates for leukotriene E4 [M−H]−), predict their CCS values with the trained SVR model, then apply a ΔCCS threshold of 2.8% to filter candidates. Confirm correct annotations are retained (100% retention expected) while a substantial fraction of incorrect structures is removed (36.1% class-matched filtering reported), and write filtered candidate lists plus filtering statistics to a CSV output.

## Related tools

- **CCSP 2.0** (Open-source Jupyter notebook implementing linear SVR CCS prediction from user-curated training sets; used to train the lipid model and filter candidates) — https://github.com/facundof2016/CCSP2.0
- **Jupyter Notebook** (Execution environment for the locally hosted CCSP 2.0 notebook (Jupyter Lab compatible, with Tk interface)) — https://github.com/facundof2016/CCSP2.0
- **Mordred** (Python package embedded in the notebook used to compute the 1613 molecular descriptors for training and candidate structures)

## Examples

```
jupyter lab "CCSP 2.0 - CCS Prediction in Jupyter.ipynb"  # load 93 lipid [M−H]− McLean Compendium CCS values, fit linear SVR, filter PubChem candidates at ΔCCS 2.8%
```

## Evaluation signals

- Median relative CCS prediction error of the trained lipid [M−H]− model is ≤1.25% on test measurements
- After ΔCCS 2.8% filtering at 10 ppm mass error, 100% of correct annotations are retained (leukotriene E4 benchmark: 4980 → 806 candidates remaining)
- Class-matched filtering removes ~36.1% of incorrect structures; isomeric filtering ~19.3%
- AUC scores of the filtering step fall around 0.75–0.78
- Filtered candidate lists and statistics are written to the expected CSV output

## Limitations

- Training set must be defined before predicting CCS values — the model is tailored to the chosen superclass and cannot generalize freely
- Current training sets cover common adducts ([M−H]−, [M+H]+, [M+Na]+, [M+K]+) but not dimers such as [2M+H]+ or [2M−H]−, or other less common adducts or in-source fragments
- Google Colab operation is limited to ~12 hours continuous and disconnects after ~20 minutes idle; requires sklearn 1.0.2 or later

## Evidence
