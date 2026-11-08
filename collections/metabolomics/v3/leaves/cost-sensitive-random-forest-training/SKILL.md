---
name: cost-sensitive-random-forest-training
description: Use when training a binary classifier on heavily imbalanced chemical
  labels (e.g. ~5% reactive bonds in EBoMD) where the Jaccard score TP/(TP+FN+FP)
  is the target metric and a random-forest base learner is desired.
when_to_use_negative:
- The label distribution is roughly balanced and plain accuracy is the target metric
  — the 0/0/1/β cost calibration is specifically designed for the 5% reactive / 95%
  non-reactive EBoMD imbalance.
- The downstream task is metabolite/byproduct generation from already-predicted bonds
  (use the MetaboGen SMIRKS-rule workflow instead).
- Only a small number of η+η (Cyclization) instances is available — the paper notes
  η+η-forming reactions occurred in only 4 of 828 η-η reactions, so they were excluded
  from the η-η training set.
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: CypBoM
  role: Target module whose η-η, η-H and η-SPN classifiers are trained via the cost-sensitive
    super-learner over CDK-derived features
  repo: https://bitbucket.org/wishartlab/cyproduct/src/master/
- name: WEKA
  role: Random-forest base learner that the SL refits with optimal [β*, t*, N*] on
    the full dataset D(α)
- name: Chemistry Development Kit (CDK)
  role: Computes the 473 per-instance features fed to the cost-sensitive random forest
- name: EBoMD
  role: Training corpus of 828 η-η, 1348 η-H and 86 η-SPN BoM annotations over 679
    Zaretzki-derived substrates (EBoMD.sdf)
provenance:
  source_task_ids:
  - task_002
  - task_006
  source_papers:
  - doi: 10.1021/acs.jcim.1c00144
    title: 'CyProduct: A Software Tool for Accurately Predicting the Byproducts of
      Human Cytochrome P450 Metabolism'
  source_content_hash: sha256:dff21b45dcf91a7d8ee9fb43417834bf115934dcc18fff23196cb1876c09f84b
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Chemistry Development Kit (CDK)
  - CyProduct
  - CypBoM
  - EBoMD
  - WEKA
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 17
---

# cost-sensitive-random-forest-training

## Summary

Trains a WEKA random-forest classifier with a 0/0/1/β cost matrix via a super-learner (SL) that grid-searches 189 [β, t, N] combinations by internal 5-fold cross-validation, optimizing the Jaccard score rather than accuracy. Used in CyProduct's CypBoM module to handle the extreme 5%/95% reactive/non-reactive BoM class imbalance.

## When to use

Use when training a binary classifier on heavily imbalanced chemical labels (e.g. ~5% reactive bonds in EBoMD) where the Jaccard score TP/(TP+FN+FP) is the target metric and a random-forest base learner is desired.

## When NOT to use

- The label distribution is roughly balanced and plain accuracy is the target metric — the 0/0/1/β cost calibration is specifically designed for the 5% reactive / 95% non-reactive EBoMD imbalance.
- The downstream task is metabolite/byproduct generation from already-predicted bonds (use the MetaboGen SMIRKS-rule workflow instead).
- Only a small number of η+η (Cyclization) instances is available — the paper notes η+η-forming reactions occurred in only 4 of 828 η-η reactions, so they were excluded from the η-η training set.

## Inputs

- Per-isoform training dataset D(α) of [compound, bond] instances with binary BoMX−Y/non-BoMX−Y labels (from EBoMD.sdf, 828 η-η BoMs over 679 Zaretzki-derived substrates)
- 473 CDK-derived feature columns per instance (7 molecular descriptors, 14 fingerprint bits, 2 ConnectedAtoms, 30 BondFingerprint, 23 BondAtomTypes, 20 BondAtomDescriptors, 92 NeighborAtomTypes, 280 NeighborAtomDescriptor features)
- Hyperparameter grids: β ∈ {2, 2.5, 3, 3.5, 4, 5, ..., 8}, t ∈ {90, 95, 100}, N ∈ {50, 100, 150, 200, 300, 400, All}

## Outputs

- Trained WEKA random-forest classifier CypBoMη−η(α, ·) per isoform with optimal [β*, t*, N*]
- Per-bond reactive/non-reactive probability scores rescaled as (prob−threshold)/(1−threshold)
- Cross-validated Jaccard scores per isoform (e.g. written to cypbom_cv_jaccard.csv / cypbom_eta_eta_cv_scores.csv)

## How to apply

First remove constant features, rank remaining features by information gain against the BoMX−Y/non-BoMX−Y label, and retain the top-N with N ∈ {50, 100, 150, 200, 300, 400, All} learned by internal cross-validation. Then grid-search all 9 × 3 × 7 = 189 combinations of false-positive cost β ∈ {2, 2.5, 3, 3.5, 4, 5, ..., 8}, batch size t ∈ {90, 95, 100} and feature count N using internal 5-fold cross-validation over WEKA random-forest base learners, optimizing the Jaccard score with cost matrix 0/0/1/β for TP/TN/FN/FP. Refit the winning random forest with optimal [β*, t*, N*] on the full isoform training set D(α). Convert predicted probabilities to a reactive/non-reactive decision at threshold τ = 1/(β+1), and rescale scores as Score = (prob−threshold)/(1−threshold). Repeat per bond-type (η-η, η-H, η-SPN) and per CYP450 isoform (1A2, 2A6, 2B6, 2C8, 2C9, 2C19, 2D6, 2E1, 3A4).

## Related tools

- **CypBoM** (Target module whose η-η, η-H and η-SPN classifiers are trained via the cost-sensitive super-learner over CDK-derived features) — https://bitbucket.org/wishartlab/cyproduct/src/master/
- **WEKA** (Random-forest base learner that the SL refits with optimal [β*, t*, N*] on the full dataset D(α))
- **Chemistry Development Kit (CDK)** (Computes the 473 per-instance features fed to the cost-sensitive random forest)
- **EBoMD** (Training corpus of 828 η-η, 1348 η-H and 86 η-SPN BoM annotations over 679 Zaretzki-derived substrates (EBoMD.sdf))

## Evaluation signals

- Each of the nine per-isoform external 5-fold cross-validated Jaccard scores falls in the reported 0.380–0.452 range for CypBoM.
- All per-isoform Jaccard scores exceed the random-classifier baseline, consistent with the reported significance p < 3.2E-10.
- Decision threshold used equals τ = 1/(β+1) for the selected β*, and reported scores equal (prob−τ)/(1−τ).
- Cost-sensitive calibration reflects the reactive-class distribution (~5% reactive, ~95% non-reactive) in the classifier's predicted positive rate.
- Feature count N passed to the final refit is one of {50, 100, 150, 200, 300, 400, All} and was selected by internal cross-validation, not on the external test folds.

## Limitations

- No software version numbers are stated for CDK or WEKA, so exact reproducibility of the random-forest implementation is not guaranteed.
- Class imbalance is extreme (5% reactive η-η bonds), making the Jaccard metric and cost matrix essential; without cost sensitivity the learner would predict almost all bonds non-reactive.
- The discussion section content was absent from the provided document, so limitations stated by the authors themselves could not be extracted.

## Evidence
