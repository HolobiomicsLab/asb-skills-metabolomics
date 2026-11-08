---
name: cheminformatics-bond-feature-generation
description: Use when building or reproducing a bond-of-metabolism (BoM) or site-of-metabolism
  predictor that requires numeric features per [compound, bond] instance from SDF/SMILES
  structures, following the CypBoM/CDK feature scheme.
when_to_use_negative:
- The goal is generating metabolite structures from predicted bonds — that is MetaboGen's
  SMIRKS transformation step, not bond featurization.
- The input is already a numeric feature table or descriptor matrix — re-running CDK
  feature computation would be redundant.
- Predicting whether a compound is a reactant at all (compound-level task) — use CypReact,
  which operates before CypBoM in the CyProduct pipeline.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_2258
- http://edamontology.org/topic_3474
tools:
- name: Chemistry Development Kit (CDK)
  role: Computes all 473 bond-level features (descriptors, fingerprint bits, ConnectedAtoms,
    BondFingerprint, BondAtomTypes, BondAtomDescriptors, NeighborAtomTypes, NeighborAtomDescriptors)
    for each [compound, bond] instance.
- name: CypBoM
  role: Consumer of the generated feature matrix; trains cost-sensitive random-forest
    super-learners over [β, t, N] parameter grids to predict reactive BoMX−Y pairs.
  repo: https://bitbucket.org/wishartlab/cyproduct/src/master/
- name: WEKA
  role: Machine-learning backend used by the CypBoM super-learner over the generated
    features.
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.jcim.1c00144
    title: 'CyProduct: A Software Tool for Accurately Predicting the Byproducts of
      Human Cytochrome P450 Metabolism'
  source_content_hash: sha256:8535f16c185ec93b3c14bde8c4e9d0552c8b5a025432b11d28fe45c7a30f76b9
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Chemistry Development Kit (CDK)
  - CyProduct
  - CypBoM
  - WEKA
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_2258
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# cheminformatics-bond-feature-generation

## Summary

Generates per-bond (BoMX−Y) feature vectors from 2D molecular structures for machine-learning prediction of CYP450 reactive bonds, as implemented in the CypBoM module of CyProduct: 473 CDK-derived features computed for each canonically ordered η-η (and η-H, η-SPN) bond instance.

## When to use

Use when building or reproducing a bond-of-metabolism (BoM) or site-of-metabolism predictor that requires numeric features per [compound, bond] instance from SDF/SMILES structures, following the CypBoM/CDK feature scheme.

## When NOT to use

- The goal is generating metabolite structures from predicted bonds — that is MetaboGen's SMIRKS transformation step, not bond featurization.
- The input is already a numeric feature table or descriptor matrix — re-running CDK feature computation would be redundant.
- Predicting whether a compound is a reactant at all (compound-level task) — use CypReact, which operates before CypBoM in the CyProduct pipeline.

## Inputs

- EBoMD.sdf — compounds from Zaretzki's CYP450 substrate dataset (679 compounds) with BoM annotations (828 η-η, 1348 η-H, 86 η-SPN bonds)
- Molecular structures in SDF format or SMILES strings with candidate bonds identified
- Binary BoMX−Y/non-BoMX−Y labels per CYP450 isoform (1A2, 2A6, 2B6, 2C8, 2C9, 2C19, 2D6, 2E1, 3A4)

## Outputs

- Feature matrix of 473 CDK-derived numeric/binary features per [compound, bond] instance
- Canonical atom-pair ordering per η-η bond for orientation-invariant features
- Filtered, information-gain-ranked feature table (constant features removed, top-N retained)

## How to apply

For every [compound, bond] instance in the labeled dataset (e.g. EBoMD.sdf with BoMX−Y/non-BoMX−Y labels per isoform among the nine human CYP450s), canonically order each η-η bond atom pair: smaller atomic number first, then ConnectedAtoms count CA(·), then lexicographic AtomDescriptor 10-tuple order, so features are orientation-invariant. Compute 473 CDK-derived features per instance: 7 molecular descriptors, 14 molecular fingerprint bits, 2 ConnectedAtoms, 30 BondFingerprint, 23 BondAtomTypes, 20 BondAtomDescriptors, 92 NeighborAtomTypes and 280 NeighborAtomDescriptor features; features for η-H and η-SPN bond types are computed analogously. Because reactive η-η bonds are rare (~5% vs ~95% non-reactive in EBoMD), pair the feature table with a cost-sensitive learner downstream rather than resampling here. After assembly, drop features constant across all instances, rank the remainder by information gain with respect to the BoMX−Y label, and retain the top-N features with N learned by internal cross-validation (N ∈ {50, 100, 150, 200, 300, 400, All} in CypBoM's super-learner grid).

## Related tools

- **Chemistry Development Kit (CDK)** (Computes all 473 bond-level features (descriptors, fingerprint bits, ConnectedAtoms, BondFingerprint, BondAtomTypes, BondAtomDescriptors, NeighborAtomTypes, NeighborAtomDescriptors) for each [compound, bond] instance.)
- **CypBoM** (Consumer of the generated feature matrix; trains cost-sensitive random-forest super-learners over [β, t, N] parameter grids to predict reactive BoMX−Y pairs.) — https://bitbucket.org/wishartlab/cyproduct/src/master/
- **WEKA** (Machine-learning backend used by the CypBoM super-learner over the generated features.)

## Evaluation signals

- Feature count per η-η bond instance equals 473 (7 + 14 + 2 + 30 + 23 + 20 + 92 + 280 CDK-derived features).
- Canonical ordering is orientation-invariant: swapping the two bond atoms yields the identical feature vector (smaller atomic number first, then CA(·), then lexicographic AtomDescriptor 10-tuple).
- No feature in the final table is constant across all instances, and remaining features are ordered by information gain with respect to the BoMX−Y label.
- Downstream check: external 5-fold CV Jaccard scores (TP/(TP+FN+FP)) per isoform fall in the reported 0.380–0.452 range and beat the random baseline (p < 3.2E-10).

## Limitations

- Severe class imbalance: reactive η-η bonds are only ~5% of instances vs ~95% non-reactive in EBoMD, so features alone cannot fix the skew — cost-sensitive learning (β up to 8) is required downstream.
- No software version numbers stated for CDK or WEKA, so exact feature-value reproducibility across library versions is not guaranteed.
- The 10-tuple AtomDescriptor lexicographic ordering requires a fully specified descriptor tuple; η+η-forming reactions are very rare (4 of 828 in EBoMD), limiting training signal for bond-breaking/cyclization cases.

## Evidence

- [methods] Feature selection pipeline: [evidence span withheld — anchor status: not_found]
