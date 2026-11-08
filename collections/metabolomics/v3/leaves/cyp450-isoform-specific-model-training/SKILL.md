---
name: cyp450-isoform-specific-model-training
description: Use when a training dataset of substrate molecules with bond-of-metabolism
  (BoM) annotations is available for a specific CYP450 isoform and the goal is a per-isoform
  classifier that scores each bond as reactive or non-reactive.
when_to_use_negative:
- The goal is to predict whether a compound is a reactant at all (reactant/non-reactant
  classification) — use CypReact, which is a separate module and acts as a filter
  upstream of CypBoM.
- The input annotations are site-of-metabolism (SoM) atom annotations rather than
  bond (BoM) annotations, or the reactions are η+η (Cyclization) pairs, which were
  excluded because they occur in only 4 of 828 η-η reactions.
- The dataset is not imbalanced enough to require cost-sensitive learning — with balanced
  classes the 0/0/1/β cost matrix and Jaccard optimization would be unnecessary.
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3370
- http://edamontology.org/topic_3474
tools:
- name: Chemistry Development Kit (CDK)
  role: Computes the 473 per-[compound, bond] features (molecular descriptors, fingerprints,
    atom/bond types, neighbor descriptors) used as classifier input.
- name: WEKA
  role: Provides the random-forest algorithm trained as the final per-isoform classifier
    with the optimal [β*, t*, N*] parameters.
- name: CypBoM
  role: 'The module itself: trained per isoform on EBoMD, predicts reactive bonds
    (〈X, Y〉 pairs) with scores for a given isoform and molecule; invoked downstream
    by CyProduct/MetaboGen.'
  repo: https://bitbucket.org/wishartlab/cyproduct/src/master/
- name: EBoMD
  role: The Edmonton Bond-of-Metabolism Dataset (828 η-η BoMs) that supplies the labeled
    training data for CypBoM.
  repo: https://bitbucket.org/wishartlab/cyproduct/src/master/
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.jcim.1c00144
    title: 'CyProduct: A Software Tool for Accurately Predicting the Byproducts of
      Human Cytochrome P450 Metabolism'
  source_content_hash: sha256:21bc2cc5aaa22255710f02a1295ae67220abb7303633af3abf2cb315de6e12bc
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Chemistry Development Kit (CDK)
  - CyProduct
  - CypBoM
  - EBoMD
  - WEKA
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3370
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# cyp450-isoform-specific-model-training

## Summary

Trains a cost-sensitive random-forest classifier (CypBoM) per CYP450 isoform to predict reactive bond locations (BoMX−Y) on substrate molecules, using CDK-derived bond features and a super-learner over 189 [β, t, N] hyperparameter combinations optimized for Jaccard score rather than accuracy.

## When to use

Use when a training dataset of substrate molecules with bond-of-metabolism (BoM) annotations is available for a specific CYP450 isoform and the goal is a per-isoform classifier that scores each bond as reactive or non-reactive.

## When NOT to use

- The goal is to predict whether a compound is a reactant at all (reactant/non-reactant classification) — use CypReact, which is a separate module and acts as a filter upstream of CypBoM.
- The input annotations are site-of-metabolism (SoM) atom annotations rather than bond (BoM) annotations, or the reactions are η+η (Cyclization) pairs, which were excluded because they occur in only 4 of 828 η-η reactions.
- The dataset is not imbalanced enough to require cost-sensitive learning — with balanced classes the 0/0/1/β cost matrix and Jaccard optimization would be unnecessary.

## Inputs

- EBoMD.sdf — η-η BoM annotations (828 reactive η-η BoMs, 16418 η-η bonds) derived from the Zaretzki dataset of 679 CYP450 substrates
- Per-isoform training dataset D(α) of [compound, bond] instances with BoMX−Y / non-BoMX−Y labels
- 473-feature CDK feature table per [compound, bond] instance

## Outputs

- Per-isoform WEKA random-forest classifier CypBoMη−η(α, ·) with optimal hyperparameters [β*, t*, N*]
- Reactive-bond predictions (bond scores and reactive/non-reactive calls at threshold τ = 1/(β+1)) for a query substrate
- External 5-fold cross-validated Jaccard scores per isoform (written to cypbom_eta_eta_cv_scores.csv)

## How to apply

Load the EBoMD η-η BoM annotations (828 reactive η-η BoMs out of 16418 η-η bonds from 679 Zaretzki-derived compounds) from EBoMD.sdf, excluding the rare η+η pairs, and canonically order each bond's atom pair (by atomic number, then ConnectedAtoms value, then lexicographic AtomDescriptor order) to avoid duplicate encodings. Extract 473 CDK features per [compound, bond] instance (7 molecular descriptors, 14 fingerprint bits, 2 ConnectedAtoms, 30 BondFingerprint, 23 SYBYL BondAtomTypes, 20 BondAtomDescriptors, 92 NeighborAtomTypes, 280 NeighborAtomDescriptor features). Remove constant features, rank the rest by information gain against the BoMX−Y/non-BoMX−Y label, and retain the top-N features with N chosen by internal cross-validation. Run the cost-sensitive super-learner over all 9 × 3 × 7 = 189 combinations of [β ∈ {2, 2.5, 3, ..., 8}, batch size t ∈ {90, 95, 100}, N ∈ {50, 100, 150, 200, 300, 400, All}] with 5-fold internal cross-validation, using the cost matrix 0/0/1/β (TP/TN/FN/FP) and optimizing the Jaccard score TP/(TP+FN+FP) — this compensates for the extreme class imbalance (~5% reactive vs ~95% non-reactive bonds). Train the final WEKA random forest on the entire per-isoform dataset D(α) with the optimal [β*, t*, N*], converting each bond score to a reactive/non-reactive call via threshold τ = 1/(β+1). Evaluate with external 5-fold cross-validation per isoform (CYP1A2, 2A6, 2B6, 2C8, 2C9, 2C19, 2D6, 2E1, 3A4), recording the Jaccard score for each.

## Related tools

- **Chemistry Development Kit (CDK)** (Computes the 473 per-[compound, bond] features (molecular descriptors, fingerprints, atom/bond types, neighbor descriptors) used as classifier input.)
- **WEKA** (Provides the random-forest algorithm trained as the final per-isoform classifier with the optimal [β*, t*, N*] parameters.)
- **CypBoM** (The module itself: trained per isoform on EBoMD, predicts reactive bonds (〈X, Y〉 pairs) with scores for a given isoform and molecule; invoked downstream by CyProduct/MetaboGen.) — https://bitbucket.org/wishartlab/cyproduct/src/master/
- **EBoMD** (The Edmonton Bond-of-Metabolism Dataset (828 η-η BoMs) that supplies the labeled training data for CypBoM.) — https://bitbucket.org/wishartlab/cyproduct/src/master/

## Examples

```
java -jar CyProduct.jar CC(=O)Nc1ccc(O)cc1 1A2,2A6,2B6,2C8,2C9,2C19,2D6,2E1,3A4 E:\Users\cyproduct\
```

## Evaluation signals

- External 5-fold cross-validated Jaccard scores (TP/(TP+FN+FP)) for all nine isoforms must exceed the random classifier baseline, consistent with significance at p < 3.2E-10.
- The classifier's cost-sensitive calibration must reflect the reactive-class distribution of ~5% reactive vs ~95% non-reactive bonds.
- The super-learner must have explored all 189 [β, t, N] combinations via internal 5-fold CV before fixing [β*, t*, N*]; any shortcut over this grid invalidates the optimization.
- Canonical ordering of each η-η atom pair must produce no duplicate encodings of the same bond (ordering key: atomic number, then ConnectedAtoms, then lexicographic AtomDescriptor 10-tuple).
- The decision threshold applied to scores must equal τ = 1/(β+1) for the selected β, matching the cost matrix 0/0/1/β.

## Limitations

- No software version numbers for CDK or WEKA are stated in the paper, so exact feature-computation reproducibility is not guaranteed.
- The η+η (Cyclization) bond type is excluded — the model covers only η-η bonds, so cyclization-type SoMs are out of scope.
- Class imbalance (~5% reactive) means plain accuracy is a misleading metric; Jaccard with cost matrix must be used, and threshold τ depends on β.
- Features are CDK-computed, so results depend on CDK descriptor behavior even though no version is pinned.

## Evidence

- [methods] the learned CypBoM component first predicts the reactive bonds (〈X, Y〉 pairs): [evidence span withheld — anchor status: not_found]
