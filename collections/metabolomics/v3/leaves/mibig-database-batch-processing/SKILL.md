---
name: mibig-database-batch-processing
description: Use when you have a set of MIBiG BGC records with associated natural
  products and need to validate that a visualisation/prediction tool reproduces each
  annotated structure, or to systematically flag database records whose linked structure
  (e.
when_to_use_negative:
- The goal is a single bespoke pathway visualisation rather than systematic database-wide
  error checking — manual or interactive RAIChU use suffices
- The BGCs are not experimentally characterised or lack an associated natural product
  structure to compare against
- The tool being validated cannot process the compound class in question (e.g. RAIChU
  cannot generate mature natural products from raw antiSMASH output for all classes)
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: RAIChU
  role: Draws biosynthetic products of each MIBiG BGC for batch validation against
    reference structures
  repo: https://github.com/BTheDragonMaster/RAIChU
- name: PIKAChU
  role: Enforces chemical correctness of reaction intermediates during RAIChU runs
- name: antiSMASH
  role: Supplies BGC detection and annotation input for real-world clusters processed
    via RAIChU
- name: Python
  role: Scripting environment for batch-running RAIChU and writing the per-record
    JSON log
  repo: https://pypi.org/project/raichu
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s13321-024-00898-x
    title: 'RAIChU: automating the visualisation of natural product biosynthesis'
  source_content_hash: sha256:2c187af61f762b0f4e22603d083baaa58df2c1b1662376092e821946c023ad6b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BTheDragonMaster/RAIChU
  tools:
  - PIKAChU
  - Python
  - RAIChU
  - antiSMASH
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# mibig-database-batch-processing

## Summary

Batch-process all experimentally verified MIBiG biosynthetic gene clusters through RAIChU (with PIKAChU-enforced chemical correctness) to catch processing errors and compare each drawn product against the linked reference structure, surfacing mismatches such as the colistin A wrong-PubChem-entry discrepancy.

## When to use

Use when you have a set of MIBiG BGC records with associated natural products and need to validate that a visualisation/prediction tool reproduces each annotated structure, or to systematically flag database records whose linked structure (e.g. PubChem entry) disagrees with the cluster's biosynthetic logic.

## When NOT to use

- The goal is a single bespoke pathway visualisation rather than systematic database-wide error checking — manual or interactive RAIChU use suffices
- The BGCs are not experimentally characterised or lack an associated natural product structure to compare against
- The tool being validated cannot process the compound class in question (e.g. RAIChU cannot generate mature natural products from raw antiSMASH output for all classes)

## Inputs

- MIBiG database records (experimentally verified BGCs with DNA sequence and associated natural product)
- Reference structure SMILES strings from MIBiG, PubChem, or NP Atlas
- antiSMASH predictions of real-world BGCs (optional input to RAIChU)

## Outputs

- Spaghetti diagrams (drawn biosynthetic products) per MIBiG entry
- Structured JSON error/mismatch log with one entry per processed MIBiG record, flagging discrepancies such as colistin A

## How to apply

Create a conda environment with Python 3.9.12 and install RAIChU via pip. Collect SMILES strings for each MIBiG entry directly from MIBiG where possible, otherwise from PubChem, then NP Atlas, in that order of preference. Run RAIChU (which uses PIKAChU to enforce chemical correctness of reaction intermediates) on the entire database rather than manually curating select examples, catching unexpected processing errors. For each entry, compare RAIChU's drawn product against the reference structure, recording processing errors and structure mismatches in a structured per-record JSON log. Flag records like the colistin A BGC whose MIBiG entry points to the wrong PubChem entry — there the drawn cluster matches an analogue with a d-2,4-diaminobutyric acid residue at position 3, consistent with the original authors' postulate.

## Related tools

- **RAIChU** (Draws biosynthetic products of each MIBiG BGC for batch validation against reference structures) — https://github.com/BTheDragonMaster/RAIChU
- **PIKAChU** (Enforces chemical correctness of reaction intermediates during RAIChU runs)
- **antiSMASH** (Supplies BGC detection and annotation input for real-world clusters processed via RAIChU)
- **Python** (Scripting environment for batch-running RAIChU and writing the per-record JSON log) — https://pypi.org/project/raichu

## Examples

```
conda create -n raichu python==3.9.12; conda activate raichu; pip install raichu
```

## Evaluation signals

- The JSON log exists and contains an entry for every processed MIBiG record
- The log explicitly flags the colistin A record as pointing to the wrong PubChem entry (analogue with d-2,4-diaminobutyric acid at position 3)
- Cluster drawing correctness across entries reaches RAIChU's benchmark of 100% (readability ~97.66%)
- No unexpected processing errors occur for any MIBiG entry beyond those explicitly logged
- SMILES sources follow the documented preference order: MIBiG directly, then PubChem, then NP Atlas

## Limitations

- Manually assessing every diagram is impractical at database scale, so only a few select MIBiG examples (e.g. 10 NRPS, 10 PKS, 10 hybrid) are typically manually curated; automated fingerprinting-based validation was considered but not implemented
- RAIChU can generate natural product scaffolds but not mature natural products from raw antiSMASH output, so predicted and real-world structures may not have high fingerprinting similarity
- Cationic cascade reactions in terpenes require programmatic shortcuts, yielding simplified representations of reaction intermediates
- Double bond stereochemistry following dehydration reactions in PKS systems (e.g. CMC-thuggacins) is a source of discrepancy

## Evidence

- [results] Batch RAIChU run over entire MIBiG: "First, we ran RAIChU on the entire database to ensure no unexpected errors occurred"
- [results] Colistin A PubChem discrepancy: "the colistin A BGC in the MIBiG database [36] points to the wrong PubChem entry"
- [discussion] SMILES source preference order: "We collected SMILES strings directly from the MIBiG database where possible, and otherwise retrieved them from PubChem [44] or NP Atlas [45]"
- [readme] Environment setup: "First, create a new environment with python version 3.9.12, for instance with conda"
- [abstract] Validation benchmark: "RAIChU's cluster drawing correctness (100%) and drawing readability (97.66%) were validated on 5000 randomly generated PKS/NRPS systems, and on the MIBiG database"
