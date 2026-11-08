---
name: nrps-a-domain-substrate-extraction
description: Use when parsing MIBiG JSON-formatted BGC entries to quantify NRPS A-domain
  substrate annotations (expect 2775 A-domains, 274 unique substrates) or to build
  substrate-SMILES training data.
when_to_use_negative:
- You need PKS AT-domain substrate annotations or other protein-domain selectivities
  rather than NRPS A-domain substrates
- You need experimentally validated full BGC entries rather than the derived substrate-SMILES
  tally (use the MIBiG Repository directly)
- The A-domains of interest are not present in MIBiG entries — this skill only counts
  annotations already curated in MIBiG, it does not predict substrates for novel sequences
  (use predictors such as SANDPUMA or NRPSPredictor2 for prediction)
edam_operation: http://edamontology.org/operation_0222
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: PIKAChU
  role: Validates the correctness of the isomeric SMILES syntax of each extracted
    A-domain substrate structure
- name: MIBiG 3.0
  role: Source database of NRPS A-domain substrate annotations, distributed as JSON
    via the mibig-secmet/mibig-json repository
  repo: https://mibig.secondarymetabolites.org/
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkac1049
    title: 'MIBiG 3.0: a community-driven effort to annotate experimentally validated
      biosynthetic gene clusters'
  source_content_hash: sha256:de682c5a9c90365c0e03f98a507c6c3598be6879da5bd2791a8977159d8eee1d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MIBiG 3.0
  - PIKAChU
  edam_operation: http://edamontology.org/operation_0222
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# nrps-a-domain-substrate-extraction

## Summary

Extracts and tallies NRPS adenylation (A)-domain substrate annotations from MIBiG 3.0 JSON entries, validating each substrate's isomeric SMILES with PIKAChU and producing a substrate-to-SMILES mapping with per-substrate A-domain counts.

## When to use

Use when parsing MIBiG JSON-formatted BGC entries to quantify NRPS A-domain substrate annotations (expect 2775 A-domains, 274 unique substrates) or to build substrate-SMILES training data.

## When NOT to use

- You need PKS AT-domain substrate annotations or other protein-domain selectivities rather than NRPS A-domain substrates
- You need experimentally validated full BGC entries rather than the derived substrate-SMILES tally (use the MIBiG Repository directly)
- The A-domains of interest are not present in MIBiG entries — this skill only counts annotations already curated in MIBiG, it does not predict substrates for novel sequences (use predictors such as SANDPUMA or NRPSPredictor2 for prediction)

## Inputs

- MIBiG 3.0 JSON-formatted BGC entries from the mibig-secmet/mibig-json repository `data` directory
- Isomeric SMILES strings linked to each NRPS A-domain substrate name within each entry

## Outputs

- a_domain_substrates.csv mapping each unique substrate name to its isomeric SMILES and per-substrate A-domain count
- Total A-domain annotation count (expected 2775) and unique substrate count (expected 274)

## How to apply

Download the JSON-formatted MIBiG 3.0 entries from the mibig-secmet/mibig-json GitHub repository, where the current datasets live in the `data` directory with entry status tracked via the `cluster.status` field. Recursively parse each JSON entry with Python's json library, extracting the NRPS A-domain substrate annotations together with the isomeric SMILES string linked to each substrate name. Count the total number of annotated A-domains and verify the total equals 2775 (versus 755 in MIBiG 2.0), since A-domain substrate prediction in NRPS systems is harder than AT-domain prediction in PKS systems and requires substantially more data. Deduplicate substrate names, validate each isomeric SMILES string's syntax with PIKAChU exactly as done in the MIBiG 3.0 paper, and tally A-domain counts per unique substrate (274 unique names expected). Write the resulting substrate-to-SMILES mapping with per-substrate A-domain counts to a_domain_substrates.csv.

## Related tools

- **PIKAChU** (Validates the correctness of the isomeric SMILES syntax of each extracted A-domain substrate structure)
- **MIBiG 3.0** (Source database of NRPS A-domain substrate annotations, distributed as JSON via the mibig-secmet/mibig-json repository) — https://mibig.secondarymetabolites.org/

## Examples

```
python -c "import json; e=json.load(open('data/BGC0000001.json')); [print(g, s) for g in e.get('biosynthetic_class',[]) for s in [e]]" > a_domain_substrates.csv
```

## Evaluation signals

- Total parsed A-domain annotations must equal 2775 (MIBiG 3.0), not 755 (the MIBiG 2.0 figure)
- Deduplicated substrate count must equal 274 unique substrate names
- Every extracted substrate SMILES must pass PIKAChU syntax validation
- Output a_domain_substrates.csv must contain one row per unique substrate with a non-empty isomeric SMILES and its per-substrate A-domain tally

## Limitations

- Counts are specific to MIBiG 3.0 (2775 A-domains, 274 unique substrates); parsing MIBiG 2.0 or later releases will change the totals
- Only a handful of GenBank sequence files in the repository's `genbanks` directory are not available from NCBI GenBank/RefSeq, so cross-checking against NCBI may be incomplete
- The skill tallies curated annotations only; A-domain substrate prediction remains a greater challenge requiring substantially more data than is present here

## Evidence

- [intro] Correctness of SMILES syntax was validated with PIKAChU (20): "Correctness of SMILES syntax was validated with PIKAChU (20)"
- [intro] A-domain challenge motivation: "for A-domains in NRPS systems, which recognise over 500 known substrates (22), substrate prediction is a greater challenge, which will require substantially more data"
- [readme] JSON data location and status tracking: "The current datasets for [MIBiG] live in the `data` directory, entry status is now tracked via the `cluster.status` field."
- [supplementary] GitHub availability of JSON data standard: "The source code components, JSON-formatted data standard, and SQL schema for the MIBiG Repository are available on GitHub"
