---
name: umap-embedding-of-metabolic-profiles
description: Use when you have a per-cell normalized metabolic profile matrix (e.g.
  SpaceM's MORPHnMOL.csv with ~30k cells) and need a 2D visualization of resolved
  metabolic states after graph-based clustering.
when_to_use_negative:
- Input is raw MALDI-imaging .imzML/.ibd data or pre/post-MALDI microscopy tiles —
  this skill operates downstream on the already-generated per-cell spatio-molecular
  matrix
- You need cell-type classification from metabolomes alone rather than visualization
  of within-population state structure (the article used supervised prediction for
  the 91.3% cell-type accuracy, not UMAP)
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: SCANPY
  role: Provides scanpy.pp.neighbors graph construction, Leiden clustering, scanpy.tl.umap
    embedding and scanpy.tl.paga state-relationship inference applied to the metabolic
    profiles
- name: SpaceM
  role: Produces the upstream MORPHnMOL.csv spatio-molecular matrix that is embedded;
    the SpaceM Colab notebook demonstrates downstream processing of these matrices
  repo: github.com/alexandrovteam/SpaceM
- name: Python
  role: Runtime for the Scanpy-based UMAP pipeline and pandas tabulation of state
    fractions
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41592-021-01198-0
    title: SpaceM reveals metabolic states of single cells
  source_content_hash: sha256:1624f56910bada952a4dc8199b44fe9cd0683e82b4b8962b7309edd3841cd2c2
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Python
  - SCANPY
  - SpaceM
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# UMAP embedding of metabolic profiles

## Summary

Nonlinear dimensionality reduction of SpaceM spatio-molecular matrices (e.g. MORPHnMOL.csv) with Scanpy UMAP, after Leiden clustering, to visualize single-cell metabolic states (homeostatic, intermediate, steatotic) of dHepaRG hepatocytes in 2D.

## When to use

Use when you have a per-cell normalized metabolic profile matrix (e.g. SpaceM's MORPHnMOL.csv with ~30k cells) and need a 2D visualization of resolved metabolic states after graph-based clustering.

## When NOT to use

- Input is raw MALDI-imaging .imzML/.ibd data or pre/post-MALDI microscopy tiles — this skill operates downstream on the already-generated per-cell spatio-molecular matrix
- You need cell-type classification from metabolomes alone rather than visualization of within-population state structure (the article used supervised prediction for the 91.3% cell-type accuracy, not UMAP)

## Inputs

- SpaceM spatio-molecular matrix MORPHnMOL.csv (from the scAnalysis sub-folder) containing normalized metabolic profiles and microscopy-derived phenotypic features for single cells
- Leiden cluster/state assignments over the single-cell metabolic profiles
- Culture-condition labels per cell (e.g. +FA, +FA+IL17A, +FA+IL17A+TPCA1)

## Outputs

- 2D UMAP coordinates for all ~29,738 hepatocytes embedded from their metabolic profiles
- UMAP plots colored by metabolic state (homeostatic/intermediate/steatotic) and by culture condition
- PAGA graph of state relationships (scanpy.tl.paga)

## How to apply

Load the spatio-molecular matrix MORPHnMOL.csv from the scAnalysis sub-folder with pandas, retaining normalized metabolite intensities and microscopy-derived phenotypic features. Preprocess the metabolic profiles and build a nearest-neighbor graph with scanpy.pp.neighbors, since UMAP operates on this graph rather than the raw high-dimensional metabolite matrix. Run Leiden clustering first (scanpy Leiden) so states (homeostatic, intermediate, steatotic) are defined, then embed with scanpy.tl.umap and optionally run scanpy.tl.paga to infer state relationships between clusters. Color the UMAP by cluster/state and by culture condition (+FA, +FA+IL17A, +FA+IL17A+TPCA1) to compare state balance across conditions, e.g. the 75.2% intermediate / 24.3% steatotic split under +FA. Judgement of correctness: the embedding should spatially separate the resolved metabolic states in a way consistent with Leiden cluster assignments, and state fractions tabulated per condition should match reported balances when reproducing the hepatocyte study.

## Related tools

- **SCANPY** (Provides scanpy.pp.neighbors graph construction, Leiden clustering, scanpy.tl.umap embedding and scanpy.tl.paga state-relationship inference applied to the metabolic profiles)
- **SpaceM** (Produces the upstream MORPHnMOL.csv spatio-molecular matrix that is embedded; the SpaceM Colab notebook demonstrates downstream processing of these matrices) — github.com/alexandrovteam/SpaceM
- **Python** (Runtime for the Scanpy-based UMAP pipeline and pandas tabulation of state fractions)

## Examples

```
import scanpy as sc; adata = sc.read_csv('MORPHnMOL.csv'); sc.pp.neighbors(adata); sc.tl.umap(adata); sc.tl.paga(adata)
```

## Evaluation signals

- UMAP embedding derives from a nearest-neighbor graph built on normalized metabolic profiles, not raw intensity columns
- The three Leiden-defined metabolic states (homeostatic, intermediate, steatotic) are spatially separated in the 2D UMAP
- State fractions computed on the embedded clusters reproduce the reported balances: +FA 75.2% intermediate / 24.3% steatotic; +FA+IL17A ~93% steatotic; +FA+IL17A+TPCA1 75.2% intermediate
- PAGA relationships between states are consistent with the UMAP topology (no disconnected/contradictory cluster links)
- Cell count in the embedded matrix matches the source MORPHnMOL.csv (~29,738 hepatocytes across the three conditions)

## Limitations

- The SpaceM implementation requires tile stitching and is optimized for Nikon Ti-E microscope output format, constraining which upstream matrices can be generated for embedding
- Bimodality of intermediate-state cells on the embedding could reflect a clustering artifact or biological heterogeneity of lipid-droplet (LD) compositions, so UMAP/cluster structure should not be over-interpreted
- Excluding co-sampling ablation marks reduces the number of cells and the statistical significance of metabolic state markers, which can affect embedding-based visual comparisons

## Evidence

- [other] Embed the clustered cells with UMAP and infer state relationships with PAGA (scanpy.tl.umap, scanpy.tl.paga) to visualize the three metabolic states.: [evidence span withheld — anchor status: not_found]
- [results] Among fatty-acid-stimulated (+FA) hepatocytes, unsupervised clustering resolved an intermediate and a steatotic metabolic state, with 75.2% and 24.3% of cells residing in each state, respectively.: [evidence span withheld — anchor status: not_found]
- [other] Preprocess and build a nearest-neighbor graph over the single-cell metabolic profiles using Scanpy (scanpy.pp and scanpy.pp.neighbors).: [evidence span withheld — anchor status: not_found]
