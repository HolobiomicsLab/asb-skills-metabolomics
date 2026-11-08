---
name: cytoscape-network-export
description: Use when a MEANtools reaction network has been converted to per-substrate
  DAGs and each DAG must be exported in a Cytoscape-importable format for visualization,
  inspection, and qualitative pathway assessment.
when_to_use_negative:
- The reaction network still contains cycles among metabolites at the same reaction
  distance — cycle resolution (removing the weakest enzyme-metabolite correlation
  edge) must happen before export, otherwise the exported graph is not a valid DAG
- The goal is MS/MS spectral-library or molecular-networking annotation rather than
  reaction-rule-based biosynthetic pathway DAGs — the article notes MEANtools currently
  does not support MS/MS data loading or spectral networking-based annotation
- The pathway requires multi-substrate reactions or stereoisomer resolution — RetroRules-based
  rules are designed for single-substrate reactions only and MEANtools filters stereomeric
  conversion rules, so exported DAGs will not represent those steps correctly
edam_operation: http://edamontology.org/operation_3083
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_3318
tools:
- name: MEANtools
  role: Constructs the directed reaction network, resolves cycles into per-substrate
    DAGs, extracts longest paths, and writes each DAG in a Cytoscape-importable graph
    format
  repo: https://github.com/IntegrativeBioinformaticsLab/meantools
- name: Cytoscape
  role: Target application into which the exported per-substrate DAG files are imported
    for visualization and inspection
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1371/journal.pbio.3003307
    title: MEANtools integrates multi-omics data to identify metabolites and predict
      biosynthetic pathways
  source_content_hash: sha256:7a9721c3b27c7bbb42a253f0fbecb4ada40615b442f3964afe2d1086ddec3e9e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/IntegrativeBioinformaticsLab/meantools
  tools:
  - Cytoscape
  - MEANtools
  edam_operation: http://edamontology.org/operation_3083
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# cytoscape-network-export

## Summary

Export each per-substrate directed acyclic reaction graph (DAG) produced by MEANtools pathway prediction into a Cytoscape-importable graph format for visualization and manual inspection of predicted biosynthetic pathways.

## When to use

Use when a MEANtools reaction network has been converted to per-substrate DAGs and each DAG must be exported in a Cytoscape-importable format for visualization, inspection, and qualitative pathway assessment.

## When NOT to use

- The reaction network still contains cycles among metabolites at the same reaction distance — cycle resolution (removing the weakest enzyme-metabolite correlation edge) must happen before export, otherwise the exported graph is not a valid DAG
- The goal is MS/MS spectral-library or molecular-networking annotation rather than reaction-rule-based biosynthetic pathway DAGs — the article notes MEANtools currently does not support MS/MS data loading or spectral networking-based annotation
- The pathway requires multi-substrate reactions or stereoisomer resolution — RetroRules-based rules are designed for single-substrate reactions only and MEANtools filters stereomeric conversion rules, so exported DAGs will not represent those steps correctly

## Inputs

- Directed reaction network (reaction_network.csv) linking initial metabolites to products via annotated mass shifts
- Per-substrate cyclic reaction networks awaiting cycle resolution (weakest enzyme-metabolite correlation edge removal)
- RetroRules-derived reaction rules dataset (Combined_small_middle_big_datasets.csv from Zenodo record 15697913, combining loose, medium, and strict entries)
- Sets of correlated enzyme-coding genes and metabolites used to filter mass shift-reaction associations, including ghost (unmeasured) mass signatures

## Outputs

- Per-metabolite DAG files in a Cytoscape-importable graph format (one graph per initial metabolite)
- Longest valid reaction path extracted per initial metabolite
- Acyclic per-substrate graphs ready for Cytoscape visualization and inspection

## How to apply

First ensure the directed reaction network (reaction_network.csv, linking initial metabolites to products via annotated mass shifts) has been built from RetroRules-derived rules cross-checked with correlated enzyme-coding genes and metabolites. For each initial metabolite, detect cycles among metabolites at the same reaction distance and eliminate the edge with the weakest enzyme-metabolite correlation to obtain an acyclic DAG; note that gene-metabolite pairs with edge score below 0.01 were already excluded from the Ni→j networks. Traverse each DAG to extract the longest valid reaction path starting from its initial metabolite. Then export each per-metabolite DAG in a Cytoscape-importable graph format, writing one graph per substrate so each can be loaded and inspected independently in Cytoscape. Correctness is confirmed by checking that each exported graph is acyclic, that the longest path per initial metabolite is reported, and that all files import cleanly into Cytoscape; pathway quality is then judged qualitatively (the reference analysis anticipated 5 of 7 characterized falcarindiol pathway steps in tomato).

## Related tools

- **MEANtools** (Constructs the directed reaction network, resolves cycles into per-substrate DAGs, extracts longest paths, and writes each DAG in a Cytoscape-importable graph format) — https://github.com/IntegrativeBioinformaticsLab/meantools
- **Cytoscape** (Target application into which the exported per-substrate DAG files are imported for visualization and inspection)

## Evaluation signals

- Each exported per-substrate graph is confirmed acyclic (cycle removal removed the weakest enzyme-metabolite correlation edge among same-distance metabolites)
- The longest valid reaction path per initial metabolite is reported for every exported DAG
- All exported graph files import cleanly into Cytoscape without errors
- Gene-metabolite pairs with edge score below 0.01 are absent from the exported Ni→j networks
- Exported pathways qualitatively reproduce the reference performance, e.g. 5 of 7 characterized falcarindiol biosynthetic steps anticipated in the tomato dataset

## Limitations

- Feature cluster (FC) size may be insufficient to form a complete biosynthetic pathway, so exported DAGs can cover only part of a pathway
- Mass spectrometric data does not capture stereoisomers and RetroRules-based rules handle single-substrate reactions only, hindering reconstruction of steps such as phenylalanine to p-coumaroyl-CoA or p-coumaroyltyramine formation
- Loose RetroRules rulesets can retain spurious reaction-Pfam links even after a Pfam cutoff of 6, so exported DAGs built from loose rules may contain spurious enzyme annotations; use the strict rules dataset when possible

## Evidence

- [discussion] Cycle-removal rationale for DAG validity before export: "In instances where cycles occur among metabolites at the same reaction distance from the initial metabolite, the edge featuring the weakest enzyme-metabolite correlation is eliminated."
- [discussion] Edge-score threshold applied in Ni→j networks underlying the exported DAGs: "Gene–metabolite pairs that show lower edge score than 0.01 are excluded in the Ni→j networks."
- [discussion] Purpose of the downstream pathway graphs the DAG exports support: "MEANtools can generate testable hypotheses on metabolic pathways with little to no prior knowledge, by integrating metabolomics and transcriptomics data."
- [readme] Tool repository and documentation source for the exporting software: "Read the documentation here: https://meantools.readthedocs.io/en/latest/"
