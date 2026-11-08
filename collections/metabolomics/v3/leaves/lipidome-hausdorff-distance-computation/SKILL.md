---
name: lipidome-hausdorff-distance-computation
description: Use when you have two or more lipidomes (e.g., csv/xlsx or mzTab-M tables
  with lipid species abundances) and need a global structural comparison relating
  all lipidomes, rather than only exact lipid-name matching.
when_to_use_negative:
- When you only need quantitative differential-abundance statistics between sample
  groups rather than structural comparison across whole lipidomes — use the statistics/feature-analysis
  modules instead.
- When lipid species lists cannot be parsed into structures (e.g., shorthand names
  unsupported by Goslin grammars), since structural distances depend on graph comparison
  of lipid structures.
- When a single lipidome is analyzed — Hausdorff distances require at least two lipidomes
  to relate to each other.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: LipidSpace
  role: Standalone workstation application that computes lipid structural distances,
    lipidome Hausdorff distances, and the hierarchical dendrogram of lipidomes
  repo: https://github.com/lifs-tools/lipidspace
- name: lambda-lanczos
  role: Header-only C++ library used to compute only the first n' < n principal components
    of the lipid distance matrix for the structural space model
  repo: https://github.com/mrcdr/lambda-lanczos
- name: cppgoslin
  role: Parses and standardizes heterogeneous lipid name dialects before structural
    distance computation
  repo: https://github.com/lifs-tools/cppgoslin
- name: openBLAS
  role: Provides fast matrix multiplication for the PCA/linear algebra underlying
    the structural space
  repo: https://www.openblas.net/
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.3c02449
    title: 'LipidSpace: Simple Exploration, Reanalysis, and Quality Control of Large-Scale
      Lipidomics Studies'
  source_content_hash: sha256:cf857638bde2080c4708fb843cd617b1dc5f33cdd4d8df3d6a8d6c099353a5e7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lifs-tools/lipidspace
  tools:
  - LipidSpace
  - cppgoslin
  - lambda-lanczos
  - openBLAS
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# lipidome-hausdorff-distance-computation

## Summary

Computes a global lipidome distance model in LipidSpace by embedding all lipids of multiple lipidomes in a graph-based structural space (MCS + Jaccard via LaWeCSE, PCA via Lanczos), then measuring pairwise Hausdorff distances between the m lipidomes over the union lipid set and clustering them hierarchically.

## When to use

Use when you have two or more lipidomes (e.g., csv/xlsx or mzTab-M tables with lipid species abundances) and need a global structural comparison relating all lipidomes, rather than only exact lipid-name matching.

## When NOT to use

- When you only need quantitative differential-abundance statistics between sample groups rather than structural comparison across whole lipidomes — use the statistics/feature-analysis modules instead.
- When lipid species lists cannot be parsed into structures (e.g., shorthand names unsupported by Goslin grammars), since structural distances depend on graph comparison of lipid structures.
- When a single lipidome is analyzed — Hausdorff distances require at least two lipidomes to relate to each other.

## Inputs

- Multi-lipidome abundance tables in flat or pivot format (csv or xlsx), or mzTab-M files
- Lipid species names parsed/standardized via the cppgoslin library
- Optional sample-associated study variables (e.g., body mass index, tissue, condition)
- Bundled example dataset, e.g., examples/Rest/Plasma-Singapore-Short.json

## Outputs

- m x m symmetric lipidome distance matrix (LipidomeDistanceMatrix) with zero diagonal
- Hierarchical clustering dendrogram of the lipidomes (interactive plot, exportable as PDF)
- Per-lipidome structural space models (PCA on the n x n lipid distance matrix)

## How to apply

Import the multi-lipidome tables (flat or pivot csv/xlsx, or mzTab-M) and let Goslin parse and standardize the lipid names. Build each lipidome's structural model: compute pairwise lipid distances with the maximum common subgraph (LaWeCSE) plus Jaccard index, then run PCA on the complete n x n distance matrix using the lambda-lanczos implementation (default 7 principal components) with openBLAS-accelerated linear algebra. Normalize each lipidome's abundances with respect to the variance of the first PC of its model and add them to the model so that low-abundant species do not dominate. Compute all pairwise Hausdorff distances between the m lipidomes over the union lipid set, yielding an m x m symmetric matrix with zero diagonal. Apply agglomerative hierarchical clustering on this matrix (unweighted average linkage by default; single and complete linkage selectable in Analysis → Clustering strategy) and render the dendrogram, which can be exported as PDF.

## Related tools

- **LipidSpace** (Standalone workstation application that computes lipid structural distances, lipidome Hausdorff distances, and the hierarchical dendrogram of lipidomes) — https://github.com/lifs-tools/lipidspace
- **lambda-lanczos** (Header-only C++ library used to compute only the first n' < n principal components of the lipid distance matrix for the structural space model) — https://github.com/mrcdr/lambda-lanczos
- **cppgoslin** (Parses and standardizes heterogeneous lipid name dialects before structural distance computation) — https://github.com/lifs-tools/cppgoslin
- **openBLAS** (Provides fast matrix multiplication for the PCA/linear algebra underlying the structural space) — https://www.openblas.net/

## Examples

```
curl -X POST -H 'Content-Type: application/json' --data @examples/Rest/Plasma-Singapore-Short.json http://localhost/lipidspace/v1/pca
```

## Evaluation signals

- The lipidome distance matrix must be symmetric with dist(A,B) = dist(B,A) and a zero diagonal.
- LipidomeDistanceMatrix values must match the reference REST response for the example dataset, e.g., 1.13403 between the first two lipidomes of Plasma-Singapore-Short.json.
- The dendrogram must reflect the selected linkage strategy (unweighted average by default; single/complete selectable) and be exportable as PDF.
- Each lipidome's abundances must be normalized to the variance of the first principal component before being added to the model.
- Structural lipid distances must lie in [0, 1] in the default bound metric (0 = identical lipids); verify the bound/unbound mode matches the analysis settings.

## Limitations

- Activating 'Ignore lipid sn-positions' compares all combinations of fatty acyl chains and picks the lowest distance — more accurate but decreased performance.
- The unbound distance metric (0 to infinity) provides more accurate results but reduces visibility of the lipid spaces because distances may become very big.
- Without adding quantity-normalized abundances, structural comparison models are heavily influenced by low abundant lipid species that do or do not appear across samples.
- Analyses cannot currently be saved or loaded in LipidSpace.

## Evidence

- [methods] MCS+Jaccard distance basis: "we decided to utilize the maximum common subgraph (MCS) approach14 in combination with the Jaccard index15"
- [methods] PCA over full distance matrix with Lanczos: "we perform a principal component analysis (PCA) on the complete matrix. We decided to use an implementation of the Lanczos method17"
- [methods] Abundance normalization before relating lipidome models: "the abundances for a lipidome are normalized concerning the variance of the first PC of its model and added to the model. Applying pairwise distance computations on m lipidomes"
- [methods] Bound distance metric range: "the distance is a value that ranges between 0 (both lipids are identical) and 1"
- [readme] Linkage alternatives for the dendrogram: "The user can switch between single linkage, unweighted average, and complete linkage clustering in the menu → Analysis → Clustering strategy."
