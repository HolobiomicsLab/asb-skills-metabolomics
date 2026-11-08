---
name: paga-trajectory-inference
description: Use when you have clustered single-cell metabolic profiles (e.g. a Leiden-partitioned
  AnnData built from a SpaceM spatio-molecular matrix like MORPHnMOL.csv) and need
  to infer connectivity/relationships between metabolic states and embed them coherently
  with UMAP.
when_to_use_negative:
- Input is still raw MALDI-imaging data (.RAW/.UDP/.imzML/.ibd) or unprocessed microscopy
  tiles — run the SpaceM pipeline (runAnalysis.py) first to obtain MORPHnMOL.csv
- No categorical cell grouping exists yet — PAGA requires pre-defined cluster labels
  (e.g. Leiden groups); it does not perform the state assignment itself
- The goal is marker-metabolite identification rather than state-relationship inference
  — METASPACE-based analysis is the appropriate step
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: SCANPY
  role: Provides the Leiden clustering, nearest-neighbor graph, PAGA graph abstraction
    (scanpy.tl.paga), and PAGA-initialized UMAP embedding (scanpy.tl.umap) used to
    resolve and relate the three metabolic states
  repo: https://github.com/alexandrovteam/SpaceM
- name: Python
  role: Execution environment; SpaceM is run via `python runAnalysis.py` and downstream
    Scanpy/PAGA analysis is demonstrated in a Google Colab notebook
  repo: https://github.com/alexandrovteam/SpaceM
- name: SpaceM
  role: Upstream method that generates the spatio-molecular matrix (MORPHnMOL.csv)
    consumed by the PAGA/Scanpy downstream analysis
  repo: https://github.com/alexandrovteam/SpaceM
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41592-021-01198-0
    title: SpaceM reveals metabolic states of single cells
  source_content_hash: sha256:ddab6ce8aca92ab9440ba8a03bf15d4fdaf69391742cecfd194e99a9a52e6923
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/alexandrovteam/SpaceM
  tools:
  - Python
  - SCANPY
  - SpaceM
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# PAGA trajectory inference

## Summary

Apply Partition-based Graph Abstraction (PAGA) in Scanpy to infer and visualize relationships between single-cell metabolic states — e.g. homeostatic, intermediate, and steatotic hepatocyte states — after Leiden clustering of SpaceM spatio-molecular profiles, using the PAGA graph to initialize a UMAP embedding.

## When to use

Use when you have clustered single-cell metabolic profiles (e.g. a Leiden-partitioned AnnData built from a SpaceM spatio-molecular matrix like MORPHnMOL.csv) and need to infer connectivity/relationships between metabolic states and embed them coherently with UMAP.

## When NOT to use

- Input is still raw MALDI-imaging data (.RAW/.UDP/.imzML/.ibd) or unprocessed microscopy tiles — run the SpaceM pipeline (runAnalysis.py) first to obtain MORPHnMOL.csv
- No categorical cell grouping exists yet — PAGA requires pre-defined cluster labels (e.g. Leiden groups); it does not perform the state assignment itself
- The goal is marker-metabolite identification rather than state-relationship inference — METASPACE-based analysis is the appropriate step

## Inputs

- Spatio-molecular matrix MORPHnMOL.csv (normalized single-cell metabolic profiles plus microscopy-derived phenotypic features, from the SpaceM scAnalysis sub-folder)
- AnnData object with normalized metabolic profiles and Leiden cluster labels for cells from +FA, +FA+IL17A, and +FA+IL17A+TPCA1 conditions

## Outputs

- PAGA graph abstraction connecting metabolic states (homeostatic, intermediate, steatotic)
- UMAP embedding of clustered single-cell metabolic states initialized from PAGA
- Per-condition table of metabolic-state fractions (e.g. metabolic_state_distributions.csv)

## How to apply

Build a nearest-neighbor graph over the single-cell metabolic profiles from the SpaceM spatio-molecular matrix (MORPHnMOL.csv from the scAnalysis sub-folder) with scanpy.pp.neighbors, then run Leiden clustering (scanpy.tl.leiden) to assign cells to metabolic states. Compute PAGA with scanpy.tl.paga using the Leiden group labels to obtain a graph abstraction of state connectivity, then embed with scanpy.tl.umap initialized from the PAGA layout so that the homeostatic, intermediate, and steatotic states appear as connected groups. In the hepatocyte NAFLD/NASH application, tabulate per-condition state fractions with pandas and compare against reported balances (+FA: 75.2% intermediate / 24.3% steatotic; +FA+IL17A: 93% steatotic; +FA+IL17A+TPCA1: 75.2% intermediate). The rationale is that PAGA preserves cluster-level connectivity while allowing a denoised 2D embedding of tens of thousands of cells (29,738 hepatocytes across conditions).

## Related tools

- **SCANPY** (Provides the Leiden clustering, nearest-neighbor graph, PAGA graph abstraction (scanpy.tl.paga), and PAGA-initialized UMAP embedding (scanpy.tl.umap) used to resolve and relate the three metabolic states) — https://github.com/alexandrovteam/SpaceM
- **Python** (Execution environment; SpaceM is run via `python runAnalysis.py` and downstream Scanpy/PAGA analysis is demonstrated in a Google Colab notebook) — https://github.com/alexandrovteam/SpaceM
- **SpaceM** (Upstream method that generates the spatio-molecular matrix (MORPHnMOL.csv) consumed by the PAGA/Scanpy downstream analysis) — https://github.com/alexandrovteam/SpaceM

## Examples

```
import scanpy as sc; sc.pp.neighbors(adata); sc.tl.leiden(adata); sc.tl.paga(adata, groups='leiden'); sc.pl.paga(adata); sc.tl.umap(adata, init_pos='paga')
```

## Evaluation signals

- The PAGA graph connects the expected states (homeostatic–intermediate–steatotic) rather than producing disconnected groups
- PAGA-initialized UMAP shows the three metabolic states as contiguous, non-fragmented groups
- Per-condition state fractions match reported balances: +FA 75.2% intermediate / 24.3% steatotic; +FA+IL17A 93% steatotic; +FA+IL17A+TPCA1 75.2% intermediate
- Excluding co-sampling ablation marks leads to identification of the same metabolic states and does not change the state balance per condition
- MORPHnMOL.csv is present in the Analysis/scAnalysis sub-folder after running the SpaceM pipeline

## Limitations

- Co-sampling of neighboring cells by the same ablation mark can occur, though it only minimally affects cell-type prediction and did not affect the metabolic state balance per condition
- Bimodality of intermediate-state cells could reflect either a clustering artifact or biological heterogeneity of lipid-droplet (LD) compositions — PAGA cannot distinguish these
- SpaceM's implementation requires stitching of microscopy tiles and the code is optimized for the Nikon Ti-E microscope output format
- CellProfiler 3.0.0 and a specific Fiji release (December 22, 2015) are required; no version numbers for Python are given in the methods

## Evidence
