---
name: compound-structure-coverage-tallying
description: Use when you need to measure how many biosynthetic gene cluster (BGC)
  entries carry compound structural data (SMILES) and how these compounds are cross-linked
  to external structure databases.
when_to_use_negative:
- You need predicted or computed chemical structures rather than the curated SMILES
  annotations stored in MIBiG entries.
- Your goal is annotating protein domain substrate selectivities (e.g. NRPS A-domains)
  instead of compound structure coverage.
edam_operation: http://edamontology.org/operation_0224
edam_topics:
- http://edamontology.org/topic_0154
tools:
- name: PIKAChU
  role: Validates that extracted SMILES strings are syntactically parseable before
    inclusion in the coverage tally
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkac1049
    title: 'MIBiG 3.0: a community-driven effort to annotate experimentally validated
      biosynthetic gene clusters'
  source_content_hash: sha256:bd3cee305bf1e2ae2b03b7c92d16b1a60e0dc5a72d6eb7df99190a6080e5a443
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PIKAChU
  edam_operation: http://edamontology.org/operation_0224
  edam_topics:
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# compound-structure-coverage-tallying

## Summary

Quantifies chemical structure annotation coverage in MIBiG 3.0 by parsing JSON entry files for SMILES identifiers and tallying cross-links to NP Atlas, PubChem, ChemSpider, LOTUS, and ChEMBL.

## When to use

Use when you need to measure how many biosynthetic gene cluster (BGC) entries carry compound structural data (SMILES) and how these compounds are cross-linked to external structure databases.

## When NOT to use

- You need predicted or computed chemical structures rather than the curated SMILES annotations stored in MIBiG entries.
- Your goal is annotating protein domain substrate selectivities (e.g. NRPS A-domains) instead of compound structure coverage.

## Inputs

- MIBiG 3.0 JSON entry files from mibig-secmet/mibig-json (data directory)
- Expected reference figures: 1188 compounds with SMILES; 1347 (v2.0) to 1860 (v3.0) BGCs with structure data

## Outputs

- structure_annotation_coverage.csv with per-database cross-link counts (NP Atlas, PubChem, ChemSpider, LOTUS, ChEMBL)
- Counted totals of distinct compounds carrying SMILES and BGC entries containing structural data

## How to apply

Download the MIBiG 3.0 JSON-formatted entries from the mibig-secmet/mibig-json GitHub repository, where curated annotations live in the `data` directory and entry status is tracked via the `cluster.status` field. Parse every entry JSON file to detect compound/structure records and extract SMILES strings; optionally validate SMILES syntax with PIKAChU before inclusion. Count BGC entries containing structural data, cross-checking against reported figures (1347 BGCs in MIBiG 2.0 rising to 1860 in MIBiG 3.0, with 1188 compounds carrying SMILES). Then tally the presence of cross-links per compound to the five structure databases — NP Atlas, PubChem, ChemSpider, LOTUS, ChEMBL — aggregating counts per database. Write per-database coverage counts and cross-link tallies to structure_annotation_coverage.csv for reuse.

## Related tools

- **PIKAChU** (Validates that extracted SMILES strings are syntactically parseable before inclusion in the coverage tally)

## Examples

```
git clone https://github.com/mibig-secmet/mibig-json; python tally_structure_coverage.py --json-dir mibig-json/data --out structure_annotation_coverage.csv
```

## Evaluation signals

- Distinct compounds carrying SMILES should equal 1188, matching the reported MIBiG 3.0 figure.
- BGC entries with structural data should count 1860 in MIBiG 3.0 versus 1347 in MIBiG 2.0.
- Cross-link tally must cover exactly the five structure databases: NP Atlas, PubChem, ChemSpider, LOTUS, ChEMBL.
- All counted SMILES strings should pass PIKAChU syntactic parse validation.
- Output CSV must contain one coverage count per database plus the aggregate cross-link totals.

## Limitations

- Counts are only valid for the curated experimentally validated BGC entries in MIBiG; they exclude predicted clusters from tools like antiSMASH.
- The mibig-json README notes the `genbanks` directory only contains sequence files not available from NCBI's GenBank/RefSeq databases, so entry completeness depends on external sequence sources.

## Evidence

- [intro] cross-link them to five databases of chemical structures: NP Atlas (9), PubChem, ChemSpider (17), LOTUS (18), and ChEMBL (19): "cross-link them to five databases of chemical structures: NP Atlas (9), PubChem, ChemSpider (17), LOTUS (18), and ChEMBL (19)"
- [intro] We focused on three features: the characterisation and cross-linking of 1188 chemical structures: "We focused on three features: the characterisation and cross-linking of 1188 chemical structures"
- [intro] Correctness of SMILES syntax was validated with PIKAChU (20): "Correctness of SMILES syntax was validated with PIKAChU (20)"
- [readme] The current datasets for [MIBiG] live in the `data` directory, entry status is now tracked via the `cluster.status` field.: "The current datasets for [MIBiG] live in the `data` directory, entry status is
now tracked via the `cluster.status` field."
