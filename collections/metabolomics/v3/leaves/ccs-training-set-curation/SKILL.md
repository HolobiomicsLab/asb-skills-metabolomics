---
name: ccs-training-set-curation
description: Use when you need CCS predictions for an adduct or compound class not
  covered by CCSP 2.0's shipped models (e.g., [2M+H]+, [2M−H]−, [M+K]+ beyond default
  coverage, or a specific superclass), and you can assemble experimental CCS values
  into a curated training table to rebuild the SVR model.
when_to_use_negative:
- The target adduct is already covered by a shipped CCSP 2.0 model ([M − H]−, [M +
  H]+, [M + Na]+, [M + K]+) with sufficient accuracy — in that case use the published
  models rather than retraining.
- You lack a defensible training set of experimentally measured CCS values for the
  target adduct or class; ML CCS prediction fundamentally requires that 'the specific
  training set to be used prior to predicting CCS values' be defined first, so retraining
  cannot proceed without curated measurements.
- You are predicting CCS for the common adducts under production filtering workflows
  where a ΔCCS threshold of 2.8% and 10 ppm mass error were validated — a custom retrain
  on a small set may not reproduce that validated filtering performance.
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_3336
tools:
- name: CCSP 2.0
  role: Open-source Jupyter notebook that trains linear SVR CCS prediction models
    from user-curated training sets and predicts CCS values; requires Sklearn 1.0.2
    or later
  repo: https://github.com/facundof2016/CCSP2.0
- name: Mordred
  role: Python package used inside CCSP 2.0 to compute the large library of 1613 molecular
    descriptors from training/target compound structures
- name: Jupyter Notebook
  role: Execution environment for the CCSP 2.0 notebook (Google Colab for small-scale
    predictions <10,000 molecules; local Jupyter Lab for large training/target sets
    or workflow integration)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.2c03491
    title: 'CCS Predictor 2.0: An Open-Source Jupyter Notebook Tool for Filtering
      Out False Positives in Metabolomics'
  source_content_hash: sha256:1dbceedf3cf63091a1d61e91b5ea7ee5d7593d17d82c80079220b96f61500513
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/facundof2016/CCSP2.0
  tools:
  - CCSP 2.0
  - Jupyter Notebook
  - Mordred
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_3336
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# ccs-training-set-curation

## Summary

Curates and uses a user-defined training set of experimental CCS values to train/retrain CCSP 2.0's linear SVR models so CCS predictions cover adduct types or molecular classes absent from the published [M − H]−, [M + H]+, and [M + Na]+ models (e.g., dimers such as [2M + H]+ or [2M − H]−). This operationalizes CCSP 2.0's design goal of letting users 'customize their own CCS predictions based on their unique training sets and workflow needs'.

## When to use

Use when you need CCS predictions for an adduct or compound class not covered by CCSP 2.0's shipped models (e.g., [2M+H]+, [2M−H]−, [M+K]+ beyond default coverage, or a specific superclass), and you can assemble experimental CCS values into a curated training table to rebuild the SVR model.

## When NOT to use

- The target adduct is already covered by a shipped CCSP 2.0 model ([M − H]−, [M + H]+, [M + Na]+, [M + K]+) with sufficient accuracy — in that case use the published models rather than retraining.
- You lack a defensible training set of experimentally measured CCS values for the target adduct or class; ML CCS prediction fundamentally requires that 'the specific training set to be used prior to predicting CCS values' be defined first, so retraining cannot proceed without curated measurements.
- You are predicting CCS for the common adducts under production filtering workflows where a ΔCCS threshold of 2.8% and 10 ppm mass error were validated — a custom retrain on a small set may not reproduce that validated filtering performance.

## Inputs

- User-curated training set table of compounds with experimentally measured CCS values for the target adduct (paired with exact mass), e.g. sourced from the McLean CCS Compendium
- Molecular structures (e.g., SMILES/InChI retrieved from PubChem) for each training compound, for Mordred descriptor computation
- Optional target compound list (small-scale (<10,000 molecules) for Google Colab; larger sets for local Jupyter Lab) whose CCS values should be predicted by the newly trained model

## Outputs

- A trained linear SVR model tailored to the curated adduct/compound class, persisted inside the CCSP 2.0 notebook environment
- Predicted CCS values for target compounds under the custom adduct type
- Cross-validation and held-out test-set performance statistics, specifically median relative error per adduct, comparable to the published 1.25–1.87% benchmarks

## How to apply

First assemble the training set: retrieve experimental CCS measurements for the target adduct (e.g., from the McLean CCS Compendium) and compile them with compound identifiers into a curated training table with paired masses and CCS values. Retrieve molecular structures (e.g., from PubChem) for the training compounds and compute the full library of 1613 molecular descriptors using the Mordred Python package inside the CCSP 2.0 Jupyter notebook. Train the linear SVR model on this user-curated training set following the notebook's workflow, evaluating under both cross-validation and held-out test-set validation conditions. Report the median relative error per adduct and judge success against the paper's own benchmarks — median relative errors of 1.25–1.87% for [M − H]−, [M + H]+, and [M + Na]+ — rather than absolute thresholds, since accuracy depends on the size and quality of the curated set. Note one key constraint of the shipped models that motivates this skill: 'Current training sets allow the prediction of the most common adduct ion types including [M − H]−, [M + H]+, [M + Na]+, and [M + K]+ but not dimers such as [2 M + H]+ or [2 M − H]−, or other less common adducts or in-source fragments.'

## Related tools

- **CCSP 2.0** (Open-source Jupyter notebook that trains linear SVR CCS prediction models from user-curated training sets and predicts CCS values; requires Sklearn 1.0.2 or later) — https://github.com/facundof2016/CCSP2.0
- **Mordred** (Python package used inside CCSP 2.0 to compute the large library of 1613 molecular descriptors from training/target compound structures)
- **Jupyter Notebook** (Execution environment for the CCSP 2.0 notebook (Google Colab for small-scale predictions <10,000 molecules; local Jupyter Lab for large training/target sets or workflow integration))

## Examples

```
Open "CCSP 2.0 - CCS Prediction in Jupyter" (.ipynb) in Jupyter Lab and follow the embedded instructions: curate the training set, compute Mordred descriptors (1613), and train the linear SVR for the custom adduct.
```

## Evaluation signals

- Cross-validation median relative error on the curated training set should be reported and, for common monomer adducts ([M − H]−, [M + H]+, [M + Na]+), should approach the paper's benchmark of 1.25–1.87% (< 2%).
- Held-out test-set predictions must show median relative error consistent with cross-validation — 'Extensive testing of the CCS prediction accuracy was conducted under both cross-validation and test set validation conditions'.
- Descriptor coverage: all 1613 Mordred descriptors computed without failures for every training compound; missing structures or descriptor failures must be resolved before SVR training.
- Prediction quality check: predicted CCS for held-out compounds of the custom adduct should fall within the ΔCCS filtering tolerance that would be applied downstream (the paper uses a 2.8% ΔCCS threshold for filtering with 10 ppm mass error).
- Environment check: Sklearn version must be 1.0.2 or later, per the README, before running the notebook.

## Limitations

- One drawback of most ML methods is the need to define the specific training set to be used prior to predicting CCS values — accuracy is bounded by the quality and size of the curated set.
- Current published training sets do not allow prediction of dimers such as [2M + H]+ or [2M −H]−, or other less common adducts or in-source fragments; custom retraining is the intended workaround but inherits its own data limitations.
- Google Colab limits continuous notebook operation to up to 12 hours and disconnects after ~20 minutes idle, so large training/target sets should use the locally hosted Jupyter Lab version.
- Future work in the paper notes exploration of CCS predictions across multiple IM-MS instrument types (drift tube vs traveling wave IM platforms); instrument-specific variance in CCS values is not fully characterized.
- The paper's published benchmarks (1.25–1.87% median relative error) were achieved on 170–138 compounds per common adduct; a user-curated set for an uncommon adduct may be smaller and less accurate.

## Evidence

- [readme] If you plan to predict CCS values using large training or target sets or if you plan to integrate the code into a larger analysis workflow, the locally hosted Jupyter Lab version is recommended.: [evidence span withheld — anchor status: external_doc]
