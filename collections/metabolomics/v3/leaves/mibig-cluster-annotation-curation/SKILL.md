---
name: mibig-cluster-annotation-curation
description: Use when you have antiSMASH annotations of curated MIBiG clusters plus
  a characterised natural-product structure, and need to diagnose why RAIChU-predicted
  scaffolds diverge from biosynthetic reality.
when_to_use_negative:
- The input BGCs are RiPP, terpene, or trans-AT PKS clusters outside the cis-AT PKS/NRPS/NRPS-PKS
  hybrid scope of the curated MIBiG subset
- The goal is de novo scaffold generation from raw antiSMASH output without a characterised
  reference structure to curate against
- Clusters lack any experimentally characterised product structure, so no predicted-vs-characterised
  comparison is possible
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: RAIChU
  role: Renders predicted end-product structures from antiSMASH annotations of the
    curated MIBiG clusters for comparison with characterised SMILES
  repo: https://github.com/BTheDragonMaster/RAIChU
- name: antiSMASH
  role: Provides the BGC detection and modular annotation input that RAIChU renders;
    its annotations are the object of the curation
- name: RDKit
  role: Structural comparison of RAIChU-rendered structures against characterised
    SMILES strings
- name: Python
  role: Execution environment (Python 3.9.12) and pipeline integration for the comparison
    and CSV-curation workflow
  repo: https://pypi.org/project/raichu
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s13321-024-00898-x
    title: 'RAIChU: automating the visualisation of natural product biosynthesis'
  source_content_hash: sha256:9c4b2ba3f08db07b09344f0a7fca3fe6927f5a7d0fbc054bd39a00e0c26c204e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BTheDragonMaster/RAIChU
  tools:
  - Python
  - RAIChU
  - RDKit
  - antiSMASH
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5333333333333333
n_claims: 15
---

# mibig-cluster-annotation-curation

## Summary

Curates MIBiG BGC annotations by running RAIChU on antiSMASH predictions of a manually curated MIBiG subset (cis-AT PKS, NRPS, NRPS/PKS hybrid clusters) and classifying every mismatch between RAIChU-rendered and experimentally characterised products into one of three cause categories.

## When to use

Use when you have antiSMASH annotations of curated MIBiG clusters plus a characterised natural-product structure, and need to diagnose why RAIChU-predicted scaffolds diverge from biosynthetic reality.

## When NOT to use

- The input BGCs are RiPP, terpene, or trans-AT PKS clusters outside the cis-AT PKS/NRPS/NRPS-PKS hybrid scope of the curated MIBiG subset
- The goal is de novo scaffold generation from raw antiSMASH output without a characterised reference structure to curate against
- Clusters lack any experimentally characterised product structure, so no predicted-vs-characterised comparison is possible

## Inputs

- antiSMASH annotations of 30 manually curated MIBiG BGCs (cis-AT PKS, NRPS, and NRPS/PKS hybrid clusters)
- Characterised natural-product SMILES strings from MIBiG (preferred), PubChem, or NP Atlas
- RAIChU environment (Python 3.9.12 via conda, raichu installed via pip)

## Outputs

- mibig_discrepancy_table.csv with 30 rows, each containing cluster ID, RAIChU-predicted vs characterised structure pair, and a non-empty cause-category field
- RAIChU-rendered predicted product structures ('spaghetti diagrams') per cluster
- Per-cluster mismatch classification into: post-assembly tailoring not encoded in the modular annotation, database/annotation errors, or RAIChU processing limitations

## How to apply

Create a conda environment with Python 3.9.12 ('conda create -n raichu python==3.9.12') and install RAIChU via pip, since the tool's biosynthetic rendering of PKS/NRPS chemistry is pinned to that interpreter version. Run RAIChU on the antiSMASH annotations of the 30 manually curated MIBiG BGCs to render predicted end-product structures, choosing the predicted product whose cyclisation site best matches the characterised natural product. Collect the characterised SMILES strings with a strict source preference: MIBiG database first, then PubChem, then NP Atlas, because database cross-links can be wrong (e.g. the colistin A MIBiG entry pointed to the wrong PubChem record). Compare each rendered structure to its characterised SMILES using RDKit structural comparison plus manual inspection of the rendered diagrams, since fingerprinting alone is unreliable when scaffolds rather than matured products are compared. Classify every mismatch into one of three cause categories: unpredicted post-assembly tailoring reactions, unassigned double-bond stereochemistry after PKS dehydration, or missing substrates in RAIChU, and record cluster ID, structure pair, and cause in mibig_discrepancy_table.csv. Validate that the CSV contains exactly 30 rows (one per curated cluster), each with a non-empty cause-category field and no cluster left unclassified.

## Related tools

- **RAIChU** (Renders predicted end-product structures from antiSMASH annotations of the curated MIBiG clusters for comparison with characterised SMILES) — https://github.com/BTheDragonMaster/RAIChU
- **antiSMASH** (Provides the BGC detection and modular annotation input that RAIChU renders; its annotations are the object of the curation)
- **RDKit** (Structural comparison of RAIChU-rendered structures against characterised SMILES strings)
- **Python** (Execution environment (Python 3.9.12) and pipeline integration for the comparison and CSV-curation workflow) — https://pypi.org/project/raichu

## Examples

```
conda create -n raichu python==3.9.12; conda activate raichu; pip install raichu
```

## Evaluation signals

- mibig_discrepancy_table.csv contains exactly 30 rows, one per curated MIBiG cluster
- Every row has a non-empty cause-category field and every mismatch falls into one of the three defined cause categories (post-assembly tailoring, double-bond stereochemistry after PKS dehydration, or missing substrates in RAIChU)
- Characterised SMILES provenance follows the documented preference order: MIBiG first, then PubChem, then NP Atlas
- Predicted product selected for comparison is the RAIChU output whose cyclisation site best matches the characterised natural product
- Each mismatch is confirmed by both RDKit structural comparison and manual inspection of the rendered diagrams

## Limitations

- Regio- and stereochemistry of tailoring enzymes cannot be predicted; users must specify reaction targets, so unpredicted tailoring systematically produces discrepancies
- Double bond stereochemistry following dehydration reactions in PKS systems is unassigned by RAIChU (e.g. as observed in the CMC-thuggacins)
- RAIChU can generate natural product scaffolds but not mature natural products from raw antiSMASH output, so predicted and real-world structures may not have high fingerprinting similarity
- MIBiG entries can contain database/annotation errors (e.g. colistin A BGC pointing to the wrong PubChem entry)
- Validation relied on manual curation of a few select MIBiG examples (10 NRPS, 10 PKS, 10 NRPS/PKS hybrid clusters) rather than automated validation of all MIBiG BGCs

## Evidence

- [methods] Environment setup pinned to Python 3.9.12: "First, create a new environment with python version 3.9.12, for instance with conda: conda create -n raichu python==3.9.12"
- [discussion] SMILES source preference order for characterised structures: "We collected SMILES strings directly from the MIBiG database where possible, and otherwise retrieved them from PubChem [44] or NP Atlas [45] in that order of preference."
- [results] Unpredicted tailoring regio/stereochemistry as a curation-relevant limitation: "As it is currently not possible to predict the regio and stereochemistry of most of these tailoring enzymes, users need to specify their reaction targets"
- [results] PKS dehydration double-bond stereochemistry as a discrepancy cause: "double bond stereochemistry following dehydration reactions in PKS systems, as observed in the CMC-thuggacins"
- [discussion] Scaffold-vs-matured-product gap limiting automated structural comparison: "As RAIChU can generate natural product scaffolds but not mature natural products from raw antiSMASH output, predicted and real-world structures may not have a high fingerprinting similarity"
- [readme] README confirming RAIChU scope as a natural-product visualisation chemoinformatics tool: "RAIChU is a chemoinformatics tool designed to illuminate the intricate processes of scaffold diversification in natural product chemistry"
