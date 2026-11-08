---
name: isomeric-smiles-validation
description: Use when a curated database (e.g. MIBiG JSON entries) assigns named substrates
  to enzyme domains and each name must be backed by a valid isomeric SMILES before
  counting or downstream ML use.
when_to_use_negative:
- Input structures are only InChI or plain names with no isomeric SMILES to validate
- Goal is generic BGC detection rather than validating chemical structure annotations
  (use antiSMASH/GECCO instead)
edam_operation: http://edamontology.org/operation_3434
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0154
tools:
- name: PIKAChU
  role: Validates correctness of SMILES syntax for each curated substrate structure
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkac1049
    title: 'MIBiG 3.0: a community-driven effort to annotate experimentally validated
      biosynthetic gene clusters'
  source_content_hash: sha256:7091c5ee93a3702c3638270a50c0c654a449e187144bc3e3a5f72eeadf99b4ff
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PIKAChU
  edam_operation: http://edamontology.org/operation_3434
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# Isomeric SMILES validation

## Summary

Validates that substrate annotations carry syntactically correct, stereochemically curated isomeric SMILES strings before they are used to build substrate-to-structure mappings (e.g. the 274 unique NRPS A-domain substrates in MIBiG 3.0).

## When to use

Use when a curated database (e.g. MIBiG JSON entries) assigns named substrates to enzyme domains and each name must be backed by a valid isomeric SMILES before counting or downstream ML use.

## When NOT to use

- Input structures are only InChI or plain names with no isomeric SMILES to validate
- Goal is generic BGC detection rather than validating chemical structure annotations (use antiSMASH/GECCO instead)

## Inputs

- MIBiG 3.0 JSON entries (from mibig-secmet/mibig-json, data directory)
- NRPS A-domain substrate annotations with associated isomeric SMILES strings

## Outputs

- Validated substrate-to-SMILES mapping (e.g. a_domain_substrates.csv) with per-substrate A-domain counts
- List of rejected/invalid SMILES annotations

## How to apply

Parse the JSON-formatted MIBiG entries (from the mibig-secmet/mibig-json repository), extracting each NRPS A-domain substrate annotation together with its linked isomeric SMILES string. Validate every SMILES string's syntax with PIKAChU, the same tool the MIBiG 3.0 curators used, rejecting or flagging entries whose SMILES cannot be parsed. Deduplicate substrates by their validated isomeric SMILES (MIBiG 3.0 collapses 2775 A-domain annotations to 274 unique substrates), then emit a substrate-to-SMILES mapping with per-substrate domain counts. Curation must preserve stereochemistry, since stereochemically curated isomeric SMILES are the key used to distinguish substrates.

## Related tools

- **PIKAChU** (Validates correctness of SMILES syntax for each curated substrate structure)

## Examples

```
import json, glob; entries=[json.load(open(f)) for f in glob.glob('data/*.json')]; smiles=[s['smiles'] for e in entries for c in e['cluster'].get('nrps',[]) for s in c.get('substrates',[])]  # then validate each with PIKAChU
```

## Evaluation signals

- Every substrate SMILES parses without error under PIKAChU
- Deduplicated unique substrate count matches the expected 274 for MIBiG 3.0 A-domains
- Total annotated A-domains equals 2775 (versus 755 in MIBiG 2.0)
- Each retained substrate name maps to exactly one stereochemically curated isomeric SMILES

## Limitations

- Stereochemistry must be curated manually by annotators; SMILES syntax validity alone does not guarantee correct stereochemical assignment
- Coverage is limited to substrates represented in MIBiG entries; A-domains recognize over 500 known substrates, so unique counts are lower-bounded by curation effort

## Evidence

- [intro] Correctness of SMILES syntax was validated with PIKAChU (20): "Correctness of SMILES syntax was validated with PIKAChU (20)"
- [intro] cross-link them to five databases of chemical structures: "cross-link them to five databases of chemical structures: NP Atlas (9), PubChem, ChemSpider (17), LOTUS (18), and ChEMBL (19)"
- [readme] JSON format tracking: "This repository tracks the current [MIBiG] annotations in JSON format."
