---
name: biosynthetic-discrepancy-classification
description: Use when you have RAIChU-predicted product scaffolds plus characterised
  SMILES for the same PKS/NRPS/NRPS-PKS hybrid BGCs and need to systematically explain
  why predictions diverge from the real natural product.
when_to_use_negative:
- You only have raw antiSMASH output without characterised product SMILES — RAIChU
  generates natural product scaffolds, not mature natural products, so predicted and
  real-world structures may not have high fingerprinting similarity
- You need automated validation of all MIBiG BGCs via fingerprinting — this was considered
  but not implemented; the workflow relies on manual curation of select examples
- The BGC class falls outside PKS/NRPS/hybrid scope with no user-provided enzymatic-step
  predictions (e.g. RiPP or terpene cationic cascades requiring programmatic shortcuts)
edam_topics:
- http://edamontology.org/topic_2258
tools:
- name: RAIChU
  role: Renders predicted end-product structures (spaghetti diagrams) from antiSMASH
    annotations of the curated MIBiG BGCs
  repo: https://github.com/BTheDragonMaster/RAIChU
- name: antiSMASH
  role: Supplies the BGC detection/annotation output that RAIChU consumes as input
    for the curated clusters
- name: RDKit
  role: Used for structural comparison between RAIChU-rendered structures and characterised
    SMILES strings
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s13321-024-00898-x
    title: 'RAIChU: automating the visualisation of natural product biosynthesis'
  source_content_hash: sha256:6f81354b88c4c0a2f6eea314f29dfc419ccd2d8eb4658f31897066be82bf81d6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BTheDragonMaster/RAIChU
  tools:
  - Python
  - RAIChU
  - RDKit
  - antiSMASH
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# biosynthetic-discrepancy-classification

## Summary

A validation workflow that compares RAIChU-rendered product structures against experimentally characterised natural products for a curated MIBiG subset, then classifies every mismatch into one of three cause categories: unpredicted post-assembly tailoring, unassigned double-bond stereochemistry after PKS dehydration, or missing substrates/RAIChU processing limitations.

## When to use

Use when you have RAIChU-predicted product scaffolds plus characterised SMILES for the same PKS/NRPS/NRPS-PKS hybrid BGCs and need to systematically explain why predictions diverge from the real natural product.

## When NOT to use

- You only have raw antiSMASH output without characterised product SMILES — RAIChU generates natural product scaffolds, not mature natural products, so predicted and real-world structures may not have high fingerprinting similarity
- You need automated validation of all MIBiG BGCs via fingerprinting — this was considered but not implemented; the workflow relies on manual curation of select examples
- The BGC class falls outside PKS/NRPS/hybrid scope with no user-provided enzymatic-step predictions (e.g. RiPP or terpene cationic cascades requiring programmatic shortcuts)

## Inputs

- antiSMASH annotations of ~30 manually curated MIBiG BGCs (NRPS, cis-AT PKS, NRPS/PKS hybrid)
- Characterised SMILES strings from MIBiG, or secondarily PubChem then NP Atlas
- RAIChU-rendered predicted product structures (spaghetti diagrams)

## Outputs

- mibig_discrepancy_table.csv with exactly 30 rows (one per curated cluster), each containing cluster ID, RAIChU-predicted vs characterised structure pair, and a non-empty cause-category field
- RDKit structural comparison results plus manual inspection notes per cluster

## How to apply

Create a conda environment with Python 3.9.12 and install RAIChU via pip, then run RAIChU on the antiSMASH annotations of the ~30 manually curated MIBiG BGCs (10 NRPS, 10 PKS/cis-AT PKS, 10 NRPS/PKS hybrid) to render predicted end-product structures, selecting the predicted product whose cyclisation site best matches the characterised natural product. Collect characterised SMILES strings for each cluster, taking them from the MIBiG database where possible and otherwise from PubChem or NP Atlas in that order of preference. Compare each rendered structure to its characterised SMILES using RDKit structural comparison plus manual inspection of the rendered diagrams. Classify every mismatch into one of the paper's three cause categories — unpredicted post-assembly tailoring reactions, unassigned double-bond stereochemistry after PKS dehydration (as seen in the CMC-thuggacins), or missing substrates/database-annotation errors — and record cluster ID, structure pair, and cause in mibig_discrepancy_table.csv. The rationale is that RAIChU generates scaffolds rather than fully maturated products, so mismatches reflect known, enumerable model limitations rather than random tool failure.

## Related tools

- **RAIChU** (Renders predicted end-product structures (spaghetti diagrams) from antiSMASH annotations of the curated MIBiG BGCs) — https://github.com/BTheDragonMaster/RAIChU
- **antiSMASH** (Supplies the BGC detection/annotation output that RAIChU consumes as input for the curated clusters)
- **RDKit** (Used for structural comparison between RAIChU-rendered structures and characterised SMILES strings)

## Examples

```
conda create -n raichu python==3.9.12 && conda activate raichu && pip install raichu
```

## Evaluation signals

- mibig_discrepancy_table.csv contains exactly 30 rows, one per curated cluster
- Every row has a non-empty cause-category field; no cluster is left unclassified
- Each mismatch maps to one of the paper's three cause categories: unpredicted post-assembly tailoring, unassigned PKS dehydration double-bond stereochemistry, or missing substrates/annotation errors
- Characterised SMILES provenance follows the specified preference order (MIBiG first, then PubChem, then NP Atlas)
- Each RAIChU-predicted product was chosen such that its cyclisation site best matches the characterised natural product

## Limitations

- Regio- and stereochemistry of tailoring enzymes cannot be predicted; users must specify reaction targets
- Double bond stereochemistry following dehydration reactions in PKS systems is a recurring source of discrepancy (observed in the CMC-thuggacins)
- RAIChU can generate natural product scaffolds but not mature natural products from raw antiSMASH output, so predicted and real-world structures may not have high fingerprinting similarity
- Terpene cationic cascade reactions necessitate programmatic shortcuts, potentially leading to simplified representations of reaction intermediates

## Evidence

- [discussion] We collected SMILES strings directly from the MIBiG database where possible, and otherwise retrieved them from PubChem [44] or NP Atlas [45] in that order of preference.: "We collected SMILES strings directly from the MIBiG database where possible, and otherwise retrieved them from PubChem [44] or NP Atlas [45] in that order of preference."
- [results] double bond stereochemistry following dehydration reactions in PKS systems, as observed in the CMC-thuggacins: "double bond stereochemistry following dehydration reactions in PKS systems, as observed in the CMC-thuggacins"
- [readme] First, create a new environment with python version 3.9.12, for instance with conda: conda create -n raichu python==3.9.12: "First, create a new environment with python version 3.9.12, for instance with conda: conda create -n raichu python==3.9.12"
- [discussion] As RAIChU can generate natural product scaffolds but not mature natural products from raw antiSMASH output, predicted and real-world structures may not have a high fingerprinting similarity: "As RAIChU can generate natural product scaffolds but not mature natural products from raw antiSMASH output, predicted and real-world structures may not have a high fingerprinting similarity"
- [results] we additionally tested RAIChU on antiSMASH predictions of real-world BGCs from the MIBiG database: "we additionally tested RAIChU on antiSMASH predictions of real-world BGCs from the MIBiG database"
