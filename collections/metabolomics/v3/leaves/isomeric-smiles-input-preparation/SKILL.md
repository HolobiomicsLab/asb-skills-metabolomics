---
name: isomeric-smiles-input-preparation
description: Use when a molecule set (e.g. McLean CCS Compendium entries) is represented
  as neutral InChI strings and must be converted to isomeric SMILES/MOL files to feed
  descriptor-based CCS predictors like CCSP 2.0, AllCCS, CCSbase or DeepCCS.
when_to_use_negative:
- The molecules are already provided as isomeric SMILES — skip InChI conversion and
  go directly to RDKit MOL encoding and descriptor calculation
- Structures lack a resolvable neutral InChI in PubChem or are not representable as
  2D small-molecule SMILES (e.g. mixtures, polymers, in-source fragments)
- 3D conformer-dependent descriptors are required — CCSP 2.0's feature space is strictly
  the 1613 2D Mordred descriptors
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: PubChem Identifier Exchange Service
  role: Converts each neutral InChI string into the corresponding isomeric SMILES
    for descriptor calculation and identical submission to all CCS predictors
- name: RDKit
  role: Encodes each isomeric SMILES into a molecular data (MOL) file as input to
    descriptor calculation
- name: Mordred V1.2.0
  role: Calculates 1613 2D molecular descriptors from each RDKit MOL file, forming
    the feature space for CCSP 2.0's linear SVR models
- name: CCSP 2.0
  role: Downstream consumer of the prepared SMILES/MOL/descriptor inputs for CCS prediction
    and benchmarking
  repo: https://github.com/facundof2016/CCSP2.0
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2022.08.09.503345
    title: 'CCS Predictor 2.0: An Open-Source Jupyter Notebook Tool for Filtering
      Out False Positives in Metabolomics'
  source_content_hash: sha256:6e62c1e8f8cbcd8ca6c136a8dc9f8c50c80a6cce586fca9d009ea68d6010ebab
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: ''
  repo_url: https://github.com/facundof2016/CCSP2.0
  tools:
  - AllCCS
  - CCSP 2.0
  - CCSbase
  - DeepCCS
  - Mordred
  - Mordred V1.2.0
  - PubChem Identifier Exchange Service
  - Python
  - RDKit
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# Isomeric SMILES input preparation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Converts neutral InChI strings from a CCS reference compendium into isomeric SMILES via the PubChem Identifier Exchange Service, then encodes each SMILES as a MOL file with RDKit so that 1613 2D Mordred descriptors can be computed for CCSP 2.0 SVR-based CCS prediction.

## When to use

Use when a molecule set (e.g. McLean CCS Compendium entries) is represented as neutral InChI strings and must be converted to isomeric SMILES/MOL files to feed descriptor-based CCS predictors like CCSP 2.0, AllCCS, CCSbase or DeepCCS.

## When NOT to use

- The molecules are already provided as isomeric SMILES — skip InChI conversion and go directly to RDKit MOL encoding and descriptor calculation
- Structures lack a resolvable neutral InChI in PubChem or are not representable as 2D small-molecule SMILES (e.g. mixtures, polymers, in-source fragments)
- 3D conformer-dependent descriptors are required — CCSP 2.0's feature space is strictly the 1613 2D Mordred descriptors

## Inputs

- Neutral InChI strings for each molecule (e.g. from the McLean Unified CCS Compendium, 567 [M-H]-, 518 [M+H]+, 461 [M+Na]+ ions after scrubbing)
- PubChem Identifier Exchange Service (InChI-to-SMILES conversion)
- SMILES strings requiring encoding as MOL files

## Outputs

- Isomeric SMILES specification per molecule (stereochemistry preserved)
- RDKit-encoded molecular data (MOL) file per SMILES
- 1613 2D molecular descriptors per molecule computed with Mordred V1.2.0

## How to apply

Take each molecule's neutral InChI string from the McLean Unified CCS Compendium and convert it to the corresponding isomeric SMILES using the PubChem Identifier Exchange Service, preserving stereochemistry. Encode each isomeric SMILES into a molecular data (MOL) file with RDKit. Feed each MOL file into Mordred V1.2.0 to calculate the full library of 1613 2D molecular descriptors, which are the feature space for CCSP 2.0's linear support vector regression models. The identical isomeric SMILES specifications should be reused when submitting compounds to external predictors (AllCCS, CCSbase web servers, DeepCCS command line) so all tools are benchmarked on the same chemical input.

## Related tools

- **PubChem Identifier Exchange Service** (Converts each neutral InChI string into the corresponding isomeric SMILES for descriptor calculation and identical submission to all CCS predictors)
- **RDKit** (Encodes each isomeric SMILES into a molecular data (MOL) file as input to descriptor calculation)
- **Mordred V1.2.0** (Calculates 1613 2D molecular descriptors from each RDKit MOL file, forming the feature space for CCSP 2.0's linear SVR models)
- **CCSP 2.0** (Downstream consumer of the prepared SMILES/MOL/descriptor inputs for CCS prediction and benchmarking) — https://github.com/facundof2016/CCSP2.0

## Examples

```
from rdkit import Chem; mol = Chem.MolFromSmiles(isomeric_smiles); Chem.MolToMolFile(mol, 'compound.mol')
```

## Evaluation signals

- Every converted SMILES must be parseable by RDKit (Chem.MolFromSmiles returns a valid Mol object) and writable to a MOL file
- Isomeric (stereochemistry-bearing) SMILES must be produced, not non-isomeric SMILES — chirality from the original InChI must be retained
- Mordred descriptor calculation must yield the full 1613 2D descriptor library per molecule before SVR training
- The same isomeric SMILES strings must be reused for AllCCS/CCSbase web submissions and DeepCCS command-line predictions to keep benchmark splits identical
- Spot-check that converted SMILES round-trip to the expected structure for known Compendium entries

## Limitations

- The Identifier Exchange conversion depends on PubChem resolution of each neutral InChI; molecules not represented in PubChem cannot be converted this way
- The descriptor feature space covers only standard monomeric adduct structures — CCSP 2.0's training sets do not support dimers such as [2M+H]+ or [2M-H]-, less common adducts, or in-source fragments found in ESI-MS experiments
- The workflow uses 2D descriptors only; stereochemical/3D gas-phase conformational effects on CCS are not captured by the encoding step

## Evidence

- [readme] CCSP 2.0 as the downstream prediction tool consuming curated inputs: [evidence span withheld — anchor status: external_doc]
