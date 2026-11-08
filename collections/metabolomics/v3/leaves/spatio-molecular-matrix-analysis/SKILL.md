---
name: spatio-molecular-matrix-analysis
description: Use when you have a per-cell spatio-molecular matrix (MORPHnMOL.csv)
  from SpaceM and need to identify metabolic states, rank differential metabolites
  between conditions, or classify cell types from single-cell metabolomes.
when_to_use_negative:
- Input is raw MALDI-imaging (.imzML/.ibd) or pre/post-MALDI microscopy tiles — run
  the SpaceM pipeline (runAnalysis.py) first to generate MORPHnMOL.csv before downstream
  single-cell analysis
- Data are bulk LC-MS/MS tissue lipidomics (e.g. ND vs WD mouse liver) rather than
  per-cell SpaceM matrices — use bulk differential-analysis methods instead
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_2229
tools:
- name: SpaceM
  role: Generates the spatio-molecular matrix (MORPHnMOL.csv) that this skill analyzes,
    integrating MALDI-imaging MS with light microscopy
  repo: github.com/alexandrovteam/SpaceM
- name: SCANPY
  role: Clustering of metabolic states and Wilcoxon rank-sum differential metabolite
    testing (rank_genes_groups) with multiple-testing correction
- name: Python
  role: Loads MORPHnMOL.csv with pandas, applies co-sampling filters, and orchestrates
    downstream analysis
provenance:
  source_task_ids:
  - task_007
  - task_006
  - task_005
  source_papers:
  - doi: 10.1038/s41592-021-01198-0
    title: SpaceM reveals metabolic states of single cells
  source_content_hash: sha256:613bfcc32271ae11db17c03ef33019a3c74093ac3823a1d9cd0f5d967de23381
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Fiji
  - Python
  - SCANPY
  - SpaceM
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# spatio-molecular-matrix-analysis

## Summary

Downstream single-cell analysis of the SpaceM spatio-molecular matrix (MORPHnMOL.csv), which pairs normalized per-cell metabolite intensities with microscopy-derived morphology/fluorescence features, to cluster metabolic states, test differential metabolite abundance, and classify cell types in Python/Scanpy.

## When to use

Use when you have a per-cell spatio-molecular matrix (MORPHnMOL.csv) from SpaceM and need to identify metabolic states, rank differential metabolites between conditions, or classify cell types from single-cell metabolomes.

## When NOT to use

- Input is raw MALDI-imaging (.imzML/.ibd) or pre/post-MALDI microscopy tiles — run the SpaceM pipeline (runAnalysis.py) first to generate MORPHnMOL.csv before downstream single-cell analysis
- Data are bulk LC-MS/MS tissue lipidomics (e.g. ND vs WD mouse liver) rather than per-cell SpaceM matrices — use bulk differential-analysis methods instead

## Inputs

- MORPHnMOL.csv — per-cell spatio-molecular matrix with normalized metabolite intensities and morphology/fluorescence features (from MF/Analysis/scAnalysis)
- Per-cell metadata columns encoding culture condition (e.g. control, +FA, +FA+IL17A) and cell type for co-cultures
- Optionally, co-sampling annotations identifying ablation marks that sampled neighboring cells

## Outputs

- Metabolic-state cluster assignments per cell (reproducible against Supp. Fig. 11A)
- Ranked differential abundance table (log fold-change, adjusted p-value, metabolite annotation), e.g. differential_abundance_inflammatory_hepatocytes.csv
- Cell-type classification accuracy metrics for co-culture metabolomes (Supp. Fig. 6)
- Per-condition metabolic-state fraction comparisons between filtered and unfiltered runs

## How to apply

Load MORPHnMOL.csv from the MF/Analysis/scAnalysis sub-folder with pandas; it contains normalized metabolite intensities plus microscopy-derived phenotypic properties (fluorescence intensities and morpho-spatial features) per cell. Annotate cells by culture-condition metadata (e.g. +FA benign steatotic vs +FA+IL17A inflammatory steatotic hepatocytes) and run clustering in Scanpy to recover metabolic states, mirroring Supp. Fig. 11A. For per-metabolite comparisons, run a non-parametric Wilcoxon rank-sum test via scanpy's rank_genes_groups with multiple-testing correction, then subset features of interest (e.g. ceramide phosphocholines/sphingomyelins) and assess direction and significance via log fold-change and adjusted p-values. For mixed co-cultures (HeLa/NIH3T3), train a cell-type classifier on the single-cell metabolomes; expect ~91.3% accuracy. Optionally repeat the analysis after dropping cells whose ablation mark co-sampled a neighboring cell — the paper shows states and balance are preserved, though marker significance drops due to fewer cells (29,738 → 21,009).

## Related tools

- **SpaceM** (Generates the spatio-molecular matrix (MORPHnMOL.csv) that this skill analyzes, integrating MALDI-imaging MS with light microscopy) — github.com/alexandrovteam/SpaceM
- **SCANPY** (Clustering of metabolic states and Wilcoxon rank-sum differential metabolite testing (rank_genes_groups) with multiple-testing correction)
- **Python** (Loads MORPHnMOL.csv with pandas, applies co-sampling filters, and orchestrates downstream analysis)

## Examples

```
import pandas as pd; adata = sc.AnnData(pd.read_csv('MF/Analysis/scAnalysis/MORPHnMOL.csv')); sc.tl.rank_genes_groups(adata, 'condition', method='wilcoxon')
```

## Evaluation signals

- MORPHnMOL.csv exists in MF/Analysis/scAnalysis and loads in pandas with both metabolite-intensity and morphology/fluorescence columns
- Differential abundance table contains required columns (log fold-change, adjusted p-value, metabolite annotation) and sphingomyelin/ceramide-phosphocholine features show positive log fold-changes with significant adjusted p-values in +FA+IL17A cells, matching Supp. Fig. 12
- Clustering recovers the same metabolic states, and per-condition state fractions are near-unchanged, when co-sampled ablation marks are excluded (Supp. Figs 6 and 11A)
- Cell-type classification on single-cell metabolomes reaches ~91.3% accuracy in the HeLa/NIH3T3 co-culture
- Filtered run keeps expected cell counts (21,009 filtered vs 29,738 unfiltered hepatocytes) and marker identities, with reduced marker significance attributable to lower cell number

## Limitations

- Excluding co-sampling ablation marks reduces statistical significance of metabolic state markers, likely due to the lower number of cells
- Bimodality of intermediate-state cells could reflect a clustering artifact or biological heterogeneity of lipid-droplet (LD) compositions
- The upstream SpaceM implementation requires tile stitching optimized for Nikon Ti-E microscope output and visible, non-overlapping ablation marks in post-MALDI microscopy

## Evidence
