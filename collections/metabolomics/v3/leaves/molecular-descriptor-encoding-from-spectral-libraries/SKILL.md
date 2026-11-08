---
name: molecular-descriptor-encoding-from-spectral-libraries
description: 'Use when building IDSL_MINT training targets from a reference .msp MS/MS
  library whose blocks contain `SMILES: ` or `InChI: ` rows but no user-provided `Fingerprint:
  ` bit-location rows, and decoder targets must be derived computationally.'
when_to_use_negative:
- 'MSP blocks already contain `Fingerprint: ` rows — IDSL_MINT parses those user-provided
  fingerprints directly instead of generating descriptors from structure'
- The task is the inverse translation direction — predicting canonical SMILES from
  MS/MS spectra, or forecasting MS/MS fragments from fingerprints
- 'Input is not in .msp format or blocks lack `PrecursorMZ: ` rows, which IDSL_MINT
  requires'
tools:
- name: RDKit
  role: Converts InChI row entries to canonical SMILES and computes the Extended-connectivity
    fingerprints (ECFPs, adjustable radius) or MACCS Keys from which bit-location
    targets are extracted
  repo: https://www.rdkit.org/
- name: IDSL_MINT
  role: 'Parses .msp blocks (case-insensitive `SMILES: `/`InChI: ` detection, longest-content
    selection for duplicate headers), tokenizes fingerprints, writes tokenized_fingerprints.csv
    and saves mint_fingerprint_encoder.py'
  repo: https://github.com/idslme/IDSL_MINT
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s13321-024-00804-5
    title: 'IDSL_MINT: a deep learning framework to predict molecular fingerprints
      from mass spectra'
  source_content_hash: sha256:2b1a0e6884375e0bf168f8b81a301f2b6051b3cb671a64326a314263bfb680cc
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - IDSL_MINT
  - RDKit
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# molecular-descriptor-encoding-from-spectral-libraries

## Summary

Reconstructs the IDSL_MINT module that generates molecular fingerprint training targets (e.g., ECFP2 with radius 2, or MACCS Keys) directly from SMILES/InChI rows in .msp reference MS/MS library blocks when precomputed `Fingerprint:` rows are absent, producing dash-separated bit-location strings for the transformer decoder.

## When to use

Use when building IDSL_MINT training targets from a reference .msp MS/MS library whose blocks contain `SMILES: ` or `InChI: ` rows but no user-provided `Fingerprint: ` bit-location rows, and decoder targets must be derived computationally.

## When NOT to use

- MSP blocks already contain `Fingerprint: ` rows — IDSL_MINT parses those user-provided fingerprints directly instead of generating descriptors from structure
- The task is the inverse translation direction — predicting canonical SMILES from MS/MS spectra, or forecasting MS/MS fragments from fingerprints
- Input is not in .msp format or blocks lack `PrecursorMZ: ` rows, which IDSL_MINT requires

## Inputs

- Reference MS/MS library in NIST .msp format with `SMILES: ` or `InChI: ` rows inside each block (e.g., LipidMaps, MoNA, GNPS exports)
- Optional user-provided dash-separated `Fingerprint: ` rows in the .msp blocks
- YAML model configuration specifying fingerprint type and radius/bit count (e.g., ECFP2, MACCS Keys)

## Outputs

- tokenized_fingerprints.csv with per-compound dash-separated fingerprint bit-location strings (e.g., `15-53-85-157-...`)
- mint_fingerprint_encoder.py — saved fingerprint encoder used to decode/encode bit-location tokens
- Canonical SMILES per compound derived from InChI rows via RDKit

## How to apply

Parse the reference .msp file into blocks, locating `SMILES: ` or `InChI: ` labels case-insensitively between the Name and Num Peaks rows and selecting the longest content when duplicate headers occur, per IDSL_MINT parsing rules. Convert InChI row entries to canonical SMILES with RDKit, then compute the fingerprint for each canonical structure using RDKit Extended-connectivity fingerprints (ECFPs) with an adjustable radius (e.g., ECFP2) or MACCS Keys, honoring dash-separated `Fingerprint: ` rows where the user supplied them. Serialize each fingerprint as a dash-separated bit-location string in the same format as MSP `Fingerprint: ` rows (as in the Aspirin example block: `15-53-85-157-...-3996`), entries allowed anywhere between the Name and Num Peaks rows. Write the per-compound tokenized fingerprint targets to tokenized_fingerprints.csv and save the encoder as mint_fingerprint_encoder.py so the transformer decoder can consume the bit-location tokens as translation targets.

## Related tools

- **RDKit** (Converts InChI row entries to canonical SMILES and computes the Extended-connectivity fingerprints (ECFPs, adjustable radius) or MACCS Keys from which bit-location targets are extracted) — https://www.rdkit.org/
- **IDSL_MINT** (Parses .msp blocks (case-insensitive `SMILES: `/`InChI: ` detection, longest-content selection for duplicate headers), tokenizes fingerprints, writes tokenized_fingerprints.csv and saves mint_fingerprint_encoder.py) — https://github.com/idslme/IDSL_MINT

## Examples

```
MINT_workflow --yaml /path/to/yaml/file
```

## Evaluation signals

- Every emitted target is a dash-separated bit-location string consistent with the MSP `Fingerprint: ` row format
- Spot-check the generated encoding against the example Aspirin MSP block, whose Fingerprint row reads `15-53-85-157-246-322-329-343-444-464-553-708-763-785-799-821-847-1040-1139-1240-1250-1317-1348-1439-1450-1460-1475-1479-1502-1674-1693-1734-1841-1866-2046-2310-2329-2413-2627-2750-2755-2777-2782-2799-2901-2911-2915-3028-3049-3394-3412-3442-3514-3535-3557-3700-3737-3785-3972-3996`
- Each InChI-derived structure round-trips through RDKit as a parseable canonical SMILES before fingerprint computation
- tokenized_fingerprints.csv contains one tokenized target per compound and mint_fingerprint_encoder.py is saved alongside it
- Bit locations fall within the bounds of the configured fingerprint size (ECFP bit count / MACCS 167-key space) and block entries sit between the Name and Num Peaks rows

## Limitations

- No numeric accuracy threshold is reported in the article for this encoding step — validation is format/spot-check based, not benchmarked
- MSP blocks must include `PrecursorMZ: ` row entries, and parsing depends on case-insensitive `SMILES: `/`InChI: ` labels; when multiple similar headers appear in a block, IDSL_MINT selects the one with the longest content, which could misresolve
- Only the ECFP/MACCS RDKit methods and user-provided dash-separated `Fingerprint: ` rows are supported as descriptor sources

## Evidence

- [other] When customized binary fingerprint bit locations are absent from the MSP blocks, IDSL_MINT generates the fingerprint descriptors itself from the SMILES or InChI strings: [evidence span withheld — anchor status: not_found]
- [methods] InChI-to-SMILES conversion step: "InChI and SMILES row entries in the MSP blocks are converted into canonical SMILES using [RDKit]"
- [readme] README: descriptor options: [evidence span withheld — anchor status: not_found]
- [readme] README: structure-label parsing rules: "IDSL_MINT identifies chemical structures through `SMILES: ` or `InChI: ` labels without case sensitivity."
- [readme] README: duplicate-header selection: "In case multiple similar headers are present in a MSP block, the one with the longest content is selected for parsing."
