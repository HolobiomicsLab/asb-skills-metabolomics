---
name: metabolic-state-annotation-in-hepatocytes
description: Use when a SpaceM spatio-molecular matrix (MORPHnMOL.csv) of single-cell
  metabolic profiles must be partitioned into unsupervised metabolic states and per-condition
  state proportions quantified, e.g. CTRL vs +FA vs +FA+IL17A vs +FA+IL17A+TPCA1 NAFLD/NASH
  cultures.
when_to_use_negative:
- The goal is to distinguish different cell types rather than resolve states within
  one isogenic population — the article addresses cell-type prediction separately
  (91.3% accuracy from metabolomes alone) and this state-resolution pipeline targets
  within-population heterogeneity.
- Input is raw MALDI-imaging data (.RAW/.imzML/.ibd) or microscopy tiles — SpaceM
  preprocessing must first be run via runAnalysis.py to produce MORPHnMOL.csv before
  state annotation.
- The population lacks detectable substructure (no fatty-acid stimulation / no metabolic
  perturbation), so forcing three Leiden states would be unjustified.
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3391
tools:
- name: SpaceM
  role: Generates the per-cell spatio-molecular matrix (MORPHnMOL.csv) that is the
    direct input for clustering metabolic states
  repo: github.com/alexandrovteam/SpaceM
- name: SCANPY
  role: Nearest-neighbor graph construction (scanpy.pp.neighbors), Leiden clustering,
    UMAP embedding (scanpy.tl.umap), and PAGA state-relationship inference (scanpy.tl.paga)
- name: Python
  role: Execution environment; SpaceM runs via 'python runAnalysis.py' and downstream
    analysis uses pandas for state-fraction tabulation
  repo: github.com/alexandrovteam/SpaceM
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41592-021-01198-0
    title: SpaceM reveals metabolic states of single cells
  source_content_hash: sha256:2e864ef3df70cca3134f3057a4778e3a66c8d1f26a1bb1f2ce018ef6dace912d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Python
  - SCANPY
  - SpaceM
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3391
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# metabolic-state-annotation-in-hepatocytes

## Summary

Resolves unsupervised metabolic states (homeostatic, intermediate, steatotic) within isogenic fatty-acid-stimulated dHepaRG hepatocytes from SpaceM spatio-molecular matrices using Scanpy graph-based clustering (neighbors + Leiden), UMAP embedding, and PAGA state-relationship inference, then tabulates per-condition state fractions.

## When to use

Use when a SpaceM spatio-molecular matrix (MORPHnMOL.csv) of single-cell metabolic profiles must be partitioned into unsupervised metabolic states and per-condition state proportions quantified, e.g. CTRL vs +FA vs +FA+IL17A vs +FA+IL17A+TPCA1 NAFLD/NASH cultures.

## When NOT to use

- The goal is to distinguish different cell types rather than resolve states within one isogenic population — the article addresses cell-type prediction separately (91.3% accuracy from metabolomes alone) and this state-resolution pipeline targets within-population heterogeneity.
- Input is raw MALDI-imaging data (.RAW/.imzML/.ibd) or microscopy tiles — SpaceM preprocessing must first be run via runAnalysis.py to produce MORPHnMOL.csv before state annotation.
- The population lacks detectable substructure (no fatty-acid stimulation / no metabolic perturbation), so forcing three Leiden states would be unjustified.

## Inputs

- SpaceM spatio-molecular matrix MORPHnMOL.csv from the scAnalysis sub-folder (per-cell normalized metabolic profile + microscopy-derived fluorescence/morpho-spatial features)
- Condition labels for hepatocyte cultures: CTRL, +FA, +FA+IL17A, +FA+IL17A+TPCA1

## Outputs

- Leiden cluster labels assigning each cell to homeostatic, intermediate, or steatotic metabolic state
- UMAP embedding with PAGA state-relationship graph of the three metabolic states
- metabolic_state_distributions.csv tabulating the fraction of cells per metabolic state per culture condition

## How to apply

Load MORPHnMOL.csv from the scAnalysis sub-folder with pandas, retaining the normalized metabolic profiles and microscopy-derived phenotypic features for the ~29,738 hepatocytes across conditions. Preprocess the metabolic features and build a nearest-neighbor graph with scanpy.pp.neighbors, then cluster with Leiden (scanpy.tl.leiden) choosing a resolution that resolves three states: homeostatic, intermediate, and steatotic. Embed with scanpy.tl.umap and order states with scanpy.tl.paga to visualize relationships (intermediate lying between homeostatic and steatotic). Finally, use pandas to compute the fraction of cells in each state per culture condition and validate against reported balances — +FA: 75.2% intermediate / 24.3% steatotic; +FA+IL17A: 93% steatotic; +FA+IL17A+TPCA1: shift back to ~75.2% intermediate. Decision points: exclude co-sampling ablation marks as a robustness check (the article reports this changes neither the states nor the state balance); consider whether bimodality of intermediate cells is a clustering artifact or biological heterogeneity of lipid-droplet compositions.

## Related tools

- **SpaceM** (Generates the per-cell spatio-molecular matrix (MORPHnMOL.csv) that is the direct input for clustering metabolic states) — github.com/alexandrovteam/SpaceM
- **SCANPY** (Nearest-neighbor graph construction (scanpy.pp.neighbors), Leiden clustering, UMAP embedding (scanpy.tl.umap), and PAGA state-relationship inference (scanpy.tl.paga))
- **Python** (Execution environment; SpaceM runs via 'python runAnalysis.py' and downstream analysis uses pandas for state-fraction tabulation) — github.com/alexandrovteam/SpaceM

## Examples

```
import pandas as pd, scanpy as sc; adata = sc.read_csv('MF/Analysis/scAnalysis/MORPHnMOL.csv'); sc.pp.neighbors(adata); sc.tl.leiden(adata); sc.tl.paga(adata); sc.tl.umap(adata)
```

## Evaluation signals

- State fractions for +FA reproduce the reported balance: 75.2% intermediate / 24.3% steatotic
- +FA+IL17A shifts toward ~93% steatotic; +FA+IL17A+TPCA1 returns to ~75.2% intermediate — fractions should reproduce these condition-specific balances in metabolic_state_distributions.csv
- UMAP shows three spatially coherent states with intermediate positioned between homeostatic and steatotic on the PAGA graph
- Robustness check: excluding co-sampling ablation marks identifies the same metabolic states and does not change the state balance within each condition (marker statistical significance may drop due to fewer cells)
- Cell count sanity check: the +FA/+FA+IL17A/+FA+IL17A+TPCA1 analysis covers 29,738 hepatocytes

## Limitations

- Bimodality of intermediate cells could either reflect a clustering artifact or biological heterogeneity of LD (lipid droplet) compositions — unexplained
- Excluding co-sampling ablation marks reduces statistical significance of metabolic state markers, likely due to the lower number of cells
- SpaceM downstream code is optimized for Nikon Ti-E microscope output format and requires tiled, stitched microscopy with visible, non-overlapping ablation marks in post-MALDI images

## Evidence

- [results] State fractions reported in task card: [evidence span withheld — anchor status: not_found]
