---
name: cdk-molecular-descriptor-feature-generation
description: Use when building a bond-of-metabolism (BoM) or site-of-metabolism classifier
  from annotated SDF compounds (e.g. EBoMD.sdf) and you need per-bond feature vectors
  for training/evaluation.
when_to_use_negative:
- Input is not bond-annotated metabolite/substrate data but raw reaction records without
  BoM labels — no per-bond instances can be formed
- The task is whole-molecule reactant classification (CypReact's job) rather than
  locating reactive bonds within a substrate
- Compounds contain η+η (Cyclization) BoMs only — the paper found these too rare (4
  of 828 reactions) to model and excluded them
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_2258
tools:
- name: Chemistry Development Kit (CDK)
  role: Computes all 473 molecular/bond features per [compound, bond] instance for
    the CypBoM training table
  repo: https://bitbucket.org/wishartlab/cyproduct/src/master/
- name: WEKA
  role: 'Consumes the CDK-generated feature table: constant-feature removal, information-gain
    ranking, top-N selection, and cost-sensitive random-forest training'
  repo: https://bitbucket.org/wishartlab/cyproduct/src/master/
- name: CyProduct
  role: Suite whose CypBoMη−η(α, ·) classifiers are trained on the CDK-generated features;
    EBoMD.sdf in its Datasets folder provides the training input
  repo: https://bitbucket.org/wishartlab/cyproduct/src/master/
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.jcim.1c00144
    title: 'CyProduct: A Software Tool for Accurately Predicting the Byproducts of
      Human Cytochrome P450 Metabolism'
  source_content_hash: sha256:7ab7e5f88d8ba092df1ad5b56f980fd5c2075b4a9ba8da2756dc1a42d46545e1
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Chemistry Development Kit (CDK)
  - CyProduct
  - CypBoM
  - EBoMD
  - WEKA
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# cdk-molecular-descriptor-feature-generation

## Summary

Generates the 473-feature vector per [compound, bond] instance that underlies the CypBoM η−η bond-site predictor, using the Chemistry Development Kit (CDK) to compute molecular descriptors, fingerprints, bond atom types, and neighbor atom features from canonically ordered atom pairs.

## When to use

Use when building a bond-of-metabolism (BoM) or site-of-metabolism classifier from annotated SDF compounds (e.g. EBoMD.sdf) and you need per-bond feature vectors for training/evaluation.

## When NOT to use

- Input is not bond-annotated metabolite/substrate data but raw reaction records without BoM labels — no per-bond instances can be formed
- The task is whole-molecule reactant classification (CypReact's job) rather than locating reactive bonds within a substrate
- Compounds contain η+η (Cyclization) BoMs only — the paper found these too rare (4 of 828 reactions) to model and excluded them

## Inputs

- EBoMD.sdf — annotated compounds derived from the Zaretzki CYP450 substrate dataset of 679 compounds, with η−η BoM labels (828 reactive η−η BoMs out of 16418 η−η bonds; η+η pairs excluded since only 4 of 828 were Cyclization)
- Per-[compound, bond] atom-pair instances with canonically ordered atom indices

## Outputs

- 473-feature table per [compound, bond] instance (7 molecular descriptors, 14 molecular fingerprint bits, 2 ConnectedAtoms, 30 BondFingerprint, 23 SYBYL BondAtomTypes, 20 BondAtomDescriptors, 92 NeighborAtomTypes at depths 1-4, 280 NeighborAtomDescriptor features)
- Feature-ranked instance table with BoMX−Y/non-BoMX−Y labels, ready for information-gain feature selection and cost-sensitive random-forest training in WEKA

## How to apply

First canonically order each η−η bond atom pair (by atomic number, then ConnectedAtoms value, then lexicographic AtomDescriptor 10-tuple order) so the same bond is never encoded twice. Then use CDK to extract 473 features per [compound, bond] instance: 7 molecular descriptors, 14 molecular fingerprint bits, 2 ConnectedAtoms, 30 BondFingerprint, 23 BondAtomTypes (SYBYL), 20 BondAtomDescriptors, 92 NeighborAtomTypes (depths 1-4), and 280 NeighborAtomDescriptor features. Downstream, remove constant features, rank the rest by information gain against the BoMX−Y/non-BoMX−Y label, and retain the top-N (N ∈ {50,100,150,200,300,400,All}) chosen by internal cross-validation. The class distribution (~5% reactive η−η bonds vs ~95% non-reactive in EBoMD) is why the downstream learner must be cost-sensitive with cost matrix 0/0/1/β rather than accuracy-driven.

## Related tools

- **Chemistry Development Kit (CDK)** (Computes all 473 molecular/bond features per [compound, bond] instance for the CypBoM training table) — https://bitbucket.org/wishartlab/cyproduct/src/master/
- **WEKA** (Consumes the CDK-generated feature table: constant-feature removal, information-gain ranking, top-N selection, and cost-sensitive random-forest training) — https://bitbucket.org/wishartlab/cyproduct/src/master/
- **CyProduct** (Suite whose CypBoMη−η(α, ·) classifiers are trained on the CDK-generated features; EBoMD.sdf in its Datasets folder provides the training input) — https://bitbucket.org/wishartlab/cyproduct/src/master/

## Evaluation signals

- Feature count per instance is exactly 473 (7+14+2+30+23+20+92+280); any deviation indicates a dropped or duplicated feature family
- No duplicate encodings: each η−η bond appears once because pairs are canonically ordered by atomic number, then CA, then AtomDescriptor lexicographic order
- Constant features (same value for all instances) are removed before information-gain ranking
- Reactive:non-reactive class ratio in the labeled table approximates 5%:95%, matching EBoMD's known imbalance
- Downstream external 5-fold CV Jaccard scores for all nine isoforms (CYP1A2…CYP3A4) exceed the random classifier baseline, consistent with p < 3.2E-10 significance

## Limitations

- No CDK or WEKA version numbers are stated in the paper, complicating exact reproduction
- Feature scheme is specific to η−η bonds; η+η reactions were excluded as too rare (4/828) to train on
- Only one bond of each pair is encoded per the canonical ordering rule — non-canonical atom indexing will silently produce inconsistent features

## Evidence

- [methods] All features above are computed using the Chemistry Development Kit (CDK)27: "All features above are computed using the Chemistry Development Kit (CDK)27"
- [methods] if the atomic numbers of atom i and atom j are different, then the pair is ordered to start with the atom having the smaller atomic number: "if the atomic numbers of atom i and atom j are different, then the pair is ordered to start with the atom having the smaller atomic number"
- [methods] the percentage of reactive η-η bonds (5%) is much less than the number of non-reactive η-η bonds (95%): "the percentage of reactive η-η bonds (5%) is much less than the number of non-reactive η-η bonds (95%)"
