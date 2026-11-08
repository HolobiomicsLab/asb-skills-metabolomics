---
name: fingerprint-bit-tokenization-for-seq2seq-targets
description: 'Use when an .msp reference MS/MS library lacks precomputed `Fingerprint:
  ` rows, and bit-location descriptor targets must be derived from SMILES/InChI to
  train the MS/MS-to-fingerprint transformer.'
when_to_use_negative:
- The desired decoder output is canonical SMILES rather than fingerprint bit locations
  — use the MS/MS-to-SMILES translation workflow instead
- 'The .msp blocks contain no `SMILES: ` or `InChI: ` labels and no user-provided
  `Fingerprint: ` rows, so no descriptor source exists'
- The input is already a tokenized fingerprint target file rather than .msp reference
  spectra
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: RDKit
  role: Converts InChI row entries to canonical SMILES and computes ECFP (adjustable
    radius) or MACCS Keys molecular fingerprints from the canonical structures
  repo: https://www.rdkit.org/docs/Install.html
- name: IDSL_MINT
  role: Parses .msp blocks (case-insensitive SMILES/InChI labels, longest-content
    selection on duplicate headers), serializes fingerprints as dash-separated bit-location
    strings, and emits tokenized_fingerprints.csv plus mint_fingerprint_encoder.py
  repo: https://github.com/idslme/IDSL_MINT
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s13321-024-00804-5
    title: 'IDSL_MINT: a deep learning framework to predict molecular fingerprints
      from mass spectra'
  source_content_hash: sha256:6a2b216573f47090a171542318443e2319bf5b8b500fde210f897f9c6151df86
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - IDSL_MINT
  - RDKit
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# fingerprint-bit-tokenization-for-seq2seq-targets

## Summary

Generates molecular fingerprint bit-location targets (e.g., ECFP with adjustable radius such as ECFP2, or MACCS Keys) from SMILES/InChI entries in NIST .msp reference MS/MS blocks, so the IDSL_MINT transformer decoder can be trained to translate MS/MS spectra into descriptor tokens.

## When to use

Use when an .msp reference MS/MS library lacks precomputed `Fingerprint: ` rows, and bit-location descriptor targets must be derived from SMILES/InChI to train the MS/MS-to-fingerprint transformer.

## When NOT to use

- The desired decoder output is canonical SMILES rather than fingerprint bit locations — use the MS/MS-to-SMILES translation workflow instead
- The .msp blocks contain no `SMILES: ` or `InChI: ` labels and no user-provided `Fingerprint: ` rows, so no descriptor source exists
- The input is already a tokenized fingerprint target file rather than .msp reference spectra

## Inputs

- Reference MS/MS library in NIST .msp text format with Name, SMILES:/InChI:, and PrecursorMZ: rows (blocks split between Name and Num Peaks rows)
- User-provided dash-separated `Fingerprint: ` rows (optional alternative to computed descriptors)
- YAML model configuration file specifying the fingerprint type and ECFP radius

## Outputs

- tokenized_fingerprints.csv with per-compound tokenized fingerprint targets
- mint_fingerprint_encoder.py (saved fingerprint encoder)
- Dash-separated bit-location strings in the same format as MSP `Fingerprint` rows

## How to apply

Parse the reference .msp into blocks and locate `SMILES: ` or `InChI: ` labels case-insensitively between the Name and Num Peaks rows; when duplicate headers occur in a block, select the entry with the longest content, and confirm each block carries a `PrecursorMZ: ` row. Convert InChI row entries to canonical SMILES with RDKit, then compute Extended-connectivity fingerprints (ECFPs) with an adjustable radius (e.g., ECFP2) or MACCS Keys as the alternative RDKit descriptor method. If the user has supplied custom fingerprints, instead parse the dash-separated `Fingerprint: ` rows, which are permitted anywhere between the Name and Num Peaks rows. Serialize each fingerprint as a dash-separated bit-location string (e.g., Aspirin's `15-53-85-...-3996`), write the per-compound tokenized targets to tokenized_fingerprints.csv, and save the encoder as mint_fingerprint_encoder.py.

## Related tools

- **RDKit** (Converts InChI row entries to canonical SMILES and computes ECFP (adjustable radius) or MACCS Keys molecular fingerprints from the canonical structures) — https://www.rdkit.org/docs/Install.html
- **IDSL_MINT** (Parses .msp blocks (case-insensitive SMILES/InChI labels, longest-content selection on duplicate headers), serializes fingerprints as dash-separated bit-location strings, and emits tokenized_fingerprints.csv plus mint_fingerprint_encoder.py) — https://github.com/idslme/IDSL_MINT

## Examples

```
MINT_workflow --yaml /path/to/yaml/file
```

## Evaluation signals

- Every emitted target is a dash-separated bit-location string consistent with the MSP `Fingerprint` row format
- Spot-check output against the example Aspirin MSP block (e.g., its `Fingerprint: 15-53-85-...-3996` row); no numeric accuracy threshold is reported in the text for this encoding step
- Every InChI entry produces a canonical SMILES via RDKit before fingerprint computation
- Duplicate headers in a block resolve to the longest content; every retained block contains a `PrecursorMZ: ` row
- tokenized_fingerprints.csv and mint_fingerprint_encoder.py are produced, and the encoder reproduces the same bit locations when re-run on the same SMILES

## Limitations

- Validation of the encoding step is by format spot-check only; no numeric accuracy threshold is reported in the text
- MSP blocks must include `PrecursorMZ: ` row entries, and chemical structures must be discoverable via `SMILES: ` or `InChI: ` labels or user-provided `Fingerprint: ` rows
- Fingerprint descriptor choice (ECFP radius vs. MACCS Keys vs. custom rows) is user-configurable, so targets are only reproducible if the same descriptor method/radius is reused

## Evidence

- [methods] InChI conversion to canonical SMILES and fingerprint computation are done via RDKit: "InChI and SMILES row entries in the MSP blocks are converted into canonical SMILES using [RDKit]"
- [readme] IDSL_MINT locates structures case-insensitively in .msp blocks: "**IDSL_MINT** identifies chemical structures through `SMILES: ` or `InChI: ` labels without case sensitivity."
- [readme] Duplicate .msp headers are resolved by longest content: "In case multiple similar headers are present in a MSP block, the one with the longest content is selected for parsing."
- [readme] ECFP or MACCS Keys computed from InChI/SMILES, or user-provided fingerprints parsed: "This method offers the option to calculate fingerprints using the [Extended-connectivity fingerprints (ECFPs)]"
- [abstract] Overall purpose: translating MS/MS spectra into fingerprint descriptors: "IDSL_MINT can translate MS/MS spectra into molecular fingerprint descriptors."
