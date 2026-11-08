---
name: reaction-type-classification
description: Use when counting or categorizing BoM records by ReactionType in an annotated
  SDF dataset (e.g., EBoMD.sdf), or when verifying reported reaction-category frequencies
  such as η+η-forming Cyclization counts.
when_to_use_negative:
- The input is raw structures without BoM annotations and no bond-pair predictions
  exist — ReactionType rules operate on 〈X, Y〉 pairs, not unannotated molecules
- The goal is SoM localization (which atom reacts) rather than reaction-type assignment
  — use the CypBoM / FAME comparison workflow instead
- Analyzing non-CYP450 metabolism where the η-η / η-H / η-SPN BoM taxonomy and SMIRKS
  rule set do not apply
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_3370
tools:
- name: CyProduct
  role: End-to-end tool whose MetaboGen module assigns ReactionType and ReactionID
    from predicted 〈X, Y〉 pairs and applies SMIRKS transformations
  repo: https://bitbucket.org/wishartlab/cyproduct/src/master/
- name: CypBoM
  role: Predicts the reactive BoMX−Y bond pairs with scores that feed the reaction-type
    rules
  repo: https://bitbucket.org/wishartlab/cyproduct/src/master/
- name: MetaboGen
  role: Generates the ReactionType and ReactionID from the 〈X, Y〉 pairs and applies
    the SMIRKS rules from Reaction_rules.csv
  repo: https://bitbucket.org/wishartlab/cyproduct/src/master/
- name: EBoMD
  role: Annotated BoM dataset (828 η-η records) used to train CypBoM and to validate
    reaction-type frequency findings
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.jcim.1c00144
    title: 'CyProduct: A Software Tool for Accurately Predicting the Byproducts of
      Human Cytochrome P450 Metabolism'
  source_content_hash: sha256:0ba084d4bfc9670869b6fd1fe5713fa22e7e224305b03bed0776446b6463e93a
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - CyProduct
  - CypBoM
  - EBoMD
  - MetaboGen
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_3370
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# reaction-type-classification

## Summary

Classifies CYP450 bond-of-metabolism (BoM) records into ReactionType categories (e.g., Cyclization, EpOxidation, Reduction, Dealkylation, Deamination, RingRearrangement) by applying structural rules to predicted η-η / η-H bond pairs, enabling rarity and distribution analyses like the '4 of 828 η+η-forming reactions' finding in EBoMD.

## When to use

Use when counting or categorizing BoM records by ReactionType in an annotated SDF dataset (e.g., EBoMD.sdf), or when verifying reported reaction-category frequencies such as η+η-forming Cyclization counts.

## When NOT to use

- The input is raw structures without BoM annotations and no bond-pair predictions exist — ReactionType rules operate on 〈X, Y〉 pairs, not unannotated molecules
- The goal is SoM localization (which atom reacts) rather than reaction-type assignment — use the CypBoM / FAME comparison workflow instead
- Analyzing non-CYP450 metabolism where the η-η / η-H / η-SPN BoM taxonomy and SMIRKS rule set do not apply

## Inputs

- EBoMD.sdf — annotated BoM dataset derived from Zaretzki's 679-compound CYP450 substrate set (Datasets folder of the repository)
- Predicted BoMX−Y bond pairs from CypBoM for a query molecule and CYP450 isoform
- Reaction_rules.csv — SMIRKS strings used by the MetaboGen module

## Outputs

- Per-record ReactionType labels (Cyclization, EpOxidation, Reduction, Dealkylation, Deamination, RingRearrangement, etc.)
- Category frequency counts (e.g., 4 η+η-forming out of 828 η-η records, all Cyclization)
- CSV of counts and boolean verification results, e.g., ebomd_eta_eta_cyclization_counts.csv

## How to apply

Load the annotated SDF (e.g., EBoMD.sdf with 679 Zaretzki-derived compounds) and parse each BoM record's bond pair type — 〈η-η〉, 〈η-H〉, or 〈η-SPN〉 — and its ReactionType label. For classification of new predictions, MetaboGen assigns the ReactionType from the predicted 〈X, Y〉 pairs by applying rule-based conditions: EpOxidation if the 〈η-η〉 pair is a C=C double bond, Reduction if atom i is S/N/P paired with O, Dealkylation if an 〈η-η〉 〈i, j〉 co-occurs with an 〈η-H〉 〈i, H〉 on N/O/S, Deamination for C–N pairs with an η-H on the carbon, and RingRearrangement when all 〈η-η〉 bonds in a 6-member ring are predicted. For rarity analyses, filter bond-forming records (η+η type) from bond-modifying ones, tally per-category counts, and corroborate with dataset-wide statistics (η-H ≈ 1348, η-SPN ≈ 86, η-η = 828 in EBoMD). Write counts and verification booleans to a CSV (e.g., ebomd_eta_eta_cyclization_counts.csv).

## Related tools

- **CyProduct** (End-to-end tool whose MetaboGen module assigns ReactionType and ReactionID from predicted 〈X, Y〉 pairs and applies SMIRKS transformations) — https://bitbucket.org/wishartlab/cyproduct/src/master/
- **CypBoM** (Predicts the reactive BoMX−Y bond pairs with scores that feed the reaction-type rules) — https://bitbucket.org/wishartlab/cyproduct/src/master/
- **MetaboGen** (Generates the ReactionType and ReactionID from the 〈X, Y〉 pairs and applies the SMIRKS rules from Reaction_rules.csv) — https://bitbucket.org/wishartlab/cyproduct/src/master/
- **EBoMD** (Annotated BoM dataset (828 η-η records) used to train CypBoM and to validate reaction-type frequency findings)

## Examples

```
java -jar CyProduct.jar CC(=O)Nc1ccc(O)cc1 1A2,2A6,2B6,2C8,2C9,2C19,2D6,2E1,3A4 E:\Users\cyproduct\
```

## Evaluation signals

- η-η record count over EBoMD must equal 828; η+η-forming subset must equal 4 with all four labeled Cyclization
- Corroborating tallies: η-H BoM count ≈ 1348 and η-SPN BoM count ≈ 86
- Each assigned ReactionType must be derivable from its rule condition (e.g., EpOxidation only when the 〈i, j〉 pair is a carbon double bond)
- Reactive vs non-reactive η-η split should be ~5% / ~95% as reported for EBoMD
- Rule applications must be consistent with the SMIRKS strings shipped in Reaction_rules.csv

## Limitations

- The η+η-forming class is extremely rare (4 of 828), so category-level classifiers and frequency claims are sensitive to small counts
- RingRearrangement requires ALL 〈η-η〉 bonds in a 6-member ring to be predicted as BoMX−Ys, making it dependent on complete bond-level prediction
- No software version numbers are stated for CDK or WEKA; no explicit repository URL is given in the methods despite references to 'the repository'

## Evidence

- [methods] η+η-forming reactions are very rare -- occurring in only 4 of the 828 〈η, η〉 reactions: [evidence span withheld — anchor status: not_found]
- [methods] MetaboGen generates ReactionType from 〈X, Y〉 pairs: [evidence span withheld — anchor status: not_found]
- [methods] EpOxidation rule condition: [evidence span withheld — anchor status: not_found]
