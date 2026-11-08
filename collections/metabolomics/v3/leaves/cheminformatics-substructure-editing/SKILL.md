---
name: cheminformatics-substructure-editing
description: Use when generating metabolite structures by applying a SMIRKS reaction
  rule to the substructure of a substrate identified by predicted reactive bond pairs
  (BoM pairs), then merging the product back onto the parent scaffold.
when_to_use_negative:
- The input compound is not first classified as a reactant by CypReact — the CyProduct
  pipeline only proceeds to bond prediction and metabolite generation when CypReact
  predicts R
- You need SoM (site-of-metabolism) ranking only, without structure editing — that
  is the CypBoM task, not the SMIRKS-based metabolite generation step
- The transformation of interest is not among the 27 SMIRKS rules derived from EBoMD
  (e.g. rare η+η-forming reactions, which occurred in only 4 of 828 EBoMD reactions)
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_2258
- http://edamontology.org/topic_3336
tools:
- name: MetaboGen
  role: Module that explores the predicted BoM set, classifies reaction types, applies
    SMIRKS transformations to the substructure, and merges products back onto the
    substrate
  repo: https://bitbucket.org/wishartlab/cyproduct/src/master/
- name: CypBoM
  role: Upstream predictor that supplies the scored reactive 〈X, Y〉 bond pairs used
    to select the substructure to edit
  repo: https://bitbucket.org/wishartlab/cyproduct/src/master/
- name: CyProduct
  role: Command-line wrapper (CyProduct.jar) and Java API (BioTransformerAPI static
    functions) that runs CypReact filtering, CypBoM prediction, and MetaboGen structure
    editing end to end
  repo: https://bitbucket.org/wishartlab/cyproduct/src/master/
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.jcim.1c00144
    title: 'CyProduct: A Software Tool for Accurately Predicting the Byproducts of
      Human Cytochrome P450 Metabolism'
  source_content_hash: sha256:c3b7011b6e6bdfe21f61d4ffdb868edbbe9d14afcd3d8cdc3d1006f2cbc7cb6d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - CyProduct
  - CypBoM
  - MetaboGen
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_2258
  - http://edamontology.org/topic_3336
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# cheminformatics-substructure-editing

## Summary

Apply SMIRKS transformation rules to a predicted reactive substructure of a substrate and merge the edited structure back onto the original molecule, as done by MetaboGen in CyProduct to generate CYP450 metabolites from CypBoM's predicted bonds-of-metabolism (BoMs).

## When to use

Use when generating metabolite structures by applying a SMIRKS reaction rule to the substructure of a substrate identified by predicted reactive bond pairs (BoM pairs), then merging the product back onto the parent scaffold.

## When NOT to use

- The input compound is not first classified as a reactant by CypReact — the CyProduct pipeline only proceeds to bond prediction and metabolite generation when CypReact predicts R
- You need SoM (site-of-metabolism) ranking only, without structure editing — that is the CypBoM task, not the SMIRKS-based metabolite generation step
- The transformation of interest is not among the 27 SMIRKS rules derived from EBoMD (e.g. rare η+η-forming reactions, which occurred in only 4 of 828 EBoMD reactions)

## Inputs

- Set S of predicted reactive 〈X, Y〉 BoM pairs with scores p from the CypBoM module for a given substrate and CYP450 isoform
- Substrate molecule as a SMILES string (format SMILES=yourSmilesString) or an .sdf file
- Reaction_rules.csv containing the 27 SMIRKS transformation strings used by MetaboGen
- Isoform- and bond-type-specific β values from the CypBoM cost matrix (to compute threshold τ = 1/(β+1))

## Outputs

- Predicted metabolite structure(s) obtained by applying SMIRKS transformations to the modified substructure and merging back onto the original substrate
- Per-metabolite attributes: normalized score (p − τ)/(1 − τ) of the highest-scoring contributing 〈X, Y〉 pair and the assigned ReactionType (e.g. Dealkylation, EpOxidation)
- ReactionID / reaction annotation recorded with each generated metabolite

## How to apply

Take the set S of reactive 〈X, Y〉 pairs predicted by CypBoM for a substrate-isoform pair (a bond is reactive when its score p >= τ = 1/(β+1), with isoform- and bond-type-specific β from the cost matrix). Classify each predicted bond into one of MetaboGen's 10 reaction types (RingRearrangement, Reduction, Dealkylation, Deamination, C-O Oxidation, EpOxidation, Hydroxylation, S-Oxidation, N-Oxidation, P-Oxidation) by applying rules in descending importance — e.g. Dealkylation needs an 〈η-η〉 BoM〈i, j〉 plus an 〈η-H〉 BoM〈i, H〉 with i a non-terminal N, O or S; EpOxidation needs a C=C double bond. Extract the subset S' of 〈X, Y〉 pairs modified in one reaction, load the matching SMIRKS string from the 27 transformations in Reaction_rules.csv (derived from the EBoMD substrate-metabolite pairs), and apply it to the substructure. Merge the resulting structure back onto the original substrate, normalize the bond score as (p − τ)/(1 − τ), and assign the highest normalized score among contributing 〈X, Y〉 components to the metabolite along with its reaction type. Normalization can be skipped when calling the tool via API with useCypReact disabled if only structures are needed, but score and reaction type are recorded as metabolite attributes in the standard workflow.

## Related tools

- **MetaboGen** (Module that explores the predicted BoM set, classifies reaction types, applies SMIRKS transformations to the substructure, and merges products back onto the substrate) — https://bitbucket.org/wishartlab/cyproduct/src/master/
- **CypBoM** (Upstream predictor that supplies the scored reactive 〈X, Y〉 bond pairs used to select the substructure to edit) — https://bitbucket.org/wishartlab/cyproduct/src/master/
- **CyProduct** (Command-line wrapper (CyProduct.jar) and Java API (BioTransformerAPI static functions) that runs CypReact filtering, CypBoM prediction, and MetaboGen structure editing end to end) — https://bitbucket.org/wishartlab/cyproduct/src/master/

## Examples

```
java -jar CyProduct.jar CC(=O)Nc1ccc(O)cc1 1A2,2A6,2B6,2C8,2C9,2C19,2D6,2E1,3A4 E:\Users\cyproduct\
```

## Evaluation signals

- For olanzapine R1, the 〈20, 19; Cleavage; R1〉 and 〈20, H; Oxidation; R1〉 BoMs must yield both N-desmethyl olanzapine and formaldehyde
- Running the module on phenacetin, nicotine, and Diuron reproduces the metabolites documented in the article (e.g. Diuron bond 〈6, 5〉 handling)
- Each output metabolite carries a normalized score in (0, 1] computed as (p − τ)/(1 − τ) with τ = 1/(β+1), and an assigned ReactionType among the 10 MetaboGen types
- Only bonds with raw score p >= τ = 1/(β+1) should be treated as reactive and passed to the SMIRKS transformation stage
- Applied transformations should match exactly one of the 27 SMIRKS strings in Reaction_rules.csv, with the product merged back onto the unmodified scaffold of the original substrate

## Limitations

- η+η-forming reactions are very rare — occurring in only 4 of the 828 〈η, η〉 reactions in EBoMD (all were Cyclization) — so such transformations are poorly represented by the 27 SMIRKS rules
- Only 10 reaction types are covered (RingRearrangement, Reduction, Dealkylation, Deamination, C-O/S-O/N-O/P-Oxidation, EpOxidation, Hydroxylation); other CYP450 chemistry is outside MetaboGen's rule set
- Metabolite scores are inherited from the highest-scoring contributing bond pair, so the score reflects bond-level confidence, not metabolite-level likelihood
- The tool requires Java and enzyme names without the 'CYP' prefix (e.g. 1A2, not CYP1A2); errors otherwise

## Evidence

- [methods] MetaboGen: explore predicted BoMX−Y set, extract subset S' modified in one reaction, apply SMIRKS to substructure, merge back to substrate: [evidence span withheld — anchor status: not_found]
- [methods] Dealkylation rule requires paired eta-eta and eta-H BoMs: [evidence span withheld — anchor status: not_found]
- [methods] EpOxidation rule requires carbon-carbon double bond: [evidence span withheld — anchor status: not_found]
