---
name: bom-record-categorization
description: Use when validating rare BoM-class counts or dataset-level statistics
  derived from EBoMD.sdf — e.g. verifying that η+η-forming reactions are 4 of 828
  η-η records and all are Cyclization.
when_to_use_negative:
- Input is EBoMD2.sdf or another hold-out dataset (e.g. 60Reactants.sdf, 13321_2018_324_MOESM6_ESM.sdf)
  — the 828 η-η count and its η+η breakdown are properties of EBoMD only
- Goal is predicting metabolites for a query molecule rather than auditing dataset-level
  BoM record statistics — use the CyProduct.jar pipeline instead
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_0154
tools:
- name: EBoMD
  role: The annotated BoM dataset being parsed and counted; derived by converting
    Zaretzki's SoM dataset to BoM records
- name: CyProduct
  role: Source tool suite whose repository distributes EBoMD.sdf in its Datasets folder
  repo: https://bitbucket.org/wishartlab/cyproduct/src/master/
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.jcim.1c00144
    title: 'CyProduct: A Software Tool for Accurately Predicting the Byproducts of
      Human Cytochrome P450 Metabolism'
  source_content_hash: sha256:f29ae638c1862f52c7e41521a07c6601db3f92bf43b76f71d9516daa0cad881c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - CyProduct
  - EBoMD
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# bom-record-categorization

## Summary

Parse annotated Bond-of-Metabolism (BoM) records from CyProduct's EBoMD.sdf dataset, tally records by BoM type (η-η, η-H, η-SPN), distinguish bond-forming (η+η) from bond-modifying records, and verify reported dataset statistics such as 'only 4 of the 828 η-η reactions form bonds, all Cyclization'.

## When to use

Use when validating rare BoM-class counts or dataset-level statistics derived from EBoMD.sdf — e.g. verifying that η+η-forming reactions are 4 of 828 η-η records and all are Cyclization.

## When NOT to use

- Input is EBoMD2.sdf or another hold-out dataset (e.g. 60Reactants.sdf, 13321_2018_324_MOESM6_ESM.sdf) — the 828 η-η count and its η+η breakdown are properties of EBoMD only
- Goal is predicting metabolites for a query molecule rather than auditing dataset-level BoM record statistics — use the CyProduct.jar pipeline instead

## Inputs

- EBoMD.sdf (annotated BoM records for 679 Zaretzki-derived CYP450 substrates, from the repository's Datasets folder)

## Outputs

- ebomd_eta_eta_cyclization_counts.csv containing η-η, η-H (1348), η-SPN (86) BoM tallies, the η+η-forming count, their ReactionType labels, and a boolean result for the '4 of 828, all Cyclization' claim

## How to apply

Load EBoMD.sdf from the repository's Datasets folder and parse the annotated BoM records for the 679 Zaretzki-derived compounds. Count the η-η BoM records to confirm the reported total of 828, and separately tally the η-H and η-SPN counts (expected 1348 and 86) as corroborating dataset statistics. Filter the η-η records to keep only those where the bond is formed (η+η type) rather than merely modified, then classify each by its ReactionType label (e.g. Cyclization, EpOxidation, Dealkylation). Write all counts plus a boolean verification of the '4 of 828, all Cyclization' claim to a CSV such as ebomd_eta_eta_cyclization_counts.csv; the validation rule is that the η+η-forming total must equal exactly 4 out of 828 η-η reactions with all four classified as Cyclization.

## Related tools

- **EBoMD** (The annotated BoM dataset being parsed and counted; derived by converting Zaretzki's SoM dataset to BoM records)
- **CyProduct** (Source tool suite whose repository distributes EBoMD.sdf in its Datasets folder) — https://bitbucket.org/wishartlab/cyproduct/src/master/

## Examples

```
python tally_bom_records.py --input Datasets/EBoMD.sdf --bom-type eta-eta --form-only --out ebomd_eta_eta_cyclization_counts.csv
```

## Evaluation signals

- η-η BoM record count equals 828
- η+η-forming record count equals exactly 4, all labeled Cyclization
- η-H BoM count equals 1348
- η-SPN BoM count equals 86
- Verification boolean for the '4 of 828, all Cyclization' claim is True in the output CSV

## Limitations

- η+η-forming reactions are extremely rare (4 of 828), so per-class statistics for η+η records have no statistical power for subtype generalization
- No explicit repository URL is stated in the paper's methods text despite references to 'the repository', so locating EBoMD.sdf depends on the Bitbucket/GitHub distribution
- BoM annotation depends on the curation rules used to convert Zaretzki's SoM dataset; errors in that conversion propagate into the counts

## Evidence

- [methods] η+η-forming reactions are very rare -- occurring in only 4 of the 828 〈η, η〉 reactions: [evidence span withheld — anchor status: not_found]
- [readme] EBoMD.sdf contains the compounds derived from the Zaretzki’s CYP450 substrate datasets of 679 compounds. It is used to train the CypBoM module.: "EBoMD.sdf contains the compounds derived from the Zaretzki’s CYP450 substrate datasets of 679 compounds. It is used to train the CypBoM module."
- [methods] we converted Zaretzki’s SoM dataset to a corresponding BoM dataset, by applying the following rules: "we converted Zaretzki’s SoM dataset to a corresponding BoM dataset, by applying the following rules"
- [methods] The EBoMD ("Edmonton Bond-of-Metabolism Dataset") collection contains the 828 η-η BoMs: [evidence span withheld — anchor status: not_found]
