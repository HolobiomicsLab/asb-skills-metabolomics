---
name: eta-eta-bond-canonical-ordering
description: Use when encoding unordered atom-pair bonds (e.g., η-η bonds of metabolism)
  as feature rows for machine learning, to prevent duplicate encodings of the same
  bond from inconsistent atom ordering.
when_to_use_negative:
- When the bond endpoints are already ordered (e.g., directed reaction transformations
  or SMIRKS rules in MetaboGen) — reordering would destroy the intended directionality.
- When working with η+η or η-H bond types, which are encoded by different bond representations
  than the η-η pairs this ordering scheme covers.
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3332
- http://edamontology.org/topic_0091
tools:
- name: Chemistry Development Kit (CDK)
  role: Computes the atom descriptors, ConnectedAtoms values, and bond-level features
    from the canonically ordered atom pairs
- name: CyProduct
  role: Suite in which the CypBoM η-η bond-site predictor consumes the canonically
    ordered bond instances
  repo: https://bitbucket.org/wishartlab/cyproduct/src/master/
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.jcim.1c00144
    title: 'CyProduct: A Software Tool for Accurately Predicting the Byproducts of
      Human Cytochrome P450 Metabolism'
  source_content_hash: sha256:c013e3d04b1df1f4d608378b7985a6bae7f7bede3f79d123829510c5d871383e
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
  - http://edamontology.org/topic_3332
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# eta-eta-bond-canonical-ordering

## Summary

Canonical ordering scheme for η-η bond atom pairs used when building the CypBoM BoM feature instances from EBoMD, ensuring each unordered bond {i,j} maps to exactly one encoding. Without it, the same bond would appear twice with different feature values, corrupting training and evaluation.

## When to use

Use when encoding unordered atom-pair bonds (e.g., η-η bonds of metabolism) as feature rows for machine learning, to prevent duplicate encodings of the same bond from inconsistent atom ordering.

## When NOT to use

- When the bond endpoints are already ordered (e.g., directed reaction transformations or SMIRKS rules in MetaboGen) — reordering would destroy the intended directionality.
- When working with η+η or η-H bond types, which are encoded by different bond representations than the η-η pairs this ordering scheme covers.

## Inputs

- Molecule structures from EBoMD.sdf (679 Zaretzki-derived CYP450 substrate compounds) with η-η bond annotations
- Atom-level properties per bond endpoint: atomic number, ConnectedAtoms (CA) value, 10-tuple AtomDescriptor

## Outputs

- Canonically ordered [compound, bond] instances ready for CDK-based 473-feature extraction
- One unique feature row per η-η bond (no duplicate encodings of the same bond)

## How to apply

For each η-η bond between atoms i and j in a substrate molecule, first order the pair by atomic number so the atom with the smaller atomic number comes first; if atomic numbers are equal, break the tie by the ConnectedAtoms (CA) value; if still tied, order the pair by lexicographic comparison of the 20-dimension AtomDescriptor (10-tuple per atom). This canonical ordering is applied before extracting the 473 CDK-based features (7 molecular descriptors, 14 molecular fingerprint bits, 2 ConnectedAtoms, 30 BondFingerprint, 23 BondAtomTypes, 20 BondAtomDescriptors, 92 NeighborAtomTypes, 280 NeighborAtomDescriptor features) for each [compound, bond] instance in the EBoMD dataset (828 reactive η-η BoMs out of 16418 η-η bonds from 679 Zaretzki-derived compounds). Rationale: bond-site labels are unordered pairs, so a deterministic atom ordering is required to keep one bond = one instance and to make feature extraction reproducible across training and prediction.

## Related tools

- **Chemistry Development Kit (CDK)** (Computes the atom descriptors, ConnectedAtoms values, and bond-level features from the canonically ordered atom pairs)
- **CyProduct** (Suite in which the CypBoM η-η bond-site predictor consumes the canonically ordered bond instances) — https://bitbucket.org/wishartlab/cyproduct/src/master/

## Evaluation signals

- Each η-η bond appears exactly once per compound in the feature table — no duplicate [compound, bond] rows after ordering.
- For a symmetric bond (same atomic numbers and identical AtomDescriptors for both endpoints), swapping the two atoms yields an identical feature row.
- Total instance count matches the expected 16418 η-η bonds from the 679 Zaretzki-derived compounds in EBoMD.sdf.
- Reactive/non-reactive label distribution after ordering matches the ~5% reactive / ~95% non-reactive split of the 828 η-η BoMs.

## Limitations

- The scheme only covers η-η bonds; the paper notes η+η-forming reactions are very rare (4 of 828) and were excluded, so other bond-type orderings are not defined here.
- The ordering rule is defined for the paper's specific AtomDescriptor tuple; changing descriptor definitions breaks the lexicographic tie-break.

## Evidence

- [methods] Workflow step: canonically order each η-η bond atom pair to avoid duplicate encodings: [evidence span withheld — anchor status: not_found]
- [methods] Dataset scope: 828 η-η BoMs in EBoMD from 679 compounds: [evidence span withheld — anchor status: not_found]
