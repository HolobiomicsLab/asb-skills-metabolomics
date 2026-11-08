---
name: reaction-rule-network-construction
description: Use when transcript-metabolite correlated mass-shift associations must
  be turned into a directed reaction network, resolved into per-substrate acyclic
  DAGs, and mined for the longest valid reaction path starting from each initial metabolite.
when_to_use_negative:
- The pathway of interest involves multi-substrate reactions or stereoisomeric conversions
  — RetroRules-based rules are designed for single-substrate reactions only, and stereoisomers
  are not captured by mass spectrometric data
- The goal is MS/MS-based spectral annotation of mass features rather than reaction-path
  prediction — MEANtools currently supports no MS/MS data loading or spectral library/networking-based
  annotation
- The network already exists as an acyclic graph — cycle resolution via weakest-edge
  elimination would unnecessarily drop correlated edges
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3366
tools:
- name: MEANtools
  role: Constructs the directed reaction network from filtered mass shift–reaction
    associations, detects and resolves same-distance cycles by removing the weakest
    enzyme–metabolite correlation edge, extracts the longest valid reaction path per
    DAG, and exports Cytoscape-importable graphs
  repo: github.com/IntegrativeBioinformaticsLab/meantools
- name: RetroRules
  role: Source of general reaction rules (loose/medium/strict rules–enzyme association
    datasets) that define the candidate mass shift–reaction associations feeding the
    network
- name: Cytoscape
  role: Target visualization/inspection tool for the exported per-substrate DAG files
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1371/journal.pbio.3003307
    title: MEANtools integrates multi-omics data to identify metabolites and predict
      biosynthetic pathways
  source_content_hash: sha256:b746c53da4299311b79aedd65517542c4b8f796b64a507b84f5d2cd2976d9ede
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Cytoscape
  - MEANtools
  - RetroRules
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3366
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# reaction-rule-network-construction

## Summary

Builds the directed reaction network from mass shift–reaction associations (nodes are measured or unmeasured 'ghost' mass signatures, edges are annotated mass shifts) and converts it into per-substrate directed acyclic graphs (DAGs) by cycle resolution, from which the longest valid reaction path per initial metabolite is extracted.

## When to use

Use when transcript-metabolite correlated mass-shift associations must be turned into a directed reaction network, resolved into per-substrate acyclic DAGs, and mined for the longest valid reaction path starting from each initial metabolite.

## When NOT to use

- The pathway of interest involves multi-substrate reactions or stereoisomeric conversions — RetroRules-based rules are designed for single-substrate reactions only, and stereoisomers are not captured by mass spectrometric data
- The goal is MS/MS-based spectral annotation of mass features rather than reaction-path prediction — MEANtools currently supports no MS/MS data loading or spectral library/networking-based annotation
- The network already exists as an acyclic graph — cycle resolution via weakest-edge elimination would unnecessarily drop correlated edges

## Inputs

- RetroRules-derived reaction rules dataset (loose, medium, and strict entries combined), e.g. Combined_small_middle_big_datasets.csv from Zenodo record 15697913
- Reaction–enzyme association datasets (RetroRules-derived loose set, experimentally validated strict set, medium set with ECDomainMiner predictions)
- Sets of correlated enzyme-coding genes and metabolites (MR-based correlation of transcriptomics with metabolomics mass features)
- Candidate mass shift–reaction associations, including unmeasured 'ghost' mass signatures as intermediates

## Outputs

- Directed reaction network linking initial metabolites to products via annotated mass shifts (reaction_network.csv)
- Per-substrate acyclic DAG (one per initial metabolite) after weakest-edge cycle removal
- Longest valid reaction path extracted from each per-substrate DAG
- Cytoscape-importable graph files for each per-metabolite DAG

## How to apply

Load the RetroRules-derived reaction rules dataset (loose, medium, and strict entries combined in Combined_small_middle_big_datasets.csv from Zenodo record 15697913) and cross-check the reaction–enzyme association datasets against the sets of correlated enzyme-coding genes and metabolites to filter candidate mass shift–reaction associations, including unassigned 'ghost' mass signatures as intermediates. Construct the directed reaction network linking initial metabolites to products via the annotated mass shifts and write it out as reaction_network.csv. For each initial metabolite, detect cycles among metabolites at the same reaction distance and eliminate the edge with the weakest enzyme–metabolite correlation, yielding an acyclic per-substrate DAG; this resolution step is needed because correlation-based edges can create same-distance cycles that prevent path traversal. Traverse each DAG to extract the longest valid reaction path starting from its initial metabolite, then export every per-metabolite DAG in a Cytoscape-importable graph format for visualization and inspection.

## Related tools

- **MEANtools** (Constructs the directed reaction network from filtered mass shift–reaction associations, detects and resolves same-distance cycles by removing the weakest enzyme–metabolite correlation edge, extracts the longest valid reaction path per DAG, and exports Cytoscape-importable graphs) — github.com/IntegrativeBioinformaticsLab/meantools
- **RetroRules** (Source of general reaction rules (loose/medium/strict rules–enzyme association datasets) that define the candidate mass shift–reaction associations feeding the network)
- **Cytoscape** (Target visualization/inspection tool for the exported per-substrate DAG files)

## Evaluation signals

- reaction_network.csv exists and every edge connects an initial metabolite to a product via an annotated mass shift
- Each exported per-substrate graph is confirmed acyclic after weakest-edge cycle removal
- The longest valid reaction path is reported for every initial metabolite in each DAG
- All exported graph files import cleanly into Cytoscape
- Pathway quality judged qualitatively against the paper's benchmark: 5 of 7 characterized falcarindiol pathway steps anticipated

## Limitations

- Cycle resolution removes the weakest enzyme–metabolite correlation edge, which may drop true edges when correlation strength is uninformative
- Mass spectrometric data does not capture stereoisomers, hindering reconstruction of steps such as phenylalanine to p-coumaroyl-CoA
- RetroRules-based rules are designed for single-substrate reactions only, blocking steps like the two-substrate conversion to p-coumaroyltyramine
- Spurious reaction–Pfam links from the loose RetroRules ruleset can persist after the Pfam cutoff of 6, so the strict ruleset is preferable for confident predictions

## Evidence

- [discussion] cycle resolution by weakest-edge elimination: "In instances where cycles occur among metabolites at the same reaction distance from the initial metabolite, the edge featuring the weakest enzyme-metabolite correlation is eliminated."
- [discussion] filtering mass shift–reaction associations: "By cross-checking these reaction-enzyme association datasets with sets of correlated enzyme-coding genes and metabolites, MEANtools effectively filters the set of possible mass shift-reaction"
- [abstract] connecting transcript-correlated mass features via reactions: "MEANtools thus assesses whether these reactions can connect transcript-correlated mass features"
- [readme] documentation pointer for the MEANtools repository: "Read the documentation here: https://meantools.readthedocs.io/en/latest/"
