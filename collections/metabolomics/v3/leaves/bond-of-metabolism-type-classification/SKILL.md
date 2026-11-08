---
name: bond-of-metabolism-type-classification
description: Use when loading EBoMD.sdf (Zaretzki-derived 679-compound substrate set)
  to audit, reproduce, or re-label BoM annotations by category — e.g. to confirm 828
  η-η + 1348 η-H + 86 η-SPN = 2262 records.
when_to_use_negative:
- The goal is metabolite structure generation or scoring from predicted BoM pairs
  — that is the MetaboGen/CypBoM workflow, not annotation auditing of the curated
  EBoMD labels.
- The input is EBoMD2.sdf, 60Reactants.sdf, 30NonReactants.sdf, or the BioTransformer
  hold-out SDF — these hold-out sets are for testing/reactant-filtering evaluation,
  not the 679-compound BoM annotation census.
- The task is predicting whether a bond is reactive (CypBoM model training/inference
  with cost-matrix β and threshold τ = 1/(β+1)) rather than classifying already-annotated
  BoM records.
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3345
tools:
- name: RDKit
  role: SDF parser used to read EBoMD.sdf compound records and their stored BoM annotations
- name: pandas
  role: Tabulate and export the per-category BoM counts to ebomd_bond_counts.csv
- name: CyProduct
  role: Source repository providing the EBoMD.sdf dataset being classified and counted
  repo: https://bitbucket.org/wishartlab/cyproduct/src/master/
provenance:
  source_task_ids:
  - task_004
  - task_007
  source_papers:
  - doi: 10.1021/acs.jcim.1c00144
    title: 'CyProduct: A Software Tool for Accurately Predicting the Byproducts of
      Human Cytochrome P450 Metabolism'
  source_content_hash: sha256:0960a750fc0883cdd1eaccbb114309025d92aa52e658589685da51b46f1a774d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - CyProduct
  - CypBoM
  - MetaboGen
  - RDKit
  - pandas
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3345
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# bond-of-metabolism-type-classification

## Summary

Classifies every annotated bond-of-metabolism (BoM) record in the CyProduct EBoMD dataset into one of three 4-tuple categories (η-η, η-H, η-SPN) and counts them per category, enabling verification of the dataset's reported composition of 2262 BoM annotations derived from 679 compounds.

## When to use

Use when loading EBoMD.sdf (Zaretzki-derived 679-compound substrate set) to audit, reproduce, or re-label BoM annotations by category — e.g. to confirm 828 η-η + 1348 η-H + 86 η-SPN = 2262 records.

## When NOT to use

- The goal is metabolite structure generation or scoring from predicted BoM pairs — that is the MetaboGen/CypBoM workflow, not annotation auditing of the curated EBoMD labels.
- The input is EBoMD2.sdf, 60Reactants.sdf, 30NonReactants.sdf, or the BioTransformer hold-out SDF — these hold-out sets are for testing/reactant-filtering evaluation, not the 679-compound BoM annotation census.
- The task is predicting whether a bond is reactive (CypBoM model training/inference with cost-matrix β and threshold τ = 1/(β+1)) rather than classifying already-annotated BoM records.

## Inputs

- EBoMD.sdf (CyProduct repository Datasets folder; compounds derived from Zaretzki's CYP450 substrate dataset of 679 compounds, with stored BoM annotations)
- BoM 4-tuple annotation records (atom indices and bond categories per compound)

## Outputs

- ebomd_bond_counts.csv with three per-category counts (η-η, η-H, η-SPN)
- Per-record category labels (η-η / η-H / η-SPN) keyed by compound and ReactionID

## How to apply

Load the deposited EBoMD.sdf from the Datasets folder of the CyProduct repository with an SDF parser such as RDKit and iterate over every annotated bond record in each compound. Classify each record by its BoM 4-tuple category: η-η (bond between two connected non-hydrogen atoms), η-H (atom-to-hydrogen bond replaced by a heteroatom), and η-SPN (newly formed bond to sulfur, phosphorus, or nitrogen via lone-pair sharing). For η-η records, deduplicate by applying the canonical ordering rule — if the atomic numbers of atoms i and j differ, order the pair to start with the atom of smaller atomic number, then by CA, then by AtomDescriptor lexicographic order — so 〈i, j〉 and 〈j, i〉 are counted once. Associate multi-product reactions by their shared ReactionID so a reaction is not double-counted across product entries. Write the three per-category counts to ebomd_bond_counts.csv and validate against the reported totals of 828 η-η, 1348 η-H, and 86 η-SPN (sum 2262), noting the η-η total must be consistent with the finding that only 4 of 828 reactions were η+η-forming (all Cyclizations).

## Related tools

- **RDKit** (SDF parser used to read EBoMD.sdf compound records and their stored BoM annotations)
- **pandas** (Tabulate and export the per-category BoM counts to ebomd_bond_counts.csv)
- **CyProduct** (Source repository providing the EBoMD.sdf dataset being classified and counted) — https://bitbucket.org/wishartlab/cyproduct/src/master/

## Examples

```
from rdkit import Chem; mols = Chem.SDMolSupplier('Datasets/EBoMD.sdf'); counts = {'η-η': 0, 'η-H': 0, 'η-SPN': 0}  # then classify each stored BoM annotation and sum per category
```

## Evaluation signals

- Computed counts must equal the reported totals: 828 η-η + 1348 η-H + 86 η-SPN = 2262 BoM annotations from 679 compounds.
- The η-η total of 828 must be consistent with η+η-forming reactions occurring in only 4 of 828 reactions (all Cyclizations).
- Deduplication invariant: 〈i, j〉 and 〈j, i〉 η-η pairs collapse to a single record after canonical ordering (atomic number, then CA, then AtomDescriptor lexicographic).
- Class balance sanity check: reactive η-η bonds should account for roughly 5% of η-η records vs ~95% non-reactive.
- Multi-product reactions grouped by shared ReactionID must contribute exactly one reaction record each, not one per product.

## Limitations

- Severe class imbalance: only ~5% of η-η bonds are reactive vs ~95% non-reactive, so per-category counts alone do not indicate reactivity.
- η+η-forming reactions are extremely rare (4 of 828 η-η reactions), so counts of this subtype are statistically fragile.
- Without the canonical-ordering dedup rule, η-η bonds can be double-counted (〈i, j〉 vs 〈j, i〉), inflating counts above 828.

## Evidence

- [readme] EBoMD.sdf contains the compounds derived from the Zaretzki’s CYP450 substrate datasets of 679 compounds: "EBoMD.sdf contains the compounds derived from the Zaretzki’s CYP450 substrate datasets of 679 compounds"
- [methods] The EBoMD ("Edmonton Bond-of-Metabolism Dataset") collection contains the 828 η-η BoMs: [evidence span withheld — anchor status: not_found]
- [methods] η+η-forming reactions are very rare -- occurring in only 4 of the 828 〈η, η〉 reactions: [evidence span withheld — anchor status: not_found]
- [methods] Canonical ordering rule for η-η pairs: "if the atomic numbers of atom i and atom j are different, then the pair is ordered to start with the atom having the smaller atomic number"
- [methods] Reactive vs non-reactive η-η balance: "the percentage of reactive η-η bonds (5%) is much less than the number of non-reactive η-η bonds (95%)"
