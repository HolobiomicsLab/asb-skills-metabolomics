---
name: rdkit-ecfp-fingerprint-generation
description: 'Use when training data are NIST .msp MS/MS blocks with SMILES: or InChI:
  rows but no dash-separated Fingerprint: bit-location entries, and decoder targets
  must be derived for supervised fingerprint prediction.'
when_to_use_negative:
- 'MSP blocks already contain dash-separated Fingerprint: rows — parse them directly
  instead of recomputing descriptors.'
- The goal is structure prediction (MS/MS to canonical SMILES) or spectra prediction
  (fingerprints to MS/MS fragments), not fingerprint-target construction.
- Input is raw LC-MS data rather than curated .msp reference blocks with structural
  identifiers.
edam_operation: http://edamontology.org/operation_3859
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3316
tools:
- name: RDKit
  role: Converts InChI row entries to canonical SMILES and computes ECFP (adjustable
    radius) or MACCS Keys fingerprints from those structures
  repo: https://www.rdkit.org/docs/Install.html
- name: IDSL_MINT
  role: Wraps the fingerprint generation in the MSP-parsing pipeline and consumes
    the resulting bit-location targets to train the transformer decoder
  repo: https://github.com/idslme/IDSL_MINT
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s13321-024-00804-5
    title: 'IDSL_MINT: a deep learning framework to predict molecular fingerprints
      from mass spectra'
  source_content_hash: sha256:c5ee931b74d505b775638ffb4ed4c4528e6e59c68afcbb12e6425a3c338fd312
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - IDSL_MINT
  - RDKit
  edam_operation: http://edamontology.org/operation_3859
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# rdkit-ecfp-fingerprint-generation

## Summary

Generates molecular fingerprint bit-location targets from SMILES/InChI structures using RDKit extended-connectivity fingerprints (ECFP, e.g. ECFP2 with adjustable radius) or MACCS Keys, so that IDSL_MINT transformer decoders can be trained when MSP blocks lack precomputed Fingerprint rows.

## When to use

Use when training data are NIST .msp MS/MS blocks with SMILES: or InChI: rows but no dash-separated Fingerprint: bit-location entries, and decoder targets must be derived for supervised fingerprint prediction.

## When NOT to use

- MSP blocks already contain dash-separated Fingerprint: rows — parse them directly instead of recomputing descriptors.
- The goal is structure prediction (MS/MS to canonical SMILES) or spectra prediction (fingerprints to MS/MS fragments), not fingerprint-target construction.
- Input is raw LC-MS data rather than curated .msp reference blocks with structural identifiers.

## Inputs

- NIST .msp file split into blocks containing Name, SMILES: or InChI: rows and PrecursorMZ: entries
- InChI strings (converted to canonical SMILES via RDKit)
- Optional user-provided dash-separated Fingerprint: rows in MSP blocks
- Fingerprint type and radius/bit-count configuration (e.g. ECFP2, MACCS Keys) from the model YAML file

## Outputs

- tokenized_fingerprints.csv with per-compound dash-separated fingerprint bit-location strings
- mint_fingerprint_encoder.py (saved fingerprint encoder for training and inference)

## How to apply

Parse each .msp block and locate `SMILES: ` or `InChI: ` labels case-insensitively between the Name and Num Peaks rows, selecting the longest content when duplicate headers appear in a block as IDSL_MINT requires; convert InChI entries to canonical SMILES with RDKit. Compute the fingerprint for each canonical structure with RDKit using extended-connectivity fingerprints with an adjustable radius (e.g. ECFP2) or MACCS Keys as an alternative descriptor, or honor a user-provided dash-separated `Fingerprint:` row when present. Serialize each fingerprint as a dash-separated bit-location string matching the MSP `Fingerprint` row format (e.g. Aspirin: 15-53-85-157-...-3996), which may sit anywhere between the Name and Num Peaks rows. Write the per-compound tokenized targets to tokenized_fingerprints.csv and save the encoder as mint_fingerprint_encoder.py for reuse at inference time.

## Related tools

- **RDKit** (Converts InChI row entries to canonical SMILES and computes ECFP (adjustable radius) or MACCS Keys fingerprints from those structures) — https://www.rdkit.org/docs/Install.html
- **IDSL_MINT** (Wraps the fingerprint generation in the MSP-parsing pipeline and consumes the resulting bit-location targets to train the transformer decoder) — https://github.com/idslme/IDSL_MINT

## Examples

```
from rdkit import Chem; from rdkit.Chem import AllChem; mol = Chem.MolFromSmiles(Chem.MolToSmiles(Chem.MolFromInchi('InChI=1S/C9H8O4/c1-6(10)13-8-5-3-2-4-7(8)9(11)12/h2-5H,1H3,(H,11,12)'))); bits = sorted(fp.GetOnBits()) for fp in [AllChem.GetMorganFingerprintAsBitVect(mol, 2)]
```

## Evaluation signals

- Every emitted target is a dash-separated string of integer bit locations consistent with the MSP Fingerprint row format.
- Bit locations fall within the configured fingerprint size (e.g. values up to 3996 in the Aspirin example ECFP string).
- Spot-check the generated string against the example Aspirin MSP block fingerprint (no numeric accuracy threshold is reported for this encoding step).
- Each compound with a parseable SMILES/InChI yields a fingerprint row; blocks lacking PrecursorMZ: or structural rows should be caught as parse failures.
- The saved mint_fingerprint_encoder.py reproduces the same bit strings when re-applied to the same canonical SMILES.

## Limitations

- When multiple duplicate headers occur in an MSP block, the longest content is selected — shorter alternative annotations are discarded.
- The paper reports no numeric accuracy threshold for the encoding step; validation is by format consistency and spot-checking.
- MSP blocks must include PrecursorMZ: row entries, so blocks without it are unusable.
- Fingerprint type and radius are user-configurable (YAML), so target dimensions vary between trained models and are not interchangeable.

## Evidence

- [methods] convert InChI to canonical SMILES: "InChI and SMILES row entries in the MSP blocks are converted into canonical SMILES using [RDKit]"
- [readme] fingerprint computation options: "calculate fingerprints using the [Extended-connectivity fingerprints (ECFPs)](https://doi.org/10.1021/ci100050t) or [MACCS Keys](https://doi.org/10.1021/ci200081k) RDKit methods from InChI and SMILES"
- [readme] duplicate header rule: "In case multiple similar headers are present in a MSP block, the one with the longest content is selected for parsing."
- [readme] user-provided fingerprints alternative: "Another option to obtain molecular fingerprints is to parse the MSP files for the user-provided fingerprints."
- [intro] token translation framing: "Converting a MS/MS spectrum to molecular descriptor is in a way, a language translation problem, in which token A (m/z and intensity) are translated into token B (descriptors)"
