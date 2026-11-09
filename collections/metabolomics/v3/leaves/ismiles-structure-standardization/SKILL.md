---
name: ismiles-structure-standardization
description: Use when building or normalizing a training/prediction corpus of chemical
  structures (e.g. MS2 dataset SMILES) for a sequence-to-sequence transformer whose
  output vocabulary is ISMILES, or when comparing predicted structures against PubChem
  candidates.
when_to_use_negative:
- The structure contains inorganic/metal elements (K, Na, Au, Se, B, Fe, As, Ca, Mg,
  Cr, Al) or disconnected fragments ('.') — the article explicitly excludes these
  as out of scope for organic pollutants
- A SMILES cannot be converted to ISMILES or its ISMILES exceeds 130 characters —
  it is dropped rather than truncated
- 'Offline raw-data processing is required: the README states the current version
  does not support offline processing of raw data'
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: PubChemPy
  role: Performs SMILES→ISMILES conversion and remote PubChem retrieval to standardize
    the output alphabet across the whole dataset
  repo: https://github.com/LQZ0123/MSThunder
- name: RDKit
  role: Generates molecular fingerprints (PubChem 449-bit, MACCS 105-bit, Klekota-Roth
    1633-bit) from the standardized ISMILES for the downstream weighted similarity
    match
  repo: https://github.com/LQZ0123/MSThunder
- name: PubChem
  role: Source database from which candidate structures (ISMILES) are retrieved by
    precursor formula
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.enceco.2025.07.022
    title: 'MSThunder: A deep learning-based nontargeted analytical framework for
      the accurate and rapid identification of unknown organic pollutants in water'
  source_content_hash: sha256:b6079474211b24398fa592d7f3da55659e5c3b0067b960c1588aa5c39e6feef1
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/LQZ0123/MSThunder
  repo_url: https://github.com/LQZ0123/MSThunder
  tools:
  - PubChem
  - PubChemPy
  - Python 3.10.5
  - RDKit
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# ismiles-structure-standardization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Converts SMILES strings retrieved from PubChem into Isomeric SMILES (ISMILES) via PubChemPy so that transformer-predicted sequences and candidate structures share a single canonical character alphabet, with length and element filters applied.

## When to use

Use when building or normalizing a training/prediction corpus of chemical structures (e.g. MS2 dataset SMILES) for a sequence-to-sequence transformer whose output vocabulary is ISMILES, or when comparing predicted structures against PubChem candidates.

## When NOT to use

- The structure contains inorganic/metal elements (K, Na, Au, Se, B, Fe, As, Ca, Mg, Cr, Al) or disconnected fragments ('.') — the article explicitly excludes these as out of scope for organic pollutants
- A SMILES cannot be converted to ISMILES or its ISMILES exceeds 130 characters — it is dropped rather than truncated
- Offline raw-data processing is required: the README states the current version does not support offline processing of raw data

## Inputs

- SMILES strings retrieved from PubChem via PubChemPy for a confirmed precursor molecular formula
- MS2 dataset SMILES sequences to be standardized into ISMILES
- Paired high-resolution MS2 spectra with mass accuracy to two decimal places

## Outputs

- ISMILES sequences (≤130 characters, organic elements only) ready for transformer input/output and fingerprint generation
- Filtered candidate ISMILES list per precursor formula for similarity ranking

## How to apply

Retrieve the candidate SMILES list from PubChem for the confirmed precursor molecular formula using PubChemPy (version 1.0.4 in the released MSThunder build). Convert every SMILES to its ISMILES form so the whole dataset uses one standardized output alphabet — the article states all SMILES sequences from the MS2 dataset were converted to ISMILES 'for standardizing output and maintaining consistency with the whole dataset'. Apply the dataset filter rules: exclude ISMILES containing K, Na, Au, Se, B, Fe, As, Ca, Mg, Cr, Al or the '.' disconnected-fragment token because the scope is organic compounds; exclude SMILES that cannot be replaced by ISMILES or whose ISMILES exceeds 130 characters; and require accompanying high-resolution MS2 spectra with mass accuracy to two decimal places. The ISMILES strings then feed the fingerprint generator (RDKit) for the downstream weighted similarity ranking.

## Related tools

- **PubChemPy** (Performs SMILES→ISMILES conversion and remote PubChem retrieval to standardize the output alphabet across the whole dataset) — https://github.com/LQZ0123/MSThunder
- **RDKit** (Generates molecular fingerprints (PubChem 449-bit, MACCS 105-bit, Klekota-Roth 1633-bit) from the standardized ISMILES for the downstream weighted similarity match) — https://github.com/LQZ0123/MSThunder
- **PubChem** (Source database from which candidate structures (ISMILES) are retrieved by precursor formula)

## Examples

```
import pubchempy as pcp; c = pcp.get_compounds('C6H6', 'formula'); smiles = [c_.isomeric_smiles for c_ in c]
```

## Evaluation signals

- No standardized ISMILES contains the excluded tokens K, Na, Au, Se, B, Fe, As, Ca, Mg, Cr, Al or '.'
- All retained ISMILES strings have length ≤ 130 characters
- Every SMILES in the corpus has a corresponding ISMILES replacement (no 'cannot be replaced' cases remain)
- Predicted ISMILES sequences and candidate ISMILES share the same character alphabet, enabling exact string comparison and RDKit fingerprint generation without re-conversion
- Each retained structure is backed by high-resolution MS2 spectra with mass accuracy to two decimal places

## Limitations

- SMILES that cannot be replaced by ISMILES, or whose ISMILES length exceeds 130 characters, are simply excluded — no fallback representation is provided
- ISMILES with metal/metalloid elements or '.' fragments are outside the organic-compound scope of the skill
- The current MSThunder release does not support offline processing of raw data; raw files must be converted via MSConvert and processed on a Linux system

## Evidence

- [other] Standardization rationale: "All SMILES sequences from the MS2 dataset were converted to ISMILES sequences for standardizing output and maintaining consistency with the whole dataset using PubChemPy."
- [other] Candidate retrieval scope: "Upon initial confirmation of the precursor formula, it was utilized for structural (ISMILES) retrieval in PubChem via PubChemPy, which allows remote access to the database and generates a list of"
- [other] Dataset filter rules: "initial dataset was filtered to ensure the quality of dataset according to the following rules: i) ISMILES containing K, Na, Au, Se, B, Fe, As, Ca, Mg, Cr, Al, and “.” were excluded due to the"
- [other] Length filter: "ii) SMILES that cannot be replaced by ISMILES and/or ISMILES with a character length exceeding 130 were excluded"
- [other] Spectral quality gate: "iii) the input data were excluded unless accompanied by high-resolution MS2 spectra demonstrating mass accuracy to two decimal places"
