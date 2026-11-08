---
name: chemical-database-curation
description: Use when curating or auditing a natural product database where each BGC
  record has an associated product structure (e.g. MIBiG/PubChem links), and you need
  to detect entries whose linked SMILES/structure mismatches the biosynthetically
  predicted product.
when_to_use_negative:
- Records where the product is a scaffold rather than a mature natural product — RAIChU
  cannot generate mature natural products from raw antiSMASH output, so predicted
  and real-world structures may not have high fingerprinting similarity
- Database audits requiring fully automated structure validation of all BGCs — fingerprinting-based
  automatic validation (e.g. Morgan fingerprinting) was considered but not implemented;
  validation relied on manual curation of select examples
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3314
tools:
- name: RAIChU
  role: Redraws the biosynthetic product of each BGC entry to enable comparison against
    the database's linked reference structure
  repo: https://github.com/BTheDragonMaster/RAIChU
- name: PIKAChU
  role: Enforces chemical correctness of reaction intermediates during RAIChU's redraws
- name: antiSMASH
  role: Provides BGC detection and annotation predictions of real-world MIBiG clusters
    used as RAIChU input
- name: MIBiG
  role: Source database of experimentally verified BGCs with associated natural products
    being curated
- name: PubChem
  role: Source of reference SMILES strings when not available directly from MIBiG
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s13321-024-00898-x
    title: 'RAIChU: automating the visualisation of natural product biosynthesis'
  source_content_hash: sha256:d589431285e2f7db03d5f768ea74441b474b8cc58e209efd623b14faab46fbd9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BTheDragonMaster/RAIChU
  tools:
  - MIBiG
  - PIKAChU
  - PubChem
  - Python
  - RAIChU
  - antiSMASH
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# chemical-database-curation

## Summary

Batch-validate biosynthetic gene cluster entries in a chemical reference database (MIBiG) by re-drawing each annotated product with RAIChU and comparing the drawn structure to the linked reference (PubChem SMILES), surfacing wrong-structure links such as the colistin A entry.

## When to use

Use when curating or auditing a natural product database where each BGC record has an associated product structure (e.g. MIBiG/PubChem links), and you need to detect entries whose linked SMILES/structure mismatches the biosynthetically predicted product.

## When NOT to use

- Records where the product is a scaffold rather than a mature natural product — RAIChU cannot generate mature natural products from raw antiSMASH output, so predicted and real-world structures may not have high fingerprinting similarity
- Database audits requiring fully automated structure validation of all BGCs — fingerprinting-based automatic validation (e.g. Morgan fingerprinting) was considered but not implemented; validation relied on manual curation of select examples

## Inputs

- MIBiG database entries (experimentally verified BGCs with DNA sequence and associated natural product)
- Linked PubChem (or NP Atlas) SMILES strings for each natural product
- antiSMASH predictions of real-world BGCs from MIBiG

## Outputs

- Structured JSON log with one entry per processed database record, flagging processing errors and reference-structure mismatches (e.g. colistin A flagged as pointing to the wrong PubChem entry)
- Per-record RAIChU-drawn product structures ('spaghetti diagrams') for side-by-side comparison with reference structures

## How to apply

Create a conda environment with Python 3.9.12 and install RAIChU via 'pip install raichu'. Download the database entries — experimentally verified BGCs from MIBiG with their associated natural products and linked PubChem SMILES strings (retrieved from MIBiG first, then PubChem or NP Atlas in that order of preference). Batch-process every entry through RAIChU, which uses PIKAChU to enforce chemical correctness of reaction intermediates, catching unexpected processing errors per record. For each entry, compare RAIChU's drawn product against the reference structure; flag mismatches such as the colistin A case, where the cluster likely produces an analogue with a D-2,4-diaminobutyric acid residue at position 3 instead of the L-form. Record all processing errors and structure mismatches per database record in a structured JSON log.

## Related tools

- **RAIChU** (Redraws the biosynthetic product of each BGC entry to enable comparison against the database's linked reference structure) — https://github.com/BTheDragonMaster/RAIChU
- **PIKAChU** (Enforces chemical correctness of reaction intermediates during RAIChU's redraws)
- **antiSMASH** (Provides BGC detection and annotation predictions of real-world MIBiG clusters used as RAIChU input)
- **MIBiG** (Source database of experimentally verified BGCs with associated natural products being curated)
- **PubChem** (Source of reference SMILES strings when not available directly from MIBiG)

## Examples

```
conda create -n raichu python==3.9.12; conda activate raichu; pip install raichu
```

## Evaluation signals

- The JSON log exists and contains an entry for every processed MIBiG record
- The colistin A record is explicitly flagged as pointing to the wrong PubChem entry, consistent with the analogue bearing a D-2,4-diaminobutyric acid residue at position 3
- Drawing correctness across processed entries matches RAIChU's reported 100% cluster drawing correctness benchmark
- All unexpected processing errors are caught and logged per record rather than silently dropped
- Comparison of each drawn product against its reference structure confirms no further mismatches beyond the flagged ones

## Limitations

- Regio- and stereochemistry of tailoring enzymes cannot currently be predicted; users must specify reaction targets, so stereocentre discrepancies (like the colistin A D/L-2,4-diaminobutyric acid case) require manual interpretation
- Fingerprinting-based automatic validation of MIBiG BGCs was considered but not implemented; validation relied on manual curation of select examples (10 NRPS, 10 PKS, 10 NRPS/PKS hybrid clusters)
- RAIChU generates natural product scaffolds but not mature natural products from raw antiSMASH output, limiting automated similarity checks against real-world structures

## Evidence

- [results] colistin A discrepancy: "the colistin A BGC in the MIBiG database [36] points to the wrong PubChem entry"
- [discussion] full-database validation run: "For validation against characterised biosynthetic products, we ran RAIChU on the entire MIBiG database [36] to catch general processing errors."
- [discussion] reference SMILES sourcing priority: "We collected SMILES strings directly from the MIBiG database where possible, and otherwise retrieved them from PubChem [44] or NP Atlas [45] in that order of preference."
- [results] PIKAChU chemical correctness: "we implemented the most commonly occurring elongation and tailoring reactions using PIKAChU, a tool which enforces chemical correctness of reaction intermediates"
- [readme] installation via conda and pip: "First, create a new environment with python version 3.9.12, for instance with conda:"
