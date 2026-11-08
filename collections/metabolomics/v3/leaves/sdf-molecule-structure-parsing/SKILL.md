---
name: sdf-molecule-structure-parsing
description: Use when a workflow starts from SDF-formatted molecular structure files
  (e.g. chembl_28.sdf.gz or HMDB structures.zip) and downstream steps require per-molecule
  SMILES or fingerprints as NEIMS model inputs.
when_to_use_negative:
- Molecules are already provided as SMILES strings or another structure format, so
  SDF parsing is unnecessary
- The target library is experimental spectra (e.g. NIST 2017 measured spectra) rather
  than in-silico predicted spectra generated from structures
- Molecules fall outside the paper's filterable scope (e.g. molecular mass ≥ 1000
  Da or elements beyond H, C, O, N, P, S, Cl, F, Br, I, Si), since they will be excluded
  anyway
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: RDKit
  role: Parses SDF molecular structures and reads SMILES strings, and generates the
    ECFPs used as NEIMS inputs
  repo: https://github.com/Qiong-Yang/FastEI
- name: NEIMS
  role: 'Downstream consumer: takes ECFPs derived from the parsed SDF structures to
    predict EI mass spectra without re-training'
  repo: https://github.com/brain-research/deep-molecular-massspec
- name: FastEI
  role: Packaged pipeline whose library-construction steps (parse SDF → ECFP → NEIMS
    prediction → IN_SILICO_LIBRARY.db) this skill initiates
  repo: https://github.com/Qiong-Yang/FastEI
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-39279-7
    title: Ultra-fast and accurate electron ionization mass spectrum matching for
      compound identification with million-scale in-silico library
  source_content_hash: sha256:c83072fc807666ffbb3ff00eef6237fec298081aaf6add2f6281d04b2def40b3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Qiong-Yang/FastEI
  tools:
  - FastEI
  - NEIMS
  - RDKit
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# sdf-molecule-structure-parsing

## Summary

Reads molecular structures from structure-data files (SDF) such as ChEMBL 28 and HMDB 5.0 into RDKit molecule objects and extracts SMILES strings, as the prerequisite step for generating ECFPs and NEIMS-predicted EI mass spectra in the FastEI million-scale in-silico library pipeline.

## When to use

Use when a workflow starts from SDF-formatted molecular structure files (e.g. chembl_28.sdf.gz or HMDB structures.zip) and downstream steps require per-molecule SMILES or fingerprints as NEIMS model inputs.

## When NOT to use

- Molecules are already provided as SMILES strings or another structure format, so SDF parsing is unnecessary
- The target library is experimental spectra (e.g. NIST 2017 measured spectra) rather than in-silico predicted spectra generated from structures
- Molecules fall outside the paper's filterable scope (e.g. molecular mass ≥ 1000 Da or elements beyond H, C, O, N, P, S, Cl, F, Br, I, Si), since they will be excluded anyway

## Inputs

- chembl_28.sdf.gz — gzipped structure-data file of ChEMBL 28
- HMDB 5.0 SDF zip archive from https://hmdb.ca/system/downloads/current/structures.zip
- NIST 2017-derived molecule sets (f-NIST, 255,821 compounds from NEIMS train/validation/test splits)

## Outputs

- Per-molecule SMILES strings parsed from the SDF records
- RDKit molecule objects suitable for ECFP generation and property filtering (mass, elements, ionic character, LogP)
- Parsed structure collection feeding NEIMS spectrum prediction (ultimately contributing to IN_SILICO_LIBRARY.db with 2,146,690 molecules)

## How to apply

Download the gzipped SDF files (ChEMBL 28 from https://ftp.ebi.ac.uk/pub/databases/chembl/ChEMBLdb/releases/chembl_28/chembl_28.sdf.gz; HMDB 5.0 from https://hmdb.ca/system/downloads/current/structures.zip) and parse them with RDKit v2022.03.3 to read each molecule's SMILES string. From the parsed molecules, generate extended circular fingerprints (ECFPs) that are then fed into the NEIMS fully connected neural network to obtain predicted EI mass spectra. Apply the paper's five filtering steps (molecular mass < 1000 Da; only H, C, O, N, P, S, Cl, F, Br, I, Si; non-ionic; LogP from −12 to 24; deduplication) before or while parsing so downstream library counts stay consistent. Rationale: SDF is the native distribution format of ChEMBL and HMDB, and RDKit provides the SMILES/ECFP extraction that NEIMS requires without re-training (weights used as-is from massspec_weights.zip).

## Related tools

- **RDKit** (Parses SDF molecular structures and reads SMILES strings, and generates the ECFPs used as NEIMS inputs) — https://github.com/Qiong-Yang/FastEI
- **NEIMS** (Downstream consumer: takes ECFPs derived from the parsed SDF structures to predict EI mass spectra without re-training) — https://github.com/brain-research/deep-molecular-massspec
- **FastEI** (Packaged pipeline whose library-construction steps (parse SDF → ECFP → NEIMS prediction → IN_SILICO_LIBRARY.db) this skill initiates) — https://github.com/Qiong-Yang/FastEI

## Examples

```
from rdkit import Chem; mols = [m for m in Chem.SDMolSupplier('chembl_28.sdf.gz') if m is not None]; smiles = [Chem.MolToSmiles(m) for m in mols]
```

## Evaluation signals

- The parsed record count matches the source dataset after the five filtering steps (e.g. f-NIST yields 255,821 compounds; the combined in-silico library contains exactly 2,146,690 molecules)
- Every parsed molecule yields a valid SMILES string and a computable ECFP via RDKit v2022.03.3
- Parsed molecules satisfy the filter criteria: mass < 1000 Da, only the 11 allowed elements, non-ionic, LogP between −12 and 24, no duplicates within a dataset
- IN_SILICO_LIBRARY.db exists and contains exactly 2,146,690 molecule entries, matching the library size reported in the article

## Limitations

- Molecules excluded by the five filtering steps are only itemized in Supplementary Data 1, which is not present in the main text
- RDKit version pinning matters: the article reports RDKit v2022.03.3, and other versions may parse or fingerprint differently
- The NEIMS pre-trained weights are used without re-training and their version/build date is not stated, so structure-parsing outputs must remain compatible with the original ECFP format

## Evidence

- [discussion] The SMILES strings of molecules were read by RDKit (v2022.03.3): "The SMILES strings of molecules were read by RDKit (v2022.03.3)"
- [discussion] The gzipped structure-data file (SDF) of ChEMBL 28 was downloaded from https://ftp.ebi.ac.uk/pub/databases/chembl/ChEMBLdb/releases/chembl_28/chembl_28.sdf.gz: "The gzipped structure-data file (SDF) of ChEMBL 28 was downloaded from https://ftp.ebi.ac.uk/pub/databases/chembl/ChEMBLdb/releases/chembl_28/chembl_28.sdf.gz"
- [discussion] The ECFPs of all molecules in f-NIST, f-ChEMBL, f-HMDB, and the extra test set were fed into the NEIMS model to obtain their predicted mass spectra: "The ECFPs of all molecules in f-NIST, f-ChEMBL, f-HMDB, and the extra test set were fed into the NEIMS model to obtain their predicted mass spectra"
- [other] 5. Generate extended circular fingerprints (ECFPs) for every molecule using RDKit.: [evidence span withheld — anchor status: not_found]
- [readme] $ python make_spectra_prediction.py --input_file=examples/pentachlorobenzene.sdf --output_file=/tmp/annotated.sdf: [evidence span withheld — anchor status: not_found]
