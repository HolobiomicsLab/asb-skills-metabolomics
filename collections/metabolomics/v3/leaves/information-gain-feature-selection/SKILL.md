---
name: information-gain-feature-selection
description: Use when a high-dimensional per-instance feature table (here, 473 CDK-derived
  bond features over 679 Zaretzki-derived substrates) must be reduced before training
  a classifier, and the retained dimensionality should be learned per class via internal
  cross-validation rather than fixed a priori.
when_to_use_negative:
- Input compounds are unlabeled (no BoMX−Y/non-BoMX−Y annotation), so information
  gain against the label cannot be computed.
- The prediction target is not the per-isoform reactive-bond binary label defined
  here — e.g., reactant/non-reactant classification, which CypReact handles, or byproduct
  generation, which MetaboGen handles.
- A single global feature subset is required across all nine CYP450 isoforms; this
  pipeline learns a separate N* and ranking for each D(α).
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_2258
tools:
- name: Chemistry Development Kit (CDK)
  role: Computes the 473 per-instance bond/molecular features that are subsequently
    filtered and ranked
- name: WEKA
  role: Supplies the information-gain attribute evaluation and the random-forest base
    learners used with the selected top-N features
- name: CypBoM
  role: Consumer of the selected feature subsets; its cost-sensitive super-learner
    chooses N among {50, 100, 150, 200, 300, 400, All} by internal 5-fold cross-validation
  repo: https://bitbucket.org/wishartlab/cyproduct/src/master/
provenance:
  source_task_ids:
  - task_002
  - task_006
  source_papers:
  - doi: 10.1021/acs.jcim.1c00144
    title: 'CyProduct: A Software Tool for Accurately Predicting the Byproducts of
      Human Cytochrome P450 Metabolism'
  source_content_hash: sha256:161aa3a52894d50b5708ec87009852ba3332546efc5f89a60e98f3c44d7c9543
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
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# information-gain-feature-selection

## Summary

Per-class feature reduction used by the CypBoM reactive-bond predictor: drop CDK features that are constant across all [compound, bond] instances, rank the remainder by information gain against the binary BoMX−Y/non-BoMX−Y label, and retain the top-N features with N learned by internal cross-validation.

## When to use

Use when a high-dimensional per-instance feature table (here, 473 CDK-derived bond features over 679 Zaretzki-derived substrates) must be reduced before training a classifier, and the retained dimensionality should be learned per class via internal cross-validation rather than fixed a priori.

## When NOT to use

- Input compounds are unlabeled (no BoMX−Y/non-BoMX−Y annotation), so information gain against the label cannot be computed.
- The prediction target is not the per-isoform reactive-bond binary label defined here — e.g., reactant/non-reactant classification, which CypReact handles, or byproduct generation, which MetaboGen handles.
- A single global feature subset is required across all nine CYP450 isoforms; this pipeline learns a separate N* and ranking for each D(α).

## Inputs

- EBoMD.sdf — BoM-annotated compounds from Zaretzki's 679-substrate CYP450 dataset (828 η-η, 1348 η-H, 86 η-SPN bonds)
- Per-[compound, bond] feature matrix of 473 CDK-derived features
- Binary BoMX−Y / non-BoMX−Y labels per CYP450 isoform D(α)

## Outputs

- Information-gain-ranked feature list per isoform
- Reduced top-N feature matrix (N ∈ {50, 100, 150, 200, 300, 400, All}) per isoform
- Selected N* per isoform, consumed by the cost-sensitive super-learner grid search

## How to apply

First compute the 473 CDK features for each [compound, bond] instance (7 molecular descriptors, 14 molecular fingerprint bits, 2 ConnectedAtoms, 30 BondFingerprint, 23 BondAtomTypes, 20 BondAtomDescriptors, 92 NeighborAtomTypes, 280 NeighborAtomDescriptor features). Remove features whose values are identical for every instance in the dataset. For each CYP450 isoform α ∈ {1A2, 2A6, 2B6, 2C8, 2C9, 2C19, 2D6, 2E1, 3A4}, compute the information gain of each remaining feature with respect to the BoMX−Y/non-BoMX−Y label defined on dataset D(α), and rank features by that gain. Retain the top-N attributes and drop the rest, with N drawn from {50, 100, 150, 200, 300, 400, All}; N is learned by internal 5-fold cross-validation so that the downstream cost-sensitive super-learner can grid-search [beta ∈ {2, 2.5, ..., 8}, batch size t ∈ {90, 95, 100}, N] over all 189 combinations optimizing the Jaccard score. The selection must be redone per isoform because labels D(α) differ across enzymes; the selected feature subset then feeds the final random-forest refit with optimal [beta*, t*, N*].

## Related tools

- **Chemistry Development Kit (CDK)** (Computes the 473 per-instance bond/molecular features that are subsequently filtered and ranked)
- **WEKA** (Supplies the information-gain attribute evaluation and the random-forest base learners used with the selected top-N features)
- **CypBoM** (Consumer of the selected feature subsets; its cost-sensitive super-learner chooses N among {50, 100, 150, 200, 300, 400, All} by internal 5-fold cross-validation) — https://bitbucket.org/wishartlab/cyproduct/src/master/

## Evaluation signals

- Feature count before selection equals 473 (7 + 14 + 2 + 30 + 23 + 20 + 92 + 280 CDK features); any feature constant across all instances is removed before ranking.
- The chosen N* for every isoform must be one of {50, 100, 150, 200, 300, 400, All}.
- Ranking and N selection are repeated for each of the nine isoforms (1A2, 2A6, 2B6, 2C8, 2C9, 2C19, 2D6, 2E1, 3A4), yielding up to nine distinct feature subsets.
- With the selected features, external 5-fold cross-validated Jaccard scores per isoform fall in the reported 0.380–0.452 range and beat the random baseline (p < 3.2E-10).
- Class imbalance (~5% reactive vs ~95% non-reactive η-η bonds) must be preserved in the selected-feature training data, matching the cost-sensitive calibration.

## Limitations

- N is only searched over the fixed grid {50, 100, 150, 200, 300, 400, All}; other dimensionalities are never considered.
- Feature selection is per-isoform; results do not transfer directly to enzymes outside the nine major human CYP450 isoforms.
- No software version numbers are stated for CDK or WEKA, so exact information-gain implementations may vary across versions.
- Selection depends on the internal 5-fold cross-validation split; running it outside the training folds risks leaking label information into the external evaluation.

## Evidence

- [methods] rank the remaining features according to their information gain values ... retain the top-N attributes ... where the value N is learned by internal cross-validation: [evidence span withheld — anchor status: not_found]
- [methods] our SL takes as input a training dataset D, explores all 9 * 3 * 7 = 189 possible combinations for the parameters [beta, t, N] using an internal 5-fold cross-validation: [evidence span withheld — anchor status: not_found]
