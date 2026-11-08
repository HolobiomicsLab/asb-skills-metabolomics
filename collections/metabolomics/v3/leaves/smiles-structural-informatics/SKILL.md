---
name: smiles-structural-informatics
description: Use when validating predicted PKS/NRPS product scaffolds against experimentally
  characterised natural products in MIBiG, and you need to systematically classify
  structural discrepancies between prediction and reality.
when_to_use_negative:
- 'Retroactively applying to raw antiSMASH output for mature natural products: RAIChU
  generates natural product scaffolds, not mature products, so fingerprinting similarity
  between predicted and real-world structures may be low'
- When you need automatic validation of all MIBiG BGCs via fingerprinting — this was
  considered but not implemented; validation relied on manual curation of a few select
  examples
- When the goal is predicting regio- and stereochemistry of tailoring enzymes — users
  must specify reaction targets themselves
edam_operation: http://edamontology.org/operation_2929
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3314
tools:
- name: RAIChU
  role: Renders predicted end-product structures ('spaghetti diagrams') from antiSMASH
    annotations of PKS/NRPS BGCs for comparison against characterised SMILES
  repo: https://github.com/BTheDragonMaster/RAIChU
- name: antiSMASH
  role: Provides BGC detection and modular annotations of the curated MIBiG clusters
    that RAIChU consumes as input
- name: Python
  role: Hosts the RAIChU pip installation and RDKit-based structural comparison of
    predicted vs characterised SMILES
  repo: https://pypi.org/project/raichu
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s13321-024-00898-x
    title: 'RAIChU: automating the visualisation of natural product biosynthesis'
  source_content_hash: sha256:28006596dd4b972b0dfe151ca0fe25c72015d34752416634969e2525aaa58a4d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BTheDragonMaster/RAIChU
  tools:
  - Python
  - RAIChU
  - antiSMASH
  edam_operation: http://edamontology.org/operation_2929
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# smiles-structural-informatics

## Summary

Compare RAIChU-rendered predicted natural product structures against characterised SMILES strings (from MIBiG, PubChem, or NP Atlas) using RDKit structural comparison plus manual diagram inspection, classifying every mismatch into the paper's three discrepancy causes.

## When to use

Use when validating predicted PKS/NRPS product scaffolds against experimentally characterised natural products in MIBiG, and you need to systematically classify structural discrepancies between prediction and reality.

## When NOT to use

- Retroactively applying to raw antiSMASH output for mature natural products: RAIChU generates natural product scaffolds, not mature products, so fingerprinting similarity between predicted and real-world structures may be low
- When you need automatic validation of all MIBiG BGCs via fingerprinting — this was considered but not implemented; validation relied on manual curation of a few select examples
- When the goal is predicting regio- and stereochemistry of tailoring enzymes — users must specify reaction targets themselves

## Inputs

- antiSMASH annotations of 30 manually curated MIBiG BGCs (cis-AT PKS, NRPS, NRPS/PKS hybrid)
- Characterised SMILES strings from MIBiG, PubChem, or NP Atlas
- RAIChU-rendered predicted end-product structure diagrams

## Outputs

- mibig_discrepancy_table.csv with exactly 30 rows (one per curated cluster), each recording cluster ID, predicted/characterised structure pair, and a non-empty cause-category field

## How to apply

Install RAIChU in a Python 3.9.12 conda environment via pip, then run it on antiSMASH annotations of the 30 manually curated MIBiG BGCs spanning cis-AT PKS, NRPS, and NRPS/PKS hybrid clusters to render predicted end-product structures. For each cluster, select the predicted product whose cyclisation site best matches the characterised natural product. Collect characterised SMILES strings, preferring MIBiG first, then PubChem, then NP Atlas in that order. Compare each RAIChU-rendered structure to its characterised SMILES using RDKit structural comparison supplemented by manual inspection of the rendered diagrams, then classify each mismatch into one of three causes: unpredicted post-assembly tailoring reactions, unassigned double-bond stereochemistry after PKS dehydration, or missing substrates in RAIChU.

## Related tools

- **RAIChU** (Renders predicted end-product structures ('spaghetti diagrams') from antiSMASH annotations of PKS/NRPS BGCs for comparison against characterised SMILES) — https://github.com/BTheDragonMaster/RAIChU
- **antiSMASH** (Provides BGC detection and modular annotations of the curated MIBiG clusters that RAIChU consumes as input)
- **Python** (Hosts the RAIChU pip installation and RDKit-based structural comparison of predicted vs characterised SMILES) — https://pypi.org/project/raichu

## Examples

```
conda create -n raichu python==3.9.12; conda activate raichu; pip install raichu
```

## Evaluation signals

- mibig_discrepancy_table.csv contains exactly 30 rows, one per curated cluster
- Every row has a non-empty cause-category field; no cluster is left unclassified
- Each mismatch is assigned to one of the three cause categories (post-assembly tailoring, unassigned double-bond stereochemistry, missing substrates)
- Characterised SMILES provenance follows the stated preference order: MIBiG, then PubChem, then NP Atlas
- Selected RAIChU-predicted product's cyclisation site matches the characterised natural product's cyclisation site

## Limitations

- Regio- and stereochemistry of tailoring enzymes cannot currently be predicted; users must specify reaction targets
- Double bond stereochemistry following dehydration reactions in PKS systems is unassigned, a source of discrepancy (observed in the CMC-thuggacins)
- Some MIBiG entries point to wrong external records (e.g. the colistin A BGC pointed to the wrong PubChem entry), so SMILES provenance must be checked
- RAIChU can generate natural product scaffolds but not mature natural products from raw antiSMASH output, so predicted and real-world structures may not have high fingerprinting similarity

## Evidence

- [results] Discrepancies between RAIChU-predicted products and experimentally characterised products typically arise from one of three causes: unpredicted post-assembly tailoring reactions, unassigned double-bond stereochemistry after PKS dehydration, or missing substrates in RAIChU.: "double bond stereochemistry following dehydration reactions in PKS systems, as observed in the CMC-thuggacins"
- [discussion] Characterised SMILES were collected with a defined source preference order: MIBiG, then PubChem, then NP Atlas.: "We collected SMILES strings directly from the MIBiG database where possible, and otherwise retrieved them from PubChem [44] or NP Atlas [45] in that order of preference."
- [discussion] Validation was based on manual curation of a small MIBiG subset rather than automated fingerprinting of all BGCs.: "We considered using fingerprinting methods such as Morgan fingerprinting for automatic validation of MIBiG BGCs."
- [readme] Installation requires a Python 3.9.12 conda environment and pip installation of RAIChU.: "First, create a new environment with python version 3.9.12, for instance with conda: conda create -n raichu python==3.9.12"
- [results] MIBiG is the reference dataset of experimentally verified BGCs with known DNA sequence and associated natural product.: "the MIBiG database: a database of experimentally verified biosynthetic gene clusters for which both DNA sequence and the associated natural product are known"
