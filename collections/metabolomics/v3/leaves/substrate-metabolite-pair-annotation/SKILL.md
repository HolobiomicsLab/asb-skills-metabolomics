---
name: substrate-metabolite-pair-annotation
description: Use when you need to convert a site-of-metabolism (SoM) dataset into
  atom-pair BoM annotations, or when reproducing/verifying the 2262-record EBoMD annotation
  counts (828 η-η, 1348 η-H, 86 η-SPN).
when_to_use_negative:
- Input is already a BoM-annotated dataset and the goal is metabolite structure generation
  — use MetaboGen instead.
- You only have SoM labels and cannot locate the corresponding metabolite structures
  for substrate-metabolite pair curation; BoM categories cannot be assigned from SoMs
  alone.
- The task is reactant-vs-nonreactant classification for a CYP450 isoform — that is
  CypReact's scope, not BoM annotation.
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_2258
- http://edamontology.org/topic_3336
tools:
- name: RDKit
  role: SDF parser used to read EBoMD.sdf records and their stored BoM annotations
    when counting per-category records
- name: pandas
  role: Tabulation of the per-category BoM counts into ebomd_bond_counts.csv
- name: CyProduct
  role: Source suite whose CypBoM module consumes the curated EBoMD BoM annotations
    for training
  repo: https://bitbucket.org/wishartlab/cyproduct/src/master/
provenance:
  source_task_ids:
  - task_007
  - task_004
  source_papers:
  - doi: 10.1021/acs.jcim.1c00144
    title: 'CyProduct: A Software Tool for Accurately Predicting the Byproducts of
      Human Cytochrome P450 Metabolism'
  source_content_hash: sha256:7600ec9dda68823365e20a8f618e1910862270ce4e6d7be6a0f795514328e41f
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - CyProduct
  - CypBoM
  - MetaboGen
  - RDKit
  - pandas
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_2258
  - http://edamontology.org/topic_3336
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 21
---

# substrate-metabolite-pair-annotation

## Summary

Curate a Bond-of-Metabolism (BoM) dataset by converting SoM annotations into explicit substrate-metabolite pair records (η-η, η-H, η-SPN categories), the data foundation used to train CypBoM and derive MetaboGen's SMIRKS rules from the EBoMD.

## When to use

Use when you need to convert a site-of-metabolism (SoM) dataset into atom-pair BoM annotations, or when reproducing/verifying the 2262-record EBoMD annotation counts (828 η-η, 1348 η-H, 86 η-SPN).

## When NOT to use

- Input is already a BoM-annotated dataset and the goal is metabolite structure generation — use MetaboGen instead.
- You only have SoM labels and cannot locate the corresponding metabolite structures for substrate-metabolite pair curation; BoM categories cannot be assigned from SoMs alone.
- The task is reactant-vs-nonreactant classification for a CYP450 isoform — that is CypReact's scope, not BoM annotation.

## Inputs

- EBoMD.sdf (compounds derived from Zaretzki's CYP450 substrate dataset of 679 compounds, with stored BoM annotations)
- Zaretzki SoM dataset of 679 substrates (raw starting point for literature curation)
- PubChem/DrugBank entries for compound identity checking

## Outputs

- Per-category BoM counts (η-η, η-H, η-SPN) e.g. ebomd_bond_counts.csv
- Annotated BoM records classified as η-η, η-H or η-SPN, keyed by ReactionID
- Curated substrate-metabolite pairs underlying the 27 SMIRKS rules in Reaction_rules.csv

## How to apply

Start from the Zaretzki SoM dataset of 679 CYP450 substrates and convert each SoM to a corresponding BoM by curating substrate-metabolite pairs from the literature, classifying each annotation as η-η (bond between two connected non-hydrogen atoms), η-H (atom-to-hydrogen bond replaced by a heteroatom), or η-SPN (new bond to S, P or N via lone-pair sharing). Load the resulting EBoMD.sdf with RDKit and iterate over every annotated bond record, deduplicating η-η canonical ordering (〈i,j〉 and 〈j,i〉 are the same bond) and grouping multi-product reactions by their shared ReactionID. Note the strong class imbalance — reactive η-η bonds are only ~5% vs ~95% non-reactive — which motivates the cost-sensitive learning downstream. The η-η annotations also feed MetaboGen: the 27 SMIRKS strings in Reaction_rules.csv were derived by cataloging the substrate-metabolite pairs in EBoMD, and η+η-forming Cyclizations are extremely rare (4 of 828). Write per-category counts (e.g. to ebomd_bond_counts.csv) and compare against the paper's totals.

## Related tools

- **RDKit** (SDF parser used to read EBoMD.sdf records and their stored BoM annotations when counting per-category records)
- **pandas** (Tabulation of the per-category BoM counts into ebomd_bond_counts.csv)
- **CyProduct** (Source suite whose CypBoM module consumes the curated EBoMD BoM annotations for training) — https://bitbucket.org/wishartlab/cyproduct/src/master/

## Examples

```
python -c "from rdkit import Chem; supp=Chem.SDMolSupplier('EBoMD.sdf'); recs=[p for m in supp for p in m.GetProp('BoM').split(';') if p]; print(len(recs))"
```

## Evaluation signals

- Computed counts must equal the paper's totals: 828 η-η + 1348 η-H + 86 η-SPN = 2262 BoM annotations from 679 compounds.
- The η-η count of 828 must include exactly 4 η+η-forming Cyclization reactions.
- Deduplicated η-η pairs: 〈i,j〉 and 〈j,i〉 counted once, with canonical ordering by atomic number, then CA, then AtomDescriptor lexicographic order.
- Class balance check: reactive η-η bonds should be ~5% of η-η records and non-reactive ~95%.
- Multi-product reactions must be counted once per shared ReactionID, not once per product.

## Limitations

- Counts depend on correct deduplication of the η-η canonical ordering and on associating multi-product reactions by their shared ReactionID; double-counting inflates totals.
- The BoM dataset is derived from Zaretzki's 679-substrate SoM set via literature curation, so annotation coverage is limited to those curated substrate-metabolite pairs.
- No software version numbers are stated for CDK/WEKA or curation tools, and the article's discussion section (limitations) is absent from the provided document.

## Evidence

- [methods] EBoMD totals: [evidence span withheld — anchor status: not_found]
- [methods] rare cyclizations: [evidence span withheld — anchor status: not_found]
