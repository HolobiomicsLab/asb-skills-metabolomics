---
name: metabolic-pathway-hypothesis-reconstruction
description: Use when a paired transcriptomic-metabolomic dataset is available and
  the goal is unsupervised, de novo reconstruction of a specialized metabolite biosynthetic
  pathway without prior knowledge of a 'bait' compound or enzyme.
when_to_use_negative:
- 'The input requires reasoning over stereoisomers: mass spectrometric data does not
  capture stereochemistry, and RetroRules rules involving stereomeric structures are
  filtered, so stereo-specific steps (e.g., phenylalanine to p-coumaroyl-CoA) will
  be missed.'
- The target pathway contains multi-substrate reactions (e.g., tyrosine plus p-coumaroyl-CoA
  to p-coumaroyltyramine), since RetroRules-based rules are designed for single-substrate
  reactions only.
- A supervised or bait-based co-expression analysis is desired instead; this skill
  is specifically the unsupervised network/DAG-based reconstruction stage.
edam_operation: http://edamontology.org/operation_3929
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0602
tools:
- name: MEANtools
  role: Implements the directed reaction-network construction, per-substrate DAG conversion
    via weakest-edge cycle removal, longest-path extraction, and Cytoscape export
  repo: https://github.com/IntegrativeBioinformaticsLab/meantools
- name: Cytoscape
  role: Visualization and inspection of the exported per-substrate DAG graph files
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1371/journal.pbio.3003307
    title: MEANtools integrates multi-omics data to identify metabolites and predict
      biosynthetic pathways
  source_content_hash: sha256:c37cede505a6805213ad89c8a101a00e5812a8899e36d2b9fb0ccb5ace57904c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/IntegrativeBioinformaticsLab/meantools
  tools:
  - Cytoscape
  - MEANtools
  edam_operation: http://edamontology.org/operation_3929
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# metabolic-pathway-hypothesis-reconstruction

## Summary

Reconstructs biosynthetic pathway hypotheses from paired transcriptomic-metabolomic data by building a directed reaction network of mass signatures (including unmeasured 'ghost' intermediates), converting it into per-substrate directed acyclic graphs, and extracting the longest valid reaction path from each DAG.

## When to use

Use when a paired transcriptomic-metabolomic dataset is available and the goal is unsupervised, de novo reconstruction of a specialized metabolite biosynthetic pathway without prior knowledge of a 'bait' compound or enzyme.

## When NOT to use

- The input requires reasoning over stereoisomers: mass spectrometric data does not capture stereochemistry, and RetroRules rules involving stereomeric structures are filtered, so stereo-specific steps (e.g., phenylalanine to p-coumaroyl-CoA) will be missed.
- The target pathway contains multi-substrate reactions (e.g., tyrosine plus p-coumaroyl-CoA to p-coumaroyltyramine), since RetroRules-based rules are designed for single-substrate reactions only.
- A supervised or bait-based co-expression analysis is desired instead; this skill is specifically the unsupervised network/DAG-based reconstruction stage.

## Inputs

- Reaction rules dataset combining loose, medium, and strict RetroRules entries (Combined_small_middle_big_datasets.csv, Zenodo record 15697913)
- Reaction-enzyme association datasets cross-referenced with correlated enzyme-coding genes and metabolites
- Mass shift-reaction associations with annotated mass features and ghost (unmeasured) mass signatures
- Pearson/MR-derived edge weights from the paired transcriptomic-metabolomic dataset (e.g., the Jeon and colleagues (2020) tomato dataset)

## Outputs

- reaction_network.csv — the directed reaction network linking initial metabolites to products via annotated mass shifts
- Per-substrate directed acyclic graphs (DAGs), one per initial metabolite
- Longest valid reaction path per initial metabolite
- Cytoscape-importable graph files for each per-metabolite DAG

## How to apply

First cross-check the reaction-enzyme association datasets (loose, medium, and strict RetroRules-derived rules, combined in Combined_small_middle_big_datasets.csv from Zenodo record 15697913) with sets of correlated enzyme-coding genes and metabolites to filter candidate mass shift-reaction associations, incorporating ghost mass signatures as unassigned intermediates. Then construct a directed reaction network linking initial metabolites to products via the annotated mass shifts and export it as reaction_network.csv. For each initial metabolite, detect cycles among metabolites at the same reaction distance and eliminate the edge with the weakest enzyme-metabolite correlation to render the graph acyclic — this per-substrate DAG conversion is required because cycles make longest-path traversal undefined. Finally, traverse each DAG to extract the longest valid reaction path from the initial metabolite, and export each per-metabolite DAG in a Cytoscape-importable graph format for inspection. Validate by confirming acyclicity of each exported graph, confirming the longest path is reported per initial metabolite, and checking Cytoscape importability; judge pathway quality qualitatively, e.g. against the paper's benchmark where 5 of 7 characterized falcarindiol pathway steps were anticipated.

## Related tools

- **MEANtools** (Implements the directed reaction-network construction, per-substrate DAG conversion via weakest-edge cycle removal, longest-path extraction, and Cytoscape export) — https://github.com/IntegrativeBioinformaticsLab/meantools
- **Cytoscape** (Visualization and inspection of the exported per-substrate DAG graph files)

## Evaluation signals

- Each exported per-substrate graph is confirmed acyclic after weakest-edge cycle elimination
- A longest valid reaction path is reported for every initial metabolite in its DAG
- All exported graph files import cleanly into Cytoscape without errors
- reaction_network.csv contains edges linking initial metabolites to products via the annotated mass shifts
- Pathway quality judged qualitatively against the paper's benchmark: 5 of 7 characterized falcarindiol pathway steps anticipated

## Limitations

- Cycles among metabolites at the same reaction distance are resolved by removing the weakest enzyme-metabolite correlation edge, so a biologically correct edge may be removed when correlations are misleading.
- Stereoisomeric conversions are not captured by mass spectrometric data and stereomeric RetroRules are filtered, hindering reconstruction of steps like phenylalanine to p-coumaroyl-CoA.
- Multi-substrate reactions (e.g., p-coumaroyltyramine formation) cannot be modeled because RetroRules-based rules are designed for single-substrate reactions only.
- The loose RetroRules ruleset can retain spurious reaction-Pfam links after the Pfam cutoff of 6, producing predictions associated with unrelated Pfams; the strict rules dataset is preferred when available.

## Evidence

- [discussion] Cycle elimination by weakest edge: "In instances where cycles occur among metabolites at the same reaction distance from the initial metabolite, the edge featuring the weakest enzyme-metabolite correlation is eliminated."
- [discussion] Filtering mass shift-reaction associations: "By cross-checking these reaction-enzyme association datasets with sets of correlated enzyme-coding genes and metabolites, MEANtools effectively filters the set of possible mass shift-reaction"
- [abstract] Benchmark pathway performance: "MEANtools correctly anticipated five out of seven steps of the characterized pathway"
- [discussion] RetroRules rules datasets availability: "The three reaction rules datasets resulting from RetroRules, loose, medium, and strict are available in a single CSV file in the https://zenodo.org/records/15697913."
- [discussion] MEANtools scope as hypothesis generator: "MEANtools can generate testable hypotheses on metabolic pathways with little to no prior knowledge, by integrating metabolomics and transcriptomics data."
- [discussion] Spurious links in loose ruleset: "These spurious links, coming from RetroRules, have been kept in the loose dataset after applying the Pfam cutoff of 6, which highlights the importance of using the strict rules dataset when"
