---
name: cross-dataset-molecule-deduplication
description: Use when building a screening library from multiple structure databases
  (SDF/gzipped SDF) and non-organic, ionic, or duplicate molecules must be removed
  before spectrum prediction or other downstream chemistry.
when_to_use_negative:
- When the target library should cover organometallics, polymers, or charged species
  — the 11-element and non-ionic rules intentionally exclude them
- When molecules are larger than 1000 Da or fall outside LogP -12 to 24, since the
  mass and LogP cutoffs will remove them
- When only a single dataset needs cleaning and no cross-database duplicate resolution
  is required — intra-dataset dedup alone suffices and cross-set precedence assignment
  is unnecessary
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: RDKit (v2022.03.3)
  role: Parses SDF records to SMILES and computes molecular mass, atom/element composition,
    ionic-state detection, and LogP used for the five filter rules
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-39279-7
    title: Ultra-fast and accurate electron ionization mass spectrum matching for
      compound identification with million-scale in-silico library
  source_content_hash: sha256:38817c0fb5610f8c671fc335fe3c14cda1cb3dccda12587f9ad0391bb85e9308
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - RDKit (v2022.03.3)
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# cross-dataset-molecule-deduplication

## Summary

Applies five chemical filtering rules and intra- plus cross-dataset deduplication to NIST 2017, ChEMBL 28, and HMDB 5.0 structures (parsed to SMILES with RDKit), producing the f-NIST (255,821), f-ChEMBL (1,890,869), and f-HMDB (106,516) molecule sets that supply the 2,146,690 structures for the million-scale in-silico EI-MS library.

## When to use

Use when building a screening library from multiple structure databases (SDF/gzipped SDF) and non-organic, ionic, or duplicate molecules must be removed before spectrum prediction or other downstream chemistry.

## When NOT to use

- When the target library should cover organometallics, polymers, or charged species — the 11-element and non-ionic rules intentionally exclude them
- When molecules are larger than 1000 Da or fall outside LogP -12 to 24, since the mass and LogP cutoffs will remove them
- When only a single dataset needs cleaning and no cross-database duplicate resolution is required — intra-dataset dedup alone suffices and cross-set precedence assignment is unnecessary

## Inputs

- NIST 2017 training/validation/test splits (from brain-research/deep-molecular-massspec training_splits)
- Gzipped ChEMBL 28 structure-data file (chembl_28.sdf.gz)
- HMDB 5.0 SDF (structures.zip from hmdb.ca)
- RDKit-parsed SMILES strings of the molecules in these datasets

## Outputs

- f-NIST CSV molecule set (255,821 molecules)
- f-ChEMBL CSV molecule set (1,890,869 molecules)
- f-HMDB CSV molecule set (106,516 molecules)
- Combined non-redundant structure pool of 2,146,690 molecules (f-NIST + f-ChEMBL) feeding the in-silico EI-MS library

## How to apply

Load the NIST 2017 training/validation/test splits (from the NEIMS deep-molecular-massspec repository), the gzipped ChEMBL 28 SDF, and the HMDB 5.0 SDF, parsing structures to SMILES with RDKit (v2022.03.3). Apply five filter rules in order: (1) keep molecules with molecular mass < 1000 Da; (2) keep molecules composed only of the 11 common elements H, C, O, N, P, S, Cl, F, Br, I, Si; (3) exclude ionic compounds (charged species and salt/counter-ion forms); (4) keep molecules whose RDKit-computed LogP lies between -12 and 24; (5) deduplicate repeating molecules within each dataset and then across datasets so each molecule appears in only one filtered set. The rationale is to obtain 'reasonable datasets' of neutral, organic, drug-like molecules suitable for NEIMS prediction, which requires ECFPs of clean SMILES. Save each filtered set as CSV and verify counts against the reported 255,821 (f-NIST), 1,890,869 (f-ChEMBL), and 106,516 (f-HMDB).

## Related tools

- **RDKit (v2022.03.3)** (Parses SDF records to SMILES and computes molecular mass, atom/element composition, ionic-state detection, and LogP used for the five filter rules)

## Examples

```
from rdkit import Chem, Crippen, Descriptors; keep = [s for s in smiles if Descriptors.MolWt(Chem.MolFromSmiles(s)) < 1000 and -12 <= Crippen.MolLogP(Chem.MolFromSmiles(s)) <= 24]
```

## Evaluation signals

- Filtered set sizes match the reported counts: 255,821 for f-NIST, 1,890,869 for f-ChEMBL, and 106,516 for f-HMDB
- Combined f-NIST + f-ChEMBL contains exactly 2,146,690 molecules with no structure shared between the two sets
- Every retained molecule has molecular mass < 1000 Da, contains only H, C, O, N, P, S, Cl, F, Br, I, Si, no formal charges/salt components, and RDKit LogP within -12 to 24
- No duplicate molecules (by canonical SMILES) within any single filtered set
- All SMILES parse cleanly with RDKit so ECFPs can be generated for downstream NEIMS spectrum prediction

## Limitations

- Details of which molecules were excluded by each rule are only in Supplementary Data 1, which is not included in the main text
- Cross-dataset deduplication requires an arbitrary precedence order when a molecule occurs in more than one database, affecting which filtered set a structure lands in
- The 11-element and LogP -12 to 24 thresholds restrict chemical space; compounds outside this scope (e.g., organometallics, very polar or very hydrophobic species) are silently dropped

## Evidence

- [discussion] Five-rule filtering and deduplication applied to four datasets: "five filtering steps (Supplementary Fig. 6) are applied to the molecules in these four datasets to obtain reasonable datasets"
- [discussion] Element composition filter: "(2) molecules only contain 11 common elements H, C, O, N, P, S, Cl, F, Br, I, Si"
- [discussion] LogP filter threshold: "(4) molecular LogP is within the range from −12 to 24"
- [discussion] RDKit is the parsing/descriptor engine: "The SMILES strings of molecules were read by RDKit (v2022.03.3)"
- [discussion] Expected filtered counts and downstream use: "the predicted spectra of molecules in f-NIST and f-ChEMBL were used to build the in-silico library, which included 2,146,690 molecules"
