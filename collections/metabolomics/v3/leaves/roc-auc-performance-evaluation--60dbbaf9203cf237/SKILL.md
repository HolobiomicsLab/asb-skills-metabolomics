---
name: roc-auc-performance-evaluation
description: Use when you have a list of candidate structures filtered by predicted-vs-experimental
  ΔCCS and a known set of true annotations, and need to quantify threshold-dependent
  discrimination via ROC AUC.
when_to_use_negative:
- Input already contains ROC AUC values from the CCSP 2.0 paper; no need to recompute.
- The user only wants a single ΔCCS threshold applied without any ROC/AUC evaluation.
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3520
tools:
- name: CCSP 2.0
  role: predicts CCS values via linear support vector regression on 1613 Mordred descriptors
    to compute ΔCCS for candidate annotations
  repo: https://github.com/facundof2016/CCSP2.0
- name: Jupyter Notebook
  role: host environment for CCSP 2.0 notebook and threshold/ROC analysis
  repo: https://github.com/facundof2016/CCSP2.0
- name: Mordred
  role: library of 1613 molecular descriptors used by CCSP 2.0's linear SVR models
- name: PubChem
  role: source of candidate structures within a 10 ppm mass window for ΔCCS filtering
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.2c03491
    title: 'CCS Predictor 2.0: An Open-Source Jupyter Notebook Tool for Filtering
      Out False Positives in Metabolomics'
  source_content_hash: sha256:616b57e66c94dc5b1cb34859aa801df3e1f17b76f5c9f2f47fa5a661957d41d8
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/facundof2016/CCSP2.0
  tools:
  - CCSP 2.0
  - Jupyter Notebook
  - Mordred
  - PubChem
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# roc-auc-performance-evaluation

## Summary

Evaluate CCS-based metabolite annotation filtering by sweeping ΔCCS thresholds and computing ROC curves plus AUC, e.g. reproducing the reported AUC drop from 0.657 to 0.589 when mass windows shrink.

## When to use

Use when you have a list of candidate structures filtered by predicted-vs-experimental ΔCCS and a known set of true annotations, and need to quantify threshold-dependent discrimination via ROC AUC.

## When NOT to use

- Input already contains ROC AUC values from the CCSP 2.0 paper; no need to recompute.
- The user only wants a single ΔCCS threshold applied without any ROC/AUC evaluation.

## Inputs

- Set of candidate metabolite annotations with predicted and experimental CCS values (e.g. from PubChem via 10 ppm mass window)
- List of true annotations with experimental CCS values
- CCSP 2.0 notebook output with ΔCCS values per candidate
- Chosen mass error tolerance (e.g. 10 ppm)

## Outputs

- ROC curve
- AUC value(s) at each mass window (e.g. 0.657 at 10 ppm, 0.589 at tighter window)
- Table of ΔCCS threshold sweep results (candidate counts, retention %, filtering %)

## How to apply

1. For each candidate annotation compute ΔCCS = |(predicted − experimental CCS)/experimental CCS| using the CCSP 2.0 linear SVR prediction (1613 Mordred descriptors). 2. Sweep ΔCCS threshold from 0 up to a generous max (e.g. up to ~5%) and at each threshold mark candidate as accepted/rejected. 3. Tabulate at each threshold: total candidates accepted, true annotations retained (TPR), fraction of all candidates filtered. 4. Build ROC curves of correct-annotation retention versus incorrect-candidate filtering across threshold values. 5. Verify expected values such as ΔCCS 2% reducing 221 candidates to 32, ΔCCS 3% retaining 76.5% true annotations and median filtering 47.3% at 10 ppm. 6. Recompute AUC across mass tolerance windows (e.g. 10 ppm → tighter windows) and confirm the reported sequence 0.657 → 0.589.

## Related tools

- **CCSP 2.0** (predicts CCS values via linear support vector regression on 1613 Mordred descriptors to compute ΔCCS for candidate annotations) — https://github.com/facundof2016/CCSP2.0
- **Jupyter Notebook** (host environment for CCSP 2.0 notebook and threshold/ROC analysis) — https://github.com/facundof2016/CCSP2.0
- **Mordred** (library of 1613 molecular descriptors used by CCSP 2.0's linear SVR models)
- **PubChem** (source of candidate structures within a 10 ppm mass window for ΔCCS filtering)

## Examples

```
jupyter notebook "CCSP 2.0 - CCS Prediction in Jupyter.ipynb"; # sweep ΔCCS thresholds; compute ROC AUC across 10 ppm window
```

## Evaluation signals

- AUC at 10 ppm should match ~0.657 and drop to ~0.589 under tighter mass windows.
- ΔCCS 2% should reduce the 221-candidate PubChem list to ~32 candidates.
- ΔCCS 3% should retain 76.5% of true annotations and median-filter 47.3% of candidates.
- Median relative errors on McLean CCS Compendium test sets should be 1.25, 1.73, and 1.87% for [M−H]−, [M+H]+, [M+Na]+ respectively.

## Limitations

- Requires a known set of true annotations to establish ROC ground truth; cannot assess filtering when no correct annotation is available.
- Mass window choice directly changes AUC; smaller windows lower discrimination (0.657 → 0.589), so results are not transferable across tolerances.
- Only single-adduct predictions ([M−H]−, [M+H]+, [M+Na]+, [M+K]+) are supported; dimers or in-source fragments cannot be predicted.
- For chemically diverse data sets, CCSP 2.0 currently affords minimal orthogonal information with highly accurate mass measurements.

## Evidence

- [other] Decreasing mass windows reduced ROC AUC from 0.657 to 0.589.: [evidence span withheld — anchor status: not_found]
- [other] ΔCCS 3% retained 76.5% true annotations and median filtering 47.3%.: [evidence span withheld — anchor status: not_found]
