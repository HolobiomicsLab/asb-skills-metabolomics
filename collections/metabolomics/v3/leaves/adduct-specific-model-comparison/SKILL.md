---
name: adduct-specific-model-comparison
description: Use when you have trained CCSP 2.0 linear SVR models for multiple adduct
  ion types and need to understand which Mordred descriptor classes and counts each
  adduct-specific model relies on.
when_to_use_negative:
- Your goal is CCS value prediction rather than model introspection — CCSP 2.0's prediction
  workflow is a different skill.
- The training set has not yet been defined; adduct-specific models require a curated
  per-adduct training set before descriptor analysis is meaningful.
edam_operation: http://edamontology.org/operation_0314
edam_topics:
- http://edamontology.org/topic_3473
- http://edamontology.org/topic_0091
tools:
- name: CCSP 2.0
  role: host notebook containing the trained linear SVR models and RFE-selected descriptor
    subsets being compared
  repo: https://github.com/facundof2016/CCSP2.0
- name: Mordred
  role: Python package providing the 1613 molecular descriptors from which the adduct-specific
    RFE subsets are drawn
- name: Jupyter Notebook
  role: execution environment for opening the .ipynb notebook and running the comparison
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.2c03491
    title: 'CCS Predictor 2.0: An Open-Source Jupyter Notebook Tool for Filtering
      Out False Positives in Metabolomics'
  source_content_hash: sha256:66c84c9dd08c5ca11758455b79fe8fd0d9f5872cef3bacd7adb0073c04f155fd
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/facundof2016/CCSP2.0
  tools:
  - CCSP 2.0
  - Jupyter Notebook
  - Mordred
  edam_operation: http://edamontology.org/operation_0314
  edam_topics:
  - http://edamontology.org/topic_3473
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# adduct-specific-model-comparison

## Summary

Compares the per-adduct descriptor counts and feature-weight rankings across the [M−H]−, [M+H]+, and [M+Na]+ linear SVR models in CCSP 2.0 to reveal which Mordred descriptor classes (ATS, Barysz, Kier, TIC, constitutional) drive CCS predictions for each ion type.

## When to use

Use when you have trained CCSP 2.0 linear SVR models for multiple adduct ion types and need to understand which Mordred descriptor classes and counts each adduct-specific model relies on.

## When NOT to use

- Your goal is CCS value prediction rather than model introspection — CCSP 2.0's prediction workflow is a different skill.
- The training set has not yet been defined; adduct-specific models require a curated per-adduct training set before descriptor analysis is meaningful.

## Inputs

- CCSP 2.0 trained linear SVR models for [M−H]−, [M+H]+, and [M+Na]+ adducts
- Mordred descriptor library of 1613 molecular descriptors embedded in the notebook
- CCSP 2.0 Jupyter notebook (.ipynb) from github.com/facundof2016/CCSP2.0

## Outputs

- descriptor_class_rankings.csv with per-adduct RFE-selected descriptor counts and descriptor-class weight rankings

## How to apply

Open the CCSP 2.0 Jupyter notebook from facundof2016/CCSP2.0 and load the trained linear SVR models for the three adducts together with the Mordred descriptor library of 1613 molecular descriptors. Extract the recursive feature elimination (RFE)-selected descriptor subsets for each adduct and compute per-adduct descriptor counts, verifying against the reported averages of 370 ([M−H]−), 303 ([M+H]+), and 414 ([M+Na]+) descriptors. Then rank descriptor classes by the magnitude of their SVR feature weights within each model; the dominant classes should reproduce the paper's reported shape/symmetry/bulk-property descriptors (ATS, Barysz matrix, Kier, TIC, constitutional). Write per-adduct counts and class rankings to descriptor_class_rankings.csv.

## Related tools

- **CCSP 2.0** (host notebook containing the trained linear SVR models and RFE-selected descriptor subsets being compared) — https://github.com/facundof2016/CCSP2.0
- **Mordred** (Python package providing the 1613 molecular descriptors from which the adduct-specific RFE subsets are drawn)
- **Jupyter Notebook** (execution environment for opening the .ipynb notebook and running the comparison)

## Examples

```
python -c "import joblib; models = {a: joblib.load(f'{a}_svr.joblib') for a in ['[M-H]-','[M+H]+','[M+Na]+']}; print({a: len(m.coef_.ravel()[m.coef_.ravel()!=0]) for a,m in models.items()})"
```

## Evaluation signals

- RFE-selected descriptor averages match the reported 370, 303, and 414 for [M−H]−, [M+H]+, and [M+Na]+ respectively
- Dominant descriptor classes include ATS, Barysz matrix, Kier, TIC, and constitutional descriptors encoding shape, symmetry, and bulk properties
- Only descriptors from the 1613 Mordred library appear in the extracted subsets
- Output file descriptor_class_rankings.csv is produced with per-adduct counts and class rankings

## Limitations

- Current training sets allow prediction of the most common adduct ion types ([M−H]−, [M+H]+, [M+Na]+, [M+K]+) but not dimers such as [2 M + H]+ or other less common adducts or in-source fragments
- One drawback of most ML methods is the need to define the specific training set prior to predicting CCS values, limiting cross-adduct generality

## Evidence

- [results] RFE-selected descriptor counts: [evidence span withheld — anchor status: not_found]
- [results] dominant descriptor classes: [evidence span withheld — anchor status: not_found]
- [discussion] model comparison workflow: "The CCS prediction median relative errors were 1.25, 1.73, and 1.87% for [M − H]−, [M + H]+, and [M + Na]+ adducts using linear SVR models"
- [discussion] adduct limitation: "Current training sets allow the prediction of the most common adduct ion types including [M − H]−, [M + H]+, [M + Na]+, and [M + K]+ but not dimers such as [2 M + H]+ or [2 M − H]−, or other less"
- [readme] tool description: "Collision Cross Section Predictor 2.0 is an open source Python notebook intended to help ion-mobility scientists predict collision cross sections with user-curated training sets"
