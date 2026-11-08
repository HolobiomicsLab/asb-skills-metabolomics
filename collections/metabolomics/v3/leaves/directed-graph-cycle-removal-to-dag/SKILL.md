---
name: directed-graph-cycle-removal-to-dag
description: Use when a directed reaction network built from mass-shift annotations
  contains cycles among metabolites and you need per-initial-metabolite DAGs from
  which the longest valid reaction path can be extracted.
when_to_use_negative:
- 'The candidate reactions are multi-substrate conversions: RetroRules-based rules
  in MEANtools are designed for single-substrate reactions only (e.g. p-coumaroyltyramine
  formation requires two substrates), so DAG edges cannot represent them'
- 'Accurate stereochemistry is required: mass spectrometric data does not capture
  stereoisomers, and MEANtools deliberately filters stereomeric RetroRules rules,
  so the DAG cannot resolve stereoisomeric steps'
- The network is already acyclic per substrate or the goal is generic graph analytics
  unrelated to mass-shift-derived biosynthetic pathway reconstruction
edam_topics:
- http://edamontology.org/topic_0218
tools:
- name: MEANtools
  role: Constructs the directed reaction network, performs per-substrate cycle removal
    by weakest-correlation edge elimination, extracts longest paths, and exports Cytoscape-importable
    DAGs
  repo: https://github.com/IntegrativeBioinformaticsLab/meantools
- name: Cytoscape
  role: Imports and visualizes the exported per-substrate DAG files for manual inspection
    of predicted pathways
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1371/journal.pbio.3003307
    title: MEANtools integrates multi-omics data to identify metabolites and predict
      biosynthetic pathways
  source_content_hash: sha256:24f7ea0b3f104c1038b6b4409935632330032c9a7c74d9e9fe76d4eb4969ef50
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/IntegrativeBioinformaticsLab/meantools
  tools:
  - Cytoscape
  - MEANtools
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0218
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# directed-graph-cycle-removal-to-dag

## Summary

MEANtools' strategy for converting a cyclic directed reaction network — whose nodes are measured and unmeasured (ghost) mass signatures linked by annotated mass shifts — into one acyclic per-substrate DAG by eliminating the weakest enzyme–metabolite correlation edge inside each same-distance cycle, then extracting the longest valid reaction path.

## When to use

Use when a directed reaction network built from mass-shift annotations contains cycles among metabolites and you need per-initial-metabolite DAGs from which the longest valid reaction path can be extracted.

## When NOT to use

- The candidate reactions are multi-substrate conversions: RetroRules-based rules in MEANtools are designed for single-substrate reactions only (e.g. p-coumaroyltyramine formation requires two substrates), so DAG edges cannot represent them
- Accurate stereochemistry is required: mass spectrometric data does not capture stereoisomers, and MEANtools deliberately filters stereomeric RetroRules rules, so the DAG cannot resolve stereoisomeric steps
- The network is already acyclic per substrate or the goal is generic graph analytics unrelated to mass-shift-derived biosynthetic pathway reconstruction

## Inputs

- Directed reaction network (reaction_network.csv) linking initial metabolites to products via annotated mass shifts
- Per-metabolite reaction-distance assignments for each node (measured mass features and ghost mass signatures)
- Enzyme–metabolite correlation edge weights (Pearson correlations converted to mutual ranks and exponential-decay edge scores, with pairs below 0.01 excluded)

## Outputs

- One acyclic per-substrate DAG for each initial metabolite
- The longest valid reaction path extracted from each DAG
- Per-metabolite DAG files in a Cytoscape-importable graph format

## How to apply

Start from the directed reaction network (reaction_network.csv) in which initial metabolites are linked to products via annotated mass shifts, with nodes covering both measured mass features and unmeasured ghost mass signatures. For each initial metabolite, detect cycles occurring among metabolites at the same reaction distance from that substrate; this matters because cycles spanning different distances do not block a valid biosynthetic ordering. Resolve each detected cycle by deleting the single edge with the weakest enzyme–metabolite correlation, iterating until the per-substrate graph is acyclic; gene–metabolite pairs with edge score below 0.01 are already excluded from the Ni→j networks beforehand. Once acyclic, traverse each DAG to extract the longest valid reaction path starting from its initial metabolite. Export every per-metabolite DAG in a Cytoscape-importable graph format so pathway hypotheses can be inspected visually.

## Related tools

- **MEANtools** (Constructs the directed reaction network, performs per-substrate cycle removal by weakest-correlation edge elimination, extracts longest paths, and exports Cytoscape-importable DAGs) — https://github.com/IntegrativeBioinformaticsLab/meantools
- **Cytoscape** (Imports and visualizes the exported per-substrate DAG files for manual inspection of predicted pathways)

## Evaluation signals

- Confirm every exported per-substrate graph is acyclic after edge removal
- Confirm the longest path per initial metabolite is reported for each DAG
- Confirm all exported graph files import cleanly into Cytoscape
- Check that removed edges correspond to the weakest enzyme–metabolite correlation among metabolites at the same reaction distance, not across distances
- Qualitatively benchmark recovered steps against known pathways: MEANtools anticipated 5 of 7 characterized falcarindiol pathway steps in tomato

## Limitations

- Cycles are resolved solely by weakest-correlation elimination, so a biologically correct step with a weak transcript–metabolite correlation can be discarded; Pearson-only integration already yields many false-positive associations and thresholding is difficult
- FC size may be insufficient to form a complete biosynthetic pathway, so a DAG may not span the full route even after acyclicity is achieved
- Ghost mass signatures (unmeasured intermediates) keep the graph connectable but their edges are not backed by measured correlations

## Evidence

- [discussion] cycle removal rationale: "In instances where cycles occur among metabolites at the same reaction distance from the initial metabolite, the edge featuring the weakest enzyme-metabolite correlation is eliminated."
- [abstract] why pathway prediction matters: "MEANtools correctly anticipated five out of seven steps of the characterized pathway"
- [intro] pipeline scope: "Here, we introduce MEANtools, a computational pipeline that combines statistical- and reaction-rules-based integration strategies."
- [discussion] weak-correlation prefilter: "Gene–metabolite pairs that show lower edge score than 0.01 are excluded in the Ni→j networks."
