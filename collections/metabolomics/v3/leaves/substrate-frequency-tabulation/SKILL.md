---
name: substrate-frequency-tabulation
description: Use when you need to count and tabulate NRPS A-domain substrate annotations
  (e.g. per-substrate frequencies, unique substrate totals) from structured biosynthetic
  gene cluster records such as MIBiG JSON entries.
when_to_use_negative:
- You need A-domain substrate *predictions* (e.g. SANDPUMA or NRPSPredictor2 outputs)
  rather than curated experimental annotations from validated BGC entries.
- You are counting PKS AT-domain substrates or other domain selectivities; this tabulation
  targets NRPS A-domain annotations specifically.
- Input records lack isomeric SMILES or have unvalidated SMILES strings, so substrate
  identity cannot be deduplicated reliably.
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0154
tools:
- name: PIKAChU
  role: Validates correctness of SMILES syntax of substrate isomeric SMILES strings
    before they are used as deduplication keys and written to the output table
- name: MIBiG 3.0
  role: Source database of experimentally validated BGC entries whose NRPS A-domain
    substrate annotations (2775 A-domains, 274 unique substrates) are parsed and tabulated
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkac1049
    title: 'MIBiG 3.0: a community-driven effort to annotate experimentally validated
      biosynthetic gene clusters'
  source_content_hash: sha256:486b324586f132d8159dd0c52cbb504c2159e175d65bcfedbf760f0863d5dd27
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MIBiG 3.0
  - PIKAChU
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# substrate-frequency-tabulation

## Summary

Tabulates NRPS A-domain substrate annotations from MIBiG JSON entries, deduplicating substrates by stereochemically curated isomeric SMILES and counting annotated A-domains per substrate. Produces the substrate-to-SMILES frequency table used to quantify MIBiG 3.0's A-domain annotation coverage.

## When to use

Use when you need to count and tabulate NRPS A-domain substrate annotations (e.g. per-substrate frequencies, unique substrate totals) from structured biosynthetic gene cluster records such as MIBiG JSON entries.

## When NOT to use

- You need A-domain substrate *predictions* (e.g. SANDPUMA or NRPSPredictor2 outputs) rather than curated experimental annotations from validated BGC entries.
- You are counting PKS AT-domain substrates or other domain selectivities; this tabulation targets NRPS A-domain annotations specifically.
- Input records lack isomeric SMILES or have unvalidated SMILES strings, so substrate identity cannot be deduplicated reliably.

## Inputs

- MIBiG 3.0 JSON-formatted entries from the mibig-secmet/mibig-json repository (data directory)
- NRPS A-domain substrate annotation fields with linked isomeric SMILES strings

## Outputs

- a_domain_substrates.csv: substrate name to SMILES mapping with per-substrate A-domain counts
- Totals of annotated A-domains (2775) and unique substrates (274)

## How to apply

Download the JSON-formatted MIBiG 3.0 entries from the mibig-secmet/mibig-json GitHub repository, where curation data live in the `data` directory with entry status tracked via the `cluster.status` field. Recursively parse each JSON entry with Python's json library, extracting NRPS A-domain substrate annotations together with the isomeric SMILES string linked to each substrate name. Validate each isomeric SMILES string's syntax with PIKAChU, as done in the MIBiG 3.0 paper, before using it as the substrate identity key. Deduplicate substrate identities (274 unique substrates expected in MIBiG 3.0), tally the A-domain count per substrate, and verify the total annotated A-domain count (2775 in MIBiG 3.0 vs 755 in MIBiG 2.0). Write the substrate-to-SMILES mapping with per-substrate A-domain counts to a_domain_substrates.csv.

## Related tools

- **PIKAChU** (Validates correctness of SMILES syntax of substrate isomeric SMILES strings before they are used as deduplication keys and written to the output table)
- **MIBiG 3.0** (Source database of experimentally validated BGC entries whose NRPS A-domain substrate annotations (2775 A-domains, 274 unique substrates) are parsed and tabulated)

## Examples

```
import json, glob; entries = [json.load(open(f)) for f in glob.glob('data/*.json')]; print(sum(1 for e in entries for r in e['clusters']['nrps'] if 'a_domains' in str(r)))
```

## Evaluation signals

- Total annotated A-domain count equals 2775 for MIBiG 3.0 (compared to 755 in MIBiG 2.0).
- Number of unique substrates after deduplication equals 274.
- Every substrate-SMILES pair in a_domain_substrates.csv passes PIKAChU SMILES syntax validation.
- Sum of per-substrate A-domain counts equals the total number of parsed A-domain annotations (no rows dropped or double-counted).
- Output CSV contains one mapping per unique substrate with an integer A-domain count column.

## Limitations

- Counts reflect only experimentally validated MIBiG entries; A-domains in unvalidated or predicted BGCs are not covered.
- NRPS A-domain substrates number over 500 in nature, so 274 unique substrates represent only the curated subset; prediction models need substantially more data.
- The repository genbanks directory contains only a handful of sequence files not available from NCBI GenBank/RefSeq, so cross-checking against sequence files may be incomplete.

## Evidence

- [other] Workflow for parsing MIBiG JSON and counting A-domains: [evidence span withheld — anchor status: not_found]
- [intro] PIKAChU SMILES validation: "Correctness of SMILES syntax was validated with PIKAChU (20)"
- [other] Result counts for A-domain annotations: [evidence span withheld — anchor status: not_found]
- [readme] Repository layout of mibig-json: "The current datasets for [MIBiG] live in the `data` directory, entry status is now tracked via the `cluster.status` field."
- [intro] Motivation for large-scale A-domain annotation: "for A-domains in NRPS systems, which recognise over 500 known substrates (22), substrate prediction is a greater challenge, which will require substantially more data"
