---
name: mibig-cluster-data-retrieval
description: Use when a workflow needs real-world PKS/NRPS or hybrid BGC cluster definitions
  with known products — e.g. drawing MIBiG entries with RAIChU, validating predicted
  structures, or collecting SMILES for natural product visualisation.
when_to_use_negative:
- The task is to draw randomly generated or hypothetical PKS/NRPS systems rather than
  experimentally characterised MIBiG clusters
- A mature natural product must be generated directly from raw antiSMASH output without
  user specification — RAIChU generates scaffolds, not mature products, and predicted
  structures may not fingerprint-match the real product
- Tailoring regio- and stereochemistry must be predicted automatically; users must
  specify their reaction targets, so unannotated MIBiG entries alone are insufficient
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3361
tools:
- name: RAIChU
  role: Consumes the retrieved MIBiG cluster definition and draws it as an SVG spaghetti
    diagram (domains as circles, genes as rectangles, vertical backbone); installable
    as a pip package
  repo: https://github.com/BTheDragonMaster/RAIChU
- name: antiSMASH
  role: Source of BGC predictions for real-world MIBiG clusters that can be uploaded
    and edited in RAIChU
- name: MIBiG
  role: Primary database of experimentally verified BGCs supplying cluster definitions
    and SMILES strings
- name: PubChem
  role: Fallback source for SMILES strings when MIBiG is incorrect or missing them
    (second preference)
- name: NP Atlas
  role: Final fallback source for SMILES strings (third preference after MIBiG and
    PubChem)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s13321-024-00898-x
    title: 'RAIChU: automating the visualisation of natural product biosynthesis'
  source_content_hash: sha256:617f0488284701b307f8c74a816966321c5566b195b2f399d4f4f42111cc6971
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BTheDragonMaster/RAIChU
  tools:
  - MIBiG
  - NP Atlas
  - PubChem
  - Python
  - RAIChU
  - antiSMASH
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3361
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# mibig-cluster-data-retrieval

## Summary

Retrieve experimentally verified biosynthetic gene cluster definitions (DNA sequence and associated natural product) from the MIBiG database, with SMILES fallbacks to PubChem then NP Atlas, as grounded inputs for RAIChU spaghetti-diagram generation.

## When to use

Use when a workflow needs real-world PKS/NRPS or hybrid BGC cluster definitions with known products — e.g. drawing MIBiG entries with RAIChU, validating predicted structures, or collecting SMILES for natural product visualisation.

## When NOT to use

- The task is to draw randomly generated or hypothetical PKS/NRPS systems rather than experimentally characterised MIBiG clusters
- A mature natural product must be generated directly from raw antiSMASH output without user specification — RAIChU generates scaffolds, not mature products, and predicted structures may not fingerprint-match the real product
- Tailoring regio- and stereochemistry must be predicted automatically; users must specify their reaction targets, so unannotated MIBiG entries alone are insufficient

## Inputs

- MIBiG cluster entry (experimentally verified BGC with DNA sequence and known natural product)
- MIBiG-referenced SMILES string of the associated natural product
- PubChem or NP Atlas SMILES strings as fallback sources

## Outputs

- Cluster definition ready for RAIChU spaghetti-diagram drawing
- Verified SMILES string for the BGC's natural product
- Rendered spaghetti diagram (mibig_spaghetti_diagram.svg) for each retrieved cluster

## How to apply

Set up a Python 3.9.12 environment and install the RAIChU package with pip, then pull a modular PKS/NRPS BGC cluster entry from the MIBiG database to serve as the cluster definition for drawing. Prefer SMILES strings collected directly from MIBiG; where MIBiG points to an incorrect or missing structure (e.g. the colistin A BGC), fall back to PubChem, then NP Atlas, in that order of preference. Feed the retrieved cluster definition into the RaichuDrawer pipeline — alternate central-chain bond angles of 60/-60 degrees, 90 degrees for cyclic bonds, 0/180 degrees for side chains, plus clash-resolution finetuning — to render a spaghetti diagram to SVG. This retrieval step mirrors the paper's validation design, in which RAIChU was run on antiSMASH predictions of real-world MIBiG BGCs and on the entire MIBiG database to catch processing errors, achieving 100% cluster drawing correctness and 97.66% drawing readability.

## Related tools

- **RAIChU** (Consumes the retrieved MIBiG cluster definition and draws it as an SVG spaghetti diagram (domains as circles, genes as rectangles, vertical backbone); installable as a pip package) — https://github.com/BTheDragonMaster/RAIChU
- **antiSMASH** (Source of BGC predictions for real-world MIBiG clusters that can be uploaded and edited in RAIChU)
- **MIBiG** (Primary database of experimentally verified BGCs supplying cluster definitions and SMILES strings)
- **PubChem** (Fallback source for SMILES strings when MIBiG is incorrect or missing them (second preference))
- **NP Atlas** (Final fallback source for SMILES strings (third preference after MIBiG and PubChem))

## Examples

```
conda create -n raichu python==3.9.12 && conda activate raichu && pip install raichu
```

## Evaluation signals

- Depicted structure of the MIBiG cluster matches the characterised natural product recorded in the database
- Rendered SVG passes the paper's three readability criteria: correctness of depicted structures, correct bond angles, and absence of overlap
- Overall drawing correctness and readability comparable to the reported benchmarks (100% correctness, 97.66% readability on MIBiG and 5000 random systems)
- SMILES provenance follows the stated preference order: MIBiG first, then PubChem, then NP Atlas
- Ran pipeline completes without unexpected errors, as checked by running RAIChU on the entire MIBiG database

## Limitations

- The colistin A BGC in MIBiG points to the wrong PubChem entry, so SMILES strings from MIBiG references require verification
- Double bond stereochemistry following dehydration reactions in PKS systems is a source of discrepancy, as observed in the CMC-thuggacins
- Validation relied on manual curation of only a few select MIBiG examples (10 NRPS, 10 PKS, 10 NRPS/PKS hybrid clusters) rather than automated validation of all MIBiG BGCs; Morgan fingerprinting validation was considered but not implemented
- Predicted product selection for figures was manual, choosing the product that best matched the characterised natural product's cyclisation site

## Evidence

- [abstract] RAIChU's cluster drawing correctness (100%) and drawing readability (97.66%) were validated on 5000 randomly generated PKS/NRPS systems, and on the MIBiG database: "RAIChU's cluster drawing correctness (100%) and drawing readability (97.66%) were validated on 5000 randomly generated PKS/NRPS systems, and on the MIBiG database"
- [results] MIBiG is a database of experimentally verified BGCs with known products: "the MIBiG database: a database of experimentally verified biosynthetic gene clusters for which both DNA sequence and the associated natural product are known"
- [discussion] SMILES collection preference order from discussion: "We collected SMILES strings directly from the MIBiG database where possible, and otherwise retrieved them from PubChem [44] or NP Atlas [45] in that order of preference."
- [results] MIBiG clusters were tested via antiSMASH predictions: "we additionally tested RAIChU on antiSMASH predictions of real-world BGCs from the MIBiG database"
- [readme] Installation instructions from README: "Then, install raichu with pip: pip install raichu"
