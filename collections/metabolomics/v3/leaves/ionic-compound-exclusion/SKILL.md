---
name: ionic-compound-exclusion
description: Use when curating large molecule sets (SDF/SMILES from databases like
  NIST, ChEMBL, HMDB) before spectrum prediction, and neutral-only compounds are required
  for EI-MS library building.
when_to_use_negative:
- The target library explicitly admits charged or zwitterionic species (e.g., LC-MS
  metabolomics libraries where ionizable metabolites from HMDB are the focus)
- The dataset has already passed through this pipeline — re-running exclusion after
  deduplication (rule 5) is redundant since each molecule appears in only one filtered
  set
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_2258
tools:
- name: RDKit (v2022.03.3)
  role: Reads SMILES strings and detects charged species and salt/counter-ion representations
    for filter rule (3)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-39279-7
    title: Ultra-fast and accurate electron ionization mass spectrum matching for
      compound identification with million-scale in-silico library
  source_content_hash: sha256:f0337d57db317bebb6cdd62e866fcfe80498d88a177f7d2bae418be1ce557cba
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - RDKit (v2022.03.3)
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# Ionic compound exclusion

## Summary

Filter rule (3) of the five-step curation pipeline used to build f-NIST, f-ChEMBL, and f-HMDB: molecules flagged as ionic compounds — charged species and salt/counter-ion representations — are excluded so the in-silico EI-MS library contains only neutral molecules amenable to GC-EI-MS.

## When to use

Use when curating large molecule sets (SDF/SMILES from databases like NIST, ChEMBL, HMDB) before spectrum prediction, and neutral-only compounds are required for EI-MS library building.

## When NOT to use

- The target library explicitly admits charged or zwitterionic species (e.g., LC-MS metabolomics libraries where ionizable metabolites from HMDB are the focus)
- The dataset has already passed through this pipeline — re-running exclusion after deduplication (rule 5) is redundant since each molecule appears in only one filtered set

## Inputs

- SMILES strings parsed from the NIST 2017 training/validation/test splits (NEIMS deep-molecular-massspec training_splits)
- ChEMBL 28 gzipped SDF (chembl_28.sdf.gz)
- HMDB 5.0 SDF (structures.zip)

## Outputs

- Per-dataset neutral-molecule sets with ionic compounds removed, contributing to f-NIST (255,821), f-ChEMBL (1,890,869), and f-HMDB (106,516)
- Counts/records of excluded ionic molecules (Supplementary Data 1)

## How to apply

As step 3 of the five-rule pipeline (applied after rule 1, molecular mass < 1000 Da, and rule 2, restriction to the 11 elements H, C, O, N, P, S, Cl, F, Br, I, Si), parse each molecule's SMILES with RDKit (v2022.03.3) and detect ionic character: molecules carrying formal charges, or multi-component salt representations with counter-ions, are excluded. This keeps only neutral molecules whose predicted NEIMS spectra from ECFPs are meaningful for EI-MS matching. Exclusion is applied per-dataset before rule (5) deduplication within and across datasets. Detailed counts of excluded molecules are reported in Supplementary Data 1.

## Related tools

- **RDKit (v2022.03.3)** (Reads SMILES strings and detects charged species and salt/counter-ion representations for filter rule (3))

## Examples

```
from rdkit import Chem; keep = all(a.GetFormalCharge() == 0 for a in Chem.MolFromSmiles(smi).GetAtoms())
```

## Evaluation signals

- No retained molecule contains an atom with nonzero formal charge or a multi-fragment salt representation when re-checked with RDKit
- Final filtered counts match reported values: f-NIST = 255,821, f-ChEMBL = 1,890,869, f-HMDB = 106,516, with f-NIST + f-ChEMBL supplying the 2,146,690 in-silico library molecules
- Rule ordering holds: ionic exclusion applied after mass < 1000 Da and 11-element composition checks, before LogP (-12 to 24) filtering and deduplication
- Number of excluded ionic molecules per dataset is consistent with the per-rule breakdown in Supplementary Data 1

## Limitations

- The per-rule exclusion counts live in Supplementary Data 1, which is not included in the main text, so exact ionic-compound counts cannot be verified from the article alone
- Ionic-detection heuristics (formal charge / salt fragments via RDKit) may miss zwitterions or metal-coordination species not represented as formal charges in the source SDF

## Evidence

- [discussion] Rule 3 exclusion criterion: "(3) molecules are not ionic compounds"
- [discussion] Five-rule pipeline context: "five filtering steps (Supplementary Fig. 6) are applied to the molecules in these four datasets to obtain reasonable datasets"
- [discussion] Tool used for parsing and checking structures: "The SMILES strings of molecules were read by RDKit (v2022.03.3)"
- [discussion] Where excluded molecules are documented: "The detailed information on excluded molecules by data filtering and deduplicating is displayed in Supplementary Data 1"
