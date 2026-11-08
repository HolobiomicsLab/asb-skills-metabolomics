---
name: single-cell-leiden-clustering
description: Use when a per-cell spatio-molecular matrix (e.g. MORPHnMOL.csv with
  29,738 hepatocytes across +FA, +FA+IL17A, +FA+IL17A+TPCA1 conditions) must be partitioned
  into discrete metabolic states without labels.
when_to_use_negative:
- Input is raw MALDI .RAW/.UDP/.imzML/.ibd data or unstitched microscopy tiles — SpaceM
  preprocessing (METASPACE annotation, CellProfiler segmentation, co-registration)
  must run first to produce MORPHnMOL.csv
- The cell population is spatially heterogeneous across cell types and the goal is
  cell-type classification — cell types were predicted with 91.3% accuracy from single-cell
  metabolomes, which is a supervised task, not state clustering
- Cells are profiled by transcriptomics only — this skill is grounded in single-cell
  metabolomics profiles from MALDI-imaging mass spectrometry
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: SpaceM
  role: generates the per-cell spatio-molecular matrix (MORPHnMOL.csv) that is the
    clustering input
  repo: github.com/alexandrovteam/SpaceM
- name: SCANPY
  role: nearest-neighbor graph construction, Leiden clustering, UMAP embedding and
    PAGA state-relationship inference
- name: Python
  role: pandas loading of MORPHnMOL.csv and tabulation of per-condition state fractions
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41592-021-01198-0
    title: SpaceM reveals metabolic states of single cells
  source_content_hash: sha256:c73faed090887d182677cdfb4fdfd83210e2c240430624cc62eda36eda1ee10d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Python
  - SCANPY
  - SpaceM
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# single-cell-leiden-clustering

## Summary

Unsupervised Leiden graph-based clustering of SpaceM single-cell metabolic profiles (scanpy.pp.neighbors + scanpy.tl.leiden) to resolve metabolic states within isogenic hepatocyte populations and quantify the fraction of cells in each state.

## When to use

Use when a per-cell spatio-molecular matrix (e.g. MORPHnMOL.csv with 29,738 hepatocytes across +FA, +FA+IL17A, +FA+IL17A+TPCA1 conditions) must be partitioned into discrete metabolic states without labels.

## When NOT to use

- Input is raw MALDI .RAW/.UDP/.imzML/.ibd data or unstitched microscopy tiles — SpaceM preprocessing (METASPACE annotation, CellProfiler segmentation, co-registration) must run first to produce MORPHnMOL.csv
- The cell population is spatially heterogeneous across cell types and the goal is cell-type classification — cell types were predicted with 91.3% accuracy from single-cell metabolomes, which is a supervised task, not state clustering
- Cells are profiled by transcriptomics only — this skill is grounded in single-cell metabolomics profiles from MALDI-imaging mass spectrometry

## Inputs

- spatio-molecular matrix MORPHnMOL.csv (SpaceM scAnalysis sub-folder) with normalized per-cell metabolic profiles and microscopy-derived phenotypic features
- condition labels for +FA, +FA+IL17A, +FA+IL17A+TPCA1 culture conditions

## Outputs

- Leiden cluster assignment per cell (homeostatic / intermediate / steatotic metabolic state)
- UMAP embedding and PAGA state-relationship graph (scanpy AnnData)
- metabolic_state_distributions.csv tabulating cell-state fractions per culture condition

## How to apply

Load the spatio-molecular matrix MORPHnMOL.csv from the scAnalysis sub-folder with Python/pandas, retaining normalized metabolic profiles and microscopy-derived phenotypic features. Preprocess the metabolic profiles and build a nearest-neighbor graph with scanpy.pp.neighbors, then cluster with Leiden clustering (scanpy.tl.leiden) choosing a resolution that separates the homeostatic, intermediate, and steatotic states known from the dHepaRG NAFLD/NASH model. Embed the clustered cells with scanpy.tl.umap and infer state relationships with scanpy.tl.paga to visualize transitions between states (e.g. intermediate -> steatotic under IL-17A). Finally, tabulate per-condition cell fractions with pandas (expect balances such as +FA: 75.2% intermediate / 24.3% steatotic; +FA+IL17A: 93% steatotic) and save as metabolic_state_distributions.csv. Rationale: co-sampling ablation marks can be excluded or kept — the article reports exclusion led to identification of the same metabolic states without affecting state balances.

## Related tools

- **SpaceM** (generates the per-cell spatio-molecular matrix (MORPHnMOL.csv) that is the clustering input) — github.com/alexandrovteam/SpaceM
- **SCANPY** (nearest-neighbor graph construction, Leiden clustering, UMAP embedding and PAGA state-relationship inference)
- **Python** (pandas loading of MORPHnMOL.csv and tabulation of per-condition state fractions)

## Examples

```
import scanpy as sc; adata = sc.read_csv('MF/Analysis/scAnalysis/MORPHnMOL.csv'); sc.pp.neighbors(adata); sc.tl.leiden(adata); sc.tl.umap(adata); sc.tl.paga(adata)
```

## Evaluation signals

- Cluster fractions match reported balances: +FA ~75.2% intermediate / 24.3% steatotic; +FA+IL17A ~93% steatotic; +FA+IL17A+TPCA1 ~75.2% intermediate
- Leiden resolution resolves exactly three hepatocyte metabolic states (homeostatic, intermediate, steatotic) rather than merging or over-fragmenting them
- PAGA graph connects the intermediate state to the steatotic state, consistent with IL-17A-driven steatosis shift and TPCA-1 (NF-kB inhibition) reversal
- Re-running the pipeline with co-sampling ablation marks excluded yields the same metabolic states and unchanged per-condition state balances
- MORPHnMOL.csv contains profiles for 29,738 hepatocytes with no missing condition labels

## Limitations

- Bimodality of intermediate cells could either reflect a clustering artifact or biological heterogeneity of LD compositions
- Co-sampling of neighboring cells by the same ablation mark exists; excluding co-sampling marks reduced statistical significance of metabolic state markers, likely due to the lower number of cells
- No established validation approaches exist for single-cell metabolomics clustering results

## Evidence
