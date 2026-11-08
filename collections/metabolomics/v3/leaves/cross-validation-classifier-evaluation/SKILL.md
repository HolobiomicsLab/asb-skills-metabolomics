---
name: cross-validation-classifier-evaluation
description: Use when you must reproduce or estimate generalization performance of
  a per-isoform CYP450 bond-level predictor on EBoMD-style labeled [compound, bond]
  data, and need externally cross-validated Jaccard scores rather than a single hold-out
  split.
when_to_use_negative:
- When the goal is hold-out testing rather than cross-validated generalization estimates
  — use EBoMD2.sdf (98 compounds) as the separate hold-out instead of re-running CV
- When the goal is generating metabolites from predicted bonds (MetaboGen with SMIRKS
  rules) rather than evaluating classifier performance
- When the query task is reactant classification (CypReact filter) rather than bond-level
  SoM prediction
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: CypBoM
  role: Bond-of-Metabolism classifier whose per-isoform Jaccard performance is estimated
    via external 5-fold cross-validation
  repo: https://bitbucket.org/wishartlab/cyproduct/src/master/
- name: CDK (Chemistry Development Kit)
  role: Computes the 473 molecular/bond features that form the classifier input matrix
- name: WEKA
  role: Provides the random-forest base learners tuned by the cost-sensitive super-learner
- name: CyProduct
  role: Suite containing CypBoM; distributed as CyProduct.jar with EBoMD training
    data in the repository
  repo: https://bitbucket.org/wishartlab/cyproduct/src/master/
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.jcim.1c00144
    title: 'CyProduct: A Software Tool for Accurately Predicting the Byproducts of
      Human Cytochrome P450 Metabolism'
  source_content_hash: sha256:adfde33fc97c94bebcd4f163a2910006542355619c362f83c10422b47a9d8d14
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - CDK (Chemistry Development Kit)
  - Chemistry Development Kit (CDK)
  - CyProduct
  - CypBoM
  - WEKA
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# cross-validation-classifier-evaluation

## Summary

Evaluate a binary classifier (here, the CypBoM reactive-bond predictor per CYP450 isoform) with external 5-fold cross-validation, using a cost-sensitive super-learner whose hyperparameters [β, t, N] are tuned by internal 5-fold CV, and report per-class Jaccard scores TP/(TP+FN+FP).

## When to use

Use when you must reproduce or estimate generalization performance of a per-isoform CYP450 bond-level predictor on EBoMD-style labeled [compound, bond] data, and need externally cross-validated Jaccard scores rather than a single hold-out split.

## When NOT to use

- When the goal is hold-out testing rather than cross-validated generalization estimates — use EBoMD2.sdf (98 compounds) as the separate hold-out instead of re-running CV
- When the goal is generating metabolites from predicted bonds (MetaboGen with SMIRKS rules) rather than evaluating classifier performance
- When the query task is reactant classification (CypReact filter) rather than bond-level SoM prediction

## Inputs

- EBoMD.sdf — BoM-annotated compounds derived from Zaretzki's 679-substrate CYP450 dataset (η-η, η-H, η-SPN bond labels)
- Binary BoMX−Y/non-BoMX−Y labels for each [compound, bond] instance per CYP450 isoform
- CDK-computed feature table (473 features per η-η bond instance)

## Outputs

- Per-isoform external 5-fold cross-validated Jaccard scores (TP/(TP+FN+FP)) for nine CYP450 enzymes (cypbom_cv_jaccard.csv)
- Optimal hyperparameters [β*, t*, N*] per isoform and bond-type classifier
- Predicted reactive bond sets per isoform

## How to apply

Load EBoMD.sdf annotations (828 η-η, 1348 η-H, 86 η-SPN bonds over 679 Zaretzki-derived substrates) and label each [compound, bond] instance as BoMX−Y/non-BoMX−Y per isoform in {1A2, 2A6, 2B6, 2C8, 2C9, 2C19, 2D6, 2E1, 3A4}. Compute the 473 CDK-derived features (7 molecular descriptors, 14 fingerprint bits, 2 ConnectedAtoms, 30 BondFingerprint, 23 BondAtomTypes, 20 BondAtomDescriptors, 92 NeighborAtomTypes, 280 NeighborAtomDescriptor features) after canonically ordering each η-η atom pair by atomic number, then ConnectedAtoms count, then lexicographic AtomDescriptor 10-tuple. Filter constant features, rank the rest by information gain against the BoMX−Y label, and retain top-N features. Because only ~5% of η-η bonds are reactive, run the cost-sensitive super-learner: grid-search all 9 × 3 × 7 = 189 combinations of false-positive cost β ∈ {2, 2.5, 3, ..., 8}, batch size t ∈ {90, 95, 100} and feature count N ∈ {50, 100, 150, 200, 300, 400, All} using internal 5-fold CV on random-forest base learners (WEKA) optimizing Jaccard; refit the winning forest with [β*, t*, N*] on the full training fold. Report the external 5-fold cross-validated Jaccard per isoform and compare against a random classifier baseline.

## Related tools

- **CypBoM** (Bond-of-Metabolism classifier whose per-isoform Jaccard performance is estimated via external 5-fold cross-validation) — https://bitbucket.org/wishartlab/cyproduct/src/master/
- **CDK (Chemistry Development Kit)** (Computes the 473 molecular/bond features that form the classifier input matrix)
- **WEKA** (Provides the random-forest base learners tuned by the cost-sensitive super-learner)
- **CyProduct** (Suite containing CypBoM; distributed as CyProduct.jar with EBoMD training data in the repository) — https://bitbucket.org/wishartlab/cyproduct/src/master/

## Evaluation signals

- Every per-isoform external 5-fold cross-validated Jaccard score falls within the reported 0.380–0.452 range across the nine CYP450 enzymes
- Each classifier beats the random baseline with reported significance p < 3.2E-10
- Jaccard is computed exactly as TP/(TP+FN+FP) per isoform
- Hyperparameter search covers all 189 [β, t, N] combinations via internal 5-fold CV nested inside the external 5-fold splits (no feature-selection leakage across external folds)
- Constant features are removed before information-gain ranking, and only top-N (N ∈ {50, 100, 150, 200, 300, 400, All}) features are retained

## Limitations

- Extreme class imbalance (~5% reactive vs ~95% non-reactive η-η bonds in EBoMD) requires cost-sensitive learning; unweighted evaluation will under-detect reactive bonds
- No software version numbers stated for CDK or WEKA, so exact feature and learner behavior may drift across library versions
- Only 4 of 828 η-η reactions form η+η bonds, so rare bond-forming outcomes are poorly represented in cross-validation folds
- The methods section refers to 'the repository' without an explicit URL; datasets must be located via the Bitbucket/GitHub CyProduct repositories

## Evidence
