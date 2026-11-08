---
name: jaccard-score-cross-validation
description: Use when evaluating a sparse-positive molecular site-of-metabolism classifier
  (e.g. ~5% reactive bonds) where accuracy is mis and per-class/isoform Jaccard scores
  from external cross-validation are the appropriate metric.
when_to_use_negative:
- When the goal is end-to-end metabolite/byproduct prediction rather than bond-site
  classifier evaluation — use the full CyProduct pipeline (CypReact filter → CypBoM
  → MetaboGen) instead
- When classes are balanced or the positive class dominates — the cost-sensitive β
  calibration (0/0/1/β cost matrix) and threshold 1/(β+1) are designed for the ~5%
  reactive / 95% non-reactive imbalance and may distort well-balanced problems
- When the input is a generic feature table with no molecular bond structure — the
  Jaccard-over-bonds framing and per-isoform folds assume η−η bond-level instances
edam_operation: http://edamontology.org/operation_2423
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: CypBoM
  role: Bond-of-Metabolism classifier whose Jaccard scores are produced by this evaluation
    scheme
  repo: https://bitbucket.org/wishartlab/cyproduct/src/master/
- name: WEKA
  role: Random-forest base learner trained with the super-learner-selected [β, t,
    N] parameters
- name: CDK
  role: Generates the 473 features per [compound, bond] instance fed into the cross-validation
    loop
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.jcim.1c00144
    title: 'CyProduct: A Software Tool for Accurately Predicting the Byproducts of
      Human Cytochrome P450 Metabolism'
  source_content_hash: sha256:efe4bce8fe1d4717712351ed660fbd2c03f135a945d5f10badcadf97d74337a8
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - CDK
  - Chemistry Development Kit (CDK)
  - CyProduct
  - CypBoM
  - EBoMD
  - WEKA
  edam_operation: http://edamontology.org/operation_2423
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 23
---

# Jaccard-score cross-validation for imbalanced bond-site classifiers

## Summary

Evaluation methodology from CyProduct's CypBoM module: score bond-site classifiers with the Jaccard score TP/(TP+FN+FP) under external 5-fold cross-validation, using internal 5-fold CV over a cost-sensitive super-learner grid to avoid accuracy bias on 5%/95% imbalanced η−η bond data.

## When to use

Use when evaluating a sparse-positive molecular site-of-metabolism classifier (e.g. ~5% reactive bonds) where accuracy is misleading and per-class/isoform Jaccard scores from external cross-validation are the appropriate metric.

## When NOT to use

- When the goal is end-to-end metabolite/byproduct prediction rather than bond-site classifier evaluation — use the full CyProduct pipeline (CypReact filter → CypBoM → MetaboGen) instead
- When classes are balanced or the positive class dominates — the cost-sensitive β calibration (0/0/1/β cost matrix) and threshold 1/(β+1) are designed for the ~5% reactive / 95% non-reactive imbalance and may distort well-balanced problems
- When the input is a generic feature table with no molecular bond structure — the Jaccard-over-bonds framing and per-isoform folds assume η−η bond-level instances

## Inputs

- EBoMD.sdf — 679 Zaretzki-derived compounds with η−η BoM annotations (828 reactive of 16418 η−η bonds), excluding the 4 η+η Cyclization reactions
- 473 CDK features per [compound, bond] instance after constant-feature removal and information-gain ranking
- Isoform label (one of the nine CYP450 isoforms) and reactive/non-reactive bond label per instance

## Outputs

- cypbom_eta_eta_cv_scores.csv — external 5-fold cross-validated Jaccard score per isoform
- Trained WEKA random-forest classifier CypBoMη−η(α, ·) per isoform with optimal [β*, t*, N*] and threshold τ = 1/(β+1)

## How to apply

Split the labeled dataset (here 828 reactive η−η BoMs vs 16418 η−η bonds from 679 Zaretzki-derived compounds in EBoMD.sdf) into external 5-fold cross-validation folds per isoform (CYP1A2–CYP3A4). Inside each training fold, run the cost-sensitive super-learner over all 9*3*7=189 combinations of [β ∈ {2, 2.5, 3, 3.5, 4, 5, ..., 8}, batch size t ∈ {90, 95, 100}, top-N features ∈ {50, 100, 150, 200, 300, 400, All}] using internal 5-fold CV, selecting the combination that maximizes Jaccard = TP/(TP+FN+FP) rather than accuracy, with a 0/0/1/β cost matrix for TP/TN/FN/FP to compensate for the 95% non-reactive majority. Train the final WEKA random forest on the entire training fold with optimal [β*, t*, N*], and convert its bond scores to reactive/non-reactive calls using threshold τ = 1/(β+1). Record the external-CV Jaccard score per isoform (written to cypbom_eta_eta_cv_scores.csv) and compare against a random classifier baseline; the article reports significance at p < 3.2E-10.

## Related tools

- **CypBoM** (Bond-of-Metabolism classifier whose Jaccard scores are produced by this evaluation scheme) — https://bitbucket.org/wishartlab/cyproduct/src/master/
- **WEKA** (Random-forest base learner trained with the super-learner-selected [β, t, N] parameters)
- **CDK** (Generates the 473 features per [compound, bond] instance fed into the cross-validation loop)

## Examples

```
score = (prob - threshold) / (1 - threshold)  # threshold = 1/(1+beta), per CyProduct README
```

## Evaluation signals

- External 5-fold cross-validated Jaccard score for each of the nine isoforms must exceed the random classifier baseline (paper reports significance at p < 3.2E-10)
- The internal CV grid must cover all 189 [β, t, N] combinations (9 × 3 × 7) and select by Jaccard, not accuracy
- The decision threshold applied to classifier scores must equal 1/(β+1) for the chosen β*, matching the cost matrix calibration
- The reactive/non-reactive class distribution (~5%/95%) should be reflected in the classifier's cost-sensitive calibration and score distribution
- Output CSV must contain one Jaccard score per isoform for CYP1A2, CYP2A6, CYP2B6, CYP2C8, CYP2C9, CYP2C19, CYP2D6, CYP2E1, CYP3A4

## Limitations

- No software version numbers for CDK or WEKA are stated in the article, which may affect exact reproducibility of the trained forests
- η+η-forming reactions are effectively unmodelled — only 4 of 828 η−η reactions were Cyclization, so the η−η predictor's scope excludes them
- Jaccard scores are computed per isoform on 679 Zaretzki-derived compounds; performance may not transfer to compounds outside the EBoMD substrate distribution

## Evidence
