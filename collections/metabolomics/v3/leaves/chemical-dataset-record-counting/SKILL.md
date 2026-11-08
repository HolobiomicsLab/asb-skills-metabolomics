---
name: chemical-dataset-record-counting
description: Use when a deposited .sdf dataset must be audited to confirm its record
  counts match published per-category totals, especially when annotations need deduplication
  or category classification.
when_to_use_negative:
- The dataset is a plain structure file without bond-of-metabolism annotations to
  classify (e.g. EBoMD2.sdf hold-out used only for testing predictions)
- Goal is predicting new metabolites with CyProduct rather than auditing existing
  deposited annotation counts
edam_operation: http://edamontology.org/operation_3359
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_0091
tools:
- name: RDKit
  role: SDF parser used to read all compound records and their stored BoM annotations
- name: pandas
  role: Tabulates and writes the per-category counts to ebomd_bond_counts.csv
- name: CyProduct
  role: Source of the EBoMD dataset under audit, distributed via its repository and
    CyProduct.jar
  repo: https://bitbucket.org/wishartlab/cyproduct/src/master/
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.jcim.1c00144
    title: 'CyProduct: A Software Tool for Accurately Predicting the Byproducts of
      Human Cytochrome P450 Metabolism'
  source_content_hash: sha256:021b81670f7d6979f28895248409d7f1d16deb0231ed824f3e0b98b328875af0
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - CyProduct
  - RDKit
  - pandas
  edam_operation: http://edamontology.org/operation_3359
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# chemical-dataset-record-counting

## Summary

Counts and cross-checks annotated chemical records in an SDF-deposited dataset (here, EBoMD Bond-of-Metabolism annotations) per category, validating that computed totals reproduce the paper's reported figures (828 η-η, 1348 η-H, 86 η-SPN = 2262).

## When to use

Use when a deposited .sdf dataset must be audited to confirm its record counts match published per-category totals, especially when annotations need deduplication or category classification.

## When NOT to use

- The dataset is a plain structure file without bond-of-metabolism annotations to classify (e.g. EBoMD2.sdf hold-out used only for testing predictions)
- Goal is predicting new metabolites with CyProduct rather than auditing existing deposited annotation counts

## Inputs

- EBoMD.sdf from the CyProduct repository Datasets folder (compounds derived from the Zaretzki CYP450 substrate dataset of 679 compounds, with stored BoM annotations)
- Expected per-category counts from the paper (828 η-η, 1348 η-H, 86 η-SPN)

## Outputs

- ebomd_bond_counts.csv with per-category BoM record counts (η-η, η-H, η-SPN)

## How to apply

Load the deposited SDF file (e.g. EBoMD.sdf from the CyProduct repository's Datasets folder) with RDKit, iterating over every annotated bond record per compound. Classify each BoM 4-tuple record into its category — η-η (bond between two connected non-hydrogen atoms), η-H (atom-to-hydrogen bond replaced by a heteroatom), or η-SPN (new bond to S/P/N via lone-pair sharing) — and count records per category. Deduplicate η-η entries since canonical ordering means 〈i, j〉 and 〈j, i〉 denote the same bond, and associate multi-product reactions by their shared ReactionID so each annotation is counted once. Write the three per-category counts to a CSV (ebomd_bond_counts.csv) and compare against the published totals; counts must match 828/1348/86 exactly, and the η-η total of 828 should be consistent with the finding that only 4 of 828 reactions were η+η-forming Cyclizations.

## Related tools

- **RDKit** (SDF parser used to read all compound records and their stored BoM annotations)
- **pandas** (Tabulates and writes the per-category counts to ebomd_bond_counts.csv)
- **CyProduct** (Source of the EBoMD dataset under audit, distributed via its repository and CyProduct.jar) — https://bitbucket.org/wishartlab/cyproduct/src/master/

## Examples

```
python -c "import pandas as pd; from rdkit import Chem; mols=[m for m in Chem.SDMolSupplier('Datasets/EBoMD.sdf')]; print(len(mols))"
```

## Evaluation signals

- Computed η-η count equals 828, η-H count equals 1348, η-SPN count equals 86; sum equals 2262
- Total of 2262 annotated BoM records matches the paper's figure of BoM annotations derived from 679 compounds
- Deduplicated η-η counts are invariant to bond-pair orientation (〈i, j〉 vs 〈j, i〉 counted once)
- η-η total of 828 is consistent with only 4 of 828 η-η reactions being η+η-forming Cyclizations

## Limitations

- η-η records carry a canonical-ordering ambiguity (〈i, j〉 vs 〈j, i〉) that inflates counts if not deduplicated
- Multi-product reactions sharing a ReactionID risk double-counting if not grouped before tallying
- No explicit repository URL was stated in the paper for locating the dataset despite references to 'the repository'

## Evidence

- [methods] η+η-forming reactions are very rare -- occurring in only 4 of the 828 〈η, η〉 reactions: [evidence span withheld — anchor status: not_found]
