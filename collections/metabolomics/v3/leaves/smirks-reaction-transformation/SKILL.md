---
name: smirks-reaction-transformation
description: Use when you have a substrate structure plus a set of predicted reactive
  η-η / η-H bond pairs for a CYP450 isoform and need to transform the matching substructures
  into metabolite structures.
when_to_use_negative:
- The compound has not been classified as a reactant by CypReact — CyProduct proceeds
  only if CypReact returns R (reactant).
- No reactive bonds have been predicted (e.g. CypBoM found no 〈η-η〉 or 〈η-H〉 BoM above
  threshold), so there is no subset S' to transform.
- The task is reactant/substrate classification rather than metabolite generation
  — use CypReact instead.
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3370
tools:
- name: MetaboGen
  role: Module that determines ReactionType and ReactionID from predicted BoM pairs
    and applies the matching SMIRKS transformation to generate the metabolite
  repo: https://bitbucket.org/wishartlab/cyproduct/src/master/
- name: CypBoM
  role: Upstream predictor supplying the reactive 〈X, Y〉 bond pairs and scores that
    drive which SMIRKS transformation is applied
  repo: https://bitbucket.org/wishartlab/cyproduct/src/master/
- name: CyProduct
  role: CLI wrapper (CyProduct.jar) that runs CypReact → CypBoM → MetaboGen end-to-end;
    exposes the BioTransformerAPI static functions for programmatic metabolite generation
  repo: https://bitbucket.org/wishartlab/cyproduct/src/master/
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.jcim.1c00144
    title: 'CyProduct: A Software Tool for Accurately Predicting the Byproducts of
      Human Cytochrome P450 Metabolism'
  source_content_hash: sha256:72225c1d1b2d2524313d27900d4f66806f4db12bef398052f135f8cc4adb0c59
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - CyProduct
  - CypBoM
  - MetaboGen
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3370
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# smirks-reaction-transformation

## Summary

Applies SMIRKS reaction transformation strings to the reactive bond-of-metabolism (BoM) atom pairs predicted by CypBoM, generating the structure of each CYP450 metabolite as MetaboGen does in CyProduct.

## When to use

Use when you have a substrate structure plus a set of predicted reactive η-η / η-H bond pairs for a CYP450 isoform and need to transform the matching substructures into metabolite structures.

## When NOT to use

- The compound has not been classified as a reactant by CypReact — CyProduct proceeds only if CypReact returns R (reactant).
- No reactive bonds have been predicted (e.g. CypBoM found no 〈η-η〉 or 〈η-H〉 BoM above threshold), so there is no subset S' to transform.
- The task is reactant/substrate classification rather than metabolite generation — use CypReact instead.

## Inputs

- Substrate molecule (SMILES string or path to a .sdf file)
- Set of predicted reactive BoM 〈X, Y〉 pairs with scores from CypBoM for a given CYP450 isoform
- Reaction_rules.csv containing the 27 SMIRKS transformation strings used by MetaboGen

## Outputs

- Predicted metabolite structures (merged substrates with transformed substructures)
- Per-metabolite attributes: normalized score and reaction type (e.g. Hydroxylation, Dealkylation)
- Predicted-results folders (e.g. CyProductResults_For_9CYPs) holding the generated metabolites

## How to apply

Load the 27 SMIRKS transformation strings from Reaction_rules.csv, derived by cataloging the substrate-metabolite pairs curated in the EBoMD dataset. Classify each predicted bond into one of the 10 MetaboGen reaction types (RingRearrangement, Reduction, Dealkylation, Deamination, C-O Oxidation, EpOxidation, Hydroxylation, S-Oxidation, N-Oxidation, P-Oxidation) by applying rules in descending order of importance — e.g. Dealkylation requires an η-η BoM 〈i, j〉 plus an η-H BoM 〈i, H〉 with i being non-terminal N, O or S, while EpOxidation requires a C=C double bond. Extract the subset S' of η-η pairs modified in one reaction, apply the matching SMIRKS transformation to that substructure, and merge the resulting structure back onto the original substrate. Finally normalize each predicted bond score as (p − τ)/(1 − τ), where τ = 1/(β+1) uses the isoform- and bond-type-specific β from the cost matrix, and assign the highest normalized score among contributing pairs to the metabolite as an attribute alongside the reaction type.

## Related tools

- **MetaboGen** (Module that determines ReactionType and ReactionID from predicted BoM pairs and applies the matching SMIRKS transformation to generate the metabolite) — https://bitbucket.org/wishartlab/cyproduct/src/master/
- **CypBoM** (Upstream predictor supplying the reactive 〈X, Y〉 bond pairs and scores that drive which SMIRKS transformation is applied) — https://bitbucket.org/wishartlab/cyproduct/src/master/
- **CyProduct** (CLI wrapper (CyProduct.jar) that runs CypReact → CypBoM → MetaboGen end-to-end; exposes the BioTransformerAPI static functions for programmatic metabolite generation) — https://bitbucket.org/wishartlab/cyproduct/src/master/

## Examples

```
java -jar CyProduct.jar SMILES=CC(=O)Nc1ccc(O)cc1 1A2,2A6,2B6,2C8,2C9,2C19,2D6,2E1,3A4 E:\Users\cyproduct\
```

## Evaluation signals

- Run the module on example molecules (olanzapine, phenacetin, nicotine, Diuron) and confirm the documented metabolites are produced — e.g. the olanzapine R1 reaction must yield both N-desmethyl olanzapine and formaldehyde via 〈20, 19; Cleavage; R1〉 and 〈20, H; Oxidation; R1〉.
- Check that each generated metabolite carries both a normalized score of the form (p − τ)/(1 − τ) and a reaction-type label.
- Verify the Diuron bond 〈6, 5〉 case reproduces the expected transformation from the EBoMD-derived rules.
- Confirm the number of distinct SMIRKS transformations applied matches the rules listed in Reaction_rules.csv (27 strings).
- Compare predicted results against the manuscript's hold-out results stored in the PredictedResults folder for the 9 CYP450 enzymes on EBoMD2.

## Limitations

- η+η-forming reactions are very rare — occurring in only 4 of the 828 〈η, η〉 reactions in EBoMD (all were Cyclization) — so Cyclization-type transformations are poorly supported by training data.
- Reactive η-η bonds make up only ~5% of the EBoMD dataset versus ~95% non-reactive, so the underlying bond predictions feeding the transformations are heavily class-imbalanced.
- The score formula depends on the cost-matrix β chosen per isoform and bond type; if β is mis-set, the threshold τ = 1/(1+β) and hence the normalized score are wrong.

## Evidence

- [methods] MetaboGen then generates the ReactionType and ReactionID from the 〈X, Y〉 pairs, and uses that to generate the resulting metabolite: [evidence span withheld — anchor status: not_found]
- [methods] Dealkylation: occurs if there is one 〈η-η〉 BoMX−Y 〈i, j〉 and one 〈η-H〉 BoMX−Y 〈i, H〉 where atom i is N (Nitrogen), O (Oxygen) or S (Sulfur): [evidence span withheld — anchor status: not_found]
- [methods] MetaboGen (1) explores the set S of BoMX−Ys ... (2) extracts the subset S’ ... (3) applies SMIRKS transformations ... (4) merges the resulting structures: [evidence span withheld — anchor status: not_found]
