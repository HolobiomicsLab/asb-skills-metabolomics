---
name: metabolite-candidate-annotation-filtering
description: Use when you have an experimental CCS value for a metabolite adduct (e.g.
  [M−H]−, [M+H]+, [M+Na]+) from ion-mobility MS, plus a list of formula-matching database
  candidates (e.
when_to_use_negative:
- Your input is a feature table or peak list without any experimental CCS value —
  CCS filtering requires a measured drift-time-derived CCS to compare against predictions.
- Your candidate structures need CCS prediction for adducts or species outside the
  training scope, e.g. dimers such as [2M+H]+ or [2M−H]−, other less common adducts,
  or in-source fragments — the article explicitly notes current training sets do not
  cover these.
- You need a fully class-matched or superclass-matched model without tailoring — the
  article notes a drawback of ML methods is the need to define the specific training
  set prior to prediction, and superclass-matched models improve confidence.
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: CCSP 2.0
  role: Open-source Jupyter Notebook tool implementing linear SVR models on Mordred
    descriptors to predict CCS and filter candidate annotations
  repo: https://github.com/facundof2016/CCSP2.0
- name: Mordred
  role: Python package used to compute the 1613 molecular descriptors fed into the
    SVR CCS prediction models
- name: Jupyter Notebook / Jupyter Lab
  role: Execution environment for the CCSP 2.0 notebook (Google Colab variant recommended
    for <10,000 molecules; Jupyter Lab for large sets/workflow integration)
  repo: https://github.com/facundof2016/CCSP2.0
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.2c03491
    title: 'CCS Predictor 2.0: An Open-Source Jupyter Notebook Tool for Filtering
      Out False Positives in Metabolomics'
  source_content_hash: sha256:39471ec13c9f30090be9daeb7389b6d5966677ccd76e893017c36ecd9e4cab80
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/facundof2016/CCSP2.0
  tools:
  - BiomiX
  - CCSP 2.0
  - CEU Mass Mediator
  - Jupyter Notebook / Jupyter Lab
  - Mordred
  - R
  - TidyMass
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
merged_aliases:
- metabolite-annotation-candidate-filtering
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# metabolite-candidate-annotation-filtering

## Summary

Use CCS Predictor 2.0 (CCSP 2.0), an open-source Jupyter Notebook implementing linear support vector regression on 1613 Mordred molecular descriptors, to predict collision cross sections (CCS) for metabolite candidates and filter false-positive structural annotations by comparing predicted vs experimental CCS at defined ΔCCS thresholds and mass-error windows.

## When to use

Use when you have an experimental CCS value for a metabolite adduct (e.g. [M−H]−, [M+H]+, [M+Na]+) from ion-mobility MS, plus a list of formula-matching database candidates (e.g. PubChem entries within a mass tolerance such as 10 ppm), and you need to reduce false-positive structural annotations by CCS filtering.

## When NOT to use

- Your input is a feature table or peak list without any experimental CCS value — CCS filtering requires a measured drift-time-derived CCS to compare against predictions.
- Your candidate structures need CCS prediction for adducts or species outside the training scope, e.g. dimers such as [2M+H]+ or [2M−H]−, other less common adducts, or in-source fragments — the article explicitly notes current training sets do not cover these.
- You need a fully class-matched or superclass-matched model without tailoring — the article notes a drawback of ML methods is the need to define the specific training set prior to prediction, and superclass-matched models improve confidence.

## Inputs

- Experimental CCS value(s) for the target metabolite adduct from ion-mobility MS
- Accurate mass measurement of the ion (m/z)
- List of candidate molecular structures/formulas from a database such as PubChem (e.g. 598,208 candidates at 10 ppm mass tolerance for 162 deprotonated test compounds)
- Training set of metabolites with curated experimental CCS values used to build the SVR models (user-curated training sets supported)
- Molecular descriptors computed via the Mordred Python package (1613 descriptors) for each candidate structure

## Outputs

- Predicted CCS values for each candidate structure/adduct (e.g. .csv export from the notebook)
- ΔCCS values (relative percent deviation between predicted and experimental CCS) per candidate
- Filtered candidate list retaining only structures within the chosen ΔCCS threshold (e.g. 221→32 candidates at ΔCCS 2%)
- Filtering statistics: true-annotation retention (e.g. 76.5% at ΔCCS 3%) and median fraction of candidates filtered (e.g. 47.3%)
- ROC curves and AUC values across ΔCCS thresholds and mass-error windows (e.g. AUC 0.657→0.589)

## How to apply

Download the 'CCSP 2.0 - CCS Prediction in Jupyter' notebook (.ipynb) from github.com/facundof2016/CCSP2.0 and open it in Jupyter Lab or Jupyter Notebook under Python 3 (e.g. via Anaconda; Sklearn ≥1.0.2 required, see requirements.txt). For each database candidate whose accurate mass matches the measured ion within a specified mass-error window (e.g. 10 ppm), run the embedded linear SVR models built on the 1613-descriptor Mordred library to predict the CCS value for the relevant adduct type. Compute ΔCCS as the relative percent deviation between predicted and experimental CCS for each candidate, and apply a ΔCCS threshold (e.g. 2% or 3%) to retain or discard candidates. Sweep thresholds and mass windows to quantify true-annotation retention and median filtering efficacy, and use ROC AUC across ΔCCS cutoffs to judge overall discrimination (reported ~0.657 at 10 ppm, dropping to ~0.589 for tighter mass windows). For small-scale predictions (<10,000 molecules), the Google Colab notebook variant is recommended; for large training or target sets or workflow integration, use the locally hosted Jupyter Lab version.

## Related tools

- **CCSP 2.0** (Open-source Jupyter Notebook tool implementing linear SVR models on Mordred descriptors to predict CCS and filter candidate annotations) — https://github.com/facundof2016/CCSP2.0
- **Mordred** (Python package used to compute the 1613 molecular descriptors fed into the SVR CCS prediction models)
- **Jupyter Notebook / Jupyter Lab** (Execution environment for the CCSP 2.0 notebook (Google Colab variant recommended for <10,000 molecules; Jupyter Lab for large sets/workflow integration)) — https://github.com/facundof2016/CCSP2.0

## Examples

```
jupyter lab CCSP\ 2.0\ -\ CCS\ Prediction\ in\ Jupyter.ipynb
```

## Evaluation signals

- Median relative CCS prediction errors should be below ~2% on test-set validation (reported 1.25%, 1.73%, 1.87% for [M−H]−, [M+H]+, [M+Na]+ adducts respectively on 463 McLean CCS Compendium measurements).
- At ΔCCS 2% and 10 ppm mass error, a large candidate reduction should occur (e.g. 221→32 candidates for the isoorotic acid case).
- At ΔCCS 3% and 10 ppm, true-annotation retention should reach ~76.5% across the diverse 162-compound evaluation set, with median filtering of ~47.3% of candidates.
- ROC AUC across ΔCCS thresholds should be reproducible (~0.657 at 10 ppm mass tolerance; decreasing to ~0.589 when mass windows are reduced).
- Outputs must be exported as results files (e.g. ccs_filtering_results.csv) with per-candidate predicted CCS, ΔCCS, and retained/discarded flag.

## Limitations

- Requires a user-curated training set defined before prediction; one drawback of ML methods is the need to define the specific training set prior to predicting CCS values.
- Current training sets cover only the most common adduct ion types ([M−H]−, [M+H]+, [M+Na]+, [M+K]+) but not dimers such as [2M+H]+ or [2M−H]−, other less common adducts, or in-source fragments.
- Current tools afford minimal orthogonal information for diverse data sets with highly accurate mass measurements — filtering efficacy is modest (ROC AUC ~0.657) when the training set is chemically diverse rather than class-matched.
- Google Colab continuous operation is limited to up to 12 hours and disconnects after ~20 minutes idle; large training/target sets or workflow integration require the locally hosted Jupyter Lab version.
- CCSP 2.0 requires Scikit-learn version 1.0.2 or later (see requirements.txt).

## Evidence
