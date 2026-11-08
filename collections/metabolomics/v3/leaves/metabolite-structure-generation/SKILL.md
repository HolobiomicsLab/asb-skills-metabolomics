---
name: metabolite-structure-generation
description: Use when CypBoM has flagged reactive bonds (〈X, Y〉 pairs with p ≥ τ =
  1/(β+1)) for a substrate–CYP450 isoform pair and you must convert those BoM predictions
  into concrete metabolite structures with per-metabolite scores.
when_to_use_negative:
- CypReact classifies the compound as a non-reactant for the queried enzymes under
  the default pipeline — the filter gates entry, since 'the prediction process proceeds
  only if CypReact classifies the input compound as R (reactant)'.
- Only SoM localization is needed and product structures are not — run CypBoM alone;
  MetaboGen's SMIRKS step adds metabolite generation on top of bond predictions.
- The biotransformation of interest is not among the 10 MetaboGen reaction types encoded
  in the 27 SMIRKS rules of Reaction_rules.csv.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3316
- http://edamontology.org/topic_3407
tools:
- name: MetaboGen
  role: 'Core generator: classifies CypBoM 〈X, Y〉 pairs into 10 reaction types, applies
    the matching SMIRKS transformation from Reaction_rules.csv, and merges the product
    back onto the substrate'
  repo: https://bitbucket.org/wishartlab/cyproduct/src/master/
- name: CypBoM
  role: Upstream predictor supplying the reactive 〈X, Y〉 and 〈η-H〉 BoM pairs and their
    probabilities p (reactive when p ≥ τ = 1/(β+1)) that MetaboGen consumes
  repo: https://bitbucket.org/wishartlab/cyproduct/src/master/
- name: CyProduct
  role: 'Pipeline wrapper (CyProduct.jar): CypReact filter → CypBoM → MetaboGen; writes
    predicted metabolites to the given output folder'
  repo: https://bitbucket.org/wishartlab/cyproduct/src/master/
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.jcim.1c00144
    title: 'CyProduct: A Software Tool for Accurately Predicting the Byproducts of
      Human Cytochrome P450 Metabolism'
  source_content_hash: sha256:898fca83256bc0c458b6cdf3a41144e0edcf57602fa37cf9f767cdf8f984a6fe
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - CyProduct
  - CypBoM
  - MetaboGen
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3316
  - http://edamontology.org/topic_3407
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# metabolite-structure-generation

## Summary

Encapsulates the MetaboGen step of CyProduct: converting CypBoM-predicted bonds-of-metabolism into concrete metabolite structures by classifying each 〈X, Y〉 pair into one of 10 reaction types, applying the matching SMIRKS rule from Reaction_rules.csv, and merging the transformed substructure back onto the substrate with a normalized confidence score.

## When to use

Use when CypBoM has flagged reactive bonds (〈X, Y〉 pairs with p ≥ τ = 1/(β+1)) for a substrate–CYP450 isoform pair and you must convert those BoM predictions into concrete metabolite structures with per-metabolite scores.

## When NOT to use

- CypReact classifies the compound as a non-reactant for the queried enzymes under the default pipeline — the filter gates entry, since 'the prediction process proceeds only if CypReact classifies the input compound as R (reactant)'.
- Only SoM localization is needed and product structures are not — run CypBoM alone; MetaboGen's SMIRKS step adds metabolite generation on top of bond predictions.
- The biotransformation of interest is not among the 10 MetaboGen reaction types encoded in the 27 SMIRKS rules of Reaction_rules.csv.

## Inputs

- QueryMolecule: substrate as a SMILES string (format SMILES=...) or a path to an .sdf file
- enzymeList: CYP450 isoform identifiers without the 'CYP' prefix (e.g. 1A2,2A6,2B6,2C8,2C9,2C19,2D6,2E1,3A4)
- CypBoM output: set S of predicted reactive 〈X, Y〉 and 〈η-H〉 BoM pairs with probabilities p per substrate–isoform pair
- Reaction_rules.csv: the 27 SMIRKS transformation strings from the CyProduct repository

## Outputs

- Predicted metabolite structures per substrate–isoform combination (e.g. N-desmethyl olanzapine plus formaldehyde from olanzapine R1)
- Per-metabolite attributes: normalized Score = (p − τ)/(1 − τ) and ReactionType
- Prediction files written to the user-specified OutputFolderPath (cf. PredictedResults folder in the repository)

## How to apply

Load the 27 SMIRKS transformation strings from Reaction_rules.csv (curated from EBoMD substrate–metabolite pairs) and collect the set S of 〈X, Y〉 pairs CypBoM predicted reactive for each substrate–isoform pair, keeping only bonds with p ≥ τ = 1/(β+1) using the isoform- and bond-type-specific β from the cost matrix. Classify each predicted bond into one of the 10 MetaboGen reaction types (RingRearrangement, Reduction, Dealkylation, Deamination, C-O Oxidation, EpOxidation, Hydroxylation, S-Oxidation, N-Oxidation, P-Oxidation) by applying the rules in descending order of importance — Dealkylation requires an 〈η-η〉 pair 〈i, j〉 plus an 〈η-H〉 pair 〈i, H〉 with i a non-terminal N, O or S; EpOxidation requires a C=C double bond; Reduction requires S/N/P bonded to O; RingRearrangement requires all 〈η-η〉 bonds of a 6-member ring to be predicted. Extract the subset S' of pairs modified in one reaction, apply the matching SMIRKS transformation to that substructure, and merge the resulting structure back onto the original substrate. Normalize each contributing bond score as (p − τ)/(1 − τ), assign the metabolite the highest normalized score among its 〈X, Y〉 components, and record score and ReactionType as metabolite attributes. Validate by running the example molecules (olanzapine, phenacetin, nicotine, Diuron) and confirming documented metabolites are produced — e.g. olanzapine R1 must yield both N-desmethyl olanzapine and formaldehyde.

## Related tools

- **MetaboGen** (Core generator: classifies CypBoM 〈X, Y〉 pairs into 10 reaction types, applies the matching SMIRKS transformation from Reaction_rules.csv, and merges the product back onto the substrate) — https://bitbucket.org/wishartlab/cyproduct/src/master/
- **CypBoM** (Upstream predictor supplying the reactive 〈X, Y〉 and 〈η-H〉 BoM pairs and their probabilities p (reactive when p ≥ τ = 1/(β+1)) that MetaboGen consumes) — https://bitbucket.org/wishartlab/cyproduct/src/master/
- **CyProduct** (Pipeline wrapper (CyProduct.jar): CypReact filter → CypBoM → MetaboGen; writes predicted metabolites to the given output folder) — https://bitbucket.org/wishartlab/cyproduct/src/master/

## Examples

```
java -jar CyProduct.jar CC(=O)Nc1ccc(O)cc1 1A2,2A6,2B6,2C8,2C9,2C19,2D6,2E1,3A4 E:\Users\cyproduct\
```

## Evaluation signals

- Olanzapine must yield both N-desmethyl olanzapine and formaldehyde via 〈20, 19; Cleavage; R1〉 and 〈20, H; Oxidation; R1〉.
- Each metabolite's recorded score equals (prob−threshold)/(1−threshold) with threshold = 1/(1+beta) for the isoform/bond type; scores outside [0, 1] or missing ReactionType attributes flag a broken run.
- Documented metabolites for phenacetin, nicotine and Diuron are reproduced when the module runs on those example molecules.
- Every generated metabolite traces back to a subset S' of contributing 〈X, Y〉 pairs modified in one reaction; a metabolite with no contributing BoM pair indicates a merge/attachment failure.
- Only the 10 MetaboGen reaction types from the 27-SMIRKS Reaction_rules.csv appear in outputs; any other type label signals a rules-file mismatch.

## Limitations

- Coverage is bounded by the 27 SMIRKS rules and 10 reaction types derived from EBoMD; transformations outside that catalog are never generated.
- Only bonds with p ≥ τ = 1/(β+1) reach MetaboGen, and reactive η-η bonds are just ~5% of EBoMD (vs ~95% non-reactive), so β/threshold choices directly govern metabolite recall.
- Multi-enzyme runs (e.g. enzyme list 1A2,3A4) return the union of metabolites catalyzed by any listed enzyme, so isoform-specific attribution requires separate per-enzyme runs; the CypReact filter can also be disabled via the API's useCypReact=false, changing the candidate set.

## Evidence

- [methods] MetaboGen (1) explores the set S of BoMX−Ys ... (2) extracts the subset S’ ... (3) applies SMIRKS transformations ... (4) merges the resulting structures: [evidence span withheld — anchor status: not_found]
- [methods] Dealkylation: occurs if there is one 〈η-η〉 BoMX−Y 〈i, j〉 and one 〈η-H〉 BoMX−Y 〈i, H〉 where atom i is N (Nitrogen), O (Oxygen) or S (Sulfur): [evidence span withheld — anchor status: not_found]
