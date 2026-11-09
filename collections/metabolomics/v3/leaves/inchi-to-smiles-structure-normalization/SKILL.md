---
name: inchi-to-smiles-structure-normalization
description: 'Use when reference .msp blocks lack precomputed Fingerprint rows and
  contain only InChI: entries (or SMILES needing canonicalization), and dash-separated
  bit-location targets are required for IDSL_MINT transformer decoder training.'
when_to_use_negative:
- The MSP block already carries a user-provided dash-separated `Fingerprint:` row
  — IDSL_MINT parses these directly instead of recomputing from structure.
- The input is not .msp reference data (e.g., the goal is MS/MS prediction from canonical
  SMILES, the reverse translation mode of IDSL_MINT).
- The .msp block has no InChI/SMILES/structure information between the Name and Num
  Peaks rows, so no descriptor can be derived.
edam_topics:
- http://edamontology.org/topic_3452
tools:
- name: RDKit
  role: Converts InChI row entries to canonical SMILES and computes ECFP (adjustable
    radius) or MACCS Keys molecular fingerprints from those canonical structures
  repo: https://www.rdkit.org/docs/Install.html
- name: IDSL_MINT
  role: 'Parses the .msp blocks (case-insensitive SMILES:/InChI: labels, longest-content
    deduplication) and writes tokenized_fingerprints.csv plus mint_fingerprint_encoder.py'
  repo: https://github.com/idslme/IDSL_MINT
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s13321-024-00804-5
    title: 'IDSL_MINT: a deep learning framework to predict molecular fingerprints
      from mass spectra'
  source_content_hash: sha256:f13432c22e0f0f14cab7786cdb1bb33316b19ab82e035ddfe810720b0b2b9a8a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - IDSL_MINT
  - RDKit
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3452
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# inchi-to-smiles-structure-normalization

## Summary

Converts InChI row entries from NIST .msp training blocks into canonical SMILES with RDKit so that ECFP (e.g., ECFP2, radius 2) or MACCS Keys fingerprint bit-location targets can be computed when no user-provided Fingerprint rows exist.

## When to use

Use when reference .msp blocks lack precomputed Fingerprint rows and contain only InChI: entries (or SMILES needing canonicalization), and dash-separated bit-location targets are required for IDSL_MINT transformer decoder training.

## When NOT to use

- The MSP block already carries a user-provided dash-separated `Fingerprint:` row — IDSL_MINT parses these directly instead of recomputing from structure.
- The input is not .msp reference data (e.g., the goal is MS/MS prediction from canonical SMILES, the reverse translation mode of IDSL_MINT).
- The .msp block has no InChI/SMILES/structure information between the Name and Num Peaks rows, so no descriptor can be derived.

## Inputs

- NIST-format .msp file split into blocks with Name, InChI:/SMILES:, and Num Peaks rows
- MSP blocks containing `PrecursorMZ: ` row entries (required by IDSL_MINT)
- Optional user-provided dash-separated `Fingerprint: ` rows (skip generation when present)

## Outputs

- Canonical SMILES string per compound
- Dash-separated fingerprint bit-location string (e.g., ECFP2 or MACCS Keys) matching the MSP Fingerprint row format
- tokenized_fingerprints.csv with per-compound tokenized fingerprint targets
- Saved encoder script mint_fingerprint_encoder.py

## How to apply

Read the .msp file and split it into blocks, locating `SMILES: ` or `InChI: ` labels case-insensitively between the Name and Num Peaks rows; if duplicate headers appear in a block, keep the one with the longest content as IDSL_MINT does. Convert each InChI entry to canonical SMILES using RDKit (MolFromInchi followed by MolToSmiles). Compute molecular fingerprints for each canonical structure with RDKit — extended-connectivity fingerprints (ECFP) with an adjustable radius such as ECFP2, or MACCS Keys as an alternative. Serialize each fingerprint as a dash-separated bit-location string in the same format as user-provided MSP `Fingerprint:` rows (e.g., the Aspirin example: 15-53-85-157-...-3996). Write the per-compound tokenized fingerprint targets to tokenized_fingerprints.csv and save the encoder as mint_fingerprint_encoder.py for reuse at inference time.

## Related tools

- **RDKit** (Converts InChI row entries to canonical SMILES and computes ECFP (adjustable radius) or MACCS Keys molecular fingerprints from those canonical structures) — https://www.rdkit.org/docs/Install.html
- **IDSL_MINT** (Parses the .msp blocks (case-insensitive SMILES:/InChI: labels, longest-content deduplication) and writes tokenized_fingerprints.csv plus mint_fingerprint_encoder.py) — https://github.com/idslme/IDSL_MINT

## Examples

```
from rdkit import Chem; smi = Chem.MolToSmiles(Chem.MolFromInchi('InChI=1S/C9H8O4/c1-6(10)13-8-5-3-2-4-7(8)9(11)12/h2-5H,1H3,(H,11,12)'))
```

## Evaluation signals

- Every emitted target is a dash-separated bit-location string consistent with the MSP `Fingerprint:` row format (spot-check against the Aspirin block 15-53-85-...-3996)
- Each canonical SMILES parses back cleanly in RDKit (round-trip MolFromSmiles succeeds)
- tokenized_fingerprints.csv contains one tokenized fingerprint target per compound in the parsed .msp blocks
- mint_fingerprint_encoder.py is produced and reproduces identical bit locations when re-run on the same structures

## Limitations

- No numeric accuracy threshold is reported in the text for this encoding step — validation is by format consistency and spot-checking only.
- Duplicate SMILES:/InChI: headers are resolved heuristically by selecting the longest content, which may not always reflect the intended structure.
- MSP blocks must include PrecursorMZ row entries; blocks lacking structure rows between Name and Num Peaks cannot yield fingerprints via this path.

## Evidence

- [methods] InChI conversion to canonical SMILES precedes fingerprint computation: "InChI and SMILES row entries in the MSP blocks are converted into canonical SMILES using [RDKit]"
- [readme] Fingerprints computed from InChI/SMILES via RDKit ECFP or MACCS when not user-provided: "calculate fingerprints using the [Extended-connectivity fingerprints (ECFPs)](https://doi.org/10.1021/ci100050t) or [MACCS Keys](https://doi.org/10.1021/ci200081k) RDKit methods from InChI and SMILES"
- [readme] Structure label parsing is case-insensitive: "**IDSL_MINT** identifies chemical structures through `SMILES: ` or `InChI: ` labels without case sensitivity."
- [readme] Duplicate header resolution rule for MSP blocks: "In case multiple similar headers are present in a MSP block, the one with the longest content is selected for parsing."
- [readme] Fingerprint rows may be user-provided as an alternative source: "Another option to obtain molecular fingerprints is to parse the MSP files for the user-provided fingerprints."
