---
name: roc-based-filtering-evaluation
description: Use when an annotation-filtering step (e.g., ΔCCS 2.8% threshold at varying
  ppm mass tolerances) must be quantified as retained-correct-annotation vs. filtered-candidate
  trade-offs across mass windows.
when_to_use_negative:
- Candidate structures include dimers such as [2M+H]+ or [2M-H]-, other less common
  adducts, or in-source fragments — the CCSP 2.0 training sets do not support these
- Filtering is based on non-CCS evidence (retention-time matching to standards, MS/MS
  spectra, isotope clusters) rather than predicted-vs-observed ΔCCS
- CCS predictions are needed across different IM-MS instrument types (drift tube vs.
  travelling wave) — the effect of such experimental variance on the CCS threshold
  is not yet investigated
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_2269
tools:
- name: CCSP 2.0
  role: Produces the SVR-predicted CCS values whose ΔCCS-based filtering decisions
    are evaluated by ROC/AUC
  repo: https://github.com/facundof2016/CCSP2.0
- name: SciKit-Learn
  role: Generates ROC curves and area-under-curve scores across the 10–0 ppm mass
    windows; also hosts the underlying linear-kernel SVR
- name: Mordred V1.2.0
  role: Supplies the 1613 2D molecular descriptors from which candidate CCS predictions
    (and hence filter scores) are derived
- name: PubChemPy
  role: Retrieves each candidate CID from the exact-mass PubChem queries so descriptors
    and predicted CCS can be assigned per candidate
- name: Biopython (Entrez)
  role: Queries PubChem for all compounds with exact mass within 10 ppm to build the
    candidate pools being scored
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1101/2022.08.09.503345
    title: 'CCS Predictor 2.0: An Open-Source Jupyter Notebook Tool for Filtering
      Out False Positives in Metabolomics'
  source_content_hash: sha256:4e1ecf1e4b9c3d5b14495d6c0010aa7cf2733ad732a9942da5060779d5b380d1
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
  - Biopython (Entrez)
  - CCSP 2.0
  - Jupyter Notebook
  - Mordred
  - Mordred V1.2.0
  - PubChemPy
  - Python
  - RDKit
  - SciKit-Learn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 19
---

# roc-based-filtering-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Evaluates how well a candidate-annotation filter (e.g., ΔCCS-based filtering from CCSP 2.0) discriminates correct annotations from false positives by generating receiver operating characteristic curves and area-under-curve scores across multiple precursor mass windows (10, 7, 5, 3, 1, and 0 ppm).

## When to use

Use when an annotation-filtering step (e.g., ΔCCS 2.8% threshold at varying ppm mass tolerances) must be quantified as retained-correct-annotation vs. filtered-candidate trade-offs across mass windows.

## When NOT to use

- Candidate structures include dimers such as [2M+H]+ or [2M-H]-, other less common adducts, or in-source fragments — the CCSP 2.0 training sets do not support these
- Filtering is based on non-CCS evidence (retention-time matching to standards, MS/MS spectra, isotope clusters) rather than predicted-vs-observed ΔCCS
- CCS predictions are needed across different IM-MS instrument types (drift tube vs. travelling wave) — the effect of such experimental variance on the CCS threshold is not yet investigated

## Inputs

- Per-validation-compound PubChem candidate lists (43–13,984 candidates per compound; 136,940 total at 10 ppm exact-mass tolerance)
- Predicted CCS values (linear-kernel SVR, CCSP 2.0) for every candidate structure computed from 1613 Mordred 2D descriptors
- Known correct annotation identities for the 33 validation compounds
- ΔCCS threshold (2.8%) and set of mass windows (10, 7, 5, 3, 1, 0 ppm)

## Outputs

- Per-mass-window percentage of annotation candidates filtered
- Per-mass-window percentage of correct annotations retained
- Receiver operating characteristic (ROC) curves per lipid class
- Area-under-curve (AUC) scores across mass windows (expected 0.748–0.776 for the lipid class-matched model)

## How to apply

For each validation compound, first assemble the full candidate set (exact-mass candidates retrieved from PubChem within the 10 ppm window, plus CCS values predicted for every candidate by the trained SVR model). Label the known correct structure as the positive and treat the ΔCCS filter outcome as the classifier score/decision. Sweep the mass window from 10 ppm down to 0 ppm, computing at each window the percentage of candidates filtered and the percentage of correct annotations retained at the ΔCCS 2.8% threshold, then generate receiver operating characteristic curves and area-under-curve scores across these windows with SciKit-Learn. Correct application is judged by matching the class-matched lipid benchmark: 100% of correct annotations retained at ΔCCS 2.8% and 10 ppm, a median of 36.1% of candidates filtered (19.3% of isomeric candidates at 0 ppm), and lipid-class ROC AUC scores in the range 0.748–0.776.

## Related tools

- **CCSP 2.0** (Produces the SVR-predicted CCS values whose ΔCCS-based filtering decisions are evaluated by ROC/AUC) — https://github.com/facundof2016/CCSP2.0
- **SciKit-Learn** (Generates ROC curves and area-under-curve scores across the 10–0 ppm mass windows; also hosts the underlying linear-kernel SVR)
- **Mordred V1.2.0** (Supplies the 1613 2D molecular descriptors from which candidate CCS predictions (and hence filter scores) are derived)
- **PubChemPy** (Retrieves each candidate CID from the exact-mass PubChem queries so descriptors and predicted CCS can be assigned per candidate)
- **Biopython (Entrez)** (Queries PubChem for all compounds with exact mass within 10 ppm to build the candidate pools being scored)

## Examples

```
from sklearn.metrics import roc_curve, auc; fpr, tpr, _ = roc_curve(y_true, scores); roc_auc = auc(fpr, tpr)
```

## Evaluation signals

- At ΔCCS 2.8% and 10 ppm mass error, 100% of the correct annotations are retained (no false negatives of true annotations)
- Median percentage of candidates filtered across validation compounds equals 36.1% (19.3% of isomeric candidates at 0 ppm)
- ROC AUC values for the lipid class-matched evaluation fall within 0.748–0.776
- All six mass windows (10, 7, 5, 3, 1, 0 ppm) are reported, with filtered-fraction monotonicity checked as the window tightens
- Validation set dimensions match the protocol: 33 scored validation compounds from the 36-compound 30% split (3 excluded for lacking exact PubChem matches)

## Limitations

- The specific training set must be defined prior to CCS prediction; changing the training set (e.g., class-matched lipid superclass) changes the ROC/AUC outcomes
- Effect of experimental sources of variance on the choice of the CCS threshold is not yet investigated, so threshold-based filter performance may vary with instrument conditions
- CCSP 2.0 use requires familiarity with a programming language such as Python, and the current version requires Scikit-learn V1.0.2 or later

## Evidence

- [readme] Collision Cross Section Predictor 2.0 is an open source Python notebook intended to help ion-mobility scientists predict collision cross sections with user-curated training sets: [evidence span withheld — anchor status: external_doc]
