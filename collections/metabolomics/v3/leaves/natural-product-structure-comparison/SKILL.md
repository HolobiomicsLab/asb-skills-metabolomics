---
name: natural-product-structure-comparison
description: Use when you have antiSMASH annotations of PKS, NRPS, or NRPS/PKS hybrid
  BGCs and need to validate RAIChU-predicted product structures against characterised
  natural product structures, classifying any mismatches.
when_to_use_negative:
- You need automatic fingerprinting-based validation across all MIBiG BGCs — the article
  notes fingerprinting was considered but not implemented and that RAIChU generates
  scaffolds, not mature natural products, so predicted and real-world structures may
  not show high fingerprinting similarity
- Your BGCs are RiPP, terpene, or other classes outside the cis-AT PKS/NRPS/hybrid
  modular scope of this comparison workflow
- Regio- and stereochemistry of tailoring enzymes must be predicted automatically
  — RAIChU requires users to specify reaction targets
edam_operation: http://edamontology.org/operation_2929
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3314
tools:
- name: RAIChU
  role: Renders predicted end-product structures from antiSMASH annotations of PKS/NRPS
    BGCs for comparison with characterised products
  repo: https://github.com/BTheDragonMaster/RAIChU
- name: antiSMASH
  role: Provides BGC detection and annotations that are uploaded as input to RAIChU
    for the curated MIBiG clusters
- name: Python
  role: Environment (conda Python 3.9.12, pip-installed RAIChU) and RDKit-based structural
    comparison of predicted vs characterised SMILES
  repo: https://pypi.org/project/raichu
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s13321-024-00898-x
    title: 'RAIChU: automating the visualisation of natural product biosynthesis'
  source_content_hash: sha256:b72a1bd645f5a51e1412e456f56d6b190fd364463a2c65549f1a556abe415b7e
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
yes_rate: 0.8
n_claims: 10
---

# natural-product-structure-comparison

## Summary

Compares RAIChU-rendered biosynthetic product structures against experimentally characterised natural products (SMILES from MIBiG, PubChem, or NP Atlas) to identify and classify discrepancies between predicted and real-world biosynthetic scaffolds.

## When to use

Use when you have antiSMASH annotations of PKS, NRPS, or NRPS/PKS hybrid BGCs and need to validate RAIChU-predicted product structures against characterised natural product structures, classifying any mismatches.

## When NOT to use

- You need automatic fingerprinting-based validation across all MIBiG BGCs — the article notes fingerprinting was considered but not implemented and that RAIChU generates scaffolds, not mature natural products, so predicted and real-world structures may not show high fingerprinting similarity
- Your BGCs are RiPP, terpene, or other classes outside the cis-AT PKS/NRPS/hybrid modular scope of this comparison workflow
- Regio- and stereochemistry of tailoring enzymes must be predicted automatically — RAIChU requires users to specify reaction targets

## Inputs

- antiSMASH annotations of curated MIBiG BGCs (e.g. 30 clusters spanning cis-AT PKS, NRPS, and NRPS/PKS hybrids)
- RAIChU-rendered predicted end-product structure diagrams
- Characterised SMILES strings from MIBiG (preferred), PubChem, or NP Atlas

## Outputs

- mibig_discrepancy_table.csv with one row per cluster: cluster ID, predicted/characterised structure pair, and cause category (post-assembly tailoring, double-bond stereochemistry, or RAIChU processing limitation)
- Per-cluster mismatch classification (e.g. post-assembly tailoring not encoded in the modular annotation, database/annotation errors, RAIChU processing limitations)

## How to apply

Run RAIChU (installed via 'pip install raichu' in a Python 3.9.12 conda environment) on antiSMASH annotations of each curated BGC to render predicted end-product structures, selecting the predicted product whose cyclisation site best matches the characterised natural product. Collect characterised SMILES strings, taking them from the MIBiG database where possible and otherwise from PubChem or NP Atlas in that order of preference. Compare each RAIChU-rendered structure to its characterised SMILES using structural comparison with RDKit plus manual inspection of the rendered diagrams. Classify every mismatch into one of three cause categories: unpredicted post-assembly tailoring reactions, unassigned double-bond stereochemistry after PKS dehydration, or missing substrates/processing limitations in RAIChU. Record cluster ID, structure pair, and cause in a CSV, verifying one complete, non-empty row per cluster.

## Related tools

- **RAIChU** (Renders predicted end-product structures from antiSMASH annotations of PKS/NRPS BGCs for comparison with characterised products) — https://github.com/BTheDragonMaster/RAIChU
- **antiSMASH** (Provides BGC detection and annotations that are uploaded as input to RAIChU for the curated MIBiG clusters)
- **Python** (Environment (conda Python 3.9.12, pip-installed RAIChU) and RDKit-based structural comparison of predicted vs characterised SMILES) — https://pypi.org/project/raichu

## Examples

```
conda create -n raichu python==3.9.12; conda activate raichu; pip install raichu
```

## Evaluation signals

- mibig_discrepancy_table.csv contains exactly one row per curated cluster (30 rows for the 30-cluster validation set)
- Every row has a non-empty cause-category field; no cluster is left unclassified
- Each mismatch is assigned to one of the three predefined cause categories (post-assembly tailoring, PKS dehydration double-bond stereochemistry, RAIChU processing/database limitations)
- Characterised SMILES were sourced in the stated preference order: MIBiG first, then PubChem, then NP Atlas
- Predicted product figures were chosen so the cyclisation site best matches the characterised natural product's cyclisation site

## Limitations

- Discrepancies typically arise from unpredicted post-assembly tailoring reactions, unassigned double-bond stereochemistry after PKS dehydration, or missing substrates in RAIChU
- Regio- and stereochemistry of tailoring enzymes cannot be predicted; users must specify their reaction targets
- RAIChU can generate natural product scaffolds but not mature natural products from raw antiSMASH output, so predicted and real-world structures may not have high fingerprinting similarity
- Validation relied on manual curation of only a few select MIBiG examples (10 NRPS, 10 PKS, 10 NRPS/PKS hybrid clusters) rather than automated validation of all MIBiG BGCs
- Database errors occur, e.g. the colistin A BGC in MIBiG points to the wrong PubChem entry

## Evidence

- [discussion] We collected SMILES strings directly from the MIBiG database where possible, and otherwise retrieved them from PubChem [44] or NP Atlas [45] in that order of preference.: "We collected SMILES strings directly from the MIBiG database where possible, and otherwise retrieved them from PubChem [44] or NP Atlas [45] in that order of preference."
- [discussion] For our figures, we chose the predicted product that best matched the cyclisation site found in the characterised natural product.: "For our figures, we chose the predicted product that best matched the cyclisation site found in the characterised natural product."
- [results] double bond stereochemistry following dehydration reactions in PKS systems, as observed in the CMC-thuggacins: "double bond stereochemistry following dehydration reactions in PKS systems, as observed in the CMC-thuggacins"
- [readme] First, create a new environment with python version 3.9.12, for instance with conda: conda create -n raichu python==3.9.12: "First, create a new environment with python version 3.9.12, for instance with conda: conda create -n raichu python==3.9.12"
- [results] the colistin A BGC in the MIBiG database [36] points to the wrong PubChem entry: "the colistin A BGC in the MIBiG database [36] points to the wrong PubChem entry"
