---
name: dendrogram-visualization-of-lipidome-clustering
description: Use when multiple lipidomes have been loaded and structurally embedded
  in a shared lipid space model, and you need to see which lipidomes cluster by structure/abundance
  or select a branch of similar lipidomes for reanalysis.
when_to_use_negative:
- Only a single lipidome is loaded — the dendrogram relates multiple lipidomes to
  one another, so there is nothing to cluster
- You want relationships between individual lipid species rather than between lipidomes
  — use the per-lipidome lipid space model view instead
- The clustering strategy choice is not part of the question and only the raw distance
  matrix values are needed
tools:
- name: LipidSpace
  role: 'Standalone workstation application performing the hierarchical clustering
    of lipidomes and rendering the interactive dendrogram (Analysis → Clustering strategy:
    single, unweighted average, complete linkage); written in C++ with Qt'
  repo: https://github.com/lifs-tools/lipidspace
- name: lambda-lanczos
  role: Lanczos-method implementation used to compute the first n' < n principal components
    of each lipidome's lipid distance matrix, feeding the structural space that underlies
    the lipidome distances
  repo: https://github.com/mrcdr/lambda-lanczos
- name: cppgoslin
  role: Parses and standardizes lipid names on import so lipid species across heterogeneous
    lipidomes are consistently identified before distance computation
  repo: https://github.com/lifs-tools/cppgoslin
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.3c02449
    title: 'LipidSpace: Simple Exploration, Reanalysis, and Quality Control of Large-Scale
      Lipidomics Studies'
  source_content_hash: sha256:a53b279b76349d10cf5aa181e6f6e77bc8d330508eae8793091f2a4b7773768e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lifs-tools/lipidspace
  tools:
  - LipidSpace
  - cppgoslin
  - lambda-lanczos
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# dendrogram-visualization-of-lipidome-clustering

## Summary

Builds the global lipidome distance model of LipidSpace by applying agglomerative hierarchical clustering to the pairwise Hausdorff distance matrix between all lipidomes, and renders it as an interactive dendrogram (PDF export). The dendrogram is the entry point for sub-selecting lipidome groups for follow-up analyses.

## When to use

Use when multiple lipidomes have been loaded and structurally embedded in a shared lipid space model, and you need to see which lipidomes cluster by structure/abundance or select a branch of similar lipidomes for reanalysis.

## When NOT to use

- Only a single lipidome is loaded — the dendrogram relates multiple lipidomes to one another, so there is nothing to cluster
- You want relationships between individual lipid species rather than between lipidomes — use the per-lipidome lipid space model view instead
- The clustering strategy choice is not part of the question and only the raw distance matrix values are needed

## Inputs

- Multi-lipidome dataset imported into LipidSpace (flat or pivot tables in csv/xlsx, or mzTab-M; e.g. the bundled example dataset via File → Import example dataset)
- m x m symmetric lipidome distance matrix with zero diagonal computed from the per-lipidome lipid space models (pairwise Hausdorff distances)

## Outputs

- Interactive dendrogram plot relating all input lipidomes hierarchically
- PDF export of the dendrogram figure
- Selected lipidome subset (via 'Select these lipidomes in sample selection') for follow-up analysis

## How to apply

First assemble the union set of all distinct lipid species across all lipidomes, and for each lipidome compute the PCA of its lipid distance matrix (maximum common subgraph (MCS) with Jaccard index) using the Lanczos method, adding lipid abundances normalized to the variance of the first principal component. Then compute all pairwise Hausdorff distances over the m lipidomes with the fast Hausdorff implementation, yielding an m x m symmetric matrix with zero diagonal. Apply agglomerative hierarchical clustering to this matrix with unweighted average linkage as the default; single linkage and complete linkage are selectable alternatives via Analysis → Clustering strategy. Render the result as an interactive dendrogram in the dendrogram view, from which a branch can be right-clicked to select its lipidomes in the sample selection and restart the analysis. The linkage choice changes cluster composition, so re-run the dendrogram when switching strategies.

## Related tools

- **LipidSpace** (Standalone workstation application performing the hierarchical clustering of lipidomes and rendering the interactive dendrogram (Analysis → Clustering strategy: single, unweighted average, complete linkage); written in C++ with Qt) — https://github.com/lifs-tools/lipidspace
- **lambda-lanczos** (Lanczos-method implementation used to compute the first n' < n principal components of each lipidome's lipid distance matrix, feeding the structural space that underlies the lipidome distances) — https://github.com/mrcdr/lambda-lanczos
- **cppgoslin** (Parses and standardizes lipid names on import so lipid species across heterogeneous lipidomes are consistently identified before distance computation) — https://github.com/lifs-tools/cppgoslin

## Examples

```
LipidSpace.sh  # then File → Import example dataset; Analysis → Clustering strategy → unweighted average; dendrogram view → right-click branch → 'Select these lipidomes in sample selection'
```

## Evaluation signals

- The underlying lipidome distance matrix must be symmetric with dist(A,B) = dist(B,A) and a zero diagonal
- Matrix values must match the reference REST response for the example dataset (e.g. 1.13403 between the first two lipidomes)
- Switching Analysis → Clustering strategy between single linkage, unweighted average, and complete linkage re-renders the dendrogram with correspondingly different topology
- Right-clicking a branch in the dendrogram view must populate the sample selection with exactly the lipidomes on that branch
- The dendrogram exports as a valid PDF figure via the statistics module

## Limitations

- Analyses cannot currently be saved or loaded, so the dendrogram and its settings cannot be persisted between sessions
- Comparison modes that improve accuracy (Ignore lipid sn-positions, unbound distance metric) reduce performance or reduce visibility of the lipid spaces because distances may become very large
- Clustering operates on structural comparisons of lipidomes, not on exact matches of lipid species names/identifiers; results differ from name-based lipidome comparison

## Evidence

- [readme] linkage options: "The user can switch between single linkage, unweighted average, and complete linkage clustering in the menu → Analysis → Clustering strategy."
- [readme] dendrogram-driven subselection: "Go to the dendrogram view and right-click on any branch which you are interested. In the context menu choose 'Select these lipidomes in sample selection' and restart the analysis."
- [intro] structural basis of lipidome comparison: "we applied the maximum common subgraph
(MCS) approach in combination with the Jaccard index to determine
a pairwise similarity of lipid species based on the molecular structure"
- [methods] per-lipidome PCA feeding the distance model: "we perform a principal component analysis (PCA) on the complete matrix. We decided to use an implementation of the Lanczos method17"
- [methods] abundance normalization before Hausdorff distances: "the abundances for a lipidome are normalized concerning the variance of the first PC of its model and added to the model. Applying pairwise distance computations on m lipidomes"
