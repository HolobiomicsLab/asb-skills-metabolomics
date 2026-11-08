---
name: steatosis-inflammation-metabolic-state-comparison
description: Use when single-cell metabolomics data (e.g. a SpaceM MORPHnMOL.csv matrix)
  must be split by culture condition (e.g. +FA vs +FA+IL17A) and tested for differential
  metabolite abundance or metabolic state occupancy shifts.
when_to_use_negative:
- Input is raw MALDI-imaging or microscopy data that has not yet been processed by
  SpaceM into a per-cell spatio-molecular matrix (MORPHnMOL.csv)
- The scientific goal is instrument/segmentation setup (CellProfiler pipelines, tile
  stitching, paths.json configuration) rather than downstream single-cell differential
  analysis
- Comparing bulk metabolomics samples rather than per-cell matrices from single-cell
  metabolomics
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_2229
tools:
- name: SpaceM
  role: Produces the per-cell spatio-molecular matrix (MORPHnMOL.csv) integrating
    MALDI-IMS metabolite intensities with microscopy phenotypic features that this
    comparison consumes
  repo: github.com/alexandrovteam/SpaceM
- name: SCANPY
  role: Runs the Wilcoxon rank-sum differential abundance test (rank_genes_groups)
    with multiple-testing correction on per-cell metabolite intensities
- name: Python
  role: Execution environment for loading MORPHnMOL.csv with pandas, condition annotation,
    subsetting sphingomyelin features, and writing the ranked output table
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41592-021-01198-0
    title: SpaceM reveals metabolic states of single cells
  source_content_hash: sha256:231890d4eeb4de46c8c354b35ee626cd02c359c13aa41c579d237dfda09830f0
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Python
  - SCANPY
  - SpaceM
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# steatosis-inflammation-metabolic-state-comparison

## Summary

Compares metabolic states of single hepatocytes between benign (FA-stimulated) and inflammatory (FA+IL-17A) steatotic conditions using per-cell spatio-molecular matrices from SpaceM, quantifying how inflammatory signalling shifts state occupancy and per-metabolite abundance.

## When to use

Use when single-cell metabolomics data (e.g. a SpaceM MORPHnMOL.csv matrix) must be split by culture condition (e.g. +FA vs +FA+IL17A) and tested for differential metabolite abundance or metabolic state occupancy shifts.

## When NOT to use

- Input is raw MALDI-imaging or microscopy data that has not yet been processed by SpaceM into a per-cell spatio-molecular matrix (MORPHnMOL.csv)
- The scientific goal is instrument/segmentation setup (CellProfiler pipelines, tile stitching, paths.json configuration) rather than downstream single-cell differential analysis
- Comparing bulk metabolomics samples rather than per-cell matrices from single-cell metabolomics

## Inputs

- MORPHnMOL.csv — SpaceM per-cell spatio-molecular matrix with normalized metabolite intensities and microscopy-derived morphological/fluorescence features (from the scAnalysis sub-folder)
- Culture-condition metadata columns annotating cells as +FA or +FA+IL17A
- Metabolite annotation (e.g. METASPACE identifiers) for mapping features to sphingomyelin/ceramide-phosphocholine classes

## Outputs

- differential_abundance_inflammatory_hepatocytes.csv — ranked differential abundance table with log fold-change, adjusted p-value, and metabolite annotation columns
- Per-condition metabolic state occupancy percentages (e.g. % of cells in the steatotic state per condition)

## How to apply

Load the per-cell spatio-molecular matrix MORPHnMOL.csv (normalized metabolite intensities plus morphology/fluorescence features) with pandas, then annotate and split cells into the +FA ('benign' steatotic) and +FA+IL17A ('inflammatory' steatotic) populations from culture-condition metadata columns. Run a per-metabolite non-parametric differential abundance test — Wilcoxon rank-sum via scanpy's rank_genes_groups with multiple-testing correction — comparing inflammatory vs benign cells. Subset and summarize results for ceramide phosphocholines (sphingomyelins), assessing direction and significance of their enrichment in the inflammatory state, mirroring Supp. Fig 12. Finally, write the ranked table with log fold-change, adjusted p-value, and metabolite annotation columns, and report state occupancy percentages (e.g. 93% of +FA+IL17A vs 24.3% of +FA cells in the steatotic state) to quantify how IL-17A shifts nearly all cells into the steatotic state.

## Related tools

- **SpaceM** (Produces the per-cell spatio-molecular matrix (MORPHnMOL.csv) integrating MALDI-IMS metabolite intensities with microscopy phenotypic features that this comparison consumes) — github.com/alexandrovteam/SpaceM
- **SCANPY** (Runs the Wilcoxon rank-sum differential abundance test (rank_genes_groups) with multiple-testing correction on per-cell metabolite intensities)
- **Python** (Execution environment for loading MORPHnMOL.csv with pandas, condition annotation, subsetting sphingomyelin features, and writing the ranked output table)

## Examples

```
import pandas as pd; df = pd.read_csv('MF/Analysis/scAnalysis/MORPHnMOL.csv'); sc.tl.rank_genes_groups(adata, groupby='condition', method='wilcoxon')
```

## Evaluation signals

- Output file differential_abundance_inflammatory_hepatocytes.csv exists and contains log fold-change, adjusted p-value, and metabolite annotation columns
- Sphingomyelin/ceramide-phosphocholine features show consistent enrichment in the +FA+IL17A inflammatory group: positive log fold-change with significant adjusted p-values, matching the direction reported in Supp. Fig 12
- Metabolic state occupancy fractions are reproduced: 93% of +FA+IL17A cells vs 24.3% of +FA cells in the steatotic state
- Every cell in the matrix is assigned to exactly one of the two condition populations with no unannotated cells
- Multiple-testing correction has been applied so reported p-values are adjusted p-values, not raw Wilcoxon p-values

## Limitations

- Co-sampling of neighboring cells by the same ablation mark can occur; excluding co-sampled marks preserved the same metabolic states and condition balances but reduced statistical significance of markers due to fewer cells
- Increased sphingolipids in the inflammatory/NASH-like state lack a mechanistic explanation in the source work — enrichment is a descriptive, not mechanistic, finding
- The upstream SpaceM implementation requires tile stitching and is optimized for Nikon Ti-E microscope output, constraining the raw data that can feed this comparison

## Evidence
