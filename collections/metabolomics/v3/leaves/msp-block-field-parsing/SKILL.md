---
name: msp-block-field-parsing
description: Reference for parsing .msp spectral-library blocks when you need to extract
  SMILES/InChI, Fingerprint bit locations, or metadata (PrecursorMZ, Ion_mode) per
  compound to build training targets.
when_to_use_negative:
- Input is already a parsed structure/annotation table (e.g., CSV of SMILES and spectra)
  rather than raw .msp blocks
- The .msp blocks lack PrecursorMZ rows — IDSL_MINT requires them and parsing is invalid
  without them
- You need spectra extraction from raw vendor/converted mzML files rather than parsing
  an existing spectral library
edam_operation: http://edamontology.org/operation_3359
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_0091
tools:
- name: IDSL_MINT
  role: Provides the .msp block parser that locates SMILES/InChI/Fingerprint labels
    case-insensitively, resolves duplicate headers by longest content, and reads comment/comments
    variants for MoNA and GNPS files
  repo: https://github.com/idslme/IDSL_MINT
- name: RDKit
  role: Converts InChI row entries parsed from MSP blocks into canonical SMILES so
    fingerprints can be computed downstream
  repo: https://www.rdkit.org/docs/Install.html
provenance:
  source_task_ids:
  - task_003
  - task_002
  source_papers:
  - doi: 10.1186/s13321-024-00804-5
    title: 'IDSL_MINT: a deep learning framework to predict molecular fingerprints
      from mass spectra'
  source_content_hash: sha256:83ff51d485f27a39970749b9bd81f3081fca8414d8a83c725025dd8adf564083
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/idslme/IDSL_MINT
  tools:
  - IDSL_MINT
  - RDKit
  edam_operation: http://edamontology.org/operation_3359
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 15
---

# msp-block-field-parsing

## Summary

Parses NIST .msp spectral blocks from IDSL_MINT reference libraries to extract per-compound fields — SMILES/InChI structures, user-provided Fingerprint rows, and metadata — handling duplicate headers, case-insensitive labels, and comment/comment variants.

## When to use

Reference for parsing .msp spectral-library blocks when you need to extract SMILES/InChI, Fingerprint bit locations, or metadata (PrecursorMZ, Ion_mode) per compound to build training targets.

## When NOT to use

- Input is already a parsed structure/annotation table (e.g., CSV of SMILES and spectra) rather than raw .msp blocks
- The .msp blocks lack PrecursorMZ rows — IDSL_MINT requires them and parsing is invalid without them
- You need spectra extraction from raw vendor/converted mzML files rather than parsing an existing spectral library

## Inputs

- NIST-format .msp spectral library file (e.g., MoNA/GNPS or LipidMaps exports) split into per-compound blocks
- MSP blocks with 'SMILES: ' or 'InChI: ' rows (case-insensitive) and mandatory 'PrecursorMZ: ' rows
- Optional user-provided 'Fingerprint: ' rows containing dash-separated bit-location strings

## Outputs

- Per-block extracted fields: canonical structure identifiers (SMILES/InChI), Fingerprint bit-location strings, PrecursorMZ, Ion_mode, Formula, ExactMass, collision energy
- Structures ready for RDKit canonicalization and fingerprint generation (downstream tokenized_fingerprints.csv targets)
- Parsed peak lists (m/z intensity pairs) following 'Num Peaks:' for model input spectra

## How to apply

Read the reference .msp file and split it into spectral blocks, locating 'SMILES: ' or 'InChI: ' labels case-insensitively between the Name and Num Peaks rows; when duplicate headers occur within one block, select the entry with the longest content, as IDSL_MINT requires. Also parse 'comment: ' and 'comments: ' (both spellings) so MoNA and GNPS public libraries can be processed without pre-treatment. Extract user-provided fingerprints as dash-separated bit-location strings from 'Fingerprint: ' rows, which may appear anywhere between Name and Num Peaks; when absent, the block's InChI/SMILES rows are converted to canonical SMILES with RDKit so fingerprints (e.g., ECFP radius 2 or MACCS Keys) can be computed downstream. Every block must contain a 'PrecursorMZ: ' row, so validate its presence before accepting a parsed block. The example Aspirin block shows the expected layout: Fingerprint, Synon, InChI, Precursor_type, PrecursorMZ, Ion_mode, Formula, ExactMass, then 'Num Peaks' followed by 'm/z intensity' peak lines.

## Related tools

- **IDSL_MINT** (Provides the .msp block parser that locates SMILES/InChI/Fingerprint labels case-insensitively, resolves duplicate headers by longest content, and reads comment/comments variants for MoNA and GNPS files) — https://github.com/idslme/IDSL_MINT
- **RDKit** (Converts InChI row entries parsed from MSP blocks into canonical SMILES so fingerprints can be computed downstream) — https://www.rdkit.org/docs/Install.html

## Examples

```
MINT_workflow --yaml /path/to/yaml/file
```

## Evaluation signals

- Every parsed block contains a 'PrecursorMZ: ' row — blocks without it are rejected per IDSL_MINT requirements
- When duplicate headers occur in a block (e.g., two Synon or two SMILES rows), the longest content is selected, matching IDSL_MINT parsing rules
- Extracted Fingerprint rows are dash-separated bit-location strings consistent with the MSP Fingerprint format (e.g., the Aspirin block: 15-53-85-157-…-3996)
- SMILES/InChI labels are matched case-insensitively and only between the Name and Num Peaks rows
- Spot-check parsing against the example Aspirin MSP block with 10 peaks and 61 fingerprint bit locations

## Limitations

- MSP blocks without 'PrecursorMZ: ' row entries are unsupported — IDSL_MINT parsing fails
- When multiple similar headers appear in a block, only the longest-content entry is kept, so shorter duplicate field values are silently discarded
- No numeric accuracy threshold is reported for this encoding/parsing step; validation is format consistency, not bitwise correctness against a ground truth

## Evidence

- [readme] IDSL_MINT can extract information from `comment: ` and `comments: ` entries in ***.msp*** files which enables this platform to process MoNA, GNPS: "IDSL_MINT can extract information from `comment: ` and `comments: ` entries in ***.msp*** files which enables this platform to process MoNA, GNPS"
- [readme] IDSL_MINT identifies chemical structures through `SMILES: ` or `InChI: ` labels without case sensitivity.: "IDSL_MINT identifies chemical structures through `SMILES: ` or `InChI: ` labels without case sensitivity."
- [readme] In case multiple similar headers are present in a MSP block, the one with the longest content is selected for parsing.: "In case multiple similar headers are present in a MSP block, the one with the longest content is selected for parsing."
- [readme] MSP blocks must include `PrecursorMZ: ` row entries.: "MSP blocks must include `PrecursorMZ: ` row entries."
- [methods] InChI and SMILES row entries in the MSP blocks are converted into canonical SMILES using [RDKit]: "InChI and SMILES row entries in the MSP blocks are converted into canonical SMILES using [RDKit]"
- [readme] Another option to obtain molecular fingerprints is to parse the MSP files for the user-provided fingerprints.: "Another option to obtain molecular fingerprints is to parse the MSP files for the user-provided fingerprints."
