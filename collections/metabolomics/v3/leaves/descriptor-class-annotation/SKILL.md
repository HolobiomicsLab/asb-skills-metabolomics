---
name: descriptor-class-annotation
description: Use when inspecting or interpreting a CCSP 2.0 SVR model to understand
  which Mordred descriptor classes (ATS, Barysz, Kier, TIC, constitutional) were selected
  by RFE and dominate the model, or when building a user-curated training set and
  you need to justify feature selection for a given adduct.
when_to_use_negative:
- When the input is a raw IM-MS ion mobility spectrum or a peak list rather than a
  trained SVR model — descriptor annotation only applies to fitted linear SVR models
  from CCSP 2.0, not to unprocessed data.
- When the goal is to predict CCS values for new molecules rather than interpret an
  existing model — the CCSP 2.0 prediction path (colab or Jupyter Lab notebook) should
  be used instead.
- When training sets contain dimers such as [2M + H]+, [2M − H]−, or other less common
  adducts or in-source fragments — the RFE-selected models were built only on [M−H]−,
  [M+H]+, [M+Na]+, and [M+K]+ adduct types.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3520
tools:
- name: CCSP 2.0
  role: Hosts the trained linear SVR models for the three adduct ion types; the notebook's
    embedded code is opened to extract RFE-selected descriptor subsets and their feature
    weights.
  repo: https://github.com/facundof2016/CCSP2.0
- name: Jupyter Notebook
  role: Runtime environment (Jupyter Lab or Jupyter Notebook, or Google Colab) in
    which the CCSP 2.0 .ipynb is executed to inspect models and compute descriptor-class
    rankings.
  repo: https://github.com/facundof2016/CCSP2.0
- name: Mordred
  role: Python package that supplies the 1613 molecular descriptors from which the
    RFE-selected subsets are drawn and mapped to descriptor classes (ATS, Barysz,
    Kier, TIC, constitutional, etc.).
  repo: https://github.com/facundof2016/CCSP2.0
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.2c03491
    title: 'CCS Predictor 2.0: An Open-Source Jupyter Notebook Tool for Filtering
      Out False Positives in Metabolomics'
  source_content_hash: sha256:9c2fb8200e151f10f13a2164ddb7af451e1d10a1c6002e2f98a1285efc2eefb6
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/facundof2016/CCSP2.0
  tools:
  - CCSP 2.0
  - Jupyter Notebook
  - Mordred
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# descriptor-class-annotation

## Summary

Annotate the molecular descriptors retained by recursive feature elimination in CCSP 2.0's linear SVR CCS prediction models ([M−H]−, [M+H]+, [M+Na]+ adducts) by class, and rank classes by feature-weight magnitude to interpret which molecular properties drive CCS predictions.

## When to use

Use when inspecting or interpreting a CCSP 2.0 SVR model to understand which Mordred descriptor classes (ATS, Barysz, Kier, TIC, constitutional) were selected by RFE and dominate the model, or when building a user-curated training set and you need to justify feature selection for a given adduct ion type.

## When NOT to use

- When the input is a raw IM-MS ion mobility spectrum or a peak list rather than a trained SVR model — descriptor annotation only applies to fitted linear SVR models from CCSP 2.0, not to unprocessed data.
- When the goal is to predict CCS values for new molecules rather than interpret an existing model — the CCSP 2.0 prediction path (colab or Jupyter Lab notebook) should be used instead.
- When training sets contain dimers such as [2M + H]+, [2M − H]−, or other less common adducts or in-source fragments — the RFE-selected models were built only on [M−H]−, [M+H]+, [M+Na]+, and [M+K]+ adduct types.

## Inputs

- Trained linear SVR model(s) for [M−H]−, [M+H]+, and/or [M+Na]+ adducts (embedded in the CCSP 2.0 .ipynb notebook)
- Mordred descriptor library of 1613 molecular descriptors (from the Mordred Python package, referenced in the notebook)

## Outputs

- descriptor_class_rankings.csv — per-adduct RFE-selected descriptor counts (expected ~370 / ~303 / ~414) and descriptor classes ranked by |feature weight|
- Ranked list of dominant descriptor classes per adduct ion type (ATS, Barysz matrix, Kier, TIC, constitutional, etc.)

## How to apply

Open the CCSP 2.0 Jupyter notebook (.ipynb) from the facundof2016/CCSP2.0 GitHub repository in a Python 3 Jupyter Lab or Jupyter Notebook environment. Load the trained linear SVR models for the [M−H]−, [M+H]+, and [M+Na]+ adducts together with the Mordred library of 1613 molecular descriptors embedded in the notebook. Extract the RFE-selected descriptor subsets for each adduct model and compute the per-adduct descriptor counts — the expected averages are 370, 303, and 414 descriptors respectively. Then rank descriptor classes (e.g., ATS, Barysz matrix, Kier, TIC, constitutional) by the magnitude of their SVR feature weights within each model to identify the dominant classes. Write the per-adduct counts and class rankings to descriptor_class_rankings.csv for downstream review.

## Related tools

- **CCSP 2.0** (Hosts the trained linear SVR models for the three adduct ion types; the notebook's embedded code is opened to extract RFE-selected descriptor subsets and their feature weights.) — https://github.com/facundof2016/CCSP2.0
- **Jupyter Notebook** (Runtime environment (Jupyter Lab or Jupyter Notebook, or Google Colab) in which the CCSP 2.0 .ipynb is executed to inspect models and compute descriptor-class rankings.) — https://github.com/facundof2016/CCSP2.0
- **Mordred** (Python package that supplies the 1613 molecular descriptors from which the RFE-selected subsets are drawn and mapped to descriptor classes (ATS, Barysz, Kier, TIC, constitutional, etc.).) — https://github.com/facundof2016/CCSP2.0

## Examples

```
# In a Python 3 Jupyter environment with CCSP 2.0 notebook open:
from ccsp2_0 import load_models, extract_rfe_descriptors, rank_classes
models = load_models('[M-H]-', '[M+H]+', '[M+Na]+')
ranked = {a: rank_classes(models[a]) for a in models}
ranked['[M-H]-'].head()
```

## Evaluation signals

- Per-adduct RFE-selected descriptor counts must match the reported averages of 370, 303, and 414 descriptors for [M−H]−, [M+H]+, and [M+Na]+ respectively, out of the 1613 Mordred descriptors.
- Dominant descriptor classes per adduct must reproduce the paper's reported classes: ATS, Barysz matrix, Kier, TIC, and constitutional descriptors encoding shape, symmetry, and bulk properties.
- Descriptor-class rankings must be sorted by descending magnitude of the linear SVR feature weights within each adduct model.
- descriptor_class_rankings.csv must contain rows for all three adduct ion types with both the count and the class ranking populated.
- Notebook execution must complete without error in a Python 3 environment with sklearn >= 1.0.2, as stated in the repository requirements.

## Limitations

- CCSP 2.0's current training sets cover only the most common adduct ion types — [M−H]−, [M+H]+, [M+Na]+, and [M+K]+ — so descriptor annotations cannot be generated for dimers like [2M + H]+, [2M − H]−, or other less common adducts or in-source fragments.
- Interpretation is limited to linear SVR feature weights; descriptor-class dominance reflects the models as trained on the McLean CCS Compendium training sets and may not generalize to user-curated training sets without re-running RFE.
- One drawback of most ML methods is the need to define the specific training set to be used prior to predicting CCS values, so any change in training data will change the RFE-selected descriptor subsets and their class rankings.

## Evidence

- [results] RFE-selected on average 370, 303, and 414 descriptors for the three adduct ion types: [evidence span withheld — anchor status: not_found]
- [results] dominant descriptor classes: [evidence span withheld — anchor status: not_found]
