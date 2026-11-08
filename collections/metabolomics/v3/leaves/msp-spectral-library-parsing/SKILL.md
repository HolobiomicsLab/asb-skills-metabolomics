---
name: msp-spectral-library-parsing
description: Use when reference MS/MS spectra in NIST .msp format (MoNA, GNPS, LipidMaps,
  or user-compiled libraries) must be parsed into model-ready blocks with SMILES/InChI,
  precursor MZ, and fingerprint metadata.
when_to_use_negative:
- Input is raw LC-MS/MS vendor data — extract .msp spectra first with IDSL.CSA, then
  parse
- Spectra are already in mzML/MGF or a database rather than .msp text blocks with
  Name/PrecursorMZ headers
- 'Blocks lack ''PrecursorMZ: '' entries, which IDSL_MINT requires for parsing'
edam_operation: http://edamontology.org/operation_0324
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3315
tools:
- name: IDSL_MINT
  role: Parses .msp blocks, auto-detects YAML type, and handles comment/SMILES/InChI
    header extraction for training
  repo: https://github.com/idslme/IDSL_MINT
- name: RDKit
  role: Converts InChI and SMILES row entries from MSP blocks into canonical SMILES
    and computes ECFP/MACCS Keys fingerprints
  repo: https://www.rdkit.org/docs/Install.html
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s13321-024-00804-5
    title: 'IDSL_MINT: a deep learning framework to predict molecular fingerprints
      from mass spectra'
  source_content_hash: sha256:a8feab74d260c5d650f6d91310060869cd86c16e0f3d5b1da3b7f0e7d23ca12d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/idslme/IDSL_MINT
  tools:
  - IDSL_MINT
  - PyTorch
  - RDKit
  edam_operation: http://edamontology.org/operation_0324
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# msp-spectral-library-parsing

## Summary

Parsing NIST .msp-format reference MS/MS library files (e.g. MoNA, GNPS, or LipidMaps exports) into structured blocks containing spectra plus InChI/SMILES and fingerprint metadata, as required by IDSL_MINT for training and inference.

## When to use

Use when reference MS/MS spectra in NIST .msp format (MoNA, GNPS, LipidMaps, or user-compiled libraries) must be parsed into model-ready blocks with SMILES/InChI, precursor MZ, and fingerprint metadata.

## When NOT to use

- Input is raw LC-MS/MS vendor data — extract .msp spectra first with IDSL.CSA, then parse
- Spectra are already in mzML/MGF or a database rather than .msp text blocks with Name/PrecursorMZ headers
- Blocks lack 'PrecursorMZ: ' entries, which IDSL_MINT requires for parsing

## Inputs

- NIST .msp-format MS/MS library text files (e.g. from MoNA, GNPS, LipidMaps, or user-compiled training sets)
- IDSL_MINT YAML configuration file (e.g. MINT_MS2FP_trainer.yaml) referencing the .msp training set

## Outputs

- Parsed MSP blocks with canonical SMILES, precursor MZ, ion mode, and fingerprint descriptors
- Model-ready training data consumed by IDSL_MINT (yielding trained model weights after the training run)

## How to apply

Read the .msp text file block-by-block: each block starts at a 'Name:' entry and must contain a 'PrecursorMZ: ' row entry, which IDSL_MINT requires for parsing. Identify structures via 'SMILES: ' or 'InChI: ' labels (case-insensitive); convert InChI and SMILES row entries into canonical SMILES using RDKit before descriptor calculation. Extract 'comment: ' and 'comments: ' entries so that MoNA, GNPS, and other public libraries are processed with any pre-treatment requirements. If multiple similar headers occur within one MSP block, select the one with the longest content for parsing. Fingerprints may be computed from SMILES via RDKit ECFP or MACCS Keys methods, or taken from user-provided 'Fingerprint: ' rows (hyphen-separated bit positions). The parsed blocks then feed the trainer YAML (e.g. MINT_MS2FP_trainer.yaml) executed with 'MINT_workflow --yaml /path/to/yaml/file'.

## Related tools

- **IDSL_MINT** (Parses .msp blocks, auto-detects YAML type, and handles comment/SMILES/InChI header extraction for training) — https://github.com/idslme/IDSL_MINT
- **RDKit** (Converts InChI and SMILES row entries from MSP blocks into canonical SMILES and computes ECFP/MACCS Keys fingerprints) — https://www.rdkit.org/docs/Install.html

## Examples

```
MINT_workflow --yaml /path/to/MINT_MS2FP_trainer.yaml
```

## Evaluation signals

- Every parsed MSP block contains a 'PrecursorMZ: ' row entry (a hard requirement)
- InChI/SMILES labels are resolved without case sensitivity; failures indicate malformed blocks
- Duplicate similar headers resolve to the longest-content header, matching IDSL_MINT's parsing rule
- MoNA/GNPS 'comment'/'comments' entries are extracted without pre-treatment errors
- Downstream training proceeds with a decreasing loss trajectory (e.g. ~70 h ESI+ and ~45 h ESI− on 35 CPU threads as reported)

## Limitations

- Only .msp format is supported; blocks missing 'PrecursorMZ: ' cannot be parsed
- When multiple similar headers exist in a block, only the longest-content header is kept for parsing
- User-provided fingerprints must be encoded as hyphen-separated bit positions in 'Fingerprint: ' rows

## Evidence

- [readme] PrecursorMZ requirement: "MSP blocks must include `PrecursorMZ: ` row entries."
- [readme] comment extraction for MoNA/GNPS: "**IDSL_MINT** can extract information from `comment: ` and `comments: ` entries in ***.msp*** files which enables this platform to process MoNA, GNPS, and other"
- [readme] case-insensitive structure labels: "identifies chemical structures through `SMILES: ` or `InChI: ` labels without case sensitivity."
- [readme] duplicate header rule: "In case multiple similar headers are present in a MSP block, the one with the longest content is selected for parsing."
- [methods] canonical SMILES conversion: "InChI and SMILES row entries in the MSP blocks are converted into canonical SMILES using [RDKit]"
- [results] training set source: "Spectra from MoNA and GNPS public libraries in positive and negative modes were used separately to create training sets."
