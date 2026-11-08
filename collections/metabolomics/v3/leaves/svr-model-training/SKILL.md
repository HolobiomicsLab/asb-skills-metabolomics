---
name: svr-model-training
description: Use when you need to fit a customized CCS prediction model from a curated
  training set of ion-mobility measurements (e.g. [M−H]−, [M+H]+, [M+Na]+ ions) before
  predicting or filtering candidate structures by ΔCCS threshold.
when_to_use_negative:
- You need to predict dimers such as [2M+H]+ or [2M−H]−, or other less common adducts
  or in-source fragments — the current CCSP 2.0 training sets only cover [M−H]−, [M+H]+,
  [M+Na]+, and [M+K]+
- Your training set is empty or not tailored to the data of interest — the article
  notes a drawback of ML methods is the need to define a specific training set prior
  to prediction
- The task requires drift-tube vs traveling-wave instrument transferability — the
  article lists CCS prediction across multiple IM-MS instrument types as future work,
  not current capability
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3336
tools:
- name: CCSP 2.0
  role: Open-source Jupyter notebook that packages the linear SVR training workflow
    and Mordred descriptor computation for user-curated CCS training sets
  repo: https://github.com/facundof2016/CCSP2.0
- name: Mordred
  role: Python package used inside CCSP 2.0 to compute the library of 1613 molecular
    descriptors that serve as SVR input features
- name: Jupyter Notebook
  role: Execution environment for the CCSP 2.0 notebook (Google Colab or Jupyter Lab
    variant)
  repo: https://jupyter.org/
provenance:
  source_task_ids:
  - task_002
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.2c03491
    title: 'CCS Predictor 2.0: An Open-Source Jupyter Notebook Tool for Filtering
      Out False Positives in Metabolomics'
  source_content_hash: sha256:1b0197916f6f36ba8d32b99f31202ed04deddadb207e47166d868c1f163068eb
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/facundof2016/CCSP2.0
  tools:
  - AllCCS
  - CCSBase
  - CCSP 2.0
  - DeepCCS
  - Jupyter Notebook
  - Mordred
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3336
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 17
---

# svr-model-training

## Summary

Trains a linear support vector regression model on user-curated CCS training measurements (e.g. McLean CCS Compendium adduct subsets) to predict collision cross sections for metabolite/lipid annotation filtering, as implemented in the CCSP 2.0 open-source Jupyter notebook.

## When to use

Use when you need to fit a customized CCS prediction model from a curated training set of ion-mobility measurements (e.g. [M−H]−, [M+H]+, [M+Na]+ ions) before predicting or filtering candidate structures by ΔCCS threshold.

## When NOT to use

- You need to predict dimers such as [2M+H]+ or [2M−H]−, or other less common adducts or in-source fragments — the current CCSP 2.0 training sets only cover [M−H]−, [M+H]+, [M+Na]+, and [M+K]+
- Your training set is empty or not tailored to the data of interest — the article notes a drawback of ML methods is the need to define a specific training set prior to prediction
- The task requires drift-tube vs traveling-wave instrument transferability — the article lists CCS prediction across multiple IM-MS instrument types as future work, not current capability

## Inputs

- Curated training set of CCS measurements with ion adduct labels (e.g. McLean CCS Compendium subsets: [M−H]−, [M+H]+, [M+Na]+)
- Candidate structure identifiers for descriptor computation (e.g. PubChem entries)
- CCSP 2.0 notebook file (.ipynb) and requirements.txt specifying package versions

## Outputs

- Trained linear SVR CCS prediction model for the chosen adduct/superclass
- Predicted CCS values for target molecules
- Benchmark table of median relative error and RMSE per adduct across test splits (e.g. ccs_prediction_benchmark_table.csv)

## How to apply

Curate a training set of CCS measurements for the adduct type of interest (CCSP 2.0 uses 463 test measurements from the McLean CCS Compendium: 170 [M−H]−, 155 [M+H]+, 138 [M+Na]+ ions) and stratify by chemical superclass when the goal is annotation filtering (e.g. 93 lipid [M−H]− ions for the superclass-matched lipid model). Compute 1613 molecular descriptors per structure with the Mordred Python package inside the CCSP 2.0 notebook, then fit a linear support vector regression model to the descriptor-CCS pairs using scikit-learn (the README notes the notebook requires Sklearn V1.0.2 or later). Choose the notebook variant by scale: the Google Colab version suits small predictions (<10,000 molecules), while the locally hosted Jupyter Lab notebook with Tk interface supports large training/target sets and workflow integration. Before deployment, benchmark the fitted model against competitors (CCSbase, AllCCS, DeepCCS) on identical randomly sampled test splits — e.g. 101 splits of 30% test data — and confirm the median relative error stays near the reported values (1.25% [M−H]−, 1.73% [M+H]+, 1.87% [M+Na]+, all below 2%).

## Related tools

- **CCSP 2.0** (Open-source Jupyter notebook that packages the linear SVR training workflow and Mordred descriptor computation for user-curated CCS training sets) — https://github.com/facundof2016/CCSP2.0
- **Mordred** (Python package used inside CCSP 2.0 to compute the library of 1613 molecular descriptors that serve as SVR input features)
- **Jupyter Notebook** (Execution environment for the CCSP 2.0 notebook (Google Colab or Jupyter Lab variant)) — https://jupyter.org/

## Examples

```
In Google Colab: File > Open Notebook > Upload > Click "Choose File" > Select "CCSP 2.0 - CCS Prediction for Google Colab.ipynb".
```

## Evaluation signals

- Median relative CCS prediction error ≤ ~1.25% for [M−H]−, ~1.73% for [M+H]+, and ~1.87% for [M+Na]+ on held-out test data (all below the reported 2% ceiling)
- On 101 randomly selected 30% test splits held identical across algorithms, CCSP 2.0 ranks equal to or better than CCSbase, AllCCS, and DeepCCS for most splits (e.g. [M−H]− median MRE ranking CCSP 1.251% < CCSbase 1.427% < AllCCS 1.855% < DeepCCS 1.871%)
- For a superclass-matched lipid [M−H]− model, filtering at ΔCCS 2.8% and 10 ppm mass error retains 100% of correct annotations while removing ~36.1% of incorrect class-matched structures
- Environment check: scikit-learn version ≥ 1.0.2 per requirements.txt before running the notebook

## Limitations

- Requires Sklearn V1.0.2 or later (README)
- Training set must be defined prior to prediction; performance depends on how well the curated training set matches the target chemical space
- Current training sets do not support prediction of dimers ([2M+H]+, [2M−H]−) or less common adducts/in-source fragments
- Google Colab limits continuous operation to ~12 hours and disconnects after ~20 min idle; use the local Jupyter Lab version for large training/target sets
- Cross-instrument transferability (drift-tube vs traveling-wave IM) and multi-adduct/fragment-ion CCS prediction remain future work

## Evidence
