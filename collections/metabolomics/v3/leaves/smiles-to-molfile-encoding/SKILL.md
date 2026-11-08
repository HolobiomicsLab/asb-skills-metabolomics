---
name: smiles-to-molfile-encoding
description: Use when candidate structures arrive as isomeric SMILES (e.g. retrieved
  from PubChem by CID via PubChemPy) and the next pipeline step requires MOL-file
  input for descriptor calculation.
when_to_use_negative:
- The input is already a MOL file, SDF, or descriptor table — re-encoding from SMILES
  is unnecessary and could lose registered information
- The downstream model requires 3D conformers rather than the 2D molecular descriptors
  used by CCSP 2.0's Mordred V1.2.0 step
- Working with adducts or species not representable by the neutral isomeric SMILES
  stored in PubChem (e.g. dimers, in-source fragments), which the source pipeline
  flags as unsupported
edam_topics:
- http://edamontology.org/topic_2258
tools:
- name: RDKit
  role: Reads each candidate SMILES string and writes the MOL file consumed by Mordred
    descriptor calculation
- name: PubChemPy
  role: Retrieves each candidate CID's isomeric SMILES from PubChem before MOL encoding
- name: PubChem Identifier Exchange Service
  role: Converts neutral InChI strings to the corresponding isomeric SMILES prior
    to RDKit MOL encoding
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2022.08.09.503345
    title: 'CCS Predictor 2.0: An Open-Source Jupyter Notebook Tool for Filtering
      Out False Positives in Metabolomics'
  source_content_hash: sha256:97b87a60449665f4b10eb8040457c383d10b55d1e63580c447d4dafa8e4e9f30
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: ''
  tools:
  - CCSP 2.0
  - Jupyter Notebook
  - Mordred
  - PubChem Identifier Exchange Service
  - PubChemPy
  - Python
  - RDKit
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 18
---

# SMILES-to-MOL-file encoding

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Converts isomeric SMILES strings for candidate structures into RDKit-encoded MOL files so downstream descriptor calculation (Mordred V1.2.0, 1613 2D descriptors) can proceed in the CCSP 2.0 CCS-prediction workflow.

## When to use

Use when candidate structures arrive as isomeric SMILES (e.g. retrieved from PubChem by CID via PubChemPy) and the next pipeline step requires MOL-file input for descriptor calculation.

## When NOT to use

- The input is already a MOL file, SDF, or descriptor table — re-encoding from SMILES is unnecessary and could lose registered information
- The downstream model requires 3D conformers rather than the 2D molecular descriptors used by CCSP 2.0's Mordred V1.2.0 step
- Working with adducts or species not representable by the neutral isomeric SMILES stored in PubChem (e.g. dimers, in-source fragments), which the source pipeline flags as unsupported

## Inputs

- Isomeric SMILES strings for candidate structures (e.g. retrieved per PubChem CID via PubChemPy)
- Neutral InChI strings convertible to isomeric SMILES (e.g. via the PubChem Identifier Exchange Service)

## Outputs

- One RDKit-encoded MOL file per candidate SMILES
- Mordred-readable molecular structures ready for 1613 2D descriptor calculation

## How to apply

For each candidate compound (e.g. the 221 PubChem hits for m/z 155.0093 within 10 ppm), first obtain its isomeric SMILES — in the source workflow, SMILES are obtained via PubChemPy or by converting neutral InChI strings through the PubChem Identifier Exchange Service. Then use RDKit to read each SMILES and write a molecular data (MOL) file, preserving isomeric (stereochemistry-aware) representation since the source pipeline explicitly encodes 'isomeric' SMILES. Feed each MOL file into Mordred V1.2.0 to compute the 1613 2D molecular descriptors that the CCSP 2.0 linear-kernel SVR model consumes. The rationale is that the CCSP 2.0 descriptor pipeline is standardized on RDKit-encoded MOL files, so converting directly from SMILES keeps every candidate on the same representation path regardless of the original identifier format.

## Related tools

- **RDKit** (Reads each candidate SMILES string and writes the MOL file consumed by Mordred descriptor calculation)
- **PubChemPy** (Retrieves each candidate CID's isomeric SMILES from PubChem before MOL encoding)
- **PubChem Identifier Exchange Service** (Converts neutral InChI strings to the corresponding isomeric SMILES prior to RDKit MOL encoding)

## Examples

```
from rdkit import Chem; mol = Chem.MolFromSmiles(isomeric_smiles); Chem.MolToMolFile(mol, 'candidate.mol')
```

## Evaluation signals

- Every candidate SMILES (e.g. all 221 PubChem candidates for m/z 155.0093) yields a valid MOL file that RDKit can re-read without sanitization errors
- Re-reading each MOL file and round-tripping back to SMILES recovers the original isomeric SMILES, confirming stereochemistry was preserved
- Mordred V1.2.0 computes the full 1613 2D descriptor set for each encoded MOL file with no failed candidates
- The encoded structures match the neutral parent forms expected by the [M-H]- CCS model trained on the McLean Unified CCS Compendium

## Limitations

- Encoding uses the neutral form of each candidate (SMILES from neutral InChI strings), so charge-state-specific structure information for adducts like [M-H]- is not captured in the MOL file
- The CCSP 2.0 pipeline built on these MOL files uses only 2D descriptors, so this encoding step does not generate or preserve 3D conformer information
- Conversion quality depends on identifier exchange via PubChem services; failures or ambiguities in InChI-to-SMILES conversion propagate into descriptor calculation

## Evidence

- [full_text] Case-study SMILES retrieval: [evidence span withheld — anchor status: not_found]
