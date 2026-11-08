---
name: threshold-sweep-statistics-analysis
description: Decision support for choosing a ΔCCS threshold when validating a CCS-prediction-based
  annotation filter over a set of standards with known PubChem candidate lists.
when_to_use_negative:
- When candidate lists have not been generated from PubChem at a defined mass tolerance
  (e.g. 10 ppm) — the sweep statistics are meaningless without a reproducible candidate
  pool.
- When the training set does not cover the adduct being filtered (e.g. dimers such
  as [2M+H]+ or [2M-H]-, less common adducts, or in-source fragments), as CCSP 2.0's
  current training sets do not support these.
- When the user cannot run the CCSP 2.0 Jupyter notebook, e.g. lacks Python 3/Sklearn
  V1.0.2+ or is not using the provided Google Colab or locally hosted Jupyter Lab
  notebooks.
edam_operation: http://edamontology.org/operation_3501
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: CCSP 2.0
  role: Linear-kernel SVR model used to predict CCS values for every candidate structure
    and the true annotation before threshold sweeping
  repo: https://github.com/facundof2016/CCSP2.0
- name: Mordred
  role: Computes 1613 2D molecular descriptors from RDKit-encoded MOL files for feature
    generation used in CCS prediction
- name: RDKit
  role: Encodes SMILES into MOL files that Mordred uses to calculate 2D descriptors
- name: PubChem
  role: Source of candidate annotations retrieved via the Entrez Global Query Cross-Database
    Search System (Biopython) at a 10 ppm exact-mass tolerance
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1101/2022.08.09.503345
    title: 'CCS Predictor 2.0: An Open-Source Jupyter Notebook Tool for Filtering
      Out False Positives in Metabolomics'
  source_content_hash: sha256:650c955fc63a3842c24c4b321c797d3ce4295a8b1cc126ead9c5b85bc3459380
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: ''
  repo_url: https://github.com/facundof2016/CCSP2.0
  tools:
  - CCSP 2.0
  - Jupyter Notebook
  - Mordred
  - PubChem
  - Python
  - RDKit
  edam_operation: http://edamontology.org/operation_3501
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# threshold-sweep-statistics-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Sweep a ΔCCS filtering threshold from 0% to 15% while recording the fraction of true annotations retained and the median fraction of candidate structures filtered, to quantify the sensitivity-specificity trade-off of CCS-based metabolite annotation filtering.

## When to use

Decision support for choosing a ΔCCS threshold when validating a CCS-prediction-based annotation filter over a set of standards with known PubChem candidate lists.

## When NOT to use

- When candidate lists have not been generated from PubChem at a defined mass tolerance (e.g. 10 ppm) — the sweep statistics are meaningless without a reproducible candidate pool.
- When the training set does not cover the adduct being filtered (e.g. dimers such as [2M+H]+ or [2M-H]-, less common adducts, or in-source fragments), as CCSP 2.0's current training sets do not support these.
- When the user cannot run the CCSP 2.0 Jupyter notebook, e.g. lacks Python 3/Sklearn V1.0.2+ or is not using the provided Google Colab or locally hosted Jupyter Lab notebooks.

## Inputs

- 70:30 split [M-H]- training/calibration data set (Supplemental Table 4) with 397 calibration compounds
- 162 deprotonated test compounds with known standard CCS values and PubChem candidate lists generated at 10 ppm exact-mass tolerance
- Predicted CCS values for all candidate structures and the true annotation

## Outputs

- filtering_stats.csv containing, for each ΔCCS threshold from 0% to 15%, the percentage of true annotations retained and the median percentage of candidates filtered
- Median filtered percentage recomputed for same-formula candidate subsets

## How to apply

Generate PubChem candidate lists for each standard's exact mass within a 10 ppm mass window, predict CCS for every candidate (including the true annotation) with the CCSP 2.0 linear-kernel SVR model, then sweep the ΔCCS threshold from 0% to 15%. At each threshold record two statistics: the percentage of true annotations retained and the median percentage of candidates filtered. Optionally restrict candidate lists to those sharing the true compound's chemical formula and recompute the median filtered percentage. Write the threshold-sweep table to filtering_stats.csv. In the 162 deprotonated ([M-H]-) test compound validation, a 3% ΔCCS threshold retained 76.5% of true annotations while filtering a median 47.3% of candidates at a 10 ppm mass window (24.9% for same-formula candidates), and the combined candidate pool was 598,208 candidates.

## Related tools

- **CCSP 2.0** (Linear-kernel SVR model used to predict CCS values for every candidate structure and the true annotation before threshold sweeping) — https://github.com/facundof2016/CCSP2.0
- **Mordred** (Computes 1613 2D molecular descriptors from RDKit-encoded MOL files for feature generation used in CCS prediction)
- **RDKit** (Encodes SMILES into MOL files that Mordred uses to calculate 2D descriptors)
- **PubChem** (Source of candidate annotations retrieved via the Entrez Global Query Cross-Database Search System (Biopython) at a 10 ppm exact-mass tolerance)

## Examples

```
python CCSP2.0_threshold_sweep.py --input filtering_stats.csv --adduct [M-H]- --pubchem_tol 10ppm --thresholds 0:0.01:15
```

## Evaluation signals

- filtering_stats.csv must contain rows for thresholds from 0% to 15% and both retention and filtered-fraction columns.
- At ΔCCS 3% with 10 ppm PubChem candidate lists, the output must show 76.5% of true annotations retained and a median 47.3% of candidates filtered.
- Restricting candidates to those sharing the true compound's chemical formula must yield a median 24.9% filtered at ΔCCS 3%.
- The combined candidate pool across the 162 test compounds must total 598,208 candidates (ranging from 15 to 18,399 per compound).
- Candidate lists must exclude the 9 compounds without exact PubChem matches.

## Limitations

- The threshold sweep depends on CCSP 2.0's prediction accuracy, which shows median relative errors of 1.25, 1.73 and 1.87% for [M-H]-, [M+H]+ and [M+Na]+ adducts; systematic prediction bias may shift optimal thresholds.
- Current CCSP 2.0 training sets do not cover dimers ([2M+H]+, [2M-H]-), less common adducts, or in-source fragments, limiting the sweep to supported adduct classes.
- CCS predictions across multiple IM-MS instrument types (drift tube vs. travelling wave) have not yet been explored, so thresholds may not transfer across platforms.
- An in-depth investigation of the effect of experimental sources of variance on the choice of CCS threshold has not been performed.
- Use of CCSP 2.0 requires familiarity with a programming language such as Python.

## Evidence

- [results] Sweep ΔCCS threshold 0–15% recording retention/filtered stats: [evidence span withheld — anchor status: not_found]
- [results] 3% threshold retains 76.5%, filters 47.3% median: [evidence span withheld — anchor status: not_found]
- [results] Same-formula candidates filtered at median 24.9%: [evidence span withheld — anchor status: not_found]
- [methods] PubChem candidates at 10 ppm via Entrez/Biopython: [evidence span withheld — anchor status: not_found]
