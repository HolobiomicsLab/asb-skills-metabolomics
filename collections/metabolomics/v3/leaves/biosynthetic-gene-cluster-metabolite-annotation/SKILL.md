---
name: biosynthetic-gene-cluster-metabolite-annotation
description: Use when curating or re-annotating biosynthetic gene cluster (BGC) entries
  that carry compound-structure or substrate SMILES strings, and you need to verify
  SMILES syntax correctness before release.
when_to_use_negative:
- The goal is detecting BGCs from DNA sequence (use antiSMASH, fungiSMASH, plantiSMASH,
  GECCO, DeepBGC, RiPPMiner, or PRISM 4 instead) — this skill only validates SMILES
  syntax of already-annotated structures
- You need to predict A-domain substrate specificity (use SANDPUMA or NRPSPredictor2);
  this skill does not predict substrates, only validates their SMILES syntax
- The input is a chemical-structure database export without SMILES fields (e.g. plain
  InChI or names) — there is nothing to parse
edam_operation: http://edamontology.org/operation_3436
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_0091
tools:
- name: PIKAChU
  role: Toolkit used to validate correctness of SMILES syntax for compound structures
    and protein-domain substrates in MIBiG 3.0 entries
- name: mibig-json
  role: Source repository of MIBiG curation data in JSON format from which SMILES
    strings are extracted
  repo: https://github.com/mibig-secmet
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkac1049
    title: 'MIBiG 3.0: a community-driven effort to annotate experimentally validated
      biosynthetic gene clusters'
  source_content_hash: sha256:692d2d514944c10aa7fff2d48b22c8b3af8e822c34d784a8b39c13a911f51256
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PIKAChU
  - mibig-json
  edam_operation: http://edamontology.org/operation_3436
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# biosynthetic-gene-cluster-metabolite-annotation

## Summary

Validates the syntactic correctness of SMILES strings annotating compound structures and protein-domain substrates in MIBiG 3.0 JSON entries by parsing them with the PIKAChU toolkit, so that chemical structure annotation in a BGC database is machine-readable and error-free.

## When to use

Use when curating or re-annotating biosynthetic gene cluster (BGC) entries that carry compound-structure or substrate SMILES strings, and you need to verify SMILES syntax correctness before release.

## When NOT to use

- The goal is detecting BGCs from DNA sequence (use antiSMASH, fungiSMASH, plantiSMASH, GECCO, DeepBGC, RiPPMiner, or PRISM 4 instead) — this skill only validates SMILES syntax of already-annotated structures
- You need to predict A-domain substrate specificity (use SANDPUMA or NRPSPredictor2); this skill does not predict substrates, only validates their SMILES syntax
- The input is a chemical-structure database export without SMILES fields (e.g. plain InChI or names) — there is nothing to parse

## Inputs

- MIBiG 3.0 JSON-formatted BGC entries from the mibig-secmet/mibig-json repository (data directory)
- SMILES strings extracted from chemical-structure fields and protein-domain substrate fields of each entry

## Outputs

- smiles_validation_report.csv with one validation status per entry and explicit flags for invalid/non-parseable SMILES

## How to apply

Retrieve the MIBiG 3.0 JSON-formatted entries from the mibig-secmet/mibig-json GitHub repository, where datasets live in the `data` directory with entry status tracked via the `cluster.status` field. Parse each JSON entry and extract all SMILES strings from both the chemical-structure fields and the protein-domain substrate fields. Feed each extracted SMILES string through the PIKAChU toolkit to check whether it parses successfully (i.e., is syntactically valid), recording per-string pass/fail status. Aggregate the per-entry outcomes into a single validation report (smiles_validation_report.csv), flagging every SMILES string that fails syntax validation or cannot be parsed; entries the paper corrected during curation should not remain as unresolved failures.

## Related tools

- **PIKAChU** (Toolkit used to validate correctness of SMILES syntax for compound structures and protein-domain substrates in MIBiG 3.0 entries)
- **mibig-json** (Source repository of MIBiG curation data in JSON format from which SMILES strings are extracted) — https://github.com/mibig-secmet

## Evaluation signals

- smiles_validation_report.csv exists and contains one validation status per JSON entry
- All flagged (invalid/non-parseable) SMILES are explicitly marked in the report
- Entries the paper corrected during curation do not appear as unresolved failures
- Every SMILES string in both chemical-structure and protein-domain substrate fields was passed through PIKAChU (no entries skipped)
- Successfully parsed SMILES are those PIKAChU accepts without syntax errors

## Limitations

- PIKAChU validation covers SMILES syntax correctness only — it does not verify that the structure is chemically correct or matches the experimentally characterized compound
- SMILES syntax validity does not guarantee correct stereochemistry or cross-links to structure databases (NP Atlas, PubChem, ChemSpider, LOTUS, ChEMBL)
- Only a handful of sequence files in the genbanks directory are not available from NCBI's GenBank/RefSeq databases; the JSON data must come from the mibig-json repository's data directory

## Evidence

- [intro] Correctness of SMILES syntax was validated with PIKAChU: "Correctness of SMILES syntax was validated with PIKAChU (20)"
- [readme] MIBiG curation data tracked in JSON format: "This repository tracks the current [MIBiG] annotations in JSON format.

The current datasets for [MIBiG] live in the `data` directory, entry status is
now tracked via the `cluster.status` field."
- [abstract] Focus on annotation of compound structures and protein domain selectivities: "Particular attention was paid to the annotation of compound structures and biological activities, as well as protein domain selectivities."
- [intro] Characterisation and cross-linking of 1188 chemical structures: "We focused on three features: the characterisation and cross-linking of 1188 chemical structures, the annotation of 1002 bioactivities of BGC products, and the validation and annotation of 2020"
