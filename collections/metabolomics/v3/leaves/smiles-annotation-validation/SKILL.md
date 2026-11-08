---
name: smiles-annotation-validation
description: Use when a database release or curated entry set adds chemical structure
  records (e.g. SMILES strings) and you need to validate their syntax and quantify
  coverage and external cross-links.
when_to_use_negative:
- The input is raw sequence data (GenBank/RefSeq files) rather than curated compound/structure
  records with SMILES annotations.
- The goal is BGC detection from DNA rather than auditing chemical structure annotation
  of already-validated BGC entries.
- Cross-link targets other than the five structure databases (NP Atlas, PubChem, ChemSpider,
  LOTUS, ChEMBL) are needed and no cross-link fields exist in the entries.
edam_operation: http://edamontology.org/operation_3436
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_0091
tools:
- name: PIKAChU
  role: Validates that counted SMILES strings are syntactically parseable before inclusion
    in the coverage tally.
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkac1049
    title: 'MIBiG 3.0: a community-driven effort to annotate experimentally validated
      biosynthetic gene clusters'
  source_content_hash: sha256:a719d8ddc90514811d0ccb47504e1641e06d694582d8a344810be76c309e31c4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PIKAChU
  edam_operation: http://edamontology.org/operation_3436
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# SMILES annotation validation

## Summary

Extracts and validates SMILES structural identifiers for compounds annotated in biosynthetic gene cluster entries, and tallies per-compound cross-links to chemical structure databases, as done to quantify structure annotation coverage in MIBiG 3.0.

## When to use

Use when a database release or curated entry set adds chemical structure records (e.g. SMILES strings) and you need to validate their syntax and quantify coverage and external cross-links.

## When NOT to use

- The input is raw sequence data (GenBank/RefSeq files) rather than curated compound/structure records with SMILES annotations.
- The goal is BGC detection from DNA rather than auditing chemical structure annotation of already-validated BGC entries.
- Cross-link targets other than the five structure databases (NP Atlas, PubChem, ChemSpider, LOTUS, ChEMBL) are needed and no cross-link fields exist in the entries.

## Inputs

- JSON-formatted BGC entry files from the mibig-secmet/mibig-json repository (MIBiG 3.0 data directory)
- compound/structure records embedded in each entry, including SMILES strings and database cross-links

## Outputs

- structure_annotation_coverage.csv with per-database cross-link counts (NP Atlas, PubChem, ChemSpider, LOTUS, ChEMBL) and coverage totals
- counts of BGCs with structural data and distinct compounds with SMILES (e.g. 1860 BGCs; 1188 compounds)

## How to apply

Download the JSON-formatted entries (e.g. from the mibig-secmet/mibig-json repository) and parse each entry file to detect compound/structure records, extracting the SMILES strings for annotated chemical structures. Validate that each SMILES is syntactically parseable using the PIKAChU cheminformatics toolkit before counting it, since malformed SMILES would inflate coverage. Count the number of BGC entries containing structural data and the number of distinct compounds carrying SMILES, cross-checking against published figures (e.g. 1347 BGCs with structural data in MIBiG 2.0 rising to 1860 in MIBiG 3.0; 1188 compounds with SMILES). For each entry, tally presence of cross-links to the five structure databases (NP Atlas, PubChem, ChemSpider, LOTUS, ChEMBL) and aggregate counts per database. Write per-database coverage counts and cross-link tallies to a tabular output such as structure_annotation_coverage.csv.

## Related tools

- **PIKAChU** (Validates that counted SMILES strings are syntactically parseable before inclusion in the coverage tally.)

## Examples

```
python count_structure_coverage.py --mibig-json mibig-secmet/mibig-json/data --validate-smiles pikachu --output structure_annotation_coverage.csv
```

## Evaluation signals

- Total compounds with SMILES matches the published count (1188 in MIBiG 3.0).
- Number of BGCs with structural data increases from 1347 (MIBiG 2.0) to 1860 (MIBiG 3.0).
- Every SMILES counted parses successfully under PIKAChU (no syntax errors).
- Per-database cross-link tallies sum to entries that carry links to exactly the five databases: NP Atlas, PubChem, ChemSpider, LOTUS, ChEMBL.
- Output CSV contains one row per structure database with a non-negative integer count.

## Limitations

- Coverage counts depend on the entry status field in the JSON data; duplicate and low-quality entries (69 removed in MIBiG 3.0) can skew counts if an older snapshot is used.
- SMILES syntax validity (checked with PIKAChU) does not guarantee chemical correctness of the annotated structure.
- A handful of genbank sequence files in the repository are not available from NCBI's GenBank/RefSeq databases, so entry provenance cannot always be cross-checked externally.

## Evidence

- [intro] SMILES syntax validated with PIKAChU: "Correctness of SMILES syntax was validated with PIKAChU (20)"
- [intro] Five cross-linked structure databases: "cross-link them to five databases of chemical structures: NP Atlas (9), PubChem, ChemSpider (17), LOTUS (18), and ChEMBL (19)"
- [intro] 1188 chemical structures characterised and cross-linked: "We focused on three features: the characterisation and cross-linking of 1188 chemical structures"
- [readme] Repository layout of JSON data: "The current datasets for [MIBiG] live in the `data` directory, entry status is now tracked via the `cluster.status` field."
