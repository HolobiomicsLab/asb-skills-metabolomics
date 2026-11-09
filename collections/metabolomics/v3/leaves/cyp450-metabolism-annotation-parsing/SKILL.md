---
name: cyp450-metabolism-annotation-parsing
description: Use when an SDF of BoM-annotated CYP450 substrate/molecule records (e.g.
  EBoMD.sdf or EBoMD2.sdf from the CyProduct repository's Datasets folder) must be
  parsed to count or cross-check η-η, η-H, or η-SPN BoM statistics and ReactionType
  labels.
when_to_use_negative:
- The goal is to predict metabolites/BoMs for a new query molecule with CyProduct
  rather than verify dataset statistics in the released EBoMD annotation files
- The input is an unannotated substrate SDF without BoM atom-pair descriptors — there
  are no η-η/η-H/η-SPN labels to parse
- The analysis targets the BioTransformer hold-out set (13321_2018_324_MOESM6_ESM.sdf)
  or HDFame 60Reactants/30NonReactants files, which are not the BoM-annotated EBoMD
  records
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3318
- http://edamontology.org/topic_2258
tools:
- name: EBoMD.sdf
  role: 'Primary input: BoM-annotated SDF of the 679 Zaretzki-derived compounds used
    to train CypBoM, containing the 828 η-η BoM records to parse'
  repo: https://bitbucket.org/wishartlab/cyproduct/src/master/
- name: CDK
  role: Chemistry toolkit used in the CyProduct pipeline for computing per-bond features
    from the parsed [compound, bond] instances; usable for SDF parsing and atom-pair
    extraction
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.jcim.1c00144
    title: 'CyProduct: A Software Tool for Accurately Predicting the Byproducts of
      Human Cytochrome P450 Metabolism'
  source_content_hash: sha256:c625b9f1d82a2c9d26c74f7bc9f7b7d0c4eb8c1cb7d3aca5f40c6333649f8d23
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - CDK
  - EBoMD
  - EBoMD.sdf
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3318
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# cyp450-metabolism-annotation-parsing

## Summary

Parse and tally the annotated Bond-of-Metabolism (BoM) records in the EBoMD SDF files (e.g. the 828 η-η records derived from Zaretzki's 679 CYP450 substrates) so that dataset statistics and per-ReactionType counts, such as the 4-of-828 η+η Cyclization rarity finding, can be independently verified.

## When to use

Use when an SDF of BoM-annotated CYP450 substrate/molecule records (e.g. EBoMD.sdf or EBoMD2.sdf from the CyProduct repository's Datasets folder) must be parsed to count or cross-check η-η, η-H, or η-SPN BoM statistics and ReactionType labels.

## When NOT to use

- The goal is to predict metabolites/BoMs for a new query molecule with CyProduct rather than verify dataset statistics in the released EBoMD annotation files
- The input is an unannotated substrate SDF without BoM atom-pair descriptors — there are no η-η/η-H/η-SPN labels to parse
- The analysis targets the BioTransformer hold-out set (13321_2018_324_MOESM6_ESM.sdf) or HDFame 60Reactants/30NonReactants files, which are not the BoM-annotated EBoMD records

## Inputs

- EBoMD.sdf — annotated BoM records for 679 Zaretzki-derived CYP450 substrates (from the CyProduct repository's Datasets folder)
- EBoMD2.sdf — hold-out set of 98 relevant compounds, if hold-out statistics must be parsed separately

## Outputs

- Count table (e.g. ebomd_eta_eta_cyclization_counts.csv) with η-η total, η+η-forming total, ReactionType labels, η-H and η-SPN tallies
- Boolean verification flag for the '4 of 828 η-η reactions are η+η-forming, all Cyclization' claim

## How to apply

Load EBoMD.sdf from the repository's Datasets folder and parse each record's annotated BoM entries for the 679 Zaretzki-derived compounds, distinguishing atom-pair descriptors such as 〈η, η〉, 〈η, H〉 and 〈η, SPN〉. Count η-η BoM records first to confirm the paper's reported total of 828, then filter the subset where the bond is formed (η+η type) rather than merely modified, and classify each by its ReactionType field. Validate against the published finding that exactly 4 of 828 η-η reactions are η+η-forming and that all four are labeled Cyclization; also tally the η-H (expected 1348) and η-SPN (expected 86) counts as corroborating dataset statistics. Note the paper reports reactive η-η bonds at ~5% versus ~95% non-reactive, which constrains the expected class balance when filtering. Write the counts and a boolean verification result (e.g. to ebomd_eta_eta_cyclization_counts.csv) so the claim is reproducible.

## Related tools

- **EBoMD.sdf** (Primary input: BoM-annotated SDF of the 679 Zaretzki-derived compounds used to train CypBoM, containing the 828 η-η BoM records to parse) — https://bitbucket.org/wishartlab/cyproduct/src/master/
- **CDK** (Chemistry toolkit used in the CyProduct pipeline for computing per-bond features from the parsed [compound, bond] instances; usable for SDF parsing and atom-pair extraction)

## Examples

```
from rdkit import Chem; recs=[r for r in Chem.SDMolSupplier('Datasets/EBoMD.sdf')]; eta_eta=[r for r in recs if r.GetProp('BoMType')=='eta-eta']; form=[r for r in eta_eta if r.GetProp('BondAction')=='eta+eta-forming']
```

## Evaluation signals

- Parsed η-η BoM record count must equal exactly 828 for EBoMD.sdf
- η+η-forming subset must contain exactly 4 records, all labeled Cyclization
- η-H BoM count must equal 1348 and η-SPN count must equal 86 as corroborating statistics
- Reactive vs non-reactive η-η split should approximate 5% / 95%, matching the paper's reported balance
- Compound-derived record set must trace to the 679 Zaretzki CYP450 substrates, not the 98-compound EBoMD2 hold-out

## Limitations

- No explicit repository URL is stated in the article text itself for the Datasets folder, so locating EBoMD.sdf depends on the README or the Bitbucket/GitHub links
- The 828 η-η figure applies to EBoMD (training) only; EBoMD2 (98 hold-out compounds) and the BioTransformer hold-out file follow different annotation bases
- Class imbalance (~5% reactive η-η) means small parsing or labeling errors can shift the rare η+η-forming subset and break the 4-of-828 verification

## Evidence

- [methods] η+η-forming reactions are very rare -- occurring in only 4 of the 828 〈η, η〉 reactions: [evidence span withheld — anchor status: not_found]
- [methods] The EBoMD ("Edmonton Bond-of-Metabolism Dataset") collection contains the 828 η-η BoMs: [evidence span withheld — anchor status: not_found]
