---
name: longest-path-extraction-in-metabolic-networks
description: Use when a directed reaction network of correlated metabolite-transcript
  mass shifts has been converted into per-substrate DAGs and you need to report the
  longest valid reaction path from each initial metabolite as a candidate pathway.
when_to_use_negative:
- The per-substrate graph still contains cycles among metabolites at the same reaction
  distance — resolve these first by removing the weakest enzyme-metabolite correlation
  edge before path extraction.
- 'The reactions of interest are multi-substrate (e.g., p-coumaroyltyramine formation
  from tyrosine plus p-coumaroyl-CoA): RetroRules-based rules in MEANtools are designed
  for single-substrate reactions only.'
- 'Stereochemical resolution is required: mass spectrometric data does not capture
  stereoisomers, and MEANtools filters rules involving stereomeric structures to avoid
  complexity.'
edam_operation: http://edamontology.org/operation_3925
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0605
tools:
- name: MEANtools
  role: Builds the directed reaction network, resolves cycles into per-substrate DAGs
    by eliminating the weakest enzyme-metabolite correlation edge, and traverses each
    DAG to extract the longest valid reaction path
  repo: https://github.com/IntegrativeBioinformaticsLab/meantools
- name: Cytoscape
  role: Import and inspect the exported per-metabolite DAG graph files for visualization
    and quality checks
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1371/journal.pbio.3003307
    title: MEANtools integrates multi-omics data to identify metabolites and predict
      biosynthetic pathways
  source_content_hash: sha256:d0d2c24e7619d213f236f0ffa70f577e99e0d64d51a02aedcf8c2b873a486c92
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/IntegrativeBioinformaticsLab/meantools
  tools:
  - Cytoscape
  - MEANtools
  edam_operation: http://edamontology.org/operation_3925
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# longest-path-extraction-in-metabolic-networks

## Summary

Extracts the longest valid reaction path from each per-substrate directed acyclic graph (DAG) built by MEANtools, where nodes are measured or ghost mass signatures and edges are mass shift-reaction associations, to propose candidate biosynthetic pathways.

## When to use

Use when a directed reaction network of correlated metabolite-transcript mass shifts has been converted into per-substrate DAGs and you need to report the longest valid reaction path from each initial metabolite as a candidate pathway.

## When NOT to use

- The per-substrate graph still contains cycles among metabolites at the same reaction distance — resolve these first by removing the weakest enzyme-metabolite correlation edge before path extraction.
- The reactions of interest are multi-substrate (e.g., p-coumaroyltyramine formation from tyrosine plus p-coumaroyl-CoA): RetroRules-based rules in MEANtools are designed for single-substrate reactions only.
- Stereochemical resolution is required: mass spectrometric data does not capture stereoisomers, and MEANtools filters rules involving stereomeric structures to avoid complexity.

## Inputs

- per-substrate directed acyclic graph (DAG) derived from reaction_network.csv, one DAG per initial metabolite
- enzyme-metabolite correlation edge weights (mutual rank converted via exponential decay function)
- RetroRules-derived mass shift-reaction associations (Combined_small_middle_big_datasets.csv from Zenodo record 15697913)
- list of initial (substrate) metabolites with measured mass features and ghost mass signatures

## Outputs

- longest valid reaction path per initial metabolite
- per-metabolite DAG exported in a Cytoscape-importable graph format
- candidate biosynthetic pathway hypotheses linking mass features via annotated reaction rules

## How to apply

First ensure the MEANtools-directed reaction network (nodes = measured or unmeasured 'ghost' mass signatures, edges = annotated mass shifts linking initial metabolites to products) has been cycle-resolved: for each initial metabolite, detect cycles among metabolites at the same reaction distance and eliminate the edge with the weakest enzyme-metabolite correlation so the graph is acyclic. Then traverse each per-substrate DAG to extract the longest valid reaction path starting from its initial metabolite; the path score is driven by the exponential-decay-weighted mutual-rank (MR) edges between mass features and biosynthetic gene transcripts. Export each per-metabolite DAG in a Cytoscape-importable graph format for visualization and inspection, and judge pathway quality qualitatively — in the paper's falcarindiol (tomato) case, 5 of 7 characterized pathway steps were anticipated.

## Related tools

- **MEANtools** (Builds the directed reaction network, resolves cycles into per-substrate DAGs by eliminating the weakest enzyme-metabolite correlation edge, and traverses each DAG to extract the longest valid reaction path) — https://github.com/IntegrativeBioinformaticsLab/meantools
- **Cytoscape** (Import and inspect the exported per-metabolite DAG graph files for visualization and quality checks)

## Evaluation signals

- Each exported per-substrate graph is confirmed acyclic after weakest-edge elimination.
- Exactly one longest valid reaction path is reported per initial metabolite.
- All exported graph files import cleanly into Cytoscape.
- Reported path steps can be cross-checked against characterized pathway steps (in the paper's benchmark, 5 of 7 falcarindiol pathway steps were anticipated).
- Removed edges correspond to the weakest enzyme-metabolite correlation among same-reaction-distance metabolites in each detected cycle.

## Limitations

- Edge removal for cycle resolution is greedy on the weakest enzyme-metabolite correlation, so a biologically correct edge may be discarded when correlations are noisy.
- FC size may be insufficient to form a complete biosynthetic pathway, limiting the span of extractable paths.
- Mass spectrometric data does not capture stereoisomers, and multi-substrate reactions cannot be represented by single-substrate RetroRules rules, so some true steps cannot appear in any DAG path.
- Pearson-based correlation integration alone yields many false positives; threshold choice (e.g., edge score cutoff 0.01) affects which edges and thus which longest paths are recovered.

## Evidence

- [discussion] cycle resolution by weakest edge: "In instances where cycles occur among metabolites at the same reaction distance from the initial metabolite, the edge featuring the weakest enzyme-metabolite correlation is eliminated."
- [abstract] longest path per initial metabolite: "MEANtools correctly anticipated five out of seven steps of the characterized pathway"
- [discussion] edge weighting from MR: "correlation values between mass features and transcripts are first converted to MRs [49], which are then subjected to an exponential decay function"
- [readme] readme pointer to docs: "Read the documentation here:

https://meantools.readthedocs.io/en/latest/"
- [supplementary] acyclic per-substrate DAG construction: [evidence span withheld — anchor status: not_found]
