---
name: biosynthetic-gene-cluster-error-detection
description: Use when auditing MIBiG or similar BGC records whose linked natural-product
  structure (PubChem or NP Atlas SMILES) may mismatch the cluster's annotated biosynthesis,
  e.g. before reusing characterised BGC-structure pairs for benchmarking or training.
when_to_use_negative:
- No characterised reference structure exists for the cluster (i.e. the entry lacks
  an associated natural product SMILES), so there is nothing to compare RAIChU's drawn
  product against.
- 'Goal is predicting mature natural products from raw antiSMASH output: RAIChU can
  generate natural product scaffolds but not mature natural products, so fingerprinting-based
  structure validation was considered but not implemented by the authors.'
- The task requires predicting regio- and stereochemistry of tailoring reactions —
  these cannot be predicted and users must specify their reaction targets.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3174
tools:
- name: RAIChU
  role: Batch-draws the biosynthetic product of each BGC record so the drawn structure
    can be compared against the linked reference SMILES to flag database errors; pip-installable
    for Python pipelines
  repo: https://github.com/BTheDragonMaster/RAIChU
- name: PIKAChU
  role: Enforces chemical correctness of reaction intermediates during drawing, so
    flagged mismatches reflect reference-structure errors rather than invalid drawings
- name: Python
  role: Host environment (Python 3.9.12 via conda) in which RAIChU is installed via
    pip to script the batch run over all MIBiG entries
  repo: https://pypi.org/project/raichu
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s13321-024-00898-x
    title: 'RAIChU: automating the visualisation of natural product biosynthesis'
  source_content_hash: sha256:d100ca680f4ffcdac0412a51313939e741276c442ccddab4faeff5a82cc70d55
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BTheDragonMaster/RAIChU
  tools:
  - PIKAChU
  - Python
  - RAIChU
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# biosynthetic-gene-cluster-error-detection

## Summary

Batch-run RAIChU (backed by PIKAChU's chemically-correct intermediate drawing) over every entry of a BGC database such as MIBiG to catch processing errors and flag records whose linked structure SMILES (PubChem/NP Atlas) disagrees with the cluster's biosynthesis, as demonstrated by the colistin A wrong-PubChem-entry discrepancy.

## When to use

Use when auditing MIBiG or similar BGC records whose linked natural-product structure (PubChem or NP Atlas SMILES) may mismatch the cluster's annotated biosynthesis, e.g. before reusing characterised BGC-structure pairs for benchmarking or training.

## When NOT to use

- No characterised reference structure exists for the cluster (i.e. the entry lacks an associated natural product SMILES), so there is nothing to compare RAIChU's drawn product against.
- Goal is predicting mature natural products from raw antiSMASH output: RAIChU can generate natural product scaffolds but not mature natural products, so fingerprinting-based structure validation was considered but not implemented by the authors.
- The task requires predicting regio- and stereochemistry of tailoring reactions — these cannot be predicted and users must specify their reaction targets.

## Inputs

- MIBiG database entries (experimentally verified BGCs with known DNA sequence and associated natural product)
- Reference structure SMILES per record, sourced from MIBiG directly, else PubChem, else NP Atlas (in that order of preference)
- RAIChU cluster definitions built from the BGC annotation (or antiSMASH results uploaded to RAIChU)
- Colistin A BGC record with its linked PubChem entry (known-error validation case)

## Outputs

- Structured JSON log with one entry per processed MIBiG record, capturing processing errors and structure mismatches
- RAIChU/PIKAChU depictions (spaghetti diagrams) of each BGC's drawn product for side-by-side comparison with the reference structure
- Flagged discrepancy reports, e.g. colistin A flagged as pointing to the wrong PubChem entry (d- vs l-2,4-diaminobutyric acid at position 3)

## How to apply

Create a conda environment with Python 3.9.12 and install RAIChU via 'pip install raichu'. For each BGC record (e.g. every MIBiG entry), collect the reference SMILES in the article's preference order — directly from MIBiG, otherwise PubChem, otherwise NP Atlas — and build the RAIChU cluster definition, letting PIKAChU enforce chemical correctness of reaction intermediates. Batch-process all entries, catching unexpected processing errors per record, and compare RAIChU's drawn product (spaghetti diagram) against the reference structure for each entry. Record processing errors and structure mismatches in a structured JSON log with one entry per record; treat a drawing mismatch as a candidate database error rather than a RAIChU failure, since RAIChU's benchmark shows 100% cluster drawing correctness (97.66% readability). For colistin A, the expected flag is that MIBiG points to the wrong PubChem entry: the cluster likely produces an analogue with a d-2,4-diaminobutyric acid residue at position 3 instead of l-2,4-diaminobutyric acid, consistent with the original authors' postulate. Cross-check each flagged record against the original publication before declaring a database error.

## Related tools

- **RAIChU** (Batch-draws the biosynthetic product of each BGC record so the drawn structure can be compared against the linked reference SMILES to flag database errors; pip-installable for Python pipelines) — https://github.com/BTheDragonMaster/RAIChU
- **PIKAChU** (Enforces chemical correctness of reaction intermediates during drawing, so flagged mismatches reflect reference-structure errors rather than invalid drawings)
- **Python** (Host environment (Python 3.9.12 via conda) in which RAIChU is installed via pip to script the batch run over all MIBiG entries) — https://pypi.org/project/raichu

## Examples

```
conda create -n raichu python==3.9.12 && conda activate raichu && pip install raichu
```

## Evaluation signals

- The JSON log exists and contains an entry for every processed MIBiG record, with unexpected processing errors caught and logged rather than crashing the batch run.
- The colistin A record is explicitly flagged as pointing to the wrong PubChem entry, with the analogue described (d-2,4-diaminobutyric acid at position 3 instead of l-).
- Cluster drawing correctness across entries matches RAIChU's reported 100% benchmark, with drawing readability near the reported 97.66%.
- Reference SMILES provenance follows the stated preference order: MIBiG directly, otherwise PubChem, otherwise NP Atlas.
- Flagged mismatches are attributable to the reference structure rather than the drawing, because PIKAChU enforces chemical correctness of reaction intermediates.

## Limitations

- Regio- and stereochemistry of most tailoring enzymes cannot be predicted; users must specify reaction targets, so discrepancies involving unspecified tailoring sites may reflect user-input gaps rather than database errors.
- Double bond stereochemistry following dehydration reactions in PKS systems is a source of discrepancy (observed in the CMC-thuggacins).
- Cationic cascade reactions in terpenes necessitate programmatic shortcuts, yielding simplified intermediate representations (the summarized pathway view shows only initial and final intermediates).
- RAIChU generates natural product scaffolds but not mature natural products from raw antiSMASH output, so automatic Morgan-fingerprint validation of MIBiG entries was considered but not implemented.

## Evidence

- [results] full-MIBiG error-catching run: "First, we ran RAIChU on the entire database to ensure no unexpected errors occurred"
- [discussion] MIBiG validation rationale: "For validation against characterised biosynthetic products, we ran RAIChU on the entire MIBiG database [36] to catch general processing errors."
- [results] colistin A discrepancy: "the colistin A BGC in the MIBiG database [36] points to the wrong PubChem entry"
- [discussion] SMILES sourcing order: "We collected SMILES strings directly from the MIBiG database where possible, and otherwise retrieved them from PubChem [44] or NP Atlas [45] in that order of preference."
- [readme] environment setup: "First, create a new environment with python version 3.9.12, for instance with conda: conda create -n raichu python==3.9.12"
- [abstract] drawing correctness benchmark: "RAIChU's cluster drawing correctness (100%) and drawing readability (97.66%) were validated on 5000 randomly generated PKS/NRPS systems, and on the MIBiG database"
