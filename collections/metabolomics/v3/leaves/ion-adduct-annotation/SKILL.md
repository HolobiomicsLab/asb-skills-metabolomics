---
name: ion-adduct-annotation
description: Use when IM-MS data contains adduct types (e.g. [M−H]−, [M+H]+, [M+Na]+,
  or uncommon dimers like [2M+H]+ / [2M−H]−) for which CCS predictions are needed
  to filter or annotate candidate structures.
when_to_use_negative:
- Input is already a curated, adduct-matched CCS dataset — CCSP 2.0's published models
  are only built for [M−H]−, [M+H]+, [M+Na]+, and [M+K]+; other adducts require user-curated
  training sets.
- The analysis goal is multi-adduct CCS filtering or fragment-ion CCS prediction in
  tandem MS; the paper lists these as future work, not supported by the current tool.
edam_operation: http://edamontology.org/operation_3767
edam_topics:
- http://edamontology.org/topic_0157
- http://edamontology.org/topic_3520
tools:
- name: CCSP 2.0
  role: Primary tool providing the open-source Jupyter notebook for building user-curated
    linear SVR CCS prediction models and computing predictions for target adduct types.
  repo: https://github.com/facundof2016/CCSP2.0
- name: Mordred
  role: Python package used inside the CCSP 2.0 notebook to compute the 1613 molecular
    descriptors that serve as features for SVR training.
- name: Jupyter Notebook
  role: Execution environment in which the CCSP 2.0 workflow (descriptor calculation,
    SVR training, CCS prediction) is run.
- name: Google Colab
  role: Recommended deployment route for small-scale predictions (<10,000 molecules);
    provides hosted Python execution without local installation.
- name: PubChem
  role: Source of molecular structures for training compounds; chosen over focused
    databases like HMDB to test CCS filtering under adverse conditions.
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.2c03491
    title: 'CCS Predictor 2.0: An Open-Source Jupyter Notebook Tool for Filtering
      Out False Positives in Metabolomics'
  source_content_hash: sha256:f25b09f6cc4a4ee49527cf862805aa9c7f1cade8c74ff716906b305bc481ce7a
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/facundof2016/CCSP2.0
  tools:
  - CCSP 2.0
  - Google Colab
  - Jupyter Notebook
  - Mordred
  - PubChem
  edam_operation: http://edamontology.org/operation_3767
  edam_topics:
  - http://edamontology.org/topic_0157
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# ion-adduct-annotation

## Summary

A method for annotating ion-mobility–derived CCS values to candidate ion adducts using CCSP 2.0's linear SVR models, including the ability to build user-curated training sets for adduct types like dimers ([2M+H]+, [2M−H]−) not covered by the published [M−H]−, [M+H]+, and [M+Na]+ models.

## When to use

Use when IM-MS data contains adduct types (e.g. [M−H]−, [M+H]+, [M+Na]+, or uncommon dimers like [2M+H]+ / [2M−H]−) for which CCS predictions are needed to filter or annotate candidate structures.

## When NOT to use

- Input is already a curated, adduct-matched CCS dataset — CCSP 2.0's published models are only built for [M−H]−, [M+H]+, [M+Na]+, and [M+K]+; other adducts require user-curated training sets.
- The analysis goal is multi-adduct CCS filtering or fragment-ion CCS prediction in tandem MS; the paper lists these as future work, not supported by the current tool.

## Inputs

- Experimental CCS measurements from the McLean CCS Compendium for the target adduct type
- Molecular structures for training compounds (e.g. from PubChem)
- .ipynb notebook file (CCSP 2.0 - CCS Prediction in Google Colab or Jupyter)
- User-curated training set table (adduct type, CCS values, compound identifiers)

## Outputs

- Trained linear SVR model for the target adduct type
- Predicted CCS values for candidate structures
- Median relative error metrics under cross-validation and test-set validation conditions

## How to apply

Retrieve experimental CCS measurements for the target adduct from the McLean CCS Compendium and assemble a user-curated training set table, leveraging CCSP 2.0's design that lets users rebuild linear SVR models from their own training sets. Retrieve molecular structures (e.g. from PubChem) for training compounds and compute the 1613 Mordred molecular descriptors inside the CCSP 2.0 Jupyter notebook. Train the linear SVR model on the curated training set following the notebook's embedded instructions, then evaluate prediction accuracy under both cross-validation and held-out test-set conditions, computing median relative error. Benchmark against the published CCSP 2.0 accuracy of median relative errors of 1.25–1.87% for [M−H]−, [M+H]+, and [M+Na]+ adducts. For adduct types not covered by the published models (e.g. [2M+H]+ or [2M−H]−), the paper explicitly notes that current training sets do not allow such predictions, so a custom model must be built via the user-curated training-set workflow.

## Related tools

- **CCSP 2.0** (Primary tool providing the open-source Jupyter notebook for building user-curated linear SVR CCS prediction models and computing predictions for target adduct types.) — https://github.com/facundof2016/CCSP2.0
- **Mordred** (Python package used inside the CCSP 2.0 notebook to compute the 1613 molecular descriptors that serve as features for SVR training.)
- **Jupyter Notebook** (Execution environment in which the CCSP 2.0 workflow (descriptor calculation, SVR training, CCS prediction) is run.)
- **Google Colab** (Recommended deployment route for small-scale predictions (<10,000 molecules); provides hosted Python execution without local installation.)
- **PubChem** (Source of molecular structures for training compounds; chosen over focused databases like HMDB to test CCS filtering under adverse conditions.)

## Examples

```
"CCSP 2.0 - CCS Prediction for Google Colab.ipynb" upload via Google Colab File > Open Notebook > Upload; or open "CCSP 2.0 - CCS Prediction in Jupyter.ipynb" in Jupyter Lab and follow embedded instructions.
```

## Evaluation signals

- Median relative error of CCS predictions should be compared against the CCSP 2.0 benchmark of 1.25–1.87% for [M−H]−, [M+H]+, and [M+Na]+ adducts.
- Model performance must be reported under both cross-validation and held-out test-set validation conditions, mirroring the paper's extensive testing workflow.
- For adduct types not covered by the published models (e.g. [2M+H]+), the training set must be explicitly user-curated from sources like the McLean CCS Compendium, not reused from the published models.
- Mordred descriptor set should contain 1613 features per molecule, matching the paper's descriptor library size.
- Notebook version requires scikit-learn V1.0.2 or later; verify the installed version before training.

## Limitations

- Current training sets allow prediction of the most common adduct ion types ([M−H]−, [M+H]+, [M+Na]+, [M+K]+) but not dimers such as [2M+H]+ or [2M−H]−, or other less common adducts or in-source fragments.
- One drawback of most ML methods is the need to define the specific training set prior to predicting CCS values.
- Future work includes exploring CCS predictions across multiple IM-MS instrument types (e.g. drift tube vs traveling wave IM platforms) and the effect of experimental sources of variance on the CCS threshold.
- No changelog is present in the repository metadata.
- The paper notes that care must be taken when interpreting results for diverse data sets with highly accurate mass measurements, as currently available tools afford minimal orthogonal information in such cases.

## Evidence

- [intro] CCSP 2.0 is illustrated using 463 test CCS measurements retrieved from the McLean CCS Compendium, resulting in relative median errors lower than 2% for sodium adducts and protonated and deprotonated molecules.: [evidence span withheld — anchor status: not_found]
