---
name: chemical-database-crosslink-counting
description: Use when a database release claims compound structure coverage (e.g.
  '1188 structures with SMILES', '1347→1860 BGCs with structure data') and you need
  to reproduce those counts and per-database cross-link tallies.
when_to_use_negative:
- The goal is annotating new BGCs or discovering structures, not auditing existing
  coverage counts — use antiSMASH/PIKAChU annotation workflows instead
- The input is not entry-level structured records (e.g. raw GenBank sequences without
  compound fields), so no SMILES or cross-link fields exist to tally
- Only one chemical database is of interest and no global coverage or per-database
  comparison is needed
edam_operation: http://edamontology.org/operation_0224
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_0091
tools:
- name: PIKAChU
  role: Validates that counted SMILES strings are syntactically parseable before inclusion
    in the coverage tally
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkac1049
    title: 'MIBiG 3.0: a community-driven effort to annotate experimentally validated
      biosynthetic gene clusters'
  source_content_hash: sha256:a1a405863c05d1801710118e8dc077763c2430935abc2edaeea30d0e8109ec0d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PIKAChU
  edam_operation: http://edamontology.org/operation_0224
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# chemical-database-crosslink-counting

## Summary

Quantifies chemical structure annotation coverage in a BGC database by parsing entry-level records for SMILES strings and tallying cross-links to chemical structure databases, as done for MIBiG 3.0 across NP Atlas, PubChem, ChemSpider, LOTUS, and ChEMBL.

## When to use

Use when a database release claims compound structure coverage (e.g. '1188 structures with SMILES', '1347→1860 BGCs with structure data') and you need to reproduce those counts and per-database cross-link tallies.

## When NOT to use

- The goal is annotating new BGCs or discovering structures, not auditing existing coverage counts — use antiSMASH/PIKAChU annotation workflows instead
- The input is not entry-level structured records (e.g. raw GenBank sequences without compound fields), so no SMILES or cross-link fields exist to tally
- Only one chemical database is of interest and no global coverage or per-database comparison is needed

## Inputs

- MIBiG 3.0 JSON-formatted entry files from the mibig-secmet/mibig-json GitHub repository (data directory)
- Compound/structure records containing SMILES strings and database cross-link identifiers

## Outputs

- Count of BGC entries with structural data (expected 1860 for MIBiG 3.0)
- Count of distinct compounds carrying SMILES (expected 1188)
- Per-database cross-link tallies for NP Atlas, PubChem, ChemSpider, LOTUS, ChEMBL
- structure_annotation_coverage.csv

## How to apply

Download the JSON-formatted entries from the mibig-secmet/mibig-json repository, where the current MIBiG annotations live in the `data` directory with entry status tracked via the `cluster.status` field. Parse every entry JSON file to detect compound/structure records and extract SMILES strings for annotated chemical structures. Count BGC entries containing structural data and distinct compounds carrying SMILES, cross-checking against reported figures (1347 BGCs with structure data in MIBiG 2.0 rising to 1860 in MIBiG 3.0; 1188 compounds with SMILES). For each entry, tally presence of cross-links to the five structure databases — NP Atlas, PubChem, ChemSpider, LOTUS, and ChEMBL — aggregating counts per database. Optionally validate SMILES syntax with PIKAChU before inclusion in the tally, mirroring the authors' own correctness check. Write per-database coverage counts and cross-link tallies to structure_annotation_coverage.csv.

## Related tools

- **PIKAChU** (Validates that counted SMILES strings are syntactically parseable before inclusion in the coverage tally)

## Examples

```
python count_structure_links.py --json-dir mibig-json/data --databases NP_Atlas,PubChem,ChemSpider,LOTUS,ChEMBL --validate-smiles PIKAChU --out structure_annotation_coverage.csv
```

## Evaluation signals

- Total compounds with SMILES equals 1188, matching the reported 'characterisation and cross-linking of 1188 chemical structures'
- Count of BGCs with structural data transitions from 1347 (MIBiG 2.0) to 1860 (MIBiG 3.0)
- Cross-link tallies cover exactly five databases: NP Atlas, PubChem, ChemSpider, LOTUS, and ChEMBL
- Every SMILES counted is syntactically parseable by PIKAChU
- structure_annotation_coverage.csv contains one row per database with non-negative integer counts and no unclassified cross-links

## Limitations

- Counts depend on entry status tracked via cluster.status — duplicate or low-quality entries (69 removed in MIBiG 3.0) can inflate coverage if not filtered
- SMILES coverage is limited to compounds the community annotated; structural data presence does not guarantee completeness or correctness of the structure itself
- Cross-links only cover the five cited structure databases; links to other chemical resources will not be captured

## Evidence

- [intro] cross-link them to five databases of chemical structures: "cross-link them to five databases of chemical structures: NP Atlas (9), PubChem, ChemSpider (17), LOTUS (18), and ChEMBL (19)"
- [intro] SMILES syntax validation with PIKAChU: "Correctness of SMILES syntax was validated with PIKAChU (20)"
- [readme] JSON data location in repository: "The current datasets for [MIBiG] live in the `data` directory, entry status is now tracked via the `cluster.status` field."
- [intro] focus on 1188 structures: "We focused on three features: the characterisation and cross-linking of 1188 chemical structures"
- [intro] duplicate/low-quality entries removed: "we added 661 novel BGCs to the MIBiG database which were published since the last database update and removed 69 duplicate and low-quality entries"
