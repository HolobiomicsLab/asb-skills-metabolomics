---
name: raichu-pathway-visualisation
description: Use when you have antiSMASH annotations for a PKS, NRPS, or hybrid PKS/NRPS
  BGC and need a module-resolution depiction of the biosynthetic pathway or the predicted
  end-product structure.
when_to_use_negative:
- You need mature natural products directly from raw antiSMASH output — RAIChU generates
  natural product scaffolds but not mature natural products without user-specified
  tailoring targets
- You need automated scaffold prediction for non-PKS/NRPS/non-hybrid classes without
  user-provided enzymatic steps
- The task is automated large-scale fingerprint-based validation of all MIBiG BGCs
  — this was considered but not implemented, and predicted vs. real-world structures
  may not show high fingerprinting similarity
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_0091
tools:
- name: RAIChU
  role: Core tool that produces depictions of biosynthetic transformations of PKS,
    NRPS, and hybrid PKS/NRPS systems and renders predicted end-product structures
  repo: https://github.com/BTheDragonMaster/RAIChU
- name: antiSMASH
  role: Provides the BGC detection and annotation output that RAIChU ingests to render
    pathway visualisations
- name: Python
  role: RAIChU is installed as a pip package and can be integrated into Python pipelines
  repo: https://pypi.org/project/raichu
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s13321-024-00898-x
    title: 'RAIChU: automating the visualisation of natural product biosynthesis'
  source_content_hash: sha256:1a323618f38d353c14bd48ccd116ce1fb2b6c32a4da99d4408e4b49e0c1e043d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BTheDragonMaster/RAIChU
  tools:
  - Python
  - RAIChU
  - antiSMASH
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# raichu-pathway-visualisation

## Summary

Run RAIChU on antiSMASH annotations of PKS, NRPS, or hybrid PKS/NRPS clusters to render automated 'spaghetti diagram' visualisations of biosynthetic transformations, and diagnose mismatches between predicted scaffolds and experimentally characterised natural products.

## When to use

Use when you have antiSMASH annotations for a PKS, NRPS, or hybrid PKS/NRPS BGC and need a module-resolution depiction of the biosynthetic pathway or the predicted end-product structure.

## When NOT to use

- You need mature natural products directly from raw antiSMASH output — RAIChU generates natural product scaffolds but not mature natural products without user-specified tailoring targets
- You need automated scaffold prediction for non-PKS/NRPS/non-hybrid classes without user-provided enzymatic steps
- The task is automated large-scale fingerprint-based validation of all MIBiG BGCs — this was considered but not implemented, and predicted vs. real-world structures may not show high fingerprinting similarity

## Inputs

- antiSMASH annotations of biosynthetic gene clusters (PKS, NRPS, or hybrid PKS/NRPS)
- Characterised natural product SMILES strings from MIBiG (preferred), PubChem, or NP Atlas
- User-specified tailoring reaction targets (since regio-/stereochemistry cannot be predicted)

## Outputs

- RAIChU 'spaghetti diagrams': module-resolution visualisations of biosynthetic pathways
- Predicted end-product scaffold structures for each cluster
- Discrepancy classification table (e.g. mibig_discrepancy_table.csv) recording cluster ID, structure pair, and cause category per mismatch

## How to apply

Create a conda environment with Python 3.9.12 and install RAIChU via pip, then run RAIChU on the antiSMASH annotation of the BGC to render the predicted end-product structure; when multiple cyclisation predictions exist, select the predicted product whose cyclisation site best matches the characterised natural product. For validation, collect characterised SMILES from MIBiG first, falling back to PubChem then NP Atlas in that order of preference, and compare RAIChU-rendered structures to the characterised SMILES using RDKit structural comparison plus manual inspection of the rendered diagrams. Classify any mismatch into the paper's three cause categories: unpredicted post-assembly tailoring reactions, unassigned double-bond stereochemistry after PKS dehydration (as in the CMC-thuggacins), or missing substrates/processing limitations in RAIChU. Note that RAIChU cannot predict regio- and stereochemistry of tailoring enzymes, so users must specify tailoring reaction targets; the tool includes 34 prevalent tailoring reactions to visualise fully maturated products.

## Related tools

- **RAIChU** (Core tool that produces depictions of biosynthetic transformations of PKS, NRPS, and hybrid PKS/NRPS systems and renders predicted end-product structures) — https://github.com/BTheDragonMaster/RAIChU
- **antiSMASH** (Provides the BGC detection and annotation output that RAIChU ingests to render pathway visualisations)
- **Python** (RAIChU is installed as a pip package and can be integrated into Python pipelines) — https://pypi.org/project/raichu

## Examples

```
conda create -n raichu python==3.9.12; conda activate raichu; pip install raichu
```

## Evaluation signals

- RAIChU achieved 100% cluster drawing correctness and 97.66% drawing readability on the validation sets; diagrams should be manually assessed for structure correctness, bond angles, and overlap
- The discrepancy table contains exactly one row per curated cluster (e.g. 30 rows for the curated MIBiG subset), each with a non-empty cause-category field and no unclassified cluster
- Every mismatch is assigned to one of the three cause categories: unpredicted post-assembly tailoring, unassigned double-bond stereochemistry after PKS dehydration, or missing substrates/RAIChU processing limitations
- Characterised SMILES provenance follows the preference order MIBiG → PubChem → NP Atlas
- No unexpected processing errors occur when running RAIChU across the full set of BGCs (validated against the entire MIBiG database)

## Limitations

- Regio- and stereochemistry of tailoring enzymes cannot be predicted; users must specify reaction targets
- Double bond stereochemistry following dehydration reactions in PKS systems is a source of discrepancy (observed in the CMC-thuggacins)
- Missing substrates in RAIChU can cause predicted/characterised structure mismatches
- Database/annotation errors occur — e.g. the colistin A BGC in MIBiG points to the wrong PubChem entry
- Cationic cascade reactions in terpenes necessitate programmatic shortcuts, leading to simplified representations of reaction intermediates

## Evidence

- [abstract] we developed RAIChU (Reaction Analysis through Illustrating Chemical Units), which produces depictions of biosynthetic transformations of PKS, NRPS, and hybrid PKS/NRPS systems: "we developed RAIChU (Reaction Analysis through Illustrating Chemical Units), which produces depictions of biosynthetic transformations of PKS, NRPS, and hybrid PKS/NRPS systems"
- [results] As it is currently not possible to predict the regio and stereochemistry of most of these tailoring enzymes, users need to specify their reaction targets: "As it is currently not possible to predict the regio and stereochemistry of most of these tailoring enzymes, users need to specify their reaction targets"
- [results] double bond stereochemistry following dehydration reactions in PKS systems, as observed in the CMC-thuggacins: "double bond stereochemistry following dehydration reactions in PKS systems, as observed in the CMC-thuggacins"
- [discussion] We collected SMILES strings directly from the MIBiG database where possible, and otherwise retrieved them from PubChem [44] or NP Atlas [45] in that order of preference.: "We collected SMILES strings directly from the MIBiG database where possible, and otherwise retrieved them from PubChem [44] or NP Atlas [45] in that order of preference."
- [readme] First, create a new environment with python version 3.9.12, for instance with conda: conda create -n raichu python==3.9.12: "First, create a new environment with python version 3.9.12, for instance with conda: conda create -n raichu python==3.9.12"
